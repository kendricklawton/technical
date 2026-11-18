# Domain 1 — Quiz 2: Vector Store Architectures & Chunking Strategies (Tasks 1.3, 1.4)

**Instructions**: This quiz contains 15 Professional-level scenario questions covering enterprise vector database architectures (Amazon OpenSearch Serverless, Amazon Aurora PostgreSQL `pgvector`, Amazon Neptune Analytics GraphRAG, Amazon MemoryDB for Redis, Amazon S3 Vectors), HNSW/IVF index internals, quantization, and document ingestion/chunking trade-offs.

---

### Question 1
A software-as-a-service (SaaS) provider hosts a shared Amazon OpenSearch Serverless vector collection for 4,000 enterprise tenants. Each document chunk includes a `tenant_id` string field and a `department` string field. When the team initially created the OpenSearch k-NN index two years ago, they configured the `knn_vector` mapping with `"engine": "nmslib"`, `"name": "hnsw"`. Now, smaller tenants who own less than 0.05% of the total vectors in the index report that when they execute filtered vector queries (`k = 10` with a boolean `filter` on `tenant_id`), the query either returns `0` results or requires an expensive `script_score` painless script that times out at 12 seconds per query.

What architectural change to the OpenSearch index mapping will enable fast, sub-50ms metadata pre-filtering without result starvation?

- **A.** Recreate the index mapping using `"engine": "faiss"` (or `"lucene"`) with `"name": "hnsw"`, which natively supports efficient metadata pre-filtering during k-NN search, and reindex the documents.
- **B.** Keep `"engine": "nmslib"`, but lower the HNSW `ef_construction` parameter from `128` to `4` and `m` from `16` to `2`.
- **C.** Keep `"engine": "nmslib"`, retrieve only `k = 1` result globally, and filter on `tenant_id` inside the client Python application.
- **D.** Disable OpenSearch Data Access Policies so `nmslib` can bypass Lucene segment filters.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
In Amazon OpenSearch Service and OpenSearch Serverless, the legacy **`nmslib` engine does NOT support efficient metadata pre-filtering** (forcing users to choose between post-filtering—which causes severe result starvation for small tenants—or slow exact brute-force `script_score` scans). Switching the `knn_vector` mapping to **`"engine": "faiss"`** (or `"lucene"`) enables native **efficient pre-filtering**, where OpenSearch dynamically evaluates the `tenant_id` filter bitmap alongside HNSW graph traversal (or automatically switches to an exact pre-filtered scan when filter selectivity is very small), guaranteeing full $k$ results in milliseconds.

#### Distractor Analysis
- **Why B is incorrect**: Lowering `ef_construction` and `m` degrades HNSW graph recall and does not add pre-filtering support to `nmslib`.
- **Why C is incorrect**: Post-filtering `k = 1` global result in client code guarantees a 99.95% failure rate (returning 0 results) for a tenant that owns 0.05% of the index.
- **Why D is incorrect**: OpenSearch Serverless Data Access Policies are mandatory IAM-level collection security controls and have nothing to do with k-NN engine filter execution.
</details>

---

### Question 2
An internal enterprise tool team is deploying four separate development and staging Amazon Bedrock Knowledge Bases for small departmental prototypes. Each Knowledge Base contains fewer than 15,000 document chunks and receives only 20 queries per day during business hours, with zero queries overnight or on weekends. When the team deployed their first prototype using the default quick-create option (Amazon OpenSearch Serverless), they were surprised by a monthly bill exceeding $350–$700 per collection due to minimum continuous OpenSearch Compute Unit (OCU) allocations.

Which vector store supported natively by Amazon Bedrock Knowledge Bases should the architect select to minimize idle compute costs while supporting full SQL relational metadata joins?

- **A.** Amazon OpenSearch Serverless with `standby_replicas = "ENABLED"` and 16 minimum Search OCUs.
- **B.** Amazon MemoryDB for Redis provisioned on `db.r6g.2xlarge` Multi-AZ cluster nodes.
- **C.** Amazon Aurora PostgreSQL Serverless v2 configured with the `pgvector` extension, RDS Data API enabled, and minimum capacity set to `0.5` ACU (or `0` ACU auto-pause).
- **D.** Amazon ElastiCache for Memcached with a custom Lambda vector calculator.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon OpenSearch Serverless (AOSS)** maintains a minimum continuous footprint of **2 OCUs** (with standby replicas disabled: 1 Indexing OCU + 1 Search OCU, or 4 halves) or **4 OCUs** (with HA standby replicas enabled), resulting in a significant monthly base cost even at zero query volume. By contrast, **Amazon Aurora PostgreSQL Serverless v2 with `pgvector`** integrates natively with Amazon Bedrock Knowledge Bases (via the **RDS Data API** and **AWS Secrets Manager**) and can scale down to **0.5 ACU** or **0 ACU (auto-pause)** during nights and weekends, reducing low-traffic dev/test vector database costs by over 80–90% while providing full PostgreSQL relational capabilities.

