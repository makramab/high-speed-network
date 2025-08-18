# Assignment 1: Dumbbell Network Topology Analysis with htsim

## **Introduction**

In this assignment, you will explore one of the most fundamental network topologies used in networking research: the **dumbbell topology**. This seemingly simple topology consists of two clusters of nodes connected by a single bottleneck link, making it an ideal testbed for understanding congestion control behavior, fairness, and network performance under resource constraints.

The dumbbell topology is widely used in networking research because it creates a controlled bottleneck scenario that allows researchers to:
- Study how different protocols compete for limited bandwidth
- Analyze fairness between flows sharing the same bottleneck
- Understand queue dynamics and buffer management
- Evaluate the impact of round-trip time (RTT) differences
- Test protocol behavior under congestion

You will use the htsim network simulator to implement dumbbell topologies through traffic matrices and conduct comprehensive performance analysis across multiple transport protocols.

## **Learning Objectives**

By completing this assignment, you will:

1. **Understand network bottlenecks** and their impact on transport protocol performance
2. **Implement dumbbell topologies** using htsim traffic matrices
3. **Compare transport protocols** (TCP, NDP, Swift) under identical network conditions
4. **Analyze congestion control behavior** in bottleneck scenarios
5. **Evaluate protocol fairness** when multiple flows compete for resources
6. **Interpret performance metrics** including throughput, latency, and queue utilization
7. **Conduct systematic experiments** with parameter variations

## **Background: Dumbbell Topology**

### What is a Dumbbell Topology?

A dumbbell topology consists of:
- **Two clusters** of nodes (left and right sides)
- **A single bottleneck link** connecting the clusters
- **Multiple flows** that traverse the bottleneck link

```
Cluster A          Bottleneck Link          Cluster B
                                           
Node 0 ────┐                          ┌──── Node 8
Node 1 ────┤                          ├──── Node 9  
Node 2 ────┼──── Router A ═══════ Router B ────┼──── Node 10
Node 3 ────┤       │                  │       ├──── Node 11
Node 4 ────┘       │    Bottleneck    │       └──── Node 12
                   │      Link        │
              Queue/Buffer        Queue/Buffer
```

### Why Dumbbell Topologies Matter

1. **Controlled Bottleneck**: Creates a predictable congestion point for analysis
2. **Fairness Testing**: Multiple flows compete for the same limited resource
3. **Protocol Comparison**: Identical conditions allow fair protocol comparison
4. **Real-world Relevance**: Models scenarios like internet backbone links, data center uplinks
5. **Educational Value**: Simple enough to understand, complex enough to be interesting

### Key Research Questions

- How do different congestion control algorithms share bottleneck bandwidth?
- What happens when flows have different round-trip times?
- How do queue sizes affect protocol performance?
- Which protocols provide better fairness under congestion?

## **Assignment Tasks**

### Task 1: Create Dumbbell Traffic Matrices (25 points)

Create traffic matrices that implement dumbbell topologies with the following specifications:

#### Subtask 1.1: Basic Dumbbell (10 points)
Create a traffic matrix for a 16-node network implementing a symmetric dumbbell:
- **Nodes 0-7**: Left cluster
- **Nodes 8-15**: Right cluster  
- **Traffic pattern**: Each node in left cluster sends to corresponding node in right cluster
- **Flow characteristics**: Infinite flows (size = 0)

**Implementation hint**: Use the existing permutation generator as a starting point, but modify the node assignments to create the dumbbell pattern.

#### Subtask 1.2: Asymmetric Dumbbell (10 points)
Create a traffic matrix with uneven cluster sizes:
- **Nodes 0-4**: Left cluster (5 nodes)
- **Nodes 5-15**: Right cluster (11 nodes)
- **Traffic pattern**: All left nodes send to different right nodes

#### Subtask 1.3: Many-to-Few Dumbbell (5 points)
Create a congestion-heavy scenario:
- **Nodes 0-11**: Senders (left cluster)
- **Nodes 12-15**: Receivers (right cluster)
- **Traffic pattern**: Multiple senders per receiver (3:1 ratio)

**Deliverable**: Three .cm traffic matrix files with clear documentation of the topology structure.

### Task 2: Protocol Performance Comparison (30 points)

Run experiments using your basic dumbbell topology (Task 1.1) with different transport protocols.

#### Subtask 2.1: TCP NewReno Baseline (10 points)
```bash
./htsim_tcp -nodes 16 -tm your_dumbbell.cm -q 50 -log sink -end 2000 -mtu 4000
```

