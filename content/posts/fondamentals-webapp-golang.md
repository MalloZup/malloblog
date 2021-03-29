+++ 
date = 2021-03-29T10:49:37+02:00
title = "Some fondamentals for web application and microservices in golang"
description = ""
slug = "" 
tags = []
categories = ["web", "go"]
externalLink = ""
series = []
+++


# Intro:

I will write in this post about some useful concept you need for building web application in golang.
Also I will compare such approaches to other languages and upstream approaches from my developer carreer into opensource world.


# 1) Web toolkit are the golang minimalist answer against complexity of frameworks

Choosing a framework is a safe choice. People has already solved the problem for you. Really ??
This can be quite considerable risk, since you embark in a mental pattern, bugs and coding style that you might don't know if it fits for you.

As pro for frameworks, it is true that one don't want  to reinvent the wheel each time. 
But as contra, you inherit a huge complexity and bugs for free!

In this context, in golang has emerged the concept of "web toolkit".

Gorilla is one developed by google. https://www.gorillatoolkit.org/

Such approach is used in other contexts like "microservices", example: https://gokit.io/


A web-toolkit is a collection of tools/library you can plug to http, incrementally, without embarking on huge dependencies risk.

If you neeed a http router, my personal suggestion is to use `gorilla/mux`webtoolkit or even `chi` as router.

https://www.gorillatoolkit.org/ or https://github.com/go-chi/chi

Both are safe choices because  they aren't a framework and they use the standard go handlers HTTP handlers signature, so in case want to revert your choices, you can still swap them without much hassle.

# 2) Templating:

WIP