#### Distractor Analysis
- **Why A is incorrect**: Enabling standby replicas and setting 16 minimum OCUs on OpenSearch Serverless will *increase* the monthly cost to several thousand dollars per month.
- **Why B is incorrect**: MemoryDB on `db.r6g.2xlarge` runs 24/7 in-memory instances (high fixed monthly cost) and does not provide relational SQL joins.
- **Why D is incorrect**: ElastiCache for Memcached has no vector search engine and is not a supported Bedrock Knowledge Base vector store.
</details>

---

### Question 3
A DevOps engineer is automating the deployment of an Amazon Bedrock Knowledge Base backed by a new Amazon OpenSearch Serverless `VECTORSEARCH` collection using AWS CloudFormation. The engineer creates an IAM Execution Role for the Knowledge Base and attaches an IAM policy granting `"Action": ["aoss:APIAccessAll"]` on `"Resource": "*"`. The engineer also configures the OpenSearch Serverless Encryption Policy and Network Policy. However, when CloudFormation attempts to create the `AWS::Bedrock::KnowledgeBase` resource, it fails with a permissions error stating that the Bedrock execution role is not authorized to describe or write to the vector index inside the collection.

What required resource is missing from the CloudFormation template?

- **A.** A VPC Peering connection between the Amazon Bedrock service VPC and the customer's default VPC.

- **B.** An Amazon Cognito User Pool attached to the OpenSearch Dashboards SAML configuration.
- **C.** An AWS WAF WebACL associated with the OpenSearch Serverless collection ARN.
- **D.** An OpenSearch Serverless **Data Access Policy** (`AWS::OpenSearchServerless::AccessPolicy` with `Type: data`) that explicitly lists the Bedrock Knowledge Base IAM Execution Role ARN in its `Principal` array and grants index/collection permissions (`aoss:CreateIndex`, `aoss:ReadDocument`, `aoss:WriteDocument`, `aoss:DescribeIndex`, `aoss:UpdateIndex`).
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
In **Amazon OpenSearch Serverless**, granting `aoss:APIAccessAll` in an IAM Identity Policy is necessary **but not sufficient** to access data inside a collection. Every OpenSearch Serverless collection requires three policies: **Encryption**, **Network**, and **Data Access** (`Type: data`). The **Data Access Policy** must explicitly include the **Bedrock Knowledge Base IAM Role ARN** in the `Principal` list with permissions on both `collection/<name>` and `index/<name>/*`.

#### Distractor Analysis
- **Why A is incorrect**: Amazon Bedrock communicates with OpenSearch Serverless via AWS PrivateLink / service-to-service network rules (`SourceServices: ["bedrock.amazonaws.com"]`), never via VPC Peering with an AWS internal service VPC.
- **Why B is incorrect**: Cognito SAML authentication is used for human browser access to OpenSearch Dashboards, not for IAM-authenticated Bedrock Knowledge Base service calls.
- **Why C is incorrect**: AWS WAF is optional and does not grant data-plane index access to the Bedrock Knowledge Base role.
</details>

---

### Question 4
A financial services company uses an Amazon Aurora PostgreSQL (`db.r6g.2xlarge`, 64 GB RAM) database with `pgvector` to store 35 million 1024-dimensional embeddings (`float32`). Initially, when the table held 5 million vectors, the HNSW index (`m = 24`, `ef_construction = 128`) fit entirely within PostgreSQL `shared_buffers` RAM, delivering 14-ms P95 query latency. As the table grew to 35 million vectors, the HNSW index size exceeded 110 GB. Query latency suddenly degraded from 14 ms to 2,100 ms, and CloudWatch shows `BufferCacheHitRatio` dropping to 38% with heavy `ReadIOPS` wait events (`IO:DataFileRead`) from Aurora shared storage. The team wants to restore sub-50ms P95 vector query latency without partitioning the table across 10 separate clusters.

Which TWO technical optimizations should the database and GenAI architects implement? (Choose TWO.)

- **A.** Migrate the Aurora instance class to an **Aurora Optimized Reads** instance family with local NVMe SSD storage (such as `db.r6gd` or `db.r8gd`) so pages evicted from `shared_buffers` are cached in the high-speed local NVMe tiered cache rather than fetched over network storage.
- **B.** Switch the vector column and HNSW index to 16-bit half-precision floating-point vectors (`halfvec(1024)` with `halfvec_cosine_ops`) or reduce Titan Text Embeddings V2 dimensions from `1024` to `512` to cut the HNSW index memory footprint by 50%, allowing the hot working set to fit in memory/NVMe cache.
- **C.** Drop the HNSW index completely and execute sequential full-table scans (`enable_indexscan = off`) with `max_parallel_workers_per_gather = 1`.
- **D.** Increase the HNSW parameter `m` from `24` to `128` so each node stores 5x more graph edges in the index.
- **E.** Replace the RDS Data API with an Amazon SQS FIFO queue.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answers: A, B**

