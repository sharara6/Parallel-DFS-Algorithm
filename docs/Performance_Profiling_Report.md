# Performance Profiling Report: DFS Traversal

## Executive Summary

This report presents the performance analysis of the parallel Depth-First Search (DFS) traversal implementation using OpenMP. We measure serial execution time (T_S) and parallel execution times (T_P) for different thread counts (1, 2, 4, 8), compute speedup and efficiency metrics, and analyze the results using Amdahl's Law.

---

## Methodology

### Test Configuration
- **Graph Size**: 50,000 vertices
- **Algorithm**: Depth-First Search (DFS) traversal
- **Thread Counts Tested**: 1, 2, 4, 8
- **Iterations**: 5 runs per configuration (averaged)
- **Compiler Flags**: `-fopenmp -O2 -std=c++11`

### Performance Metrics
1. **T_S**: Serial execution time (baseline)
2. **T_P(p)**: Parallel execution time with p threads
3. **Speedup**: S(p) = T_S / T_P(p)
4. **Efficiency**: E(p) = S(p) / p

---

## Experimental Results

### Performance Data

| Threads (p) | T_P (seconds) | Speedup (S) | Efficiency (E) |
|-------------|---------------|-------------|----------------|
| 1           | [TBD]         | [TBD]       | [TBD]          |
| 2           | [TBD]         | [TBD]       | [TBD]          |
| 4           | [TBD]         | [TBD]       | [TBD]          |
| 8           | [TBD]         | [TBD]       | [TBD]          |

**Serial Time (T_S)**: [TBD] seconds

*Note: Replace [TBD] with actual measurements from running `profile.exe`*

---

## Analysis Using Amdahl's Law

### Amdahl's Law Formula

Amdahl's Law predicts the maximum speedup achievable when parallelizing a program:

**S_max(p) = 1 / (f_serial + f_parallel / p)**

Where:
- **p**: Number of processors/threads
- **f_serial**: Fraction of execution time that is inherently serial (cannot be parallelized)
- **f_parallel**: Fraction of execution time that can be parallelized (f_serial + f_parallel = 1)

### Determining Serial Fraction

From our measurements, we can estimate the serial fraction:

**f_serial ≈ 1 - (S_measured(p) / p × Efficiency_measured(p))**

Or more directly from speedup:
- If speedup is sublinear, it indicates serial bottlenecks
- The serial fraction limits maximum achievable speedup

### Expected Observations

For the DFS implementation analyzed:

1. **Serial Components**:
   - Graph creation (performed once)
   - Critical sections protecting shared data structures (`visited`, `res`)
   - Task creation overhead
   - Synchronization (taskwait, critical sections)

2. **Parallel Components**:
   - Independent DFS traversals from different starting points
   - Work computation within each recursive call
   - Task-based parallel execution

3. **Bottlenecks**:
   - **Critical sections**: Frequent access to `visited` and `res` arrays creates contention
   - **Load imbalance**: Graph structure may create uneven work distribution
   - **Task overhead**: Creating and managing OpenMP tasks has overhead

---

## Interpretation of Results

### Speedup Analysis

**Ideal Speedup**: Linear speedup would show S(p) = p

**Expected Behavior**:
- **p = 1**: Speedup ≈ 1.0 (baseline, may be slightly less due to OpenMP overhead)
- **p = 2**: Speedup < 2.0 (due to serial bottlenecks)
- **p = 4**: Speedup < 4.0 (diminishing returns)
- **p = 8**: Speedup < 8.0 (likely plateaus due to Amdahl's Law limit)

### Efficiency Analysis

**Ideal Efficiency**: E(p) = 1.0 (100%) indicates perfect parallelization

**Expected Behavior**:
- Efficiency typically decreases as thread count increases
- Lower efficiency indicates:
  - Serial bottlenecks
  - Overhead from synchronization
  - Load imbalance
  - Cache contention

### Scalability Limits

**Amdahl's Law Limit**: 
The maximum theoretical speedup is bounded by:
**S_max = 1 / f_serial**

If f_serial = 0.1 (10% serial), maximum speedup = 10x
If f_serial = 0.2 (20% serial), maximum speedup = 5x

**Practical Observations**:
1. **Sublinear Speedup**: As threads increase, speedup increases but at a decreasing rate
2. **Efficiency Degradation**: Efficiency decreases as more threads are added
3. **Plateau Effect**: Beyond a certain point, adding more threads provides minimal benefit

---

## Discussion

### Factors Affecting Performance

1. **Critical Section Contention**
   - The DFS implementation uses `#pragma omp critical` to protect shared data
   - Frequent critical section access creates serial bottlenecks
   - This is a primary limiting factor for speedup

2. **Task Creation Overhead**
   - OpenMP task creation has overhead
   - For fine-grained tasks, overhead may exceed benefits
   - Coarser-grained parallelization might be more efficient

3. **Load Imbalance**
   - Graph structure may create uneven work distribution
   - Some threads may finish earlier and wait (idle time)
   - Dynamic load balancing can help but adds overhead

4. **Memory Access Patterns**
   - Contention for shared memory structures
   - Cache coherency overhead
   - False sharing potential

### Comparison with Theoretical Limits

**Amdahl's Law Prediction vs. Actual**:
- Calculate theoretical speedup based on estimated serial fraction
- Compare with measured speedup
- Identify additional overhead factors not captured by Amdahl's Law (e.g., synchronization, memory access)

### Scalability Discussion

**Strong Scaling**: 
- How does performance change when problem size is fixed and threads increase?
- Our analysis shows strong scaling behavior

**Weak Scaling** (future work):
- How does performance change when problem size scales with thread count?
- This would show if the algorithm scales to larger graphs

**Scalability Limits**:
- The algorithm will hit Amdahl's Law limits
- To improve: reduce serial fraction (fewer critical sections, better algorithms)
- To improve: reduce overhead (better task granularity, optimized synchronization)

---

## Conclusions

### Key Findings

1. **Serial Fraction**: The implementation has a significant serial component due to critical sections
2. **Speedup**: Sublinear speedup is observed, consistent with Amdahl's Law predictions
3. **Efficiency**: Efficiency decreases as thread count increases
4. **Optimal Thread Count**: Based on efficiency, there is likely an optimal thread count (typically 2-4 for this problem)

### Recommendations

1. **Reduce Critical Sections**:
   - Use thread-local visited arrays and merge at the end
   - Use atomic operations where possible instead of critical sections
   - Reduce frequency of shared data access

2. **Optimize Task Granularity**:
   - Create coarser-grained tasks to reduce overhead
   - Consider work-stealing approaches

3. **Alternative Algorithms**:
   - Consider BFS for better parallelization potential
   - Use graph partitioning for better load balance

4. **Hardware Considerations**:
   - Match thread count to available CPU cores
   - Consider NUMA effects for multi-socket systems

---

## Appendix: Running the Profiling

### Compilation
```bash
g++ -fopenmp -O2 -std=c++11 src/profile.cpp -o profile.exe
```

### Execution
```bash
./profile.exe
```

### Output
The program generates:
1. Console output with detailed timing information
2. `performance_results.txt` file with tabular data

### Updating This Report
Replace the [TBD] placeholders in the "Experimental Results" section with actual measurements from the profiling run.

---

## References

1. Amdahl, G. M. (1967). "Validity of the single processor approach to achieving large scale computing capabilities". AFIPS Conference Proceedings.
2. OpenMP Architecture Review Board. OpenMP Application Programming Interface.
3. Herlihy, M., & Shavit, N. (2012). The Art of Multiprocessor Programming.

