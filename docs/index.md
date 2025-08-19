# htsim Network Simulator

<figure markdown="span">
  ![Image title](https://opengraph.githubassets.com/80aa7e13e0a31a9f73574983c717257927e2e751179d14a5a988acac61f18f65/Broadcom/csg-htsim)
  <figcaption>htsim is now part of Broadcom</figcaption>
</figure>

htsim is a high-performance discrete event network simulator designed for datacenter networking research. It leverages optimized C++ implementation and specialized datacenter models to enable fast, accurate simulation of transport protocols and congestion control algorithms at scale.

htsim supports comprehensive datacenter networking research across multiple dimensions:

<table>
  <tr>
    <td><a href="#transport-protocols">TCP NewReno</a></td>
    <td><a href="#transport-protocols">NDP (Near-optimal)</a></td>
    <td><a href="#transport-protocols">Swift</a></td>
 </tr>
   <tr>
    <td><a href="#transport-protocols">RoCE (RDMA)</a></td>
    <td><a href="#transport-protocols">HPCC</a></td>
    <td><a href="#transport-protocols">DCTCP</a></td>
 </tr>
 <tr>
    <td><a href="#transport-protocols">EQDS</a></td>
    <td><a href="#datacenter-topologies">FatTree Topology</a></td>
    <td><a href="#datacenter-topologies">DragonFly</a></td>
 </tr>
 <tr>
    <td><a href="#traffic-patterns">Permutation</a></td>
    <td><a href="#traffic-patterns">Incast</a></td>
    <td><a href="#traffic-patterns">All-to-All</a></td>
 </tr>
 <tr>
    <td><a href="#routing-strategies">Source Routing</a></td>
    <td><a href="#routing-strategies">ECMP</a></td>
    <td><a href="#routing-strategies">Adaptive Routing</a></td>
 </tr>
</table>

Corresponding research papers and use cases can be found in [Multipath TCP](http://nets.cs.pub.ro/~costin/files/mptcp-nsdi.pdf), [NDP](http://nets.cs.pub.ro/~costin/files/ndp.pdf), and [EQDS](http://nets.cs.pub.ro/~costin/files/eqds.pdf) publications. For implementation details, see the [algorithm overview](algorithm.md) and [architecture documentation](architecture.md).

## **Installation**

Installation requires only a modern C++ compiler with no external dependencies:

```bash
git clone https://github.com/Broadcom/csg-htsim.git
cd csg-htsim/sim
make
```

For datacenter experiments, build the specialized simulators:

```bash
cd datacenter
make

# Available executables:
# htsim_tcp, htsim_ndp, htsim_swift, htsim_roce, htsim_hpcc, htsim_eqds
```

## **Quick Start**

We start with a simple NDP experiment on a 16-node datacenter network with permutation traffic:

```bash
# Generate traffic matrix
cd connection_matrices
python3 gen_permutation.py perm_16n_16c.cm 16 16 0 0 0

# Run NDP experiment
cd ..
./htsim_ndp -nodes 16 -tm connection_matrices/perm_16n_16c.cm -cwnd 50 -strat ecmp_host -paths 16 -log sink -q 50 -end 1000
```

After running the simulation, you can analyze the results:

```bash
# View summary statistics
../parse_output logout.dat -ndp -show

# Generate throughput plots
python3 create_throughput_plot.py logout.dat ndp experiment_results
```

The simulator outputs detailed performance metrics including throughput, latency, queue utilization, and flow completion times for comprehensive protocol evaluation.

!!! tip "Multiple Protocols"
    Replace `htsim_ndp` with `htsim_tcp`, `htsim_swift`, or any other protocol simulator to compare performance under identical conditions.

## **Core Components**

htsim's modular architecture separates concerns for maximum flexibility and performance:

### Event-Driven Simulation Engine

The core simulator provides picosecond-precision discrete event scheduling with optimized data structures for handling millions of events efficiently. The event system supports both absolute and relative time scheduling with sophisticated cancellation and rescheduling mechanisms.

### Transport Protocols  

Each protocol implements its own congestion control algorithm while sharing common infrastructure:

- **TCP NewReno**: Traditional AIMD congestion control with fast recovery
- **NDP**: Pull-based protocol with return-to-sender and path diversity  
- **Swift**: Delay-based congestion control with multi-subflow support
- **RoCE**: Rate-based protocol with credit flow control for lossless networks
- **HPCC**: High-precision control using in-band network telemetry
- **DCTCP**: ECN-based protocol for datacenter environments
- **EQDS**: Sophisticated scheduling with speculative transmission

### Datacenter Infrastructure

Comprehensive datacenter modeling including FatTree topologies, ECMP routing, PFC (Priority Flow Control), and realistic switch models with configurable buffer sizes and switching latencies.

## **Modularity**

By default, htsim experiments consist of topology generation, traffic matrix creation, protocol simulation, and result analysis run in sequence. However, the system assumes independence between these components, making htsim highly modular for research customization:

You can swap out any of these components or customize them extensively. The following aspects are completely modular:

1. **[Transport Protocols](protocols.md)** - Implement custom congestion control algorithms
2. **[Network Topologies](topologies.md)** - Design specialized datacenter architectures  
3. **[Traffic Patterns](traffic.md)** - Create realistic workload models
4. **[Routing Strategies](routing.md)** - Develop adaptive forwarding mechanisms
5. **[Queue Management](queuing.md)** - Customize buffer management and scheduling
6. **[Logging and Analysis](analysis.md)** - Extract detailed performance metrics

To learn more about the underlying simulation engine and customization options, see the [architecture guide](architecture.md).

## **Overview**

htsim provides extensive functionality for datacenter networking research. Below you'll find comprehensive method overviews organized by common usage patterns.

### Common Simulation Commands

| Operation | Command  |
|-----------------------|---|
| Run NDP experiment    |  `./htsim_ndp -nodes N -tm matrix.cm -strat ecmp_host` |
| Generate traffic matrix  |  `python3 gen_permutation.py out.cm N N 0 0 0` |
| Analyze results    |  `../parse_output logout.dat -ndp -show` |
| Create throughput plots   | `python3 create_throughput_plot.py logout.dat ndp results`  |
| Run multi-protocol comparison     |  `./compare_protocols.sh matrix.cm results/` |
| Generate incast traffic | `python3 gen_incast.py incast.cm N 1 2000000` |
| Test different queue sizes | `./htsim_ndp [params] -q 50` |
| Enable detailed logging | `./htsim_ndp [params] -log sink,queue,traffic` |
| Set simulation duration | `./htsim_ndp [params] -end 5000` |
| Configure routing strategy | `./htsim_ndp [params] -strat perm,ecmp_host,ecmp_ar` |

### Protocol Parameters

After configuring your simulation, several parameters control protocol behavior. These parameters directly affect congestion control performance and should be tuned based on your experimental goals.

| Parameter | Description |
|------------------------|-----------------------------------------------------------------------------------------------|
| `-cwnd N`               | Initial congestion window size in packets |
| `-q N` | Queue size in packets for switch buffers |
| `-strat strategy`           | Routing strategy: perm, ecmp_host, ecmp_ar, etc.                                                                      |
| `-paths N`          | Number of parallel paths for multipath protocols             |
| `-mtu N` | Maximum transmission unit in bytes                           |
| `-end T`              | Simulation duration in microseconds                                       |
| `-log types`          | Enable logging: sink, queue, traffic, switches                                  |
| `-nodes N`          | Network size (number of nodes in topology)                                                          |
| `-tm file`         | Traffic matrix file specifying communication patterns                      |
| `-hop_latency T`      | Per-hop wire latency in microseconds                                |\n| `-switch_latency T`   | Switching/processing latency in microseconds                             |

### Experiment Variations

htsim supports diverse experimental scenarios for comprehensive protocol evaluation across different network conditions and traffic patterns.

| Experiment Type | Configuration  |
|-----------------------|---|
| **[Protocol Comparison](experiments/comparison.md)** | `./compare_protocols.sh traffic_matrix.cm results/` |
| **[Traffic Pattern Analysis](experiments/traffic.md)** | `python3 gen_incast.py matrix.cm N sinks flow_size` |
| **[Topology Scaling](experiments/scaling.md)** | `./htsim_ndp -nodes 64,128,256 -tm matrix.cm` |
| **[Queue Sensitivity](experiments/queues.md)** | `./run_queue_sweep.sh 10,50,100,200 matrix.cm` |
| **[Routing Strategy Evaluation](experiments/routing.md)** | `./htsim_ndp -strat perm,ecmp_host,ecmp_ar` |
| **[Multi-Path Load Balancing](experiments/multipath.md)** | `./htsim_ndp -paths 1,4,8,16 -strat ecmp_host` |
| **[Flow Size Distribution](experiments/flows.md)** | `python3 gen_mixed_flows.py matrix.cm sizes.dist` |
| **[Network Failure Scenarios](experiments/failures.md)** | `./htsim_ndp -failure_links links.txt -failure_time T` |
| **[Congestion Control Tuning](experiments/tuning.md)** | `./parameter_sweep.sh protocol param_range matrix.cm` |

### Visualizations and Analysis

Performance evaluation in networking research requires comprehensive visualization and statistical analysis capabilities to understand protocol behavior under different conditions.

| Analysis Type | Command  |
|-----------------------|---|
| **Throughput Time Series**    |  `python3 create_throughput_plot.py logout.dat protocol results` |
| **Flow Completion Time CDF**    |  `python3 plot_fct_cdf.py logout.dat results` |
| **Queue Utilization Heatmap** | `python3 plot_queue_heatmap.py logout.dat results` |
| **Protocol Comparison**    |  `python3 compare_protocols.py results1/ results2/ comparison` |
| **Latency Distribution**  |  `python3 plot_latency_dist.py logout.dat results` |
| **Network Utilization** | `python3 plot_network_util.py logout.dat results` |
| **Congestion Analysis**  |  `python3 analyze_congestion.py logout.dat results` |
| **Path Diversity Metrics**    |  `python3 plot_path_usage.py logout.dat results` |
| **Statistical Summary** | `python3 generate_summary.py logout.dat > summary.txt` |

## **Research Applications**

htsim has been extensively used in networking research and protocol development across academia and industry:

### Protocol Development

- **Multipath TCP**: Original IETF standardization performance evaluation
- **NDP**: Complete protocol design and evaluation from conception to publication
- **EQDS**: Advanced datacenter congestion control development
- **Swift**: Delay-based congestion control algorithm validation

### Academic Research

- **Datacenter Architecture Studies**: FatTree, DragonFly, and BCube topology comparison
- **Congestion Control Evaluation**: Comprehensive protocol performance analysis
- **Traffic Engineering**: Load balancing and routing strategy optimization  
- **Network Buffer Sizing**: Switch buffer requirements and performance trade-offs

### Industry Applications

- **Product Development**: Protocol implementation validation before hardware deployment
- **Network Planning**: Capacity planning and architecture design validation
- **Performance Optimization**: Parameter tuning and configuration optimization
- **Failure Analysis**: Network resilience and failure recovery mechanism testing

### Publications and Impact

Major networking conferences have featured research using htsim including SIGCOMM, NSDI, CoNEXT, and INFOCOM. The simulator has enabled reproducible research with publicly available configurations and datasets.

!!! tip "Research Collaboration"
    htsim's deterministic results and comprehensive logging make it ideal for reproducible research. Many published studies provide complete experimental configurations for replication.

## **Getting Started with Research**

For researchers new to htsim, we recommend the following progression:

1. **[Quick Start Guide](quickstart.md)** - Basic simulation setup and execution
2. **[Protocol Tutorial](tutorials/protocols.md)** - Understanding transport protocol implementations  
3. **[Traffic Modeling](tutorials/traffic.md)** - Creating realistic workload patterns
4. **[Performance Analysis](tutorials/analysis.md)** - Extracting meaningful insights from results
5. **[Advanced Experiments](tutorials/advanced.md)** - Multi-factor studies and parameter sweeps
6. **[Custom Development](tutorials/development.md)** - Implementing new protocols and features
