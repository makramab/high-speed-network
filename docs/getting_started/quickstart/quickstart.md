## **Installation**

htsim is a high-performance discrete event network simulator written in C++ with zero external dependencies. Installation is straightforward and requires only a modern C++ compiler.

### Prerequisites

- **C++ Compiler**: g++ or clang with C++11 support
- **Operating System**: Linux or macOS
- **Build Tools**: make (typically pre-installed)

### Clone the Repository

```bash
git clone https://github.com/Broadcom/csg-htsim.git
cd csg-htsim
```

### Build the Simulator

```bash
# Build the core simulation library
cd sim
make

# Build datacenter experiments
cd datacenter
make
```

After successful compilation, you should see several executables:

```bash
ls -la htsim_*
# Expected output:
# htsim_tcp    - TCP NewReno simulator
# htsim_ndp    - NDP (Near-optimal Datacenter Protocol) simulator  
# htsim_swift  - Swift congestion control simulator
# htsim_roce   - RoCE (RDMA over Converged Ethernet) simulator
# htsim_hpcc   - HPCC (High Precision Congestion Control) simulator
# htsim_eqds   - EQDS (Earliest Queue Discharge Service) simulator
```

## **Quick Start**

We'll start with a simple experiment using NDP (Near-optimal Datacenter Protocol) on a 16-node network to demonstrate the basic workflow.

### Step 1: Generate a Traffic Matrix

Traffic matrices define the communication patterns between nodes. Let's create a permutation pattern where each node sends data to exactly one other node:

```bash
cd connection_matrices
python3 gen_permutation.py perm_16n_16c_quickstart.cm 16 16 0 0 0
```

This creates a traffic matrix file with the following parameters:

- **16 nodes** in the network
- **16 connections** (one per node)  
- **Size 0** means infinite/continuous flows
- **Start time 0** means all flows begin immediately