#### Explanation
- **Why A is correct**: HNSW graph traversal performs pointer-chasing random reads across index pages. When the HNSW index exceeds `shared_buffers` RAM, random network storage reads cause severe latency spikes. **Aurora Optimized Reads** (on `db.r6gd` / `db.r8gd` NVMe-backed instances or Serverless v2 with Optimized Reads) uses local NVMe SSDs as a tiered buffer pool extension, delivering up to **8x–9x higher vector query throughput** when indexes exceed RAM.
- **Why B is correct**: Using `pgvector` **`halfvec` (`fp16`)** or 512-dimensional **Titan Text Embeddings V2** reduces vector and HNSW index storage footprint by **50%** with less than 1% recall difference, bringing the index size from 110 GB down to ~55 GB so it fits cleanly inside RAM + NVMe cache.

#### Distractor Analysis
- **Why C is incorrect**: Performing a sequential brute-force cosine calculation across 35 million 1024-d vectors with 1 worker takes tens of seconds per query.
- **Why D is incorrect**: Increasing HNSW `m` from `24` to `128` dramatically *increases* the size of the HNSW index in memory, worsening the buffer cache thrashing.
- **Why E is incorrect**: SQS queues do not accelerate PostgreSQL buffer cache misses during HNSW index scans.
</details>

---

### Question 5
A telecommunications intelligence firm is building an investigative assistant over 500,000 incident post-mortems, BGP routing tables, submarine cable maintenance logs, and vendor SLA contracts. Investigators ask complex, multi-hop relational questions such as: *"When Submarine Cable System Alpha experienced a repeater fault in Q2, which downstream Tier-2 ISP customers suffered packet loss on services hosted in Data Center Beta, and which vendor maintenance crews had signed off on that repeater within the prior 90 days?"* When using a standard dense vector store, the RAG system retrieves chunks mentioning *"Cable System Alpha"* but completely misses the downstream ISP and maintenance crew documents because those documents only reference internal circuit IDs (`CIR-9921`) and substation codes (`SUB-44`) without ever mentioning *"Cable System Alpha"* by name.

Which AWS architecture directly solves this multi-hop entity traversal limitation?

- **A.** Keep the standard vector store, but change the vector distance metric from Cosine Similarity to Euclidean ($L_2$) distance.
- **B.** Configure an Amazon Bedrock Knowledge Base backed by **Amazon Neptune Analytics (GraphRAG)**, which automatically extracts entities and relationships (`CIR-9921`, `SUB-44`, crews, ISPs) into a property graph alongside vector embeddings and combines vector similarity lookup with multi-hop graph traversal at query time.
- **C.** Switch the chunking strategy to Fixed-Size Chunking with `maxTokens = 50` and `overlapPercentage = 0`.
- **D.** Enable Amazon Bedrock Guardrails Contextual Grounding checks with a threshold of `0.99`.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
Standard dense vector retrieval fails on **multi-hop relational queries** when intermediate bridging entities (`Cable Alpha` $\rightarrow$ `Circuit CIR-9921` $\rightarrow$ `Substation SUB-44` $\rightarrow$ `ISP Customer` / `Maintenance Crew`) are spread across separate documents that share zero lexical or semantic overlap with the original query terms. **Amazon Bedrock Knowledge Bases with Amazon Neptune Analytics (GraphRAG)** solves this by extracting entities and relationships during ingestion to build an in-memory property graph alongside vector embeddings, traversing multi-hop edges across connected documents at query time.

#### Distractor Analysis
- **Why A is incorrect**: For normalized vectors (such as Titan Text Embeddings V2), Cosine Similarity and Euclidean $L_2$ distance produce the exact same ranking order and cannot traverse multi-hop entity links across documents.
- **Why C is incorrect**: Shrinking chunks to 50 tokens with zero overlap fragments context even further, making multi-hop synthesis worse.
- **Why D is incorrect**: Guardrails Contextual Grounding blocks ungrounded hallucinations in the final output; it does not retrieve missing multi-hop documents during the search phase.
</details>

---

### Question 6
An ad-tech platform runs a real-time personalized recommendation and contextual ad-matching engine that executes 25,000 vector similarity queries per second (QPS) against an active catalog of 8 million ad embeddings. The strict latency SLA requires the vector similarity lookup itself to complete in **under 4 milliseconds at P99**, while maintaining durability across Availability Zones so a node failure does not lose recently updated ad vectors.

