# Prometheus docs

## Fundamental metrics

There are 4 fundamental metrics types:

- Gauge
- Counter
- Summary
- Histogram

## Metric vectors

- GaugeVec
- CounterVec
- SummaryVec
- HistogramVec

There are two interfaces, **Metric Interface** and the **Collector** interface.

Fundamental metrics implement both, metric Interface as well the Collector interface. but Vector versions do not implement Metric Interface.

> Note: Remember, there is something very important with **Opts** struct, GaugeOpts, CounterOps.

When you create a new metric, you are most probably implementing your own collector interface.

> Note: check goCollector, expvarCollector and processCollector.

A collector manages collection of metrics.
The collector and the metric have to describe themselves to a registry, to make sure there is no run-time error.

## Collector

In prometheus, Collector is the interface that is implemented by anything that can be used to collect metrics.

## Descriptor

A descriptor is the metadata related to a metric, they probably tell what the metrics look like.
