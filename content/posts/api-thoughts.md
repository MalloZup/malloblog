+++ 
date = 2020-10-04T11:22:10+02:00
title = "Versioning API and beyond thoughts. V0.1"
description = ""
slug = "" 
tags = []
categories = []
externalLink = ""
series = []
+++

# Introduction:

What i'm writing here comes from pragmatic experience, readings and inspiration from projects, languages (functionals), video etc.
I do have failed many times and I will do in the future.
I'm not proposing any magic formula allthought there might be some thoughts will inspire you.

I do think 80% of software projects do have a super bad design, because no one have ever thought about the API. From begin, or late. Or we don't have the time. 
Or we are **framing the problem** simply wrong.

Everything was incrementally done, nevertheless even the sprints/feedback of agile is not the magic formula.
Project are rewritten or gets abondoned because poor api, users and developers cannot consume it. 
There is lack of documentatition. People cannot even document it because they don't know what the software is doing. (developers and others)

It might sound to you a "irrealistic, pessimistic" scenario, but in reality this is how software is shipped nowdays.

We do have time constraints to deliver features, and once we deliver them, mostly they are irreversible ( I will reason and come later to this error).

We ship features tainted with support for X years, or even if the feature was a complete failure, you need to preserve the codebase which becomes tainted with zombie feature.
To be honest: I don't think that researching to the ground for a feature or a "defensife" approach towards new Ideas  is here the solution. 
In contrary, I think that beeing too defensive is just preventing good ideas to born and software innovation.

We need to be antifragile, and to fail lot of times, for creating ideas. Allthought we need to preserve to ship just failures to production.

We need to change our mental model how software is shipped and maintained.


# Why we create wrong and irreversible APIs from day 1?

To be honest, no one will ever write a good API at Version 1, an API involves different stackholders and usages. It need thinking and retrospective thinking.
No matter the skills and experience  as developer you have, you will fail, so do I.

Creating an API, involves taking critical choices with a limited knowledge. Only retrospectively you can know which was the right choice. 
Probably you can take a good decision from day 1, which involves lot of research, but in the real world, you don't have always **enough** time to research.
In the real world, the skilled developers or maintainers cannot maintain/supervise all the projects, or going to into details of X api.
No matter how you are passionated, accountable and think to be skilled developers, you will ship anyways bad API soon or later.
**Simply** because your knowledge is limited. You need to reflect retrospectively.

So soon or later your API will be broken, and removing it will be a **breaking change**. ( also how will come to this later.)

# Users don't care about the general picture, neither developers do it. * why api sucks

In the real world , in lot of legacy code or opensource code, developers are incrementally creating an API satifsying their needs, or the  User needs, compromising the whole design.
Time and effort is always the problem.
 -  people want to fix X feature in the shortest 

A practical example:

One of the project I'm involved in the opensource world, is the upstream project `terraform-libvirt`. ( https://github.com/dmacvicar/terraform-provider-libvirt)

The API of libvirt is an XML, terraform the provider needs to map it to  HCL and terraform constructs. 

In the past we had, and also in the present lot of users which send us pull-request for introducing a new feature which will change the API.
If we would merge this PRs the codebase will become un-maintainable. This is not about making people happy.

Obviously as many opensource contributors, they just need to fix their own usage, not thinking of the global picture.  I am not blaming, I might have done the same in past for X project.

It should be the role of an experienced maintainer to advise the contributors to change the code and adapt it to be generic to the codebase.
Allthough even maintainers can fail. We can't expect that all maintainers will invest the same "passion", accuracy, or even if they do , they will fail.
Some maintainers are just strategic, pushing some "showy" feature for beeing promoted or playing the "principal engineer" to stackholders and send to the hell to the "bare mortal" engineers who will need to live with that messy code,
and implement just the "details". Some devs just simple left company, or somehow you join to X codebase in a phase where no one as anymore a clue what is going on.

I think everyone is different, developers are human with their characters and organization. 

So to close the libvirt example, I think we are taking a rationale approach: trying to reviewing the PR as much we can, researching, trying to see the general picture. 
There are still failure on this.

But why do we ship to production the version 1 which becomes the `irreversible API` for all users and become a nightmare to maintain?

So the 1 version of API will be 90% globally wrong. You can be defensive, you can consult different stackholders,users, customers, opensource friends, it will be wrong. 

70% of developers simply don't care about reflecting on a API afterwards or have the time to do it.
Developers increments feature on 4/5 monts, the API is there, and it becomes the "API" officially released. 
Maybe some design choices was poor, and the API is simple exposing to much, which now you need to maintain or ship breaking changes afterwards.

On the otherside over-abstracting things beforehands is always creating super-complicated code. You can't be generic before, making code generic is something comes after you.

So in any case, overthinking or just creating features with a poor research or not enough time, will bring you to fail.

This is also why API are bad in general. 

But I think we can change this.


# shifting the mental model:


# Semantic versioning is not the whole solution of the problem

WIP:

# Why I like Kubernetes API:

WIP:

# Other solutions
