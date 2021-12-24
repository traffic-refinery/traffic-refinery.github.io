---
title: Offline (pcap) Usage
layout: default
has_children: false
parent: Usage
grand_parent: 
nav_order: 4
---

# Running Traffic Refinery Offline (pcap)

`Traffic Refinery` can also be run on pcap files that have previously been
captured. We currently support this by essentially replaying the pcap using
`tcpreplay`. Below is an example to run in a Docker container. Note that the
mounted directory is where the configuration file and output files will be
searched for / stored. In this case, the Docker image would use /out, which
would be a mount to a directory on the host.

```
export DIRECTORY=/path/to/directory 
docker run --mount type=bind,source=$DIRECTORY,destination=/out --mount \
    type=bind,source=$DIRECTORY/trconfig_replay.json,destination=/config/trconfig_replay.json,readonly \
    --rm netmicroscope:replay -c /config/trconfig_replay.json -w e4:ce:8f:01:4c:54 \
    -t /out/clean_dump.pcap
```