#### Subtask 2.2: NDP Evaluation (10 points)  
```bash
./htsim_ndp -nodes 16 -tm your_dumbbell.cm -cwnd 50 -strat ecmp_host -log sink -end 2000 -mtu 4000
```

#### Subtask 2.3: Swift Analysis (10 points)
```bash
./htsim_swift -nodes 16 -tm your_dumbbell.cm -cwnd 50 -log sink -end 2000 -mtu 4000
```

For each protocol, collect and analyze:
- **Average throughput** per flow
- **Throughput fairness** (coefficient of variation)
- **Total network utilization**
- **Flow completion behavior** (for finite flows)

**Deliverable**: Performance comparison table and throughput plots for all three protocols.

### Task 3: Bottleneck Analysis (20 points)

Investigate how the bottleneck link affects network performance.

#### Subtask 3.1: Queue Size Impact (10 points)
Using your best-performing protocol from Task 2, run experiments with different queue sizes:
- Small buffers: `-q 10`
- Medium buffers: `-q 50` 
- Large buffers: `-q 200`

#### Subtask 3.2: Congestion Behavior (10 points)
Analyze queue utilization and packet drops:
- Enable queue logging: `-log sink,queue`
- Create queue utilization plots over time
- Measure packet drop rates (if any)

**Deliverable**: Analysis of how buffer sizes affect throughput, latency, and fairness.

### Task 4: Fairness Analysis (15 points)

Evaluate how fairly different protocols share the bottleneck bandwidth.

#### Metrics to Calculate:
1. **Jain's Fairness Index**: $F = \frac{(\sum_{i=1}^{n} x_i)^2}{n \sum_{i=1}^{n} x_i^2}$ where $x_i$ is throughput of flow $i$
2. **Coefficient of Variation**: $CV = \frac{\sigma}{\mu}$ of throughput distribution
3. **Min-Max Ratio**: $\frac{\text{min throughput}}{\text{max throughput}}$

#### Implementation:
Create a Python script that:
- Parses htsim output for individual flow throughputs
- Calculates fairness metrics
- Generates fairness comparison plots

**Deliverable**: Fairness analysis report comparing all three protocols with quantitative metrics.

### Task 5: Parameter Sensitivity Study (10 points)

Investigate how key parameters affect dumbbell topology performance.

#### Parameters to Study:
- **Number of flows**: 4, 8, 16, 32 flows crossing the bottleneck
- **Flow sizes**: Infinite vs. finite flows (1MB, 10MB)
- **Network delay**: Different RTT settings using `-hop_latency`

Choose one protocol and one parameter to study in detail.

**Deliverable**: Parameter sensitivity analysis with recommendations for optimal settings.

## **Implementation Guidance**

### Creating Dumbbell Traffic Matrices

Since htsim uses traffic matrices rather than explicit topology specification, you'll create dumbbell topologies through careful traffic pattern design:

```python
# Example: Basic dumbbell traffic matrix generator
def generate_dumbbell_matrix(filename, total_nodes, left_size):
    """
    Generate a dumbbell traffic matrix.
    
    Args:
        filename: Output .cm file
        total_nodes: Total nodes in network
        left_size: Number of nodes in left cluster
    """
    right_size = total_nodes - left_size
    connections = min(left_size, right_size)
    
    with open(filename, 'w') as f:
        f.write(f"Nodes {total_nodes}\n")
        f.write(f"Connections {connections}\n")
        
        # Create left-to-right traffic flows
        for i in range(connections):
            left_node = i
            right_node = left_size + i
            f.write(f"{left_node}->{right_node} id {i+1} start 0 size 0\n")
```

### Expected Network Behavior

In a properly implemented dumbbell topology, you should observe:

1. **Bottleneck Formation**: All cross-cluster traffic must traverse the same logical link
2. **Queue Buildup**: Congestion will occur at the bottleneck routers
3. **Protocol Differences**: Different congestion control algorithms will behave distinctly
4. **Fairness Variations**: Some protocols will be more fair than others

### Analysis Tools

Use the provided analysis scripts:

```bash
# Generate throughput plots
python3 create_throughput_plot.py logout.dat protocol experiment_name

# Extract individual flow statistics  
../parse_output logout.dat -protocol -show

# Create custom analysis
python3 analyze_fairness.py logout.dat > fairness_report.txt
```

## **Expected Results**

### Protocol Performance Characteristics

**TCP NewReno**:
- Conservative congestion control
- Potential unfairness due to AIMD dynamics
- Moderate throughput utilization

**NDP**:
- Pull-based flow control
- Generally good fairness
- High throughput utilization

