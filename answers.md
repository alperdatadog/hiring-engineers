My answers to the questions are here !!!

# Prerequisites - Setup the environment
Prior to the interview I had already signed up for Datadog to play along the tool and interfaces with my own company name Cogitos Consulting. After the interview I have changed the company name to “Datadog Recruiting Candidate” and start to write down this document.

I've gone with the ready captive environmet of mine and used one of my linux installs with Ubuntu distrubution with version 18.04 Bionic on VirtualBox.

# Collecting Metrics:

* Adding tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

My host shown on the Datadog Inventory Hostmap page prior to adding tags

<img src="https://live.staticflickr.com/65535/49649359148_8620b9abcb_c.jpg" width="800" height="403"></a>

I remove the comment out and added three tags to my host via editing datadog.yaml file

```

## @param tags  - list of key:value elements - optional
## List of host tags. Attached in-app to every metric, event, log, trace, and service check emitted by this Agent.
##
## Learn more about tagging: https://docs.datadoghq.com/tagging/
#
tags:
         - environment:dev
         - hostdbapp:pgsql
         - hostwebapp:tomcat

```

<img src="https://live.staticflickr.com/65535/49649939656_36421f2927_c.jpg" width="800" height="121"></a>

My Datadog web page after adding tags:

<img src="https://live.staticflickr.com/65535/49649511328_a7dbd50269_c.jpg" width="800" height="405"></a>

* Installing a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

installed PostgreSQL

```
      cogito@devops01:~$ sudo apt update
      cogito@devops01:~$ sudo apt install postgresql postgresql-contrib
      cogito@devops01:~$ sudo -u postgres psql
      [sudo] password for cogito:
      psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
      Type "help" for help.
      postgres=#

```
Installed PostgreSQL Integration using detailed descriptions on the Integration Menu. 

* Created PostgreSQL user "datadog" for Datadog, 
* Granted pg_monitor permissions for the user and check the permissions. 
* Edited /etc/datadog-agent/conf.d/postgres.d/conf.yaml file and entered the user credidentials created in the previous step to file.
* Restarted the Datadog Agent on server

<img src="https://live.staticflickr.com/65535/49649548993_0e8b934f61_c.jpg" width="800" height="403"></a>





Used pgbench to create some metrics

```
      postgres@devops01:~$ pgbench -c 25 -T 60 -S -n
      transaction type: <builtin: select only>
      scaling factor: 1
      query mode: simple
      number of clients: 25
      number of threads: 1
      duration: 60 s
      number of transactions actually processed: 411962
      latency average = 3.643 ms
      tps = 6861.908486 (including connections establishing)
      tps = 6862.216178 (excluding connections establishing)
      postgres@devops01:~$
```

Here we go, we have some meaningfull info on the Datadog page for PostgreSQL

<img src="https://live.staticflickr.com/65535/49650123566_d62bcab201_c.jpg" width="800" height="403">


* Createing a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

1. Created the directory my_metric.d/ in the conf.d/ folder at the root of your Agent’s configuration directory.
2. In my_metric.d/ folder, created an empty configuration file named my_metric.yaml with the following content:
```
instances: [{}]
```
3. Up one level from the conf.d/ folder, went to the checks.d/ folder. Created a custom check file named my_metric.py with the content below:

```
import random

from datadog_checks.base import AgentCheck

__version__ = "1.0.0"

class MyClass(AgentCheck):
    def check(self, instance):
        self.gauge(
            "my_metric.gauge",
            random.randint(0, 1001),
            tags=["env:dev","metric_submission_type:gauge","hostname:devops01","environment:dev"],
        )
```
4. Restart the Agent and cheked if the service started correctly

```

sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent

sudo systemctl status datadog-agent
  datadog-agent.service - Datadog Agent
   Loaded: loaded (/lib/systemd/system/datadog-agent.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-03-12 02:24:26 UTC; 15min ago
 Main PID: 18050 (agent)
    Tasks: 8 (limit: 4660)
   CGroup: /system.slice/datadog-agent.service
           └─18050 /opt/datadog-agent/bin/agent/agent run -p /opt/datadog-agent/run/agent.pid



```
5. Checked the agent status for newly added metric
```
sudo datadog-agent status
```
In the result of the command above, looked for the Collector and Running Checks

