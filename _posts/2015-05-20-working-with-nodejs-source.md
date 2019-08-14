---
title: Working with NodeJS source
excerpt: I've been trying to set up a development environment for working on NodeJS source, with little luck.
tags: development nodejs
categories: Development
classes: wide
redirect_from: "/working-with-nodejs-source-9fe95b1765fe"
---

I’ve developed an interest in contributing to the NodeJS project, especially since I found some things while working on my [sslinfo](https://github.com/iamthechad/sslinfo) module that could be improved in the x509 realm.

I’m not comfortable building and running the Node source on my everyday machine since I don’t know what kinds of side effects and conflicts I might cause. It seemed only natural to use some kind of virtual machine based solution, but I’ve been having little luck getting an environment set up to my liking.

I have a modest list of requirements that I’d like to satisfy with a VM-based solution:

*   Quick and repeatable VM creation
*   Access to the Node source from within the VM, for building/testing/deploying in a “clean” environment.
*   Access to the Node source from the host machine, for using my existing dev tools (TextMate, SublimeText) to modify code and documentation, and for using my existing Git tools.

Basically, I’ve already got a set of tools in my Mac environment that I’m very accustomed to. I don’t want to reinvent that wheel by requiring a bunch more tools within the VM.

My initial thought was to simply repurpose the [Vagrant setup I use to manage this blog](https://github.com/iamthechad/vagrant-ghpages). It’s a simple setup, with a `config.vm.synced_folder "../node/", "/opt/node"` line in the Vagrantfile to map my already checked out NodeJS source into the VM.

Once I had the VM running, I SSH’ed in and ran the following:

```bash
$ cd /opt/node
$ make clean && rm -rf out
$ ./configure
$ make
$ make test
```

The code builds with no errors, but 14 tests fail.

Failed test summary -

```bash
Path: simple/test-cluster-http-pipe
Error: bind EPERM

Path: simple/test-fs-error-messages
assert.js:86
  throw new assert.AssertionError({
        ^
AssertionError: false == true

Command: out/Release/node /opt/node/test/simple/test-fs-error-messages.js
Message from syslogd@debian-7 at May 20 12:48:18 ...
 kernel:[  951.248484] Oops: 0002 [#1] SMP

Path: simple/test-fs-symlink
Error: EPERM, link '/opt/node/test/tmp/link1.js'

Path: simple/test-http-client-pipe-end
Error: listen EPERM

Path: simple/test-http-client-response-domain
Error: listen EPERM

Path: simple/test-http-unix-socket
Error: listen EPERM

Path: simple/test-net-pingpong
/opt/node/test/tmp/test.sock
Error: listen EPERM

Path: simple/test-net-pipe-connect-errors
Error: listen EPERM

Path: simple/test-pipe-address
Error: listen EPERM

Path: simple/test-pipe-stream
Error: listen EPERM

Path: simple/test-pipe-unref
Error: listen EPERM

Path: simple/test-repl
repl test
Error: listen EPERM

Path: simple/test-tls-connect-pipe
Error: listen EPERM
```

Grrr, although there is a pattern. The majority of the failures have an `EPERM` root cause.

For this attempt, I didn’t bother setting a shared folder in the Vagrantfile. Parallels tools are installed, and they give me access to my Mac’s files through `/media/psf`.

The steps this time:

```bash
$ cd /media/psf/Home/Documents/workspaces/node #Yeah, that's where I keep my code.
$ make clean && rm -rf out
$ ./configure
$ make
$ make test
```

14 tests fail, just like before.

Grrr again.

I asked Twitter. I asked the NodeJS IRC channel. I Googled. Finally, I found a single page that offered anecdotal evidence that [NodeJS cannot create sockets in a shared directory in a VM](http://samplacette.com/node-js-net-module-server-listen-socket-eperm-error/). Looking back at the failing tests, they were all socket or pipe related.

**I have not been able to find “official” confirmation of this. If anyone can confirm or deny, please let me know.**

### Attempt Three: Vagrant VM with SFTP Sync

I didn’t bother with shared folders at all this time. I installed the handy [SFTP plugin](http://wbond.net/sublime_packages/sftp) for Sublime Text and synced my local NodeJS source into the VM.

Steps again:

```bash
$ cd /home/cjohnston/node
$ make clean && rm -rf out
$ ./configure
$ make
$ make test
```

Only 1 test fails this time, but it’s a new failure.

Failed test summary -

```bash
Path: simple/test-fs-realpath
Error: EEXIST, file already exists '/home/cjohnston/node/test/fixtures/nested-index/one/symlink1-dir'
```

I honestly have no idea why this test fails. Maybe it’s file permissions due to the SFTP copy?

### Attempt Four: Give up and check out the code in the VM

As expected, building and running the tests works perfectly with this method. However, I’ve also lost the ability to expose the code to the host OS. My Vagrant VM doesn’t have a windowing system installed, so I’m down to `vi` for code edits, and checking out the source in parallel on my Mac for making and previewing doc changes.

At this point, I’d be happy for any input on how to achieve my development goals as stated at the top of the post. Solving the issue of being able to open sockets and pipes in a shared folder would be fantastic, but I don’t know how hard that problem would be to solve.