Which AWS service should the architect choose as the primary vector store for this ultra-low-latency, high-QPS workload?

- **A.** Amazon S3 Vectors with standard object storage retrieval.
- **B.** Amazon Athena federated queries executing cosine math in SQL over Parquet files in S3.
- **C.** Amazon MemoryDB for Redis with Vector Search enabled (`HNSW` index stored in-memory across sharded Redis nodes with a Multi-AZ transactional log).
- **D.** Amazon Aurora PostgreSQL Serverless v2 configured with `minCapacity = 0.5` ACU via the HTTP RDS Data API.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
**Amazon MemoryDB for Redis** is an in-memory, Redis-compatible database with a durable Multi-AZ transactional log that natively supports **Vector Search** (HNSW and FLAT indexes). Because all vector graphs reside entirely in memory across horizontally sharded nodes and clients communicate via persistent low-overhead Redis TCP connections, MemoryDB delivers **single-digit millisecond (sub-4ms) P99 latency** at tens of thousands of QPS.

#### Distractor Analysis
- **Why A is incorrect**: Amazon S3 Vectors is optimized for ultra-low storage cost at sub-second (~50–200ms) latency, not sub-4ms P99 latency at 25,000 QPS.
- **Why B is incorrect**: Amazon Athena is a batch/interactive analytical query engine with multi-second startup and scan latencies.
- **Why D is incorrect**: A 0.5 ACU Aurora Serverless v2 instance accessed over the HTTP RDS Data API cannot sustain 25,000 QPS at <4ms P99 latency.
</details>

---

### Question 7
A media preservation archive holds 1.5 billion historical newspaper article chunks and legal patent abstracts that are rarely queried (fewer than 500 queries per day by academic researchers). The researchers can tolerate a vector retrieval latency of 150 to 400 milliseconds per query, but the archive's non-profit grant budget strictly prohibits spending tens of thousands of dollars per month on terabytes of always-on DRAM in OpenSearch clusters or in-memory Redis nodes.

Which AWS vector storage architecture provides the lowest storage TCO for 1.5 billion vectors while maintaining sub-second similarity search and native integration with Amazon Bedrock Knowledge Bases?

- **A.** Deploy a 64-node `r6g.16xlarge` Amazon MemoryDB for Redis cluster so all 1.5 billion float32 vectors remain pinned in RAM 24/7.
- **B.** Use **Amazon S3 Vectors** (vector buckets and vector indexes natively stored on Amazon S3 object storage) integrated with Amazon Bedrock Knowledge Bases (or OpenSearch tiered vector storage backed by S3 Vectors).
- **C.** Store the 1.5 billion vectors as individual items in Amazon DynamoDB with Provisioned RCUs set to 500,000 and run parallel `Scan` operations on every query.
- **D.** Provision 400 OpenSearch Compute Units (OCUs) in Amazon OpenSearch Serverless with `nmslib` in-memory HNSW indexes.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
**Amazon S3 Vectors** provides purpose-built, cloud-object-store native vector storage (`s3vectors`) with dedicated APIs (`PutVectors`, `QueryVectors`) and native integration with **Amazon Bedrock Knowledge Bases** and **Amazon OpenSearch Service**. By organizing vector indexes directly on durable S3 storage rather than pinning terabytes of HNSW graphs in expensive DRAM 24/7, S3 Vectors reduces vector storage costs by **up to 90%** while delivering sub-second query latencies ideal for large-scale warm/cold RAG archives.

#### Distractor Analysis
- **Why A is incorrect**: Storing 1.5 billion 1024-d `float32` vectors (~6 TB of raw vectors plus HNSW graph overhead >10 TB) in MemoryDB DRAM costs tens of thousands of dollars per month—violating the strict budget constraint.
- **Why C is incorrect**: DynamoDB does not support native HNSW vector indexing; scanning 1.5 billion items per query would cost thousands of dollars per query and take minutes.
- **Why D is incorrect**: 400 OCUs in OpenSearch Serverless costs ~$70,000+ per month continuously, violating the non-profit archive's budget constraint.
</details>

---

### Question 8
A search architect is tuning an HNSW vector index in an Amazon OpenSearch Service cluster (`"engine": "faiss"`) for a 20-million-document technical knowledge base. Currently, the index is configured with `"m": 16`, `"ef_construction": 64`, and `"ef_search": 40`, using 32-bit floating-point vectors (`fp32`). Benchmark evaluations show that Recall@10 is `0.86`, which falls short of the required `0.94` Recall@10 target. Additionally, the cluster's JVM and native off-heap memory utilization is already at 78%, so the architect must also reduce the memory footprint per vector by approximately 50% during the reindex.

Which combination of configuration changes will simultaneously boost Recall@10 toward `0.94` AND reduce the vector index memory footprint by ~50%?

