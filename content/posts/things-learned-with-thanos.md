+++ 
date = 2020-12-22T12:26:17+01:00
title = "Things and projects learned within Thanos opensource project"
description = "A experience about thanos project"
slug = "" 
tags = ["monitoring", "observability"]
categories = []
externalLink = ""
series = []
+++

# Things learned with Thanos opensource proejct

Recently I started working and learning thanos concepts just for fun and as **opensource individual** (during my free time).

This blogpost just  summarize shortly some things I have learned, during some few days.

- 0) if you want to try out thanos check: https://katacoda.com/thanos/courses/thanos/1-globalview

- 1) Prometheus `mixins` and jsonnet concepts

- 2) Grpc protocol and middlewares

- 3) Rate limiting algorithm

- 4) Prometheus internals

- 5) CNCF Observability and other groups

- 6) Misc

## 0)

Thanos design doc: https://thanos.io/tip/thanos/design.md/

## 1) Prometheus mixin

From upstream, A mixin is a set of Grafana dashboards and Prometheus rules and alerts, packaged together in a reuseable and extensible bundle.

See for more details: https://monitoring.mixins.dev

Thanos mixins are located here:  https://github.com/thanos-io/thanos/tree/master/mixin

## 2) Grpc protocol and middlewares

I didn't used Grpc servers. ( https://grpc.io/)

I'm positive surprised by the standard of middleware and easyness of initialisation you can add on top of Grpc (middlewares) https://github.com/grpc-ecosystem/go-grpc-middleware.

# 3) Rate-Limit implementations and algorithms.

I took initiative on solving an issue on thanos with rate-limiting (is still pending), since Grpc has no method natively to do rate limiting.
During the journey, I researched various rate-limiting algorithms and mechanism. Some link about rate-limiting in golang:

* HTTP rate limiting https://dev.to/plutov/rate-limiting-http-requests-in-go-based-on-ip-address-542g
* GRPC rate limiting  https://jbrandhorst.com/post/go-semaphore/

# 4) Prometheus internals:

When reading some thanos code, you will endup learning prometheus internals since thanos import directy prometheus internals/code (via golang)

Another blog I want to read also : https://ganeshvernekar.com/blog/prometheus-tsdb-the-head-block

# 5) CNCF groups, thanos, prometheus  observability:

Thing I will want to follow-up next 2021 are definitely the opensource meetings . See https://www.cncf.io/calendar/

https://github.com/cncf/sig-observability

## 6) Misc:

Other things I appreciated in thanos:

* https://katacoda.com/thanos/courses/thanos/1-globalview is a "live" experience for using thanos and learning it. Really useful

* RFC or kind of architectural Decision log embedded in github repo: https://github.com/thanos-io/thanos/tree/master/docs/proposals

