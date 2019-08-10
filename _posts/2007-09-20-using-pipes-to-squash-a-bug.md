---
title: Using Pipes to Squash a Bug
tags: development
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

There’s been a bug in the Frame2 Eclipse plugin from day 1 that I believe I have finally squashed. The reading and writing of the `frame2-config` file happened outside of the Eclipse API, and when the file was updated using a wizard the resource got out of sync with the workspace. It hasn’t really been a big issue, but it’s annoying to always be asked by Eclipse if you’d like to reload the file since it has changed.

My original stab at fixing the problem was to change the file as it’s always been changed, but call `refreshLocal()` on the Eclipse `IFile` representation. The call never threw an exception, but it also never seemed to do anything. I finally refactored the model reading and writing code to use Eclipse APIs and platform objects. This worked fine until I got to the point where I needed to write the modified configuration to file. The existing API required an `OutputStream`, while the Eclipse API wanted an `InputStream`.

Luckily, Google turned up some suggestions, most notably using piped streams. My first attempt looked like this:

```
PipedInputStream in = new PipedInputStream();
PipedOutputStream out = new PipedOutputStream(in);
config.write(out);
modelFile.setContents(in, true, true, monitor);
```

`Config` is the Frame2 model object that knows how to write itself to an `OutputStream`, and `modelFile` is the `IFile` object representing the config file in Eclipse. Anybody who’s ever used piped streams in Java should be able to tell what happened when I ran the code - it hung. It always pays to read the JavaDoc in addition to copying somebody’s snippet of code from the web. What the JavaDoc says is this: “Attempting to use both objects from a single thread is not recommended as it may deadlock the thread.” In this case, I think _may_ is a bit soft. It deadlocked every single time I ran it.

Based on another online snippet, I modified the code to this:

```
PipedInputStream in = new PipedInputStream();
PipedOutputStream out = new PipedOutputStream(in);

new Thread(new Runnable() {
   public void run() {
      try {
         config.write(out);
      } catch (IOException e) {
         e.printStackTrace();
      }
   }
}).start();
modelFile.setContents(in, true, true, monitor);
```

This is the variant that I ran across most often online. It uses two threads to avoid deadlocking, but it still has a fatal flaw. The `setContents()` method always throws an `IOException` with this code. The `IOException`‘s message is always “Pipe Broken”. I found a \*lot\* of entries online from people asking how best to ignore this exception. There were lots of helpful explanations as to the cause of the error, but I didn’t see a single solution. Luckily, the answer was easy to find in this case.

The “Pipe Broken” message is happening because the the thread doing the writing to the `OutputStream` has terminated and nothing more is being added to it. The solution is to simply close the `OutputStream` after writing the configuration to it. The `OutputStream` is correctly marked as being finished and the exception is not thrown. An examination of the `read()` method in `PipedInputStream` shows why the `OutputStream` must be closed:

```
if (closedByWriter) {
   /* closed by writer, return EOF */
   return -1;
}

if ((writeSide != null) && (!writeSide.isAlive()) && (--trials < 0)) {
   throw new IOException("Pipe broken");
}
```

If the writing side of the pipe simply exits, `writeSide.isAlive()` returns `false` and the exception is thrown. If the `OutputStream` is closed, however, the reading side of the pipe sees it as EOF and terminates gracefully.

To sum up, the correct example for using piped streams in Java should look like this:

```
PipedInputStream in = new PipedInputStream();
PipedOutputStream out = new PipedOutputStream(in);

new Thread(new Runnable() {
   public void run() {
      try {
         doSomethingWithOut(out);
         out.close(); // Critical!
      } catch (IOException e) {
         e.printStackTrace(); // Do more than just this, OK?
      }
   }
}).start(); doSomethingWithIn(in);
```
