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
 ![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/AddingMetrics1.PNG)

![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/AddingMetrics2.PNG)
 
```rate(http_request_count_total[1m]) = average rate of increase per second```
 ![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/AddingMetrics3.PNG)

```
http_request_latency_seconds_count = count for different endpoints 

http_request_latency_seconds_sum = total amount of time spent in these different endpoints

rate(http_request_latency_seconds_sum[1m])/rate(http_request_latency_seconds_count[1m])
= average latency over the last minute 
```

## Adding Grafana
```
rate(http_request_count_total{job="webapp"}[1m])

rate(http_request_latency_seconds_sum[1m])/rate(http_request_latency_seconds_count[1m])
= average latency over the last minute
```
 ![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/AddingGrafana1.PNG)
## Infrastructure Metrics

cAdvisor (Container Advisor) provides container users an understanding of the resource usage and performance characteristics of their running containers. It is a running daemon that collects, aggregates, processes, and exports information about running containers. Specifically, for each container it keeps resource isolation parameters, historical resource usage, histograms of complete historical resource usage and network statistics. This data is exported by container and machine-wide. cAdvisor has native support for Docker containers and should support just about any other container type out of the box. (Works for Big Query as well)

```
def cpu():
    # For older machines, you may want to lower
    # this range to prevent timeouts.
    for i in range(10000):
        i**i
   return 'cpu intensive operation complete'
```
```
def memory():
    d = {}
    # For older machines, you may want to lower
    # this range to prevent timeouts.
    for i in range(10000000):
        i = str(i)
        i += "xyz"
        d[i] = i

    return 'memory intensive operation complete'
```
```
def create_app():
    main_app = Flask(__name__)
    main_app.add_url_rule('/', 'index', index)
    main_app.add_url_rule('/foo', 'foo', foo) # how to check latency, requests in basic metrics
    main_app.add_url_rule('/cpu', 'cpu', cpu)
    main_app.add_url_rule('/memory', 'memory', memory)
    setup_metrics(main_app)

    # Add prometheus wsgi middleware to route /metrics requests
```
 ![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/infrastructuremetrics.PNG)

## Adding ML Metrics Monitoring 

#### Adding ml_api GET/200 and ml_api/v1/predictions/regression 200 endpoints to Requests Rate and Latency dashboard

![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/AddingMLmetrics.PNG)

## Creating an ML System Grafana Dashboard

```
from gradient_boosting_model import __version__ as shadow_version
from regression_model import __version__ as live_version
from prometheus_client import Histogram, Gauge, Info
from api.config import APP_NAME
...
def populate_database(n_predictions: int = 500, anomaly: bool = False) -> None:


if anomaly:
        # set extremely low values to generate an outlier
        n_predictions = 1
        clean_inputs_df.loc[:, "FirstFlrSF"] = 1
        clean_inputs_df.loc[:, "LotArea"] = 1
        clean_inputs_df.loc[:, "OverallQual"] = 1
        clean_inputs_df.loc[:, "GrLivArea"] = 1

    for index, data in clean_inputs_df.iterrows():
        if index > n_predictions:
            if anomaly:
                print('Created 1 anomaly')
...
```
![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/CreatingaMLsystemDashboard.PNG)
```
Average House Price Prediction Amount(USD) = 
sum(rate(house_price_prediction_dollars_sum{job="ml_api"}[1m]))/
sum(rate(house_price_prediction_dollars_count{job="ml_api"}[1m]))

Average House Price Prediction Rate(/second)=
rate(house_price_prediction_dollars_count{job="ml_api"}[1m])

Standard Error of the Mean(SEM) =
avg_over_time(house_price_gauge_dollars[1m])---(AVG)---
stdev_over_time(house_price_gauge_dollars[1m])---(STD)---
stdev_over_time(house_price_gauge_dollars[1m])/(sqrt(count_over_time(house_price_prediction_dollars_count[1m]))) (SEM)

Z-Score = (avg_over_time(house_price_gauge_dollars[1m])-avg_over_time(house_price_gauge_dollars[1w])) / stdev_over_time(house_price_gauge_dollars[1w]) 

Min Prediction = house_price_gauge_dollars

```
#### Z-score - if the z scores goes to a high number or a negative number, it indicates that the average value in the more recent time frame is dipping multiple standard deviations above or below the long term average. And this would indicate that something is going wrong with our model.

### With --anomaly

![](https://github.com/LewisRa/Machine-Learning-Deployments/blob/master/markdownImages/CreatingaMLsystemDashboard-withOutlier.PNG)
```
