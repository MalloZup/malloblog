+++ 
date = 2020-09-27T15:46:59+02:00
title = "Self-healing systems: Instrumenting applications with Prometheus alerts"
description = "Technical overview how to fire alerts from your application"
slug = "" 
tags = []
categories = []
externalLink = ""
series = []
+++

# Selfhealing systems part 2: 

In the previous blog post I covered how to use Prometheus as basis for building selfhealing systems. (https://mallozup.github.io/posts/self-healing-systems-with-prometheus/)

Today I will explain how you can **instrument your application, creating alert, which are fired inside production code** and then handle them automatically, without human intervention.

![result](/sh.arch.png)

# Instrumenting your application with Prometheus alerts

A selfhealing system need to have, among others components, 2 components:

- 1) a "reactor" or alert-handeler component which handle automatically alerts and react with your domain logic on this alerts. 

- 2) a "sensor" component which fires alerts if something goes wrong. Distributed on your distributed system and based on your domain logic.


## A real world example:

Imagine your team build a Web-Application, and for X reasons your web-application is making the disk full.

### Traditional way:

The "traditional" way would be that at some point, the **backend** of the web-app, will write **some logs** and at some points start to failing.

Consequently the WEBUI will report some errors on the user interface and then the whole operation team will take care on this scenario.

`Note:` maybe some backends are more smarts in some errors handling, I just consider a normal scenario, for sake of simplicity.


### The self-healing approach:

The self-healing approach is **mostly automated** (unless unexpected failures which cannot solved automatically by software layer are fired).  

In the WEBAPP scenario it will be:

* When the disk is starting to be full the backend will write logs and **fire automatically an alert to prometheus Alertmanager**. ( for technical explanation see later)
* The alert handler component will handle the full-disk alert fired by your application. ( e.g cleaning up trash files, migrating resources or depending on your architecture).
  In case the alert-handler cannot handle such alert automatically, it will fire a prometheus alert with highest prio, stating that either it failed to self-healing or something went wrong.
  In this case an experienced operator need to take care, analysing logs and taking the right actionl

In a self-healing approach, people will be notfied only when the self-healling systems is broken, or when the alerts cannot be resolved automatically. 

This is why, alerts cannot replace logs but they are complementary.

Also it might be useful to instrument your application with alerts in case you don't own the application code or instrumenting this code require to much efforts upstream or in legacy code.
You can still write your own component.



## Coding snippets:

### How to send an alert from your business application.

Prometheus AlertManager offer an API which can be used by your application. (https://prometheus.io/docs/alerting/latest/clients/)

My application backend uses `golang`. **Note** the code snippets are keeped simple as possible so you can understand them quickly.  They can for sure be improved.

1) Fire alert from your `application`

```golang
     if diskFull == True {  // your critical condition
       log.Error("full disk ... etc")
       // .. 
	var a *AlertFire
	a = new(AlertFire)
	a.Status = "firing"
	a.Labels.Alertname = "FOO-ALERT"
	a.Labels.Component = "unit-test component"
	a.Labels.Severity = "critical"
	a.Labels.Instance = "test instance"
	a.Annotations.Summary = "just a test"
	a.GeneratorURL = "unit-test"
        // this is your prometheus alertmanager server
	a.sendAlert("http://10.162.31.2:9093/api/v1/alerts")
    }
```

2)  This "logic behind `sendAlert` method

```golang
// AlertFire is the payload from https://prometheus.io/docs/alerting/latest/clients/
type AlertFire struct {
	Status string `json:"status"`
	Labels struct {
		Alertname string `json:"alertname"`
		Severity  string `json:"severity"`
		Component string `json:"component"`
		Instance  string `json:"instance"`
	} `json:"labels"`
	Annotations struct {
		Summary string `json:"summary"`
	} `json:"annotations"`
	GeneratorURL string `json:"generatorURL"`
}

func (alert *AlertFire) sendAlert(url string) {
        // You can send 20 alerts, I just send 1 for simplicity
	alerts := make([]AlertFire, 1)
	alerts[0] = *alert

        // the alerts are the Json Payload you will send via POST
	body := alerts
	buf := new(bytes.Buffer)
	json.NewEncoder(buf).Encode(body)
	req, err := http.NewRequest("POST", url, buf)
	if err != nil {
		log.Errorf("Error sending http post alert %s", err)
	}
	client := &http.Client{}
	res, err := client.Do(req)
	if err != nil {
		log.Error(err)
	}
	log.Infof("Alert from handler to alertmanager sent %s", alert.Labels.Alertname)
	defer res.Body.Close()
}
```
___

The  code will create following alert. You can handle the alert then automatically or a human need to resolve it
![result](/alert.png)


# Real example:

The rationanle behind this is that, in my project, I needed to self-handle alerts and also raise an alert in case the handler crash for some reasons, since it necessary no human intervention.

https://github.com/MalloZup/hasap-alerts-handler

