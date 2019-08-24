+++ 
draft = true
date = 2019-08-15T14:13:26+02:00
title = "01-systems-drawing: Nrepl Clojure design"
description = "Unconventional systems architecture"
slug = "" 
tags = ["systems", "clojure",  "lisp"]
categories = []
externalLink = ""
series = []
+++

# Motivation

This is the 02 unconventional system design series, on a Clojure ecosystem project.
By unconventional system design I mean I will not use any automated software tool for drawing and any design schema/conventions(UML etc).

I will draw systems the form I think it fit more for describing it, like a system mindmap.

The goal is to learn more about opensource systems design.

My interest behind Nrepl or why I choosed Nrepl was trying to understand the tool that I use often for developing with Clojure.

I am impressed by the really good developer design documentation provided from NREPL project ( sofar I think this is not the normal case in software).
And I will reference here the link and quote I am using.

My system draw and description has started from the documentation, assembling and looking also in the codebases for having a real sense of how things were working.
I have also looked at code but the fact that I was looking at the codebase with a deterministic goal, it changed my perspective on how to read the codebase. I was not just having a look.
I started to looking at code differently. ( Reading non familiar codebases without a perspective, problem or goal is really hard to do, or impossible)

Drawining a system is like a map, at some point you need to choose the balance level of how much in details you will go. But having the quest for drawing a map of Nrepl, gave me lot of motivation and insights.

The "unconventional map" I draw for NREPL it should give most of the key ponts to understand NREPL general architecture. It doesn't cover all the details and I think one could draw different draw/maps for the specifics components interactions etc. 
If you feel the motivation to continue it for NREPL feel to fork this and continue it! :)

# Some details:

In case some design will change, this post is based on commit: https://github.com/nrepl/nrepl/commit/21a3200a5dee737efa0dca7529cd2b69b41c7644
so you can restart from this snapshot.

# Nrepl 
![design](/nrepl.jpeg)

We can structure the picture in 7 key points:

* 1) The arch model of Nrepl
* 2) How data is exchanged
* 3) Transport abstraction
* 4) What is a nREPL server
* 5) Server Lifecycle
* 6) A Client of Nrepl
* 7) Nrepl session

## 1) The arch model of Nrepl

Nrepl use a server/client model. In the picture the Server green is a Nrepl server.
For simplicity and personal choices, I draw EMACS as client but it could be another client. (see https://nrepl.org/nrepl/usage/clients.html)
I will use CIDER/EMACS here for sake of simplicity but the same apply for other clients.

The highlevel definition from nREPL from upstream doc:

```
nREPL is a Clojure network REPL that provides a REPL server and client, along with some common APIs of use to IDEs and other tools that may need to evaluate Clojure code in remote environments.
```

For people not familiar with REPL read [here](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) 


# 2) How data is exchanged:

nREPL is  message-oriented and asynchronous.
A message is a Clojure map with some common specification.
The client on the picture(left side) is sending a message with ID 10 to server,  for "evaluating" "eval" some code. The example is simplified.
As an example it can be when you evaluate some code on CIDER-EMACS, a message will sent to the NREPL server, which can be on your workstation machine or remotely somewhere else. The server respond with the value and a "status" of the operations.

In the 2nd part we will analyze more in deep the specification of map, especially "op" operation field and the "session" field of the maps. ( see red color).

# 3) Transport abstraction:

Nrepl use Java [sockets](https://en.wikipedia.org/wiki/Network_socket), https://github.com/nrepl/nrepl/blob/master/src/clojure/nrepl/transport.clj#L14

For encoding/decoding the message it uses by default the BENCODE format, but other are for more info read this: https://nrepl.org/nrepl/design/transports.html

The transport namespace is usally what you will use when you develop a new custom middleware for nREPL.

The transport are analog to RING adapters, https://github.com/ring-clojure/ring/wiki

# 4) What is a nREPL server:

Everthing in nREPL is message driven, and an important specification is the "op" keyword of messages.

The "op" correspond to the operation that a client requeste on the server. It can be "eval" for evaluationg code, "interrupt" for interrupting, "load-file" for loading a file, etc. 

A nREPL server by default provide some basic functionality (built-in middlewares), like session, load-file etc.

Also nREPL can be extended, in our case we are using cider-nRepl (https://github.com/clojure-emacs/cider-nrepl), but when using other clients one could/need to  extend it with additional middlwares. https://github.com/nrepl/nrepl/wiki/Extensions#middlewares

A middleware allow developer to extend nREPL functionality.

You can read more here: https://nrepl.org/nrepl/design/middleware.html

# 5) nRepl Server lifecycle:

When starting a nREPL server it basically load all the default built-in middlewares and the custom (e.g the cider), and initialize the socket-based nREPL server.
By default BENCODE as transport is used but you can overwrite this.

After server init, it basic wait for incoming messages from clients

# 6) a Client of nREPL

Cider is the client for nREPL: it extend emacs for allowing the interactive development for clojure. https://docs.cider.mx/cider/index.html

Emacs-cider  https://github.com/clojure-emacs/cider is core principles are to send message to nREPL server and perform callbacks when the message from server are back.

# 7) Session of nREPL:

Session is a concept analog to the web-browser session.
From Upstream doc

`
Sessions are for tracking remote state. They’re fundamental to allowing simultaneous activities in the same nREPL. For instance - if you want to evaluate two expressions simultaneously you’ll have to do this in separate session, as all requests within the same session are serialized.
`

If you use Clojure and Clojurescript, Piggieback uses sessions to allow host a ClojureScript REPL alongside an existing Clojure one.



# Conclusion:


# Additional resources:

https://nrepl.org/nrepl/additional_resources.html
