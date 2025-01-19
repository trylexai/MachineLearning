# Map Reduce Implementation 

### Problem

You are tasked with processing a large dataset of transactions. Each transaction contains fields like `userId`, `productId`, `amountSpent`, and `timestamp`. Your goal is to calculate the total amount spent by each user across all transactions.

---
Approach Using MapReduce: 

-- **Map Phase**: Parse the input data (`transactions`) and produce intermediate 
key-value pairs where the key is the userId and the value is the amountSpent.

```
- {"userId": "user1", "productId": "p1", "amountSpent": 50, "timestamp": "2025-01-18T10:00:00Z"}
- {"userId": "user2", "productId": "p2", "amountSpent": 20, "timestamp": "2025-01-18T11:00:00Z"}
- {"userId": "user1", "productId": "p3", "amountSpent": 30, "timestamp": "2025-01-18T12:00:00Z"}
```

**Map Output: **
```
- user1 --> 50
- user2 --> 30
- user1 --> 50
```
**Reduce Phase:** Aggregate the values for each userId to calculate the total amount spent.

**Reduce output:**
```
- user1 -> 80  
- user2 -> 20  
```
---
***Java Implementation of MapReduce***

```
import java.util.*;
import java.util.stream.Collectors;

public class TransactionProcessor {

    // Sample transaction class
    static class Transaction {
        String userId;
        String productId;
        double amountSpent;
        String timestamp;

        public Transaction(String userId, String productId, double amountSpent, String timestamp) {
            this.userId = userId;
            this.productId = productId;
            this.amountSpent = amountSpent;
            this.timestamp = timestamp;
        }
    }

    // Map function: Processes transactions and generates key-value pairs
    public static List<Map.Entry<String, Double>> map(List<Transaction> transactions) {
        List<Map.Entry<String, Double>> intermediateResults = new ArrayList<>();
        for (Transaction transaction : transactions) {
            intermediateResults.add(new AbstractMap.SimpleEntry<>(transaction.userId, transaction.amountSpent));
        }
        return intermediateResults;
    }

    // Reduce function: Aggregates the mapped key-value pairs
    public static Map<String, Double> reduce(List<Map.Entry<String, Double>> mappedData) {
        Map<String, Double> reducedResults = new HashMap<>();
        for (Map.Entry<String, Double> entry : mappedData) {
            reducedResults.merge(entry.getKey(), entry.getValue(), Double::sum);
        }
        return reducedResults;
    }

    public static void main(String[] args) {
        // Sample input data
        List<Transaction> transactions = Arrays.asList(
                new Transaction("user1", "p1", 50, "2025-01-18T10:00:00Z"),
                new Transaction("user2", "p2", 20, "2025-01-18T11:00:00Z"),
                new Transaction("user1", "p3", 30, "2025-01-18T12:00:00Z")
        );

        // Map Phase
        List<Map.Entry<String, Double>> mappedData = map(transactions);
        System.out.println("Mapped Data: " + mappedData);

        // Shuffle and Sort (Group by key)
        Map<String, List<Double>> groupedData = mappedData.stream()
                .collect(Collectors.groupingBy(Map.Entry::getKey,
                        Collectors.mapping(Map.Entry::getValue, Collectors.toList())));
        System.out.println("Grouped Data: " + groupedData);

        // Reduce Phase
        Map<String, Double> reducedData = reduce(mappedData);
        System.out.println("Reduced Data (Total Amount Spent by User): " + reducedData);
    }
}


```
Transaction Class: Represents individual transactions with fields for userId, productId, amountSpent, and timestamp.

**Map Function:** Converts each transaction into a key-value pair (userId, amountSpent).

**Grouping** (Shuffle and Sort): Groups the mapped data by userId.

**Reduce Function:** Aggregates the grouped values to calculate the total amount spent per user.

**Given Input:** 
```
[
  - {"userId": "user1", "productId": "p1", "amountSpent": 50, "timestamp": "2025-01-18T10:00:00Z"},
  - {"userId": "user2", "productId": "p2", "amountSpent": 20, "timestamp": "2025-01-18T11:00:00Z"},
  - {"userId": "user1", "productId": "p3", "amountSpent": 30, "timestamp": "2025-01-18T12:00:00Z"}

]
```

**Received Output:**
```
- Mapped Data: [user1=50.0, user2=20.0, user1=30.0]
- Grouped Data: {user1=[50.0, 30.0], user2=[20.0]}
- Reduced Data (Total Amount Spent by User): {user1=80.0, user2=20.0}
```
---
MapReduce is a programming model or paradigm for processing large-scale data in distributed systems.

It's part of the Hadoop distributed computing ecosystem, allowing massive batch computations on data stored in HDFS (Hadoop Distributed File System).

**Core Advantages of MapReduce**