```
=========
Collector
=========

  Running Checks
  ==============

    my_check (1.0.0)
    ----------------
      Instance ID: my_check:d884b5186b651429 [OK]
      Configuration Source: file:/etc/datadog-agent/conf.d/my_check.d/my_check.yaml
      Total Runs: 13
      Metric Samples: Last Run: 1, Total: 13
      Events: Last Run: 0, Total: 0
      Service Checks: Last Run: 0, Total: 0
      Average Execution Time : 0s



```

Newly added "my_metric1" is present on the Host Map Menu

<img src="https://live.staticflickr.com/65535/49650279746_4e246fe57c_c.jpg" width="800" height="407">

* Changing my check's collection interval so that it only submits the metric once every 45 seconds.

I edited my_check.yaml file and added min collection interval and restarted the agent, checked agent and collectors

```
init_config:

instances:
    - min_collection_interval: 45

```
My metric's values as after change

<img src="https://live.staticflickr.com/65535/49651848572_a9f633f33e_c.jpg" width="800" height="316">

After 45 seconds another metric is collected:

<img src="https://live.staticflickr.com/65535/49651574516_87a0af4b32_c.jpg" width="800" height="335">

Bonus Alternatively you can edit it from the metrics summary page

<img src="https://live.staticflickr.com/65535/49651574586_afdd4653cb_c.jpg" width="800" height="401">



## Visualizing Data:

Utilizing the Datadog API to create a Timeboard that contains following information:

* My custom metric scoped over my host.

```

curl --location --request POST 'https://api.datadoghq.EU/api/v1/dashboard' \
--header 'Content-Type: application/json' \
--header 'DD-API-KEY: <API_KEY_VALUE>' \
--header 'DD-APPLICATION-KEY: <APPLICATION_KEY_VALUE>' \
--data-raw '{
    "title": "EXAMPLE TIMEBOARD",
    "widgets": [
               {
            "definition": {
                "title": "My Metric Graph",
                "yaxis": {
                    "max": "auto",
                    "include_zero": true,
                    "scale": "linear",
                    "min": "auto",
                    "label": ""
                },
                "show_legend": false,
                "time": {},
                "requests": [
                    {
                        "q": "avg:my_metric.gauge{host:devops01}",
                        "style": {
                            "line_width": "normal",
                            "palette": "dog_classic",
                            "line_type": "solid"
                        },
                        "display_type": "line"
                    }
                ],
                "type": "timeseries",
                "legend_size": "0"
            }
        },
        {
            "definition": {
                "alert_id": "113834",
                "type": "alert_graph",
                "viz_type": "timeseries",
                "title": ""
            }
        },
                {
            "definition": {
                "title": "Sum of my_metric.gauge over host:devops01",
                "text_align": "right",
                "precision": 0,
                "time": {},
                "requests": [
                    {
                        "q": "hour_before(sum:my_metric.gauge{host:devops01})",
                        "aggregator": "sum"
                    }
                ],
                "type": "query_value"
            }
        }
    ],
    "layout_type": "ordered",
    "description": "Dashboard creation utilisin API test",
    "is_read_only": true,
    "notify_list": [
        "voltran13@gmail.com"
    ],
    "template_variables": [
        {
            "name": "host",
            "prefix": "host",
            "default": "<HOSTNAME_1>"
        }
    ],
    "template_variable_presets": [
        {
            "name": "Saved views for hostname 2",
            "template_variables": [
                {
                    "name": "host",
                    "value": "<HOSTNAME_2>"
                }
            ]
        }
    ]
}'

```

My newly created dashboard's API request

<img src="https://live.staticflickr.com/65535/49652656767_edf5c53561_c.jpg" width="800" height="421">

Timeboards timefreame to set to 5 mins.

<img src="https://live.staticflickr.com/65535/49651915358_aa403aff90_c.jpg" width="800" height="394">

Sending snapshot of a graph to @myself

<img src="https://live.staticflickr.com/65535/49652465296_901590f393_c.jpg" width="800" height="509">

