---
title: Cost Profiling
layout: default
has_children: false
parent: Usage
grand_parent: 
nav_order: 2
---

# Cost Profiling
`Traffic Refinery` aims to provide an intuitive platform to evaluate the system
cost effects of active data representations. We have initially included the
capacity for profiling three metrics: state, processing, and storage. We choose
these metrics as they directly affect the ability of a measurement system to
collect features from network traffic, a fundamental prerequisite for all
learning pipelines. 

Many additional cost metrics could be useful for a given environment (e.g.,
model training time, energy cost of training, model size, etc.). Users are
encouraged to extend `Traffic Refinery` profiling functionality for their
specific use case.

We use Go's built-in benchmarking features and implement dedicated tools to
profile the different costs intrinsic to the collection process. At data
representation design time, users employ the profiling method to quickly iterate
through the collection of different features in isolation and provide a fair
comparison for the three cost metrics. The relevant profiling code is located in
the [cmd](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/cmd)
directory.

`Traffic Refinery` supports two modes for profiling feature costs: 
1. Profiling from live traffic: in this setting the system captures traffic from
   a network interface and collects statistics for a configurable time interval.
2. Profiling using offline traffic traces: in this setting profiling runs over
   recorded traffic traces, which enables fine-grained inspection of specific
   traffic events (e.g., a single video streaming session) as well as
   repeatability and reproducibility of results. 
   
To select the sets of user-defined features to profile, the profiling tool takes
as input the same system configuration file used for executing the system. Upon
execution, the system creates a dedicated measurement pipeline that collects
statistics over time. 

## State costs
We aim to collect the amount of in-use memory over time for each feature class
independently. To achieve this, we use Go's `pprof` profiling tool. Using this
tool, the system can output at a desired instant a snapshot of the entire in-use
memory of the system. We extract from this snapshot the amount of memory that
has been allocated by each service class at the end of each iteration of the
collection cycle, i.e., the time the aggregation and storage module gathers the
data from the cache, which corresponds to peak memory usage for each interval.

## Processing costs 
To evaluate the CPU usage for each feature class, we monitor the amount of time
required to extract the feature information from each packet, leaving out any
operation that shares costs across all possible classes, such as processing the
packet headers or reading/writing into the cache. To do so, we build a dedicated
time execution monitoring function that tracks the execution of each `AddPacket`
function call in isolation, collecting running statistics (i.e., mean, median,
minimum, and maximum) over time. This method is similar in spirit to Go's
built-in benchmarking feature but allows for using raw packets captured from the
network for evaluation over longer periods of time.

## Storage costs
Storage costs can be compared by observing the size of the output generated over
time during the collection process. The current version of the system stores
this file in JSON format (and can support binary JSON output) without
implementing any optimization on the representation of the extracted
information.  