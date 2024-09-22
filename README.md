# Reconciliation Service Algorithm

## Algorithm Overview

```mermaid
graph TD
    A[Start] --> B[Data Loading and Partitioning]
    B --> C[Parallel Processing]
    C --> D[Transaction Matching within Buckets]
    D --> E[Aggregation of Results]
    E --> F[Result Reporting]
    F --> G[End]
```

## Detailed Process

### 1. Data Loading and Partitioning

#### System Transactions (First Dataset)
- **Partitioning**: Split the dataset into multiple buckets based on a consistent hashing function applied to the transaction IDs or dates.
- **Storage**: For each bucket, store transactions in an in-memory data structure like a HashMap for O(1) lookup.
- **Scalability**: Distribute buckets across multiple processes or servers if necessary.

#### Bank Statements (Second Dataset)
- **Partitioning**: Apply the same hashing function to ensure that corresponding transactions from both datasets end up in the same bucket.
- **Streaming**: Read and process bank transactions line by line to minimize memory usage.

```mermaid
graph TD
    A[Data Loading] --> B[System Transactions]
    A --> C[Bank Statements]
    B --> D[Partitioning]
    C --> D
    D --> E[Bucket 1]
    D --> F[Bucket 2]
    D --> G[Bucket 3]
    D --> H[...]
```

### 2. Parallel Processing
- **Thread Pool**: Utilize a pool of worker threads to process buckets in parallel.
- **Concurrency**: Ensure thread-safe operations when accessing shared resources or aggregating results.

```mermaid
graph TD
    A[Thread Pool] --> B[Worker 1]
    A --> C[Worker 2]
    A --> D[Worker 3]
    A --> E[...]
    B --> F[Process Bucket]
    C --> F
    D --> F
    E --> F
```

### 3. Transaction Matching within Buckets

#### Lookup and Matching
For each bank transaction in a bucket:
1. **Lookup**: Check if a matching system transaction exists in the bucket's HashMap.
2. **Match Found**:
   - **Amount Comparison**: Compare the transaction amounts.
   - **Exact Match**: Mark both transactions as matched.
   - **Discrepancy Detected**: Add to the 'discrepant' bucket for further analysis.
   - **Flagging**: Remove or mark the matched system transaction to prevent duplicate matching.
3. **No Match**:
   - **Unmatched Bank Transaction**: Add to the 'not found' bucket for bank transactions.

After processing all bank transactions in a bucket:
- **Unmatched System Transactions**: Remaining transactions in the HashMap are unmatched system transactions.

```mermaid
graph TD
    A[Bank Transaction] --> B{Match in System?}
    B -->|Yes| C{Amounts Match?}
    B -->|No| D[Add to Unmatched Bank]
    C -->|Yes| E[Mark as Matched]
    C -->|No| F[Add to Discrepant]
    E --> G[Remove from System HashMap]
    G --> H[Next Transaction]
    D --> H
    F --> H
```

### 4. Aggregation of Results

#### Matched Transactions
- Count the total number of matched transactions across all buckets.

#### Unmatched Transactions
- **System Unmatched**: Transactions present in the system but missing in bank statements.
- **Bank Unmatched**: Transactions present in bank statements but missing in the system.

#### Discrepancies
- Calculate the sum of absolute differences in amounts for discrepant transactions.

```mermaid
graph TD
    A[Aggregation] --> B[Count Matched]
    A --> C[Count System Unmatched]
    A --> D[Count Bank Unmatched]
    A --> E[Sum Discrepancies]
    B --> F[Final Results]
    C --> F
    D --> F
    E --> F
```

### 5. Scalability Enhancements

#### Sharding
- Use consistent hashing to distribute data evenly and ensure that related data ends up in the same shard or bucket.

#### Distributed Processing
- Deploy the system across multiple servers or use a distributed computing framework to handle large volumes of data.

#### Resource Management
- Optimize memory usage by processing data in chunks and releasing resources promptly after use.

```mermaid
graph TD
    A[Scalability] --> B[Sharding]
    A --> C[Distributed Processing]
    A --> D[Resource Management]
    B --> E[Consistent Hashing]
    C --> F[Multi-Server Deployment]
    D --> G[Chunk Processing]
    D --> H[Resource Release]
```

### 6. Result Reporting

#### Reconciliation Summary
- Total transactions processed
- Total matched transactions
- Total unmatched transactions
- Total discrepancies

#### Detailed Reports
- Lists of unmatched system transactions
- Lists of unmatched bank transactions, grouped by bank if applicable
- Lists of discrepant transactions with details

```mermaid
graph TD
    A[Result Reporting] --> B[Reconciliation Summary]
    A --> C[Detailed Reports]
    B --> D[Total Processed]
    B --> E[Total Matched]
    B --> F[Total Unmatched]
    B --> G[Total Discrepancies]
    C --> H[Unmatched System Transactions]
    C --> I[Unmatched Bank Transactions]
    C --> J[Discrepant Transactions]
```