1. Example w.r.t. SQL:
- SQL is commonly used for querying and processing data in databases.
It has predefined operators (e.g., SELECT, JOIN, GROUP BY) that limit the operations you can perform on data.
- While SQL is powerful, it might not be able to handle highly specialized or unconventional processing needs.
**Write MapReduce for your own code:**
- MapReduce allows developers to write custom logic for data processing tasks using programming languages (like Java, Python, etc.), rather than being constrained to the declarative syntax and functionality of SQL.
- MapReduce lets you define your own mapper and reducer functions.
These functions can contain any logic you need for your task, such as:
Parsing custom data formats.
- Applying specific transformations to the data.
- Performing complex computations or aggregations.
- SQL Limitation: Suppose you have a dataset of web server logs, and you want to process IP addresses to calculate the average response time for requests originating from a specific subnet.
- SQL struggles with such non-standard computations, especially if parsing or custom filtering is required.
- **MapReduce Solution:**
 - Write a mapper to extract and emit the subnet and response time.
 -  Write a reducer to aggregate and calculate the average for each subnet.

2. Data Locality
- MapReduce processes data on the node where it resides, minimizing network bandwidth usage and improving performance. This is achieved by moving computation to data rather than data to computation.

3. Fault tolerance: Designed to handle node failures gracefully by restarting only the failed part of the batch job, not the entire process. MapReduce is built to ensure data processing jobs are completed reliably, even if some parts of the system fail.
How MapReduce Handles Failures:

    Task-Level Isolation:
        Each MapReduce job is divided into smaller tasks (mappers and reducers).
        These tasks are distributed across nodes in the cluster for execution.

    Failure Detection:
        The Hadoop framework continuously monitors the health of nodes and tasks.
        If a node becomes unresponsive or a task fails (due to crashes or other issues), it is flagged as failed.

    Restarting Failed Tasks:
        Only the tasks that failed are restarted on a healthy node.
        For example:
            If 100 map tasks are running and 3 fail, only those 3 are restarted, not all 100.

    Data Resilience:
        Input data for the job is stored in HDFS (Hadoop Distributed File System), which replicates data across multiple nodes.
        This ensures that even if a node storing data fails, the same data is available from another node for reprocessing.
        Scenario:

You are running a MapReduce job to process a large dataset of customer transactions stored in HDFS. The job involves:

    A mapper that extracts customer IDs and transaction amounts.
    A reducer that calculates the total spending per customer.

Failure During Execution:

    Failure Happens:
        A node running 10 mapper tasks crashes mid-job.
    Detection:
        The framework detects the failure and marks those 10 tasks as incomplete.
    Recovery:
        The framework reschedules the 10 mapper tasks on other available nodes using the same input data from HDFS.
        The completed mapper and reducer tasks on other nodes remain unaffected.
    Completion:
        Once the failed tasks are re-executed, the reducer continues with the outputs from all mappers, ensuring the job completes successfully.


**Key Components**

Mapper

- Processes each input record and emits a key-value pair.

Example:

- Input: Unstructured logs (e.g., username + message).

- Output: Key-value pairs like (userID, message length).

Reducer

- Aggregates all values for the same key and outputs a single result for that key.

Example:

- Input: (Key = 21, Values = [15, 10, 5]).

- Output: (Key = 21, Average = 10).

**MapReduce Architecture**

Stages: 

1. Data Input:

- Reads unstructured data stored in HDFS.

- Splits it into chunks for parallel processing.

2. Mapping:

- Each mapper processes input records and emits intermediate key-value pairs.

3. huffling:

- Organizes and groups intermediate key-value pairs by key.

- Keys are hashed and sent to the appropriate reducers.

4. Reducing:

- Each reducer processes grouped key-value pairs.

- Outputs final aggregated results.

5. Output:

- Stores results back into HDFS.

- Data is replicated for fault tolerance.

**Why Sorting and Shuffling Matter**

- Sorting ensures keys are processed in a predictable order during reduction.

- Memory efficiency:

 - Sorted data allows the reducer to flush completed keys to disk once it encounters a new key.

 - Without sorting, all key-value pairs must be kept in memory, leading to higher memory overhead.

- Shuffling partitions data for reducers, ensuring values for the same key are sent to the same reducer.

**Job Chaining** Chaining MapReduce Jobs: 
- Output of one MapReduce job can be used as input for another.
- Enables complex workflows by linking multiple jobs together.
- In job chaining, the output generated in the Reduce phase of one job can be passed on to the Map phase of another job. This creates a chain of jobs where each job is dependent on the output of the previous one.
- Imagine a scenario where you want to analyze data in multiple stages:

    - Job 1: You first want to filter and sort a large dataset based on specific criteria.
    - Job 2: Then, you want to count the frequency of a particular attribute from the results of Job 1.
    - Job 3: Finally, you need to apply a statistical analysis or further processing to Job 2's output.
    - Instead of running each job independently, you can chain them together. The output of Job 1 becomes the input to Job 2, and the output of Job 2 becomes the input for Job 3.

