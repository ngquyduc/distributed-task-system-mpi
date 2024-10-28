# Distributed Task Runner - MPI Implementation

A high-performance distributed task execution system using MPI, featuring a master-worker architecture and optimized communication patterns.

## 👥 Authors

- Nguyen Quy Duc
- Pham Ba Thang

## 🎯 Overview

This project implements a distributed task runner using MPI, employing a master-worker architecture for efficient task distribution and execution. The system dynamically manages task queues and worker availability to maximize throughput.

## 🏗️ Architecture

### Master-Worker Pattern

```text
                  ┌─── Worker 1 ──► Execute Task
                  │
Master Queue ─────┼─── Worker 2 ──► Execute Task
                  │
                  └─── Worker N ──► Execute Task
```

### Key Components

1. **Master Node (Rank 0)**

   - Task queue management
   - Worker availability tracking
   - Non-blocking communication handling

2. **Worker Nodes**
   - Task execution
   - Result reporting
   - Dynamic task handling

## 💻 Technical Implementation

### Master Node Management

```cpp
// Master node data structures
queue<task_t> task_queue = get_initial_tasks(params);
vector<bool> availability(N_WORKERS, true);
vector<MPI_Request> proc_requests(N_WORKERS, MPI_REQUEST_NULL);
vector<task_t*> desc_tasks(num_procs - 1, nullptr);

// Non-blocking receive for descendant tasks
MPI_Irecv(desc_tasks[proc], Nmax, TASK_T_TYPE,
          worker_rank, 0, MPI_COMM_WORLD,
          &proc_requests[proc]);
```

### Communication Optimization

```cpp
// Non-blocking wait for multiple requests
MPI_Waitsome(unavailable_requests.size(),
             unavailable_requests.data(),
             &outcount, indices.data(),
             MPI_STATUSES_IGNORE);
```

## 📊 Performance Analysis

### Task Depth Impact

| Depth | Tasks | Sequential (ms) | MPI (ms) | Speedup | CPU Util. |
| ----- | ----- | --------------- | -------- | ------- | --------- |
| 4     | 30    | 7,002           | 4,342    | 1.61x   | 8.82      |
| 8     | 66    | 14,890          | 7,315    | 2.04x   | 9.31      |
| 12    | 115   | 21,612          | 11,901   | 1.82x   | 9.50      |
| 16    | 175   | 44,167          | 14,951   | 2.95x   | 9.64      |
| 20    | 305   | 70,305          | 18,998   | 3.70x   | 9.68      |

### Initial Tasks Impact

| Tasks | Sequential (ms) | MPI (ms) | Speedup | CPU Util. |
| ----- | --------------- | -------- | ------- | --------- |
| 5     | 20,270          | 11,363   | 1.78x   | 9.46      |
| 10    | 40,540          | 11,232   | 3.61x   | 9.49      |
| 15    | 60,643          | 11,829   | 5.13x   | 9.47      |
| 20    | 80,870          | 11,740   | 6.89x   | 9.41      |
| 25    | 101,135         | 13,346   | 7.58x   | 9.50      |

## 🔧 Optimization Journey

### 1. Communication Pattern Evolution

- **Initial**: Scatter-Gather pattern
  ```cpp
  MPI_Scatterv(task_queue.data(), send_counts.data(),
               displs.data(), task_t_type, local_tasks.data(),
               send_counts[rank], task_t_type, 0, MPI_COMM_WORLD);
  ```
- **Optimized**: Master-Worker with non-blocking communication
  - Improved load balancing
  - Reduced synchronization overhead
  - Better handling of varying task durations

### 2. Non-blocking Communication

- Implemented asynchronous task distribution
- Reduced worker idle time
- Improved overall throughput

### 3. Fixed-Size Communication

- Always send Nmax-sized vectors
- Reduced communication steps
- Minimal overhead for small Nmax values

## 📈 Performance Insights

### Scaling Characteristics

1. **Task Depth**

   - Better speedup with increased depth
   - Higher parallelism utilization
   - Linear scaling up to hardware limits

2. **Process Count**

   - Optimal performance at 12-16 processes
   - Diminishing returns beyond task pool size
   - Communication overhead impacts at higher counts

3. **Node Distribution**
   - Better performance on higher clock speed nodes
   - Communication overhead impacts multi-node setup
   - CPU utilization remains consistent

## 🚀 Getting Started

### Prerequisites

- MPI implementation (OpenMPI/MPICH)
- C++ compiler with MPI support
- SLURM workload manager (for cluster deployment)

### Building and Running

```bash
# Build the project
make

# Run with SLURM
sbatch config1.sh <depth> <Nmin> <Nmax> <P> <input_file>

# Example
sbatch config1.sh 16 1 2 0.10 tests/tinkywinky.in
```

## 📊 Test Configurations

```bash
# Configuration 1
sbatch config1.sh 16 1 2 0.10 tests/tinkywinky.in
sbatch config1.sh 5 3 5 0.50 tests/dipsy.in
sbatch config1.sh 5 2 2 0.00 tests/lala.in
sbatch config1.sh 5 2 4 0.50 tests/po.in
sbatch config1.sh 12 0 10 0.16 tests/thesun.in
```

## 🛠️ Technical Specifications

- **Processors**: Intel Xeon Silver 4114, Intel i7-7700
- **Interconnect**: High-speed cluster network
- **MPI Implementation**: OpenMPI/MPICH
- **Compiler**: GCC/G++
