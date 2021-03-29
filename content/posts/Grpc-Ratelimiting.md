+++ 
date = 2020-12-30T14:36:38+01:00
title = "an humble gRPC rate-limiting tutorial"
description = ""
slug = "" 
tags = ["grpc", "golang", "backend"]
categories = []
externalLink = ""
series = []
+++
# Summary:

- 1) Rationale 
- 2) Live Demo
- 3) Some Code details
- 4) Disclaimer 

# 1) Rationale:

Today I want to show you a possible solution, to a well-known problem when dealing with backend APIs or/and WEB applications.

**The problem**: imagine today is a special day, like 1 week before Christmas. Your servers will probably reach an abnormal higher number of client request.

If you don't have **a mechanism for limiting the number of requests**,  your servers resources, your servers or/and application will crash, or having a bad behaviour  which will be probably will led to a massive *service outage*.
 (depending on your SREs or company size :P ) 

For solving this problem, we need to focus into  the API/server backend application, usually we adopt rate-limiting techniques.. Of course one could cluster an application, but clustering or make HA a broken APP/service, doesn't seem a good idea. :P


I will show a `pragmatic` implementation  of rate-limiting and algorithms, in `golang` and `gRPC` server ecosystem.

Some useful references:

- If you aren't familiar with gRPC, I suggest you following resources: https://grpc.io/docs/what-is-grpc/introduction/

- If you aren't familiar with Rate-Limiting, check out this: https://cloud.google.com/solutions/rate-limiting-strategies-techniques

- The **code used here**  is *opensource* **avail** at: https://github.com/MalloZup/pegasus


# 2) Live Demo:

`First` **Download** the live demo video here: https://github.com/MalloZup/pegasus/blob/main/ratelimiting.mp4?raw=true  

The video showcase a grpc server/client on terminal with rate-limiting.
As you will see, I used the `interceptor` and TockenBucket approach for implementing the rate-limiting in a gRPC server. Obviously the problem can be solved in other ways.

For example using other algorithms..

# 3) Some Code details


Following code illustrate how to create your rate-limiter. 

1) creating the rate-limiter
```
type rateLimiterInterceptor struct {
        // using TockenBucket 
	TokenBucket *ratelimit.Bucket
}

// this function is the predicate which limits the requests. True -> rate-limiting
func (r *rateLimiterInterceptor) Limit() bool {
	// debug
	fmt.Printf("Token Avail %d \n", r.TokenBucket.Available())

	// if zero we reached rate limit, so return true ( report error to Grpc)
	tokenRes := r.TokenBucket.TakeAvailable(1)
	if tokenRes == 0 {
		fmt.Printf("Reached Rate-Limiting %d \n", r.TokenBucket.Available())
		return true
	}

	// if tokenRes is not zero, means gRpc request can continue to flow without rate limiting :)
	return false
}
```

2) Registering it to the gRPC server:

Here I just register my previous implemented rate-limiter as middleware to gRPC server.

Note this pattern is pretty much the standard in gRPC or http where you register middleware 
```
	limiter := &rateLimiterInterceptor{}
        // gatherTime is the second when the tocken will refilled, the capacity is the RateLimiting request we tollerate.
        // Example: if set gatherTime to 30 and capacity 20, this means following:
        // we tollerate 20 request which could be like been executed in 1 seconds,
        // and after 30 seconds a new token will added , so only 1 request can be executed.
	limiter.TokenBucket = ratelimit.NewBucket(gatherTime, int64(tokenCapacity))
	s := grpc.NewServer(
		// init the Ratelimiting middleware registration
		grpc_middleware.WithUnaryServerChain(
			grpc_ratelimit.UnaryServerInterceptor(limiter),
		),
		grpc_middleware.WithStreamServerChain(
			grpc_ratelimit.StreamServerInterceptor(limiter),
		),
	)
```

*Note*:  I took some arbritary values and implemented one possible way, for demo purposes.

I skip the whole gRPC and protobuf documentation/explication where you can find upstream.

* Interceptor is a middleware used by the grpc ecosystem.

* For the RateLimit Tocken bucket I used https://github.com/juju/ratelimit library

# Disclaimer:

During this time I researched in my free-time, as **individual opensource contributor and citzen** how to deal and implement rate-limiting in golang, for a gRPC server.
This lines represent my own thoughts, and efforts and research for improving myself and researchs as individual.

P.S: I thanks the #thanos opensource community for support in this research, which I initially picked up and started from my own initiative for researching on an upstream issue,  and which I'm not yet finished to do :) https://github.com/thanos-io/thanos

If you find something wrong or invalid, feel free to ping me on twitter or slack.