**Batch vs. Stream Processing**

Batch processing:

Executes scheduled jobs on large datasets (e.g., daily or weekly).

MapReduce is optimized for batch processing.

**Stream processing:**

Processes real-time data (e.g., Apache Kafka, Flink).

Better suited for immediate results but not MapReduce's focus.

**Limitations of MapReduce**

Inefficiency for Iterative Algorithms:
- Limitation: MapReduce processes data in a single pass and does not support iterative processing effectively. This makes it inefficient for algorithms that need multiple passes over the data, like machine learning models (e.g., training a model using gradient descent).
- Example: If you were to implement PageRank, which requires iterative updates to the rank of each page based on neighboring pages, you would need to repeatedly run multiple MapReduce jobs. This introduces significant overhead due to disk reads/writes and data shuffling, making the process slower compared to more suitable frameworks like Apache Spark, which supports in-memory processing and iterative operations.
- Handling Complex Data Dependencies:
Limitation: MapReduce is designed to handle simple key-value pairs and lacks native support for operations that require complex dependencies or multi-step computations between jobs.
Example: In graph processing (e.g., for finding the shortest path between nodes), the algorithm requires maintaining state across multiple iterations and handling non-trivial data dependencies. Implementing this with MapReduce would involve complex workarounds and would be less efficient than using specialized graph processing systems like Apache Giraph or Pregel.

High Latency Due to Disk I/O:
Limitation: The intermediate results between the Map and Reduce stages are stored on disk, introducing significant latency. Disk I/O can become a bottleneck, particularly when working with large datasets.
Example: Consider a large-scale log analysis job where each log entry needs to be mapped and aggregated (e.g., counting the number of log entries by type). Between the map and reduce stages, intermediate data is written to disk. If the dataset is huge, disk operations could result in high latency and slow processing.

Handling Small Files Inefficiently:
Limitation: MapReduce performs better with large files and datasets. When working with many small files, the overhead associated with managing them (splitting files, handling network I/O) can significantly degrade performance.
Example: In a scenario where a cloud storage system contains thousands of small log files that need to be processed, each file would be treated as a separate task, introducing excessive overhead. The system would struggle to achieve optimal throughput due to the overhead of managing multiple small files.

Resource Management Overhead:
Limitation: MapReduce can incur significant resource management overhead. The framework's resource allocation is static and might not efficiently use cluster resources, especially when workloads vary significantly.
Example: Suppose you're running a data analysis job with a large cluster. During some phases of the job, certain nodes might be idle while others are overloaded, causing inefficient resource utilization and poor performance.

Limited Fault Tolerance for Task-Level Failures:
Limitation: MapReduce handles failures at the job level (i.e., if a task or job fails, it retries the whole job or task), but finer-grained task-level failures can still cause delays or slow down the system.
Example: In a log aggregation task where each mapper processes chunks of log data, if one of the mappers crashes due to a network issue or hardware failure, MapReduce will restart the whole task, potentially causing significant delays for the entire job.

Non-Optimal Data Locality:
Limitation: MapReduce does not always ensure that data is processed where it is stored. This means that data may need to be transferred between nodes, increasing network traffic and reducing performance.
Example: In a join operation between two large datasets, MapReduce might not always place the mappers for each dataset on the nodes that store the relevant data. As a result, the system might need to shuffle large amounts of data across the network, leading to slower performance.

Limited Real-Time Processing Capabilities:
Limitation: MapReduce is designed for batch processing and is not well-suited for real-time data processing, where low-latency responses are critical.
Example: A real-time stream processing system for monitoring and alerting (e.g., monitoring web server logs in real time) would struggle with MapReduce. Instead, systems like Apache Flink or Apache Kafka Streams would be much more efficient, as they are optimized for low-latency, continuous data streams.

Programming Complexity:
Limitation: Although MapReduce is simple in theory (map function followed by reduce), writing more complex applications with MapReduce can be cumbersome and requires custom coding to handle various edge cases.
Example: For tasks like sentiment analysis of large text datasets, you might need to perform multiple stages of data cleaning, transformation, and aggregation. Doing so in MapReduce can be challenging, requiring multiple MapReduce jobs and custom logic, whereas frameworks like Apache Spark provide high-level APIs for easier implementation of such workflows.

Non-Optimal for Complex Aggregations:
Limitation: MapReduce struggles with tasks that involve multiple types of aggregation or require advanced grouping, filtering, or sorting.
Example: For a data mining job that involves finding frequent itemsets in a transactional dataset (like in the Apriori algorithm), the multiple passes over the data required for different types of aggregation and filtering can make MapReduce less efficient, especially when compared to algorithms designed for this purpose, such as FPGrowth.


