---
title: Configuration
layout: default
has_children: false
parent: Usage
grand_parent: 
nav_order: 1
---

# Configuration
`Traffic Refinery`'s configuration is defined using a JSON file. The default
configuration filename is `trconfig.json`. At runtime, `Traffic Refinery`
searches for the config file, first in the current directory and then in the
`/etc/traffic-refinery/` directory. To import a configuration from a file
located in a different location use the `-conf` option.

**NOTE: providing a configuration file is necessary.** 

For details on the configuration of `Traffic Refinery` please refer to
[`config`](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/internal/config).
Multiple examples are available in the
[configs](https://github.com/traffic-refinery/traffic-refinery/tree/main/configs)
directory in the repo. Here we provide an [example](https://github.com/traffic-refinery/traffic-refinery/tree/main/configs/trconfig_default.json)
config file, with explanations of each block.
### Sys
The `Sys` block controls whether different profiling functionality is enabled or
disabled, and where the output should be stored. 
```
{
  "Sys": {
    "CPUProf": false,
    "MemProf": false,
    "InterfaceStats": false,
    "OutFolder": "/tmp/"
  },
```
### Parsers
The `Parsers` block controls the interfaces, drivers, and mode to use for the
`DNSParser` and `TrafficParsers`, respectively. `Traffic Refinery` currently
supports `pcap`, `pfring`, and `afpacket` as drivers. Valid modes are `host`,
`router`, `mirror`, and `replay`. Modes control the behavior when determining
the direction of traffic by examining MAC addresses, details can be found in
[`network`](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/internal/network).
```
  "Parsers": {
    "DNSParser": {
      "Driver": "pcap",
      "Ifname": "eth0",
      "Mode": "router"
    },
    "TrafficParsers": [
      {
        "Driver": "afpacket",
        "Ifname": "eth0",
        "Mode": "router"
      }
    ]
  },
```
### FlowCache
The `FlowCache` block controls the size and eviction policy of the
[`flowcache`](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/internal/flowstats).
Currently, `Traffic Refinery` only supports `ConcurrentCacheMap` as the
flowcache type.
```
  "FlowCache": {
    "CacheType": "ConcurrentCacheMap",
    "EvictTime": 600000000000,
    "CleanupTime": 300000000000,
    "ShardsCount": 4096,
    "Anonymize": false
  },
```
### Stats
The `Stats` block controls the output of collected statistics. 
```
  "Stats": {
    "Run": true,
    "Mode": "video",
    "Append": true
  },
```
### DNSCache
The `DNSCache` block manages eviction and cleanup of the DNSCache.
```
  "DNSCache": {
    "EvictTime": 600000000000,
    "CleanupTime": 300000000000
  },
```
### Services
The `Services` block determines how flows are treated. Users can extend for any
service they are interested in. For each service, a name is required. Users can
input filters based on DNS domains using `DomainsString` and on IP prefixes
using `Prefixes`. `Collect` determines which features the user wants to collect
for a given service. In the example, video services such as YouTube and Netflix
collect both regular `PacketCounters` as well as video-specific `VideoCounters`
features. Lastly, `Emit` controls the interval for service statistic output.
```
  "Services": [
    {
      "Name": "Youtube",
      "Filter": {
        "DomainsString": ["youtube.com", "ytimg.com", "googlevideo.com"]
      },
      "Collect": ["PacketCounters", "VideoCounters"],
      "Emit": 10000000
    },
    {
      "Name": "Netflix",
      "Filter": {
        "DomainsString": ["netflix.com","nflxvideo.net","nflximg.net","nflxext.com","nflximg.com","nflxso.net"],
        "Prefixes": ["23.246.0.0/18", "37.77.184.0/21", "45.57.0.0/17", "64.120.128.0/17", "66.197.128.0/17", "108.175.32.0/20", "185.2.220.0/22", "185.9.188.0/22", "192.173.64.0/18", "198.38.96.0/19", "198.45.48.0/20", "208.75.79.0/24", "2620:10c:7000::/44", "2a00:86c0::/32"]
      },
      "Collect": ["PacketCounters", "VideoCounters"],
      "Emit": 10000000
    },
    {
      "Name": "Amazon",
      "Filter": {
        "DomainsString": ["amazon.com", "amazonvideo.com", "primevideo.com", "aiv-cdn.net", "avodassets-a.akamaihd.net"],
        "DomainsRegex": ["avod.*s3.*-.*.akamaihd.net", "amazon.*.llwnd.net", "amazon.*.lldns.net", ".*eu.amazon.fr"]
      },
      "Collect": ["PacketCounters", "VideoCounters"],
      "Emit": 10000000
    },
    {
      "Name": "Hulu",
      "Filter": {
        "DomainsString": ["hulu.com", "huluqa.com", "huluim.com", "hulustream", "hulu.conviva.com"],
        "DomainsRegex": [".*hulu.*.akamaihd.net", ".*hulu.*.edgekey.net",".*hulu.*.akadns.net"]
      },
      "Collect": ["PacketCounters", "VideoCounters"],
      "Emit": 10000000
    },
    {
      "Name": "Twitch",
      "Filter": {
        "DomainsString": ["twitch.tv", "ttvnw.net", "twitchcdn.net"]
      },
      "Collect": ["PacketCounters", "VideoCounters"],
      "Emit": 10000000
    }
  ]
}
```