- **A.** Keep all HNSW parameters identical, but pad every 1024-dimensional vector with 1024 zeros to create a 2048-dimensional vector.

- **B.** Decrease `"ef_construction"` to `8` and `"ef_search"` to `4`, and switch the vector data type from `float32` to `float64`.
- **C.** Switch from `"engine": "faiss"` to `"engine": "nmslib"`, set `"m": 2`, and enable 3 replica shards per primary shard.
- **D.** Increase `"ef_construction"` to `256` and `"ef_search"` to `128` to improve HNSW graph quality and search breadth, and configure `faiss` Scalar Quantization (`"encoder": {"name": "sq", "parameters": {"type": "fp16"}}`) to compress 32-bit floats into 16-bit floats.
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
In OpenSearch's `faiss` HNSW implementation:
1. **Boosting Recall@10**: Increasing **`ef_construction`** (e.g., from `64` to `256`) evaluates a larger candidate pool while building the HNSW graph layers, producing higher-fidelity navigable links; increasing **`ef_search`** (e.g., from `40` to `128`) explores more candidate nodes at query time, directly increasing Recall@10.
2. **Reducing Memory by ~50%**: Configuring **Scalar Quantization (`sq` with `fp16`)** (or `int8` for 75% reduction) compresses each 32-bit float dimension into a 16-bit float in native memory with virtually zero loss in retrieval accuracy.

#### Distractor Analysis
- **Why A is incorrect**: Zero-padding vectors to 2048 dimensions doubles memory usage without adding any semantic information.
- **Why B is incorrect**: Lowering `ef_construction` and `ef_search` reduces Recall@10 even further, and `float64` doubles memory usage instead of cutting it in half.
- **Why C is incorrect**: Setting `m = 2` severely degrades recall, and adding 3 replica shards quadruples total cluster memory consumption.
</details>

---

### Question 9
An enterprise legal department stores 80,000 contracts in Amazon S3 for an Amazon Bedrock Knowledge Base. For each contract `s3://legal-kb-bucket/contracts/contract_991.pdf`, the ingestion pipeline also generates a companion metadata file in the same S3 path containing the following JSON structure:
```json
{
  "metadataAttributes": {
    "jurisdiction": "UK",
    "practice_area": "M&A",
    "effective_year": 2025,
    "is_active": true
  }
}
```
What exact file naming convention MUST be used for this companion metadata file in Amazon S3 so Amazon Bedrock Knowledge Bases automatically ingests and indexes these attributes alongside every chunk of `contract_991.pdf`?

- **A.** `s3://legal-kb-bucket/contracts/contract_991.pdf.metadata.json`
- **B.** `s3://legal-kb-bucket/contracts/contract_991.metadata.json`
- **C.** `s3://legal-kb-bucket/metadata/contract_991.json`
- **D.** `s3://legal-kb-bucket/contracts/_meta_contract_991.pdf.yaml`

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
Amazon Bedrock Knowledge Bases requires companion S3 metadata files to reside in the **exact same S3 prefix** as the source document and append **`.metadata.json` to the full source filename including its original extension**: `<source_filename>.<ext>.metadata.json` (therefore, `contract_991.pdf.metadata.json`). If the `.pdf` extension is omitted before `.metadata.json`, Bedrock will either ignore the file or attempt to ingest the JSON file itself as a separate text document!

#### Distractor Analysis
- **Why B is incorrect**: `contract_991.metadata.json` omits the `.pdf` extension from the base filename, violating the Bedrock Knowledge Base S3 metadata naming contract.
- **Why C is incorrect**: Placing the file in a separate `/metadata/` prefix without the `.pdf.metadata.json` suffix causes Bedrock not to associate the metadata with `contract_991.pdf`.
- **Why D is incorrect**: Bedrock Knowledge Bases requires JSON format with the `.metadata.json` suffix, not YAML or `_meta_` prefixes.
</details>

---

### Question 10
A team has configured an Amazon Bedrock Knowledge Base backed by Amazon Aurora PostgreSQL Serverless v2 (`pgvector`). Dense semantic search (`overrideSearchType="SEMANTIC"`) works as expected. However, when a developer updates the `Retrieve` API call to use Hybrid Search (`overrideSearchType="HYBRID"`), the API returns a `ValidationException` / database execution error indicating that full-text lexical search could not be executed effectively on the target PostgreSQL table.

What database configuration step is required on the Aurora PostgreSQL table to support Amazon Bedrock Knowledge Base Hybrid Search?

- **A.** Change the primary key column type from `uuid` to `bytea`.
- **B.** Drop the `vector` extension and install the `pg_cron` extension instead.
- **C.** Ensure the `textField` column mapped in `storageConfiguration.rdsConfiguration.fieldMapping` is indexed with a PostgreSQL **GIN (Generalized Inverted Index)** over `to_tsvector` (e.g., `CREATE INDEX ON schema.table USING gin (to_tsvector('english', text_column));`).
- **D.** Disable the RDS Data API and connect via an RDS Proxy using IAM authentication.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: C**

#### Explanation
When `overrideSearchType="HYBRID"` is invoked against an **Amazon Aurora PostgreSQL (`pgvector`)** Knowledge Base, Bedrock combines dense k-NN vector similarity (`<=>` cosine distance operator) with PostgreSQL native **Full-Text Search** (`to_tsvector` / `to_tsquery` ranked via `ts_rank`). To support efficient full-text keyword search on the mapped `textField` column in Aurora PostgreSQL, you must create a **GIN index** using `to_tsvector('english', <text_field>)`.

#### Distractor Analysis
- **Why A is incorrect**: Bedrock Knowledge Bases requires the mapped primary key column in Aurora PostgreSQL to be of type `uuid`.
- **Why B is incorrect**: Dropping the `vector` extension breaks all vector operations, and `pg_cron` is a job scheduler unrelated to text search.
- **Why D is incorrect**: Bedrock Knowledge Bases *requires* the **RDS Data API** (`enable-http-endpoint`) to communicate with Aurora PostgreSQL; disabling it breaks the Knowledge Base completely.
</details>

---

### Question 11
A principal engineer is evaluating the vector dimension configuration of **Amazon Titan Text Embeddings V2 (`amazon.titan-embed-text-v2:0`)** for a new 80-million-chunk RAG platform. The team wants to reduce their OpenSearch Serverless Search OCU memory requirements and cut raw vector storage size by **75%** while keeping retrieval accuracy within ~95–97% of the baseline 1024-dimension configuration.

How should the engineer configure the `bedrockEmbeddingModelConfiguration` when creating the Knowledge Base (or calling `InvokeModel` on Titan Text Embeddings V2)?

- **A.** Switch to `amazon.titan-embed-text-v1`, which natively outputs 128-dimensional binary vectors.

- **B.** Set `"dimensions": 1024` and manually delete the first 768 floating-point numbers from each returned array in Python without re-normalizing the vector.
- **C.** Set `"dimensions": 4096` and `"embeddingDataType": "FLOAT64"`.
- **D.** Set `"dimensions": 256` and `"normalize": true` (since Titan Text Embeddings V2 uses Matryoshka Representation Learning to front-load the most salient semantic information into the leading 256 or 512 dimensions).
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
**Amazon Titan Text Embeddings V2** is trained using **Matryoshka Representation Learning (MRL)** and supports native output dimensions of **1024, 512, or 256** (as well as `FLOAT32` or `BINARY` embedding data types). Setting `"dimensions": 256` reduces the vector size from 1024 floats (4,096 bytes) to 256 floats (1,024 bytes)—an exact **75% reduction in vector footprint**—while preserving properly normalized unit vectors and ~97% of the 1024-d retrieval accuracy.

#### Distractor Analysis
- **Why A is incorrect**: Legacy `amazon.titan-embed-text-v1` outputs fixed **1536-dimensional** `float32` vectors and does not support configurable dimensionality or binary output.
- **Why B is incorrect**: Manually slicing off dimensions without re-normalizing breaks the unit-norm property required for dot-product/cosine equivalence, and deleting the *first* 768 dimensions destroys the most important principal components in Matryoshka embeddings!
- **Why C is incorrect**: Titan Text Embeddings V2 has a maximum dimension of 1024 (not 4096), and increasing dimensions/precision increases memory rather than reducing it by 75%.
</details>

---

### Question 12
An engineering team is ingesting 50,000 scanned PDF equipment manuals containing intricate hydraulic circuit diagrams, multi-page calibration tables, and visually formatted warning callouts into an Amazon Bedrock Knowledge Base. When using the Knowledge Base's **Default** text parser, the resulting chunks lose the structure of the calibration tables and completely omit the information depicted inside the hydraulic circuit diagrams. The team wants to configure the Bedrock Knowledge Base data source to natively parse and describe visual diagrams and complex tables during ingestion without building a separate pre-ingestion pipeline outside Bedrock.

Which two native parsing options available in the Amazon Bedrock Knowledge Base Data Source configuration (`parsingConfiguration`) will solve this problem? (Choose TWO.)

- **A.** Configure `parsingStrategy = "BEDROCK_DATA_AUTOMATION"`, which uses Amazon Bedrock Data Automation to parse multimodal PDF layouts, tables, and embedded figures during Knowledge Base ingestion.
- **B.** Configure `parsingStrategy = "BEDROCK_FOUNDATION_MODEL"` and specify a multimodal foundation model ARN (such as `anthropic.claude-3-5-sonnet-20241022-v2:0` or `amazon.nova-pro-v1:0`) with custom parsing prompt instructions to extract tables as Markdown and describe hydraulic diagrams.
- **C.** Keep the Default text parser, but change the vector store from OpenSearch Serverless to Amazon PineCone.
- **D.** Set `chunkingStrategy = "NONE"` so the Default parser skips OCR entirely.
- **E.** Enable Cross-Region Inference on the `amazon.titan-embed-text-v2:0` embedding model.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answers: A, B**