<figure markdown="span">
  ![Image title](https://i.imgur.com/FrB4rm3.png){ width="600" }
  <figcaption>You can open perm_16n_16c_quickstart to see what the generated matrix looks like</figcaption>
</figure>

### Step 2: Run Your First Experiment

Now let's run an NDP experiment using this traffic matrix:

```bash
cd .. # back to datacenter directory
./htsim_ndp -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -cwnd 50 -strat ecmp_host -paths 16 -log sink -q 50 -end 1000 -mtu 4000
```

**Parameter explanation:**

- `-nodes 16`: Create a 16-node FatTree datacenter topology
- `-tm [file]`: Use our generated traffic matrix
- `-cwnd 50`: Initial congestion window of 50 packets
- `-strat ecmp_host`: Use ECMP routing with host-based path selection
- `-paths 16`: Allow up to 16 parallel paths per flow
- `-log sink`: Enable throughput logging at receivers
- `-q 50`: Set queue size to 50 packets
- `-end 1000`: Run simulation for 1000 microseconds
- `-mtu 4000`: Set MTU to 4000 bytes

**Expected output:**

```
Starting simulation with 16 nodes...
[PLACEHOLDER: Typical simulation progress output]
Simulation completed in X.XX seconds
Results written to logout.dat
```

<figure markdown="span">
  ![Image title](https://i.imgur.com/aFp6Lte.png){ width="600" }
  <figcaption>Make sure you are inside datacenter directory when running the command</figcaption>
</figure>

### Step 3: Analyze the Results

#### Quick Summary

Get average throughput for all flows:

```bash
../parse_output logout.dat -ndp -show
```

**Expected output:**

```
[PLACEHOLDER: Average throughput results]
Flow throughputs: X.XX Gbps average
Total network utilization: XX%
```

#### Detailed Analysis with Plots

Generate time-series throughput plots:

```bash
python3 create_throughput_plot.py logout.dat ndp quickstart_results
```

This creates several files:

- `quickstart_results_timeseries.dat` - Raw throughput data over time
- `quickstart_results_plot.gp` - Gnuplot script for visualization
- `quickstart_results_throughput.pdf` - Generated plot (if gnuplot installed)

<figure markdown="span">
  ![Image title](https://i.imgur.com/vhZpcXA.png){ width="600" }
  <figcaption>One example figure that could be generated from htsim simulation</figcaption>
</figure>

## **Understanding Your Results**

### Throughput Data

The timeseries data file contains four columns:

```
# Time(ms) AvgThroughput(Gbps) TotalThroughput(Gbps) NumFlows
0.250 [PLACEHOLDER] [PLACEHOLDER] 16
0.500 [PLACEHOLDER] [PLACEHOLDER] 16
0.750 [PLACEHOLDER] [PLACEHOLDER] 16
```

### Performance Metrics

Key metrics to look for:

- **Average Throughput**: Per-flow performance
- **Total Throughput**: Network-wide data transfer rate  
- **Flow Completion Time**: How long each flow takes (for finite flows)
- **Queue Utilization**: Buffer usage at switches

### What Good Results Look Like

- **High throughput**: Close to link capacity
- **Low latency**: Fast flow completion
- **Fairness**: Similar performance across flows
- **Stability**: Consistent performance over time

## **Next Steps**

### Compare Different Protocols

Run the same experiment with different transport protocols:

```bash
# TCP NewReno
./htsim_tcp -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -q 100 -log sink -end 1000

# Swift congestion control
./htsim_swift -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -cwnd 50 -log sink -end 1000

# RoCE
./htsim_roce -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -q 200 -log sink -end 1000
```

### Try Different Traffic Patterns

#### Incast (Many-to-One)

Simulates scenarios like distributed storage or web server responses:

```bash
cd connection_matrices
python3 gen_incast.py incast_16n.cm 16 1 2000000  # 15 senders to 1 receiver, 2MB flows
cd ..
./htsim_ndp -nodes 16 -tm connection_matrices/incast_16n.cm -cwnd 50 -strat ecmp_host -log sink -end 2000
```

#### All-to-All

High network utilization scenario:

```bash
cd connection_matrices  
python3 gen_serial_alltoall.py alltoall_16n.cm 16 1000000  # 1MB flows between all pairs
cd ..
./htsim_ndp -nodes 16 -tm connection_matrices/alltoall_16n.cm -cwnd 30 -strat ecmp_host -log sink -end 5000
```

### Experiment with Parameters

Try different configurations to see their impact:

```bash
# Larger network (if you have time)
python3 connection_matrices/gen_permutation.py perm_64n.cm 64 64 0 0 0
./htsim_ndp -nodes 64 -tm connection_matrices/perm_64n.cm -cwnd 50 -strat ecmp_host -log sink -end 1000

# Different queue sizes
./htsim_ndp -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -q 10 -log sink -end 1000   # Small buffers
./htsim_ndp -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -q 200 -log sink -end 1000  # Large buffers

# Different routing strategies
./htsim_ndp -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -strat perm -log sink -end 1000      # Source routing
./htsim_ndp -nodes 16 -tm connection_matrices/perm_16n_16c_quickstart.cm -strat ecmp_ar -log sink -end 1000   # Adaptive routing
```

### Visualization with gnuplot

If you have gnuplot installed, generate publication-quality plots:

```bash
# Install gnuplot (if needed)
# Ubuntu/Debian: sudo apt-get install gnuplot
# macOS: brew install gnuplot

# Generate plots
gnuplot quickstart_results_plot.gp
open quickstart_results_throughput.pdf  # macOS
# or use your preferred PDF viewer
```

## **Common Issues & Troubleshooting**

### Build Errors

**Problem**: Compilation fails with C++11 errors

```bash
# Solution: Ensure you have a modern compiler
g++ --version  # Should be 4.8+ or equivalent clang
```

**Problem**: "make: command not found"

```bash
# Solution: Install build tools
# Ubuntu/Debian: sudo apt-get install build-essential
# macOS: Install Xcode Command Line Tools
```

### Runtime Issues

**Problem**: Simulation runs but no output generated

```bash
# Check if you included logging flags
./htsim_ndp -nodes 16 -tm your_matrix.cm -log sink -end 1000
#                                        ^^^^^^^^^ Required for analysis
```

**Problem**: "Traffic matrix file not found"

```bash
# Ensure file exists and path is correct
ls -la connection_matrices/your_file.cm
# Use absolute path if needed
./htsim_ndp -tm `pwd`/connection_matrices/your_file.cm [other options]
```

**Problem**: Very low throughput results

```bash
# Check queue sizes - too small queues cause drops
./htsim_ndp [options] -q 100  # Try larger queue sizes

# Check simulation duration - might be too short
./htsim_ndp [options] -end 5000  # Run longer simulation
```

### Analysis Issues

**Problem**: parse_output not found

```bash
# Build the parser
cd sim
make parse_output

# Or use relative path
../parse_output logout.dat -ndp -show
```

**Problem**: Python plotting script fails

```bash
# Check Python version
python3 --version  # Should be 3.6+

# Try with explicit path
python3 /full/path/to/create_throughput_plot.py logout.dat ndp results
```

!!! tip "Getting Help"
    - Check the [wiki](https://github.com/Broadcom/csg-htsim/wiki) for detailed documentation
    - Look at example experiments in `sim/EXAMPLES/` directory
    - Examine existing traffic matrices in `connection_matrices/` for patterns
    - Use smaller networks (4-8 nodes) for faster debugging

!!! warning "Performance Note"
    Large simulations (1000+ nodes) can take significant time and memory. Start small and scale up gradually.

## **What's Next?**

Now that you've run your first htsim experiment, you can:

1. **Explore the research examples** in `sim/EXAMPLES/` directory
2. **Read the detailed documentation** to understand protocol internals
3. **Run the complete experiment suites** in the `experiments/` directory
4. **Develop your own transport protocols** using the existing implementations as templates
5. **Create custom traffic patterns** for your specific research needs

Happy simulating! 🚀
