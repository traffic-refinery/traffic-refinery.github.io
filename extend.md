---
title: Extending Features
layout: default
has_children: false
parent: Usage
grand_parent: 
nav_order: 2
---

# Extending Traffic Refinery for Additional Features
`Traffic Refinery` facilitates the exploration of how different representations affect model performance and collection cost by allowing for quick implementation of user-defined collection methods for features and their aggregated statistics.

Implemented features are added as separate files inside the folder [internal/counters](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/internal/counters). Traffic Refinery uses the configuration file to obtain the list of service class definitions and the features to collect for each one of them automatically. Upon execution, the system uses Go’s language run-time reflection to load all available feature classes and select the required ones based on the system configuration. 

## Implementing a new feature
Implementing a new feature requires the definition of one structure and the implementation of five functions, i.e., a structure that implements the [internal/counters](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/internal/counters#Counter) interface.

```
type Counter interface {
	// Updates the flow states based on the packet
	AddPacket(pkt *network.Packet) error
	// Reset resets the counter statistics for periodic counting.
	// This function triggers at the "emit" time.
	Reset() error
	// Clear re-initializes the counter to its zero state
	Clear() error
	// Type returns a string with the type name of the counter.
	Type() string
	// Collect returns a []byte representation of the counter
	Collect() []byte
}
```

See [PacketCounters](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/internal/counters#PacketCounters) as a sample implementation.

### Packet processing 

The `AddPacket` function is in charge of updating the counter state based on the new packet to process. The packet information is passed using the [Packet](https://pkg.go.dev/github.com/traffic-refinery/traffic-refinery/internal/network#Packet) structure which contains both pre-processed packet information as well as access to the raw bytes of the packet.

__NOTE:__ buffers used to store the raw bytes of the packet are re-used for new incoming packets. Any information that has to be retained should be copied into new memory.

### Reset and Clear

The Reset and Clear functions have similar purposes in that they both restore the state of the counter to an initial one. The difference is the following:
* The Reset function should be used to bring the counter to the zero state. In fact, it is invoked by the packet processing pipeline right after creating the counter to make sure the counter is at the correct initial state.
* The Clear function should be used to update the state of the counter between emit cycles.

### Type

The type function is solely used to provide a string representation of the counter's name.

### Collect

The Collect function is invoked at emit time to collect a raw bytes representation of the state of the counter at the end of the emit cycle. In its current version, system assumes that the returned bytes can be manipulated as a [JSON RawMessage](https://pkg.go.dev/encoding/json#RawMessage).