**Swift**:
- Delay-based congestion control
- Potentially better fairness than TCP
- Variable performance depending on configuration

### Performance Metrics to Report

| Metric | TCP | NDP | Swift |
|--------|-----|-----|-------|
| Average Throughput (Gbps) | X.XX | X.XX | X.XX |
| Fairness Index | 0.XX | 0.XX | 0.XX |
| Network Utilization (%) | XX% | XX% | XX% |
| Buffer Utilization | High/Med/Low | High/Med/Low | High/Med/Low |

## **Deliverables**

Submit the following files and reports:

### 1. Implementation Files (20%)
- **Traffic matrices**: All .cm files created (Tasks 1.1-1.3)
- **Analysis scripts**: Any custom Python scripts for fairness calculation
- **Experiment scripts**: Bash scripts to reproduce your experiments

### 2. Experimental Results (50%)
- **Raw data**: logout.dat files for all major experiments
- **Processed data**: CSV files with throughput, latency, and fairness metrics
- **Visualizations**: Throughput plots, fairness comparisons, parameter sensitivity graphs

### 3. Technical Report (30%)
A 4-6 page report including:

#### Introduction (10%)
- Explanation of dumbbell topology importance
- Research questions you investigated

#### Methodology (20%)
- Description of traffic matrix implementation
- Experimental setup and parameters
- Analysis methods used

#### Results (50%)
- Protocol performance comparison
- Fairness analysis with quantitative metrics
- Parameter sensitivity findings
- Discussion of bottleneck behavior

#### Conclusions (20%)
- Summary of key findings
- Protocol recommendations for different scenarios
- Lessons learned about congestion control

## **Evaluation Criteria**

### Excellent (90-100%)
- All tasks completed with thorough analysis
- Clear understanding of congestion control principles
- Insightful conclusions supported by data
- Well-documented and reproducible experiments
- Professional-quality report and visualizations

### Good (80-89%)
- Most tasks completed correctly
- Good understanding of basic concepts
- Adequate analysis and conclusions
- Mostly reproducible experiments
- Clear report with minor issues

### Satisfactory (70-79%)
- Basic requirements met
- Some understanding demonstrated
- Limited analysis depth
- Experiments work but may have issues
- Report meets minimum requirements

### Needs Improvement (<70%)
- Incomplete or incorrect implementation
- Limited understanding of concepts
- Insufficient analysis
- Non-reproducible experiments
- Poor quality report

## **Advanced Challenges (Bonus)**

For students seeking additional challenge:

### Challenge 1: Dynamic Bottleneck (5% bonus)
Implement a scenario where the bottleneck capacity changes during the simulation. Investigate how different protocols adapt to capacity variations.

### Challenge 2: Multi-Bottleneck Topology (10% bonus)
Create a more complex topology with multiple potential bottlenecks and analyze how traffic load balancing affects performance.

### Challenge 3: Custom Protocol Modification (15% bonus)
Modify one of the existing protocols to improve fairness in dumbbell scenarios. Document your changes and evaluate the improvement.

### Challenge 4: Real-World Validation (10% bonus)
Compare your simulation results with published research on dumbbell topologies. Discuss similarities and differences in findings.

## **Resources and References**

### Essential Reading
- RFC 5681: TCP Congestion Control
- "Re-architecting datacenter networks and stacks for low latency and high performance" (SIGCOMM 2017)
- "Improving datacenter performance and robustness with multipath TCP" (SIGCOMM 2011)

### Helpful Tools
- **gnuplot**: For creating publication-quality plots
- **Python pandas**: For data analysis and statistics
- **Wireshark**: For understanding packet-level behavior (if needed)

### Common Issues and Solutions
- **Low throughput**: Check queue sizes and simulation duration
- **Unfair results**: Verify traffic matrix implements true dumbbell pattern
- **Missing data**: Ensure logging is enabled with `-log sink`
- **Plot generation failures**: Install gnuplot and verify file paths

!!! tip "Getting Help"
    - Review the [quickstart guide](quickstart.md) for basic htsim usage
    - Check existing traffic matrix examples in `connection_matrices/`
    - Use the course discussion forum for technical questions
    - Office hours: [Your specific times]

!!! warning "Academic Integrity"
    - You may discuss concepts with classmates but submit your own work
    - Clearly cite any external resources or code you use
    - Do not share traffic matrix files or analysis scripts with other students

---

**Due Date**: [Insert your due date]  
**Submission**: Upload all files to [your course management system]  
**Late Policy**: [Insert your late policy]

Good luck with your exploration of dumbbell topologies and congestion control! 🚀