#### Explanation
Amazon Bedrock Knowledge Bases provides two advanced native parsing strategies in `vectorIngestionConfiguration.parsingConfiguration` to overcome the limitations of standard text-only PDF extraction:
- **Why A is correct**: **`BEDROCK_DATA_AUTOMATION`** invokes Amazon Bedrock Data Automation natively during Knowledge Base ingestion to turn complex multimodal documents, tables, and visual diagrams into rich structured Markdown chunks.
- **Why B is correct**: **`BEDROCK_FOUNDATION_MODEL`** uses a multimodal FM (such as Claude 3.5 Sonnet or Amazon Nova Pro/Lite) to visually inspect each PDF page, transcribe complex nested tables accurately, and generate natural-language descriptions of diagrams and charts before chunking.

#### Distractor Analysis
- **Why C is incorrect**: Changing the downstream vector store does not fix information lost during the upstream PDF parsing phase.
- **Why D is incorrect**: Setting `chunkingStrategy = "NONE"` still uses the Default text parser and fails to extract visual hydraulic diagrams.
- **Why E is incorrect**: Cross-Region Inference routes API traffic across regions; it does not grant a text embedding model optical diagram-parsing capabilities.
</details>

---

### Question 13
A data engineer is configuring an Amazon Aurora PostgreSQL database with `pgvector` for a low-latency RAG application. The table contains 8 million vectors indexed with **IVFFlat** (`USING ivfflat (embedding vector_cosine_ops) WITH (lists = 4000)`). By default, `ivfflat.probes` is set to `1`. Users report that while queries execute in 3 milliseconds, the retrieval system frequently misses relevant documents that lie near the boundary between two Voronoi cluster cells. Furthermore, after ingesting 3 million new documents from a newly acquired subsidiary last week, overall retrieval recall dropped by another 18%.

What explains these two behaviors, and what should the engineer do?

- **A.** IVFFlat automatically deletes vectors older than 30 days; the engineer must enable Point-in-Time Recovery (PITR) in Amazon RDS.
- **B.** `ivfflat.probes = 1` only searches the single closest $k$-means centroid cell (missing boundary neighbors), and IVFFlat centroids remain static after initial index creation (suffering recall drift when data distributions shift). The engineer should increase `SET LOCAL ivfflat.probes = 40;` for immediate recall improvement, or migrate the index to **HNSW** (`USING hnsw`), which dynamically maintains graph connectivity across inserts without requiring centroid retraining.
- **C.** Setting `lists = 4000` causes PostgreSQL to encrypt 3,999 lists with KMS; the engineer should set `lists = 1` so IVFFlat searches only 1 vector.
- **D.** IVFFlat only supports Euclidean distance (`vector_l2_ops`); using `vector_cosine_ops` returns random rows.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: B**

#### Explanation
An **IVFFlat** index partitions the vector space into `lists` Voronoi cells using $k$-means clustering at index build time.
1. With default `ivfflat.probes = 1`, PostgreSQL only inspects the **1** nearest centroid bucket at query time; if a relevant vector sits just across the boundary in an adjacent bucket, it is missed. Increasing `ivfflat.probes` (e.g., to $\sqrt{\text{lists}} \approx 40\text{–}60$) searches multiple neighboring cells.
2. Because IVFFlat $k$-means centroids are never updated during `INSERT` operations, inserting 3 million new documents from a new domain causes **centroid drift** (requiring a full `REINDEX`). **HNSW (`USING hnsw`)** has no $k$-means training phase and handles dynamic inserts gracefully with superior recall-per-millisecond.

#### Distractor Analysis
- **Why A is incorrect**: `pgvector` IVFFlat indexes never auto-delete rows based on age.
- **Why C is incorrect**: `lists` controls the number of $k$-means clusters (nothing to do with KMS), and `lists = 1` degenerates into a slow full sequential scan.
- **Why D is incorrect**: IVFFlat fully supports `vector_cosine_ops`, `vector_l2_ops`, and `vector_ip_ops`.
</details>

---

### Question 14
A machine learning engineer is designing an ingestion pipeline for an Amazon Bedrock Knowledge Base. Documents stored in Amazon S3 are deleted and replaced daily as old product SKUs are discontinued. However, when a file `s3://catalog-kb/sku_104.pdf` is deleted from the S3 bucket and the engineer calls `StartIngestionJob`, the discontinued SKU's chunks remain searchable in the Amazon OpenSearch Serverless vector index, causing the customer chatbot to continue recommending discontinued products! Upon inspecting the `AWS::Bedrock::DataSource` resource configuration, the engineer finds `dataDeletionPolicy: "RETAIN"`.

