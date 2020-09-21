+++ 
date = 2020-09-21T18:19:18+02:00
title = "Intelligent systems with prometheus"
description = "How to use for fun and profit alert-manager"
slug = "" 
tags = ["code", "prometheus"]
categories = []
externalLink = ""
series = []
+++

# Intelligent systems with prometheus


Cloud native systems are complex systems. Hardware fails, software has bugs, things get wrong.

In this scenario Prometheus offer a valid help for creating more resilient systems, whichg can auto-heal themself in certains circustances.

In this demo I will show how you can use the Prometheus stack to improve your systems.


# What do you need for getting the pattern


I do simplify lot of things assuming you can setup prometehus/alertmanager and you can create an alert via prometehus ( plenty of doc upstream are avail)


1) Install prometheus. In my setup I do openSUSE Linux but Prometheus binary and alertmanager are avail everywhere


2) After you setup alertmanager and prometheus up and running, configure the alertmanager.

`/etc/prometheus/alertmanager.yml`

```
# See https://prometheus.io/docs/alerting/configuration/ for documentation.

global:
receivers:
      - name: my-receiver
        webhook_configs:
          - url: "http://10.162.30.75:9000/hooks/"

route:
  group_by: ["alertname"]
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10s
  receiver: my-receiver
  routes:
    -  receiver: my-receiver
```

This configuration say that whatever there is an alert we send a webhook .

After this restart `alertmanager` service.


3) Create the `alerthook-handler`. I used golang because I find it provides the right minimalism and efficency for my sake.

```golang

package main

import (
        "io"
        "net/http"

        log "github.com/sirupsen/logrus"
)

func main() {
        log.Infoln("starting handler")
        handler1 := func(w http.ResponseWriter, _ *http.Request) {
                io.WriteString(w, "Hello from a HandleFunc handler called via Prometheus alert\n")
                log.Infoln("handler called via prometheus alert")
        }

        http.HandleFunc("/hooks", handler1)
        err := http.ListenAndServe(":9999", nil)
        if err != nil {
                log.Fatalln(err)
        }
}
```

Build this code and deploy to the IP node


4) So assuming prometehus is up and running, and you have a fake alert for testing.

```
hana01:/tmp # ./my-handler 
INFO[0000] starting handler                             
INFO[0016] handler called via prometheus alert          
INFO[0019] handler called via prometheus alert          
INFO[0019] handler called via prometheus alert  
```

Here we do have 3 alerts fired and handled by handler

Now if you substitute the `print` function with your domain expertise! Have fun!
