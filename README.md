# ns.py: A Pythonic Discrete-Event Network Simulator

This discrete-event network simulator is based on [`simpy`](https://simpy.readthedocs.io/en/latest/), which is a general-purpose discrete event simulation framework for Python. `ns.py` is designed to be flexible and reusable, and can be used to connect multiple networking components together easily, including packet generators, network links, switch elements, schedulers, traffic shapers, traffic monitors, and demultiplexing elements.

# Installation

First, launch the terminal and create a new `conda` environment (say, called `ns.py`):

```shell
$ conda update conda
$ conda create -n ns.py python=3.8
$ conda activate ns.py
```

Then, install `ns.py` using `pip`:

```shell
$ pip install ns.py
```

That's it! You can now try to run some examples in the `examples/` directory. More examples will be added as existing components are refined and new components are introduced.

## Current Network Components

The network components that have already been implemented include:

* `Packet`: a simple representation of a network packet, carrying its creation time, size, packet id, flow id, source and destination.

* `DistPacketGenerator`: generates packets according to provided distributions of inter-arrival times and packet sizes.

* `TracePacketGenerator`: generates packets according to a trace file, with each row in the trace file representing a packet.

* `TCPPacketGenerator`: generates packets using TCP as the transport protocol.

* `PacketSink`: receives packets and records delay statistics.

* `TCPSink`: receives packets, records delay statistics, and produces acknowledgements back to a TCP sender.

* `Port`: an output port on a switch with a given rate and buffer size (in either bytes or the number of packets), using the simple tail-drop mechanism to drop packets.

* `REDPort`: an output port on a switch with a given rate and buffer size (in either bytes or the number of packets), using the Early Random Detection (RED) mechanism to drop packets.

* `Wire`: a network wire (cable) with its propagation delay following a given distribution. There is no need to model the bandwidth of the wire, as that can be modeled by its upstream `Port` or scheduling server.

* `Splitter`: a splitter that simply sends the original packet out of port 1 and sends a copy of the packet out of port 2.

* `NWaySplitter`: an n-way splitter that sends copies of the packet to *n* downstream elements.

* `TrTCM`: a two rate three color marker that marks packets as green, yellow, or red (refer to RFC 2698 for more details).

* `RandomDemux`: a demultiplexing element that chooses the output port at random.

* `FlowDemux`: a demultiplexing element that splits packet streams by flow ID.

* `FIBDemux`: a demultiplexing element that uses a Flow Information Base (FIB) to make packet forwarding decisions based on flow IDs.

* `TokenBucketShaper`: a token bucket shaper.

* `TwoRateTokenBucketShaper`: a two-rate three-color token bucket shaper with both committed and peak rates/burst sizes.

* `SPServer`: a Static Priority (SP) scheduler.

* `WFQServer`: a Weighted Fair Queueing (WFQ) scheduler.

* `DRRServer`: a Deficit Round Robin (DRR) scheduler.

* `VirtualClockServer`: a Virtual Clock scheduler.

* `SimplePacketSwitch`: a packet switch with a FIFO bounded buffer on each of the outgoing ports.

* `FairPacketSwitch`: a fair packet switch with a choice of a WFQ, DRR, or Virtual Clock scheduler, as well as bounded buffers, on each of the outgoing ports.

* `PortMonitor`: records the number of packets in a `Port`. The monitoring interval follows a given distribution.

* `ServerMonitor`: records performance statistics in a scheduling server, such as `WFQServer`, `VirtualClockServer`, `SPServer`, or `DRRServer`.

## Current utilities

* `TaggedStore`: a sorted `simpy.Store` based on tags, useful in the implementation of WFQ and Virtual Clock.

* `Config`: a global singleton instance that reads parameter settings from a configuration file. Use `Config()` to access the instance globally.

## Current Examples (in increasing levels of complexity)

* `basic.py`: A basic example that connects two packet generators to a network wire with a propagation delay distribution, and then to a packet sink. It showcases `DistPacketGenerator`, `PacketSink`, and `Wire`.

* `overloaded_switch.py`: an example that contains a packet generator connected to a downstream switch port, which is then connected to a packet sink. It showcases `DistPacketGenerator`, `PacketSink`, and `Port`.

* `mm1.py`: this example shows how to simulate a port with exponential packet inter-arrival times and exponentially distributed packet sizes. It showcases `DistPacketGenerator`, `PacketSink`, `Port`, and `PortMonitor`.

* `tcp.py`: this example shows how a two-hop simple network from a sender to a receiver, via a simple packet forwarding switch, can be configured, and how acknowledgment packets can be sent from the receiver back to the sender via the same switch. The sender uses a TCP as its transport protocol, and the congestion control algorithm is configurable (such as TCP Reno or TCP CUBIC). It showcases `TCPPacketGenerator`, `CongestionControl`, `TCPSink`, `Wire`, and `SimplePacketSwitch`.

* `token_bucket.py`: this example creates a traffic shaper whose bucket size is the same as the packet size, and whose bucket rate is one half the input packet rate. It showcases `DistPacketGenerator`, `PacketSink`, and `TokenBucketShaper`.

* `two_rate_token_bucket.py`: this example creates a two-rate three-color traffic shaper. It showcases `DistPacketGenerator`, `PacketSink`, and `TwoRateTokenBucketShaper`.

* `static_priority.py`: this example shows how to use two Static Priority (SP) schedulers to construct a more complex two-layer scheduler, turning on `zero_downstream_buffer` for the upstream scheduler and `zero_buffer` for the downstream one. It showcases `DistPacketGenerator`, `PacketSink`, and `SPServer`.

* `wfq.py`: this example shows how to use the Weighted Fair Queueing (WFQ) scheduler, and how to use a server monitor to record performance statistics with a finer granularity using a sampling distribution. It showcases `DistPacketGenerator`, `PacketSink`, `Splitter`, `WFQServer`, and `ServerMonitor`.

* `virtual_clock.py`: this example shows how to use the Virtual Clock scheduler, and how to use a server monitor to record performance statistics with a finer granularity using a sampling distribution. It showcases `DistPacketGenerator`, `PacketSink`, `Splitter`, `VirtualClockQServer`, and `ServerMonitor`.

* `drr.py`: this example shows how to use the Deficit Round Robin (DRR) scheduler. It showcases `DistPacketGenerator`, `PacketSink`, `Splitter` and `DRRServer`.

* `two_level_drr.py`, `two_level_wfq.py`, `two_level_sp.py`: these examples have shown how to construct a two-level topology consisting of Deficit Round Robin (DRR), Weighted Fair Queueing (WFQ) and Static Priority (SP) servers. They also show how to use strings for flow IDs and to use dictionaries to provide per-flow weights to the DRR, WFQ, or SP servers, so that group IDs and per-group flow IDs can be easily used to construct globally unique flow IDs.

* `red_wfq.py`: this example shows how to combine a Random Early Detection (RED) buffer (or a tail-drop buffer) and a WFQ server. The RED or tail-drop buffer serves as an upstream input buffer, configured to recognize that its downstream element has a zero-buffer configuration. The WFQ server is initialized with zero buffering as the downstream element after the RED or tail-drop buffer. Packets will be dropped when the downstream WFQ server is the bottleneck. It showcases `DistPacketGenerator`, `PacketSink`, `Port`, `REDPort`, `WFQServer`, and `Splitter`, as well as how `zero_buffer` and `zero_downstream_buffer` can be used to construct more complex network elements using elementary elements.

* `fattree_fifo.py`: an example that shows how to construct and use a FatTree topology for network flow simulation. It showcases `DistPacketGenerator`, `PacketSink`, `SimplePacketSwitch`, and `FairPacketSwitch`. If per-flow fairness is desired, `FairPacketSwitch` would be used, along with Weighted Fair Queueing, Deficit Round Robin, or Virtual Clock as the scheduling discipline at each outgoing port of the swtich.

## Writing New Network Components

To design and implement new network components in this framework, you will first need to read the [10-minute SimPy tutorial](https://simpy.readthedocs.io/en/latest/simpy_intro/index.html). It literally takes 10 minutes to read, but if that is still a bit too long, you can safely skip the section on *Process Interaction*, as this feature will rarely be used in this network simulation framework.

In the *Basic Concepts* section of this tutorial, pay attention to three simple calls: `env.process()`, `env.run()`, and `yield env.timeout()`. These are heavily used in this network simulation framework.

### Setting up a process

The first is used in our component's constructor to add this component's `run()` method to the `SimPy` environment. For example, in `scheduler/drr.py`:

```python
self.action = env.process(self.run())
```

Keep in mind that not all network components need to be run as a *SimPy* process (more discussions on processes later). While traffic shapers, packet generators, ports (buffers), port monitors, and packet schedulers definitely should be implemented as processes, a flow demultiplexer, a packet sink, a traffic marker, or a traffic splitter do not need to be modeled as processes. They just represent additional processing on packets inside a network.

### Running a process

The second call, `env.run()`, is used by our examples to run the environment after connecting all the network components together. For example, in `examples/drr.py`:

```python
env.run(until=100)
```

This call simply runs the environment for 100 seconds.

### Scheduling an event

The third call, `yield env.timeout()`, schedules an event to be fired sometime in the future. *SimPy* uses an ancient feature in Python that's not well known, *generator functions*, to implement what it called *processes*. The term *process* is a bit confusing, as it has nothing to do with processes in operating systems. In *SimPy*, each process is simply a sequence of timed events, and multiple processes occur concurrently in real-time. For example, a scheduler is a process in a network, and so is a traffic shaper. The traffic shaper runs concurrently with the scheduler, and both of these components run concurrently with other traffic shapers and schedulers in other switches throughout the network.

In order to implement these processes in a network simulation, we almost always use the `yield env.timeout()` call. Here, `yield` uses the feature of generator functions to return an iterator, rather than a value. This is just a fancier way of saying that it *yields* the *process* in *SimPy*, allowing other processes to run for a short while, and it will be resumed at a later time specified by the timeout value. For example, for a Deficit Round Robin (DRR) scheduler to send a packet (in `scheduler/drr.py`), it simply calls:

```python
yield self.env.timeout(packet.size * 8.0 / self.rate)
```

which implies that the scheduler *process* will resume its execution after the transmission time of the packet elapses. A side note: in our network components implemented so far, we assume that the `rate` (or *bandwidth*) of a link is measured in bits per second, while everything else is measured in bytes. As a result, we will need a little bit of a unit conversion here.

What a coincidence: the `yield` keyword in Python in generator functions is the same as the `yield()` system call in an operating system kernel! This makes the code much more readable: whenever a process in *SimPy* needs to wait for a shared resource or a timeout, simply call `yield`, just like calling a system call in an operating system.

**Watch out** for a potential pitfall: Make sure that you call `yield` at least once in *every* path of program execution. This is more important in an infinite loop in `run()`, which is very typical in our network components since the environment can be run for a finite amount of simulation time. For example, at the end of each iteration of the infinite loop in `scheduler/drr.py`, we call `yield`:

```python
yield self.packets_available.get()
```

This works just like a `sleep()` call on a binary semaphore in operating systems, and will make sure that other processes have a chance to run when there are no packets in the scheduler. This is, on the other hand, not a problem in our Weighted Fair Queueing (WFQ) scheduler (`scheduler/wfq.py`), since we call `yield self.store.get()` to retrieve the next packet for processing, and `self.store` is implemented as a sorted queue (`TaggedStore`). This process will not be resumed after `yield` if there are no packets in the scheduler.

### Sharing resources

The *Shared Resources* section of the 10-minute SimPy tutorial discussed a mechanism to request and release (either automatically or manually) a shared resource by using the `request()` and `release()` calls. In this network simulation framework, we will simplify this by directly calling:

```python
packet = yield store.get()
```

Here, `store` is an instance of `simpy.Store`, which is a simple first-in-first-out buffer containing shared resources in *SimPy*. We initialize one such buffer for each flow in `scheduler/drr.py`:

```python
if not flow_id in self.stores:
    self.stores[flow_id] = simpy.Store(self.env)
```

### Sending packets out

How do we send a packet to a downstream component in the network? All we need to do is to call the component's `put()` function. For example, in `scheduler/drr.py`, we run:

```python
self.out.put(packet)
```

after a timeout expires. Here, `self.out` is initialized to `None`, and it is up to the `main()` program to set up. In `examples/drr.py`, we set the downstream component of our DRR scheduler to a packet sink:

```python
drr_server.out = ps
```

By connecting multiple components this way, a network can be established with packets flowing from packet generators to packet sinks, going through a variety of schedulers, traffic shapers, traffic splitters, and flow demultiplexers.

### Flow identifiers

Flow IDs are assigned to packets when they are generated by a packet generator, which is (optionally) initialized with a specific flow ID. We use flow IDs extensively as indices of data structures, such as lists and dictionaries, throughout our framework. For example, in `scheduler/drr.py`, we use flow IDs as indices to look up our lists (or dictionaries, if strings are used for flow IDs) of deficit counters and quantum values:

```python
self.deficit[flow_id] += self.quantum[flow_id]
```

Most often, the mapping between flow IDs and per-flow parameters, such as weights in a Weighted Fair Queueing scheduler or priorities in a Static Priority scheduler, need to be stored in a dictionary, and then used to initialized these schedulers. An optional (but not recommended) style is to assign consecutive integers as flow IDs to the flows throughout the entire network, and then use simple lists of per-flow parameters to initialize the schedulers. In this case, flow IDs will be directly used as indices to look up these lists to find the parameter values.