Mail I have received

<img src="https://live.staticflickr.com/65535/49652738647_105c2dd488_c.jpg" width="800" height="397" >

Bonus answer; I created a monitor on my PostgreSQL installation with anormaly dedection. To do that I looked for an anormaly on the number of connections within the last 5 mins. So if the number of connections are above the bounds for the last 5 mins it will create an alert to me. I used basic anormaly dedection. It's name is "Connections on PostgreSQL is too high".

<img src="https://live.staticflickr.com/65535/49652789542_67d8e4b3a0_c.jpg" width="800" height="403">

I need to generate the anormaly I designed in the previous step so With the pgbench utility I have created more than 50 connections.

```

pgbench -c 50 -T 300 -S -n -d postgres

```

As a result Datadog catched the anormaly and notified me via an e-mail.

<img src="https://live.staticflickr.com/65535/49651995528_e0f980c0cf_c.jpg" width="800" height="392">

After anormaly recovered Datadog catched the recovery and notified me via an e-mail

<img src="https://live.staticflickr.com/65535/49652531741_d7c9752d7e_c.jpg" width="800" height="396">

Alert in my Timeboard graph seen as below:

<img src="https://live.staticflickr.com/65535/49652831692_9295731013.jpg" width="500" height="176">

## Monitoring Data

I have created the Metric Monitor for my previously created custom metric

<img src="https://live.staticflickr.com/65535/49653506491_b7ecc6f75e.jpg" width="500" height="252">

Received warning and recovery mails afterwards:

Warning Mail
<img src="https://live.staticflickr.com/65535/49653773952_77f3c52d55.jpg" width="500" height="239">

Recovery Mail
<img src="https://live.staticflickr.com/65535/49652957708_02decf44bf.jpg" width="500" height="237">

Bonus:
I have added downtimes for Monday to Friday 19:00 to 09:00 and for weekends. The muting of monitor notifications are carried out via downtime configuration. Hence downtime configuration can be made with a starting date of future and a time frame after the start time, one needs to make two reoccuring downtime configuration to cover these time frames:

1. Schedule Downtime weekly on Mon, Tue, Wed, Thur, Friday and Sunday from 19:00 with a duration of 14 hours.
<img src="https://live.staticflickr.com/65535/49653642646_ee6e79fc8f.jpg">

2. Schedule Downtime weekly on Sat and Sunday, from 00:00 with a duration of 24 hours. This will cover a scheduled downtime until Sunday midnight. After midnight the first scheduled downtime will work as I already added a downtime at Sunday 19:00 + 14hrs there.

<img src="https://live.staticflickr.com/65535/49653642631_f3d6abedea.jpg" width="500" height="251">

Also added another scheduled downtime for tonight so I won't get any messages until morning ;)

## Collecting APM Data:

I used ddtrace for the given flask app and collected APM metrics. Below is a screenshot of flask service on my APM. To gather some meaningfull values I used a http stress tool to generate metrics like http status codes, durations, requests per second ect. My http stress tool worked prety well, as can be seen all graphs have some values.

<img src="https://live.staticflickr.com/65535/49655169687_e50f57116f.jpg" width="500" height="262">

<img src="https://live.staticflickr.com/65535/49655169707_3ccc007f8b.jpg" width="500" height="259">

Please find link and a screenshot of a new Dashboard with both APM and Infrastructure Metrics.

https://app.datadoghq.eu/dashboard/8sk-mqv-ajw/devops01-cloned?from_ts=1584100159575&live=true&tile_size=m&to_ts=1584103759575

<img src="https://live.staticflickr.com/65535/49654342408_dfd7d3edff.jpg" width="500" height="265">

## Final Question:
I would suggest a Covid-19 (Corona virus ) Dashboard with Cases, Deaths, Recovers with anomaly detection applied to all counters. I would also like to add heatmap with geographic location as gov.uk page (they made it with ArcGIS). I couldn't find a map integration in the documents. I have seen IASM RFP's from customers that asks for map integration or datacenter floorplan integration (infrastructure). 
Geographic information would be great for giving web site hit info with geo coordinates. 



