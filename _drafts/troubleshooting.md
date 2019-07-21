---
title: Troubleshooting
description: >-
  If you’re a junior (or any level, really) developer, one of the most important
  things you can learn is how to troubleshoot when things go…
date: ''
categories: []
keywords: []
slug: ''
---

If you’re a junior (or any level, really) developer, one of the most important things you can learn is how to troubleshoot when things go wrong. I’m not talking about firing up a debugger and stepping through code. That’s a step for later. I’m talking about the steps that take you from “I got an error on the web page” to knowing where in the code to debug.

As a senior developer, I’ve been around the block a few times. This means that I have a pretty good idea of where to look when things go wrong. Unfortunately, I’ve worked with a lot of junior developers who know this and take advantage of it. I’m happy to help out and provide guidance, but nothing is quite as annoying as someone giving a vague “something broke” message in group chat then sitting back and waiting for somebody else (i.e. me) to solve the problem.

Here are some of the steps I’ve learned over the years that help me.

### Clarify what “broken” means

There are many levels to something being broken. To help others (and yourself), you need to narrow it down.

Right away, you can answer these questions:

*   Were you able to access the web page?
*   Did the web page load correctly?

If you tried to perform action X, and it didn’t work:

*   Was there an error message?
*   Did something else happen?
*   Did nothing happen?

Sometimes there can be a lot of difference between an action causing an error and an action silently (from the UI perspective) failing.

A good of understanding of how to explain what’s wrong can make the troubleshooting steps much easier for you and for anyone that’s trying to help you.

### Understand the technology

Having a basic understanding of what your application is doing is invaluable. 

If you’re building a web application, you must know how web applications work, even at a high level:

*   Understand basic HTTP
*   Know basic HTTP response codes. What’s a 404? 302? 400?
*   What is the browser doing when it first loads a URL?
*   How are assets like CSS and JavaScript loaded?

### Understand your application

How does your application load itself and get initialized?

The application that I work on:

1.  Makes a login check
2.  If the user isn’t logged in, redirects to the login service
3.  Once the user is logged in, redirects to the main app
4.  The main app initializes its dashboard, making calls to the API services

Each of these steps involves loading assets and making network calls. Understanding the sequence of events makes it easier to tell when something is awry.

### Understand your infrastructure

How your application is built and deployed.