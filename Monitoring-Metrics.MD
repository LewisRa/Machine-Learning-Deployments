# ML Monitoring - Metrics

## Metrics for Machine Learning Systems
Real time metrics configured and automatically aggregated and displayed
by our monitoring system as opposed to some of the ad hoc manual statistical tests are very important. 

Metrics could be low level usage summaries provided by the operating system or they can be higher level types of data tied to specific functionality or work of a component such as requests per second or outputs from a particular function.

#### Pros of Metrics 
1. Constant overhead-  with metrics an increase in traffic to an application will not incur a significant increase in disk utilization processing complexity speed of visualization and operational costs.

2. Ideally suited for dashboard - Unlike the case with logs, numbers are optimized for storage, processing, compression and retrieval metrics enable longer retention of data as well as easier querying.This makes metrics perfectly suited to building dashboards that reflect historical trends.

3. Well suited for alerting -  metrics better suited to report the overall health of a system and also bettersuited to trigger alerts since running queries against an in-memory time series database is far more efficient

#### Cons of metrics

1. Not as much information as logs

2. Cardinality challenges -  cardinality is the number of elements of the sets (Amount of data)  and using high cardinality values like user I.D. as metric labels can overwhelm time series databases. All popular existing time series database solutions suffer performance on the high cardinality labeling. 

3. Scoped tot a single system



Once a model is built and goes live people assume it will continue working as normal machine learning models will actually degrade in quality over time sometimes quickly.This is known as **concept drift** and this means that the predictions offered by static machine learning
models become less accurate less useful as time goes on. Metrics are central and key to tracking this degradation.



**One of the simplest things we can do is to look at the score distribution produced by the model.If that distribution changes surprisingly there is a good chance that there is an important change tothe input to the model reflecting some change in the outside world or the system.**

---
## Prometheus and Grafana
## Basic Setup
## Adding Metrics
```
localhost:9090 = web ui for prometheus
Using the Expresssion search bar, choose 'http_request_count_total'
```

rate(http_request_count_total[1m])
## Adding Grafana
## Infrastructure Metrics
## Adding ML Metrics Monitoring 
## Creating an ML System
