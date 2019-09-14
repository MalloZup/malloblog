---
title: "Monitoring an HA Cluster"
date: 2019-09-13T16:59:40+02:00
---

# Version 0.0.1:

This blog is incremental. Each version will bring new features in term of content.

# HA Cluster intro

https://clusterlabs.org/

Clusterlabs is in production since 1999 and it still works. The difference between other nowdays cluster is that is a resource cluster.

A resource can be for example Apache. A resource doesn't need to be in a container, it can but can be in a bare metal host etc.

Unlike many other cluster systems, with **only 2 nodes** you can have a full functional cluster. ( this is a big advantage, where other system require at least 3)


# Architecture with monitoring perspective of HA


![design](/monitoring_ha.jpg)


# Monitoring

The `ha_cluster_exporter` https://github.com/MalloZup/ha_cluster_exporter is a prometheus exporter which will monitor `ha-cluster` information and serve them in the prometheus format.

As you can see it is focused on data. THe source of truth will stay the cluster.xml status data (CIB).

In point (1) shows that the connector/retrivial data component can be easy changed. Currenlty the exporter uses `crm_mon`
It isn't difficult to to change/add the connector/retrivial in future , as functional language teach us or message passing systems, if you expose your data , you can easy change the interfaces/components.


