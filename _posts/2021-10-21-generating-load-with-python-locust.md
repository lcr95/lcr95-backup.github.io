---
layout: post
title: "Generating Load with Locust"
subtitle: ""
date: 2021-10-21
author: "ChenRiang"
header-style: text
tags:
    - Performance
    - Python
    - Locust
---



{% include image.html src="locust-icon.jpeg" data="group" title="" %}



**Locust** is a performance testing tools just like JMeter that generate load but written in Python. The main feature of Locust is the expandability, you can write the behavior of whole performance test in Python code without worry about the limitation of UI and domain specific language in JMeter. Because of this, Locust has a wide and fast-growing community, who prefer this framework over JMeter.



# Locust vs JMeter

|                                        | Locust                                                       | JMeter                                                       |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Language                               | Python                                                       | Java                                                         |
| License                                | MIT                                                          | Apache 2.0                                                   |
| Protocols                              | HTTP, HTTPS, [Custom plugins](https://github.com/SvenskaSpel/locust-plugins#locust-plugins) | HTTP, HTTPS, SOAP/XML-RPC, JDBC, LDAP, JMS, POP3, IMAP, SMTP, FTP |
| Test Creation                          | Python Scripting                                             | GUI tools, XML                                               |
| Concurrent User                        | Yes                                                          | Yes                                                          |
| Distributed Execution                  | Yes                                                          | Yes                                                          |
| Ramp Up                                | Yes                                                          | Yes                                                          |
| Test as Code                           | No                                                           | Yes                                                          |
| Resources needed to generate more load | High                                                         | Higher                                                       |
| Test Analysis                          | Yes                                                          | Yes                                                          |



# Locust Quickstart

In this section, we will create a quick test by spinning up Locust cluster to hit a simple Python server.



### Python Server (Target)

Run a Python simple test server with the command below :

```bash
python3 -m http.server
```



### Locust Cluster

**Installation**

```bash
pip3 install locust
```

** Locust require Python version 3.6.



**Preparation** 

Before spinning up the Locust cluster, we will need to write a Locustfile which contained the test logic.

<u>locustfile.py</u>

```python
from locust import task, FastHttpUser

class MyUser(FastHttpUser):
    @task
    def index(self):
        response = self.client.get("/")
```



**Spin up master** 

```bash
locust -f locustfile.py --master
```



**Spin up worker**

```bash
locust -f locustfile.py --worker --master-host=localhost
```



See [here](https://docs.locust.io/en/stable/running-locust-distributed.html#options) for more options.



To assure worker is successfully connected to master, we can check the log in master.

> [2021-10-20 21:28:51,738] GBNB4603/INFO/locust.main: Starting web interface at http://0.0.0.0:8089 (accepting connections from all network interfaces)
>
> [2021-10-20 21:28:51,797] GBNB4603/INFO/locust.main: Starting Locust 2.2.1
>
> [2021-10-20 21:30:48,267] GBNB4603/INFO/locust.runners: **Client 'GBNB4603_c9355b2231a94b92b2b8e3e3894b08f0' reported as ready. Currently 1 clients ready to swarm.**
>
> [2021-10-20 21:31:29,477] GBNB4603/INFO/locust.runners: Sending spawn jobs of 1 users at 1.00 spawn rate to 1 ready clients
>
> [2021-10-20 21:31:29,538] GBNB4603/INFO/locust.runners: All users spawned: {"MyUser": 1} (1 total users)
>
> [2021-10-20 21:31:39,030] GBNB4603/INFO/locust.runners: Removing GBNB4603_c9355b2231a94b92b2b8e3e3894b08f0 client from running clients
>
> [2021-10-20 21:31:39,030] GBNB4603/INFO/locust.runners: Client 'GBNB4603_c9355b2231a94b92b2b8e3e3894b08f0' reported as ready. Currently 1 clients ready to swarm.

<br/>



Once everything is up and running, we can open up the UI console by browsing, [http://localhost:8089](http://localhost:8089).

{% include image.html src="locust-ui.png" data="group" title="" %}



To start generating load to the Python server that we created just now, simply key in `http://localhost:8000` at the "**Host**" column and press the "**Start swarming**" button.



You should now see something like this.

{% include image.html src="start-generating-load.png" data="group" title="" %}

{% include image.html src="charts.png" data="group" title="" %}

<br/>



To download the report, click on "**Download Data**" > "**Download Report**". A new tab like below will appear.

{% include image.html src="download-report.png" data="group" title="" %}

<br/>



You might be noticed that the chart in the report has an interval of 5 second. Sometimes if you would run a long running test, you probably don't want the interval to be so short. Thus, you can set the config `locust.stats.HISTORY_STATS_INTERVAL_SEC ` to your desired number. 

```python
from locust import task, FastHttpUser
import locust.stats

class MyUser(FastHttpUser):
    locust.stats.HISTORY_STATS_INTERVAL_SEC = 30

    @task
    def index(self):
        response = self.client.get("/")
```

{% include image.html src="stat-interval-30s.png" data="group" title="" %}

<br/>



You now probably be asking, is there a way to pass in some parameters without restarting the cluster. Let's see the following example. 

We will now take in an extra parameter called "Endpoint" which enable us to have the flexibility calling different endpoint without restarting the cluster. 



```python
from locust import task, events, FastHttpUser

class MyUser(FastHttpUser):

    @events.init_command_line_parser.add_listener
    def _(parser):
        parser.add_argument("--custom-endpoint", type=str, default="endpoint1", help="Custom Endpoint")

    @task
    def index(self):
        response = self.client.get(f"/{self.environment.parsed_options.custom_endpoint}")
```



{% include image.html src="custom-parameters.png" data="group" title="" %}



# Conclusion 

In this blog post, we compared Locust and JMeter, learned some basic usage of Locust. I used both tools to carry out performance testing, both have its pro and cons. If you are a Python lover and prefer coding more than using a GUI tools just go with Locust. If you are not comfortable with Python scripting and prefer interacting with a GUI tool to create a performance test then JMeter is the better option.



<br/>

**Reference** 

1. JMeter vs. Locust - Which One Should You Choose? - [link](https://www.blazemeter.com/blog/jmeter-vs-locust-which-one-should-you-choose) 
2. Load Tests: Locust vs Jmeter - [link](https://azevedorafaela.com/2020/05/23/load-tests-locust-vs-jmeter/)

