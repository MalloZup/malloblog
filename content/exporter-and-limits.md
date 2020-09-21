---
title: "Creating an exporter and its limits, a retrospective"
date: 2019-10-31T11:47:54+01:00
draft: true
---

# Rationale:

This article will talk about the design, development https://github.com/ClusterLabs/ha_cluster_exporter from a technical perspective.

It will be an occasion to a retrospective on this project.


# History:

The ha_cluster_exporter 0.0.1 was first  released  on september 2019 https://github.com/ClusterLabs/ha_cluster_exporter/tree/0.0.1 .

Lot of things changed right now to the actual version `0.1.0`: 

- the architecture of the project was refactored favorizing prometheus collector API
- lot of new metrics (pacemaker, drbd, corosync) in a main pkg. The design is solid.
- the documentation has a valid structure and can easy expanded by each pr.
- logging and other small features

There is a short note on this: https://github.com/ClusterLabs/ha_cluster_exporter/blob/master/doc/design.md

# Some decisions:

As you can see, the project doesn't have a state to maintain. This was initially implemented and thought to have a functional approach on the whole exporter.

In fact this is also possible thanks to the pacemaker cluster which maintain the state.