How should the engineer remediate this issue so deleted S3 objects are automatically purged from the vector store during synchronization, and what happens if a manual `DeleteDataSource` call is made?

- **A.** Change `dataDeletionPolicy` on the Bedrock Knowledge Base Data Source from `"RETAIN"` to `"DELETE"`. With `"DELETE"`, removing an object from S3 and running `StartIngestionJob` purges its vector embeddings from the vector index (and deleting the data source deletes all its indexed vectors).
- **B.** Keep `dataDeletionPolicy: "RETAIN"`, and set `temperature = 0.0` on the foundation model so it ignores discontinued products.
- **C.** Enable S3 Object Lock in Compliance Mode on `s3://catalog-kb/`.
- **D.** Delete and recreate the entire OpenSearch Serverless collection every 24 hours via an AWS Step Functions state machine.

<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: A**

#### Explanation
In Amazon Bedrock Knowledge Bases, the data source `dataDeletionPolicy` controls how vector embeddings are lifecycle-managed when source files are removed or when the data source is deleted:
- **`DELETE`** (Default): When source files are deleted from S3 and `StartIngestionJob` runs (or when the data source is deleted), Bedrock deletes all corresponding vector chunks from the vector store.
- **`RETAIN`**: Retains the underlying vector embeddings in the vector store even after the source S3 objects or data source resource are deleted. Because the engineer had set `"RETAIN"`, deleted S3 files remained in OpenSearch! Switching `dataDeletionPolicy` to `"DELETE"` and running an ingestion sync purges the stale vectors.

#### Distractor Analysis
- **Why B is incorrect**: Setting `temperature = 0.0` makes decoding deterministic, but the FM will deterministically use the stale discontinued SKU chunks if they are retrieved from the vector store.
- **Why C is incorrect**: S3 Object Lock *prevents* S3 files from being deleted; it does not purge stale embeddings from OpenSearch.
- **Why D is incorrect**: Dropping and recreating the entire OpenSearch collection daily causes downtime and re-embedding costs for the entire catalog when native incremental sync with `dataDeletionPolicy="DELETE"` handles deletions cleanly.
</details>

---

### Question 15
An enterprise security architect is reviewing a custom vector search pipeline where an AWS Lambda function invokes **Cohere Embed v3 English (`cohere.embed-english-v3`)** on Amazon Bedrock to embed both 500-word technical manual chunks during offline ingestion AND short 8-word user troubleshooting questions at query time. During evaluation, retrieval accuracy is unexpectedly low (`Recall@5 = 0.61`). When the architect inspects the Lambda code, both the offline document ingestion function and the online query function pass `"input_type": "classification"` in the request body to `cohere.embed-english-v3`.

How should the `input_type` parameter be configured in the two Lambda functions to maximize retrieval accuracy?

- **A.** Remove `"input_type"` completely from the JSON payload so `cohere.embed-english-v3` defaults to BM25 sparse vectors.

- **B.** Set `"input_type": "clustering"` for both ingestion and query time.
- **C.** Set `"input_type": "search_query"` during document ingestion and `"input_type": "search_document"` when embedding the user's question.
- **D.** Set `"input_type": "search_document"` when embedding the 500-word knowledge base chunks during ingestion, and set `"input_type": "search_query"` when embedding the user's question at query time.
<details>
<summary><strong>Reveal Answer & Architectural Explanation</strong></summary>

**Correct Answer: D**

#### Explanation
**Cohere Embed v3 (`cohere.embed-english-v3` and `cohere.embed-multilingual-v3`)** uses asymmetric task-specific embedding adapters controlled by the mandatory `input_type` parameter:
- **`"search_document"`**: Optimizes embeddings for passage/document chunks stored in a vector database.
- **`"search_query"`**: Optimizes embeddings for short asymmetrical search queries to align with `"search_document"` vectors in cosine space.
Using `"classification"` for both forces symmetric classification representations rather than asymmetric passage retrieval representations, severely degrading RAG retrieval recall.

#### Distractor Analysis
- **Why A is incorrect**: `input_type` is a required field in the Cohere Embed v3 request schema on Amazon Bedrock (omitting it throws a `ValidationException`), and Cohere Embed v3 produces dense vectors, not BM25 sparse vectors.
- **Why B is incorrect**: `"clustering"` optimizes embeddings for unsupervised topic clustering, not asymmetric query-to-passage RAG retrieval.
- **Why C is incorrect**: Reversing `"search_query"` and `"search_document"` misaligns the asymmetric projections between short queries and long passages.
</details>
