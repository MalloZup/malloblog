---
date: 2018-09-10T12:00:00-05:00
description: "The importance of having a code coverage tool for large codebases"
featured_image: "/images/codecoverage.jpeg"
tags: ["codebase analysis", "golang"]
title: "Codebase Forensinc Analysis with help of codecoverage"
---

This article is splitted in 2 parts:

  - Code coverage motivation
  - Case study: https://github.com/dmacvicar/terraform-provider-libvirt

first we will take a look on code coverage theory for studying and analyse codebases.
Then we will see a concrete case study, which is an opensource project I co-maintain:
https://github.com/dmacvicar/terraform-provider-libvirt.

___
# Why do we need a code coverage tool ? 

A modern codebase is like a city. (https://wettel.github.io/codecity.html).

Differents people use the city infrastructures and services without carying or knowing about the quality, or the internal functionality of things;
we take the metro for accomplish an objective ( going to city center, work etc).

In a codebase this is the same. When i call an external library like the `schema` library of terraform (https://godoc.org/github.com/hashicorp/terraform/helper/schema), i use this for another objective without knowing  how the library is coded internally.

For maintaining a project driven by a community or company project, tools like code coverage are essential for keep the codebase healty.

We can use code coverage for enforcing our scepticism. Scepticism is generally any questioning attitude or doubt towards one or more items of putative knowledge or belief.

A code coverage report is a **citymap**. You can observe which part of your city/codebase are mostly covered and which not. You can observe the flow and discover new patterns and regression in your code.
( It will be more clear on the case study).

You need always scepticism when analysing codecoverage reports: having a 100% covered codebase doesn't prevent you to have poor written tests that will be always pass, and let bugs explodes on productions.
(For the fun of green-tests and nosense tests have a look here: https://github.com/auchenberg/volkswagen/).

A codecoverage is a fantastic tool that give us a new  perspective about what is **not tested** and the risky parts of our city. 
Upon this information we can start an analyse and implementing a new tests that will cover that part of code.

If the tests is well written etc, this is out of scope of codecoverage tools.

What is usefull to keep in mind: a percentage like 80% code covered is just a statistic number. You need always to question how the test is written and don't be tempted to base your PR reviews on how code coverage is increased/decreased. This is from my pov a bad usage of code-coverage tools.
We need always to thinkg about why that part of code is not covered, think about a good test that could cover it, and if even a test is needed/worth to cover that part of code. 

If your project doesn't have a code coverage tool or a integrating it is difficult,  is a codesmell for bad architecture on your codebase.


# Case study: a [golang project](https://github.com/dmacvicar/terraform-provider-libvirt)

Before we start, we assume following:

* 1) Our tests (unit and integration and acceptance) should be **easy to run**.
* 2) The tests should live *in the same* (GitHub/GitLab) repository as the codebase.

Every language has it's different coverage tool. In the case study since the project is golang we stick on the go tools.

For the golang the 1 and 2 prerequisites are implemented by default by golang design choices.

If your codebase doesn't satisify the 2 prerequisites from my personal experience the project will be harder to maintain.

### 1st condition 
In the [terraform-provider-libvirt](https://github.com/dmacvicar/terraform-provider-libvirt) we satisfy the **first condition** with 1 command.

So assuming you are on the source code directory with
```bash 
make testacc
```
 
we then run the terraform-acceptance test framework and the code-coverage tool. 

The interesting part of the `make testacc` command :
```bash
export TF_ACC=true

go test -v -covermode=count -coverprofile=profile.cov -timeout=1200s ./libvirt
```
As you can see we activate the code coverage and run the tests with one command.

For reading more about golang coverage take look here: https://blog.golang.org/cover.

### 2nd condition 
The **2nd condition** the golang language enforce this by its natural design. The tests files are living in same dir as the codebase files.
For example: 

 * - codebase file ```resource_libvirt_domain.go``` 
 * - test_file. `resource_libvirt_domain_test.go` 

You can recognize this pattern on the repo. (https://github.com/dmacvicar/terraform-provider-libvirt/tree/master/libvirt)

## Analyze code coverage reports.

So once we have run our acceptance tests we have also a report file `profile.cov` from  our code coverage.

In golang you can do it with:
```bash
go tool cover -html=profile.cov
```
