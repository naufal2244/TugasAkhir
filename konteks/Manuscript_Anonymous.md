**"The System Knows Your Team's Performance, But Takes Forever to Tell You": A Comparative Study of Caching Strategy and Query Optimization on a Decision Support System in a Collaborative Virtual Office Application**

This study evaluates the impact of the Redis Cache-Aside pattern and ORM query optimization on the Decision Support System (DSS) execution path in Koladi, a Laravel-based collaborative virtual office application. Baseline profiling showed that the DSS snapshot mechanism still relied on PostgreSQL queries, while several ORM patterns caused excessive query execution. A controlled before-and-after benchmarking experiment was conducted on two endpoints, GET /statistik/suggestions and GET /statistik/modal-data, across three data volumes, two cache conditions, and ten valid repetitions per combination (240 measurements total). Four metrics were evaluated: response time, query count, memory utilization, and cache hit ratio. The optimized configuration consistently reduced query count, stabilized response time as data volume increased, controlled memory utilization, and showed more effective caching once cache hit ratio was read alongside query count. A paired-sample t-test confirmed that the reduction in query count was statistically significant across all combinations (*p* \< 0.001), while response time differed significantly mainly under cache-miss conditions and on the modal-data endpoint. These results indicate that combining Redis Cache-Aside with ORM query optimization meaningfully improves DSS execution efficiency for aggregation-heavy backend workflows. 

**Keywords:** Decision Support System, Laravel, ORM optimization, query count, Redis Cache-Aside, web application performance.

## **Introduction**

The global shift toward distributed work has pushed organizations to adopt collaborative platforms for structured team coordination across geographical boundaries \[1\]. Within such platforms, the Decision Support System (DSS) plays a distinct computational role: rather than presenting static data, it runs a staged aggregation pipeline over relational team-activity data to produce actionable productivity indicators. This web-based DSS paradigm is increasingly dominant \[2\]. It is defined as a system that assists managers in semi-structured decisions through human-computer interaction and data \[3\], accessed through a browser to support multi-user access at scale \[4\]. Koladi is a concrete instance of this paradigm: a Laravel 12.x / PostgreSQL 15 SaaS collaborative virtual office application combining communication, task management, scheduling, and document collaboration. Its DSS module computes 22 raw productivity metrics per workspace on every request, including task completion rate, workload distribution, and progress percentage, making it the most computationally intensive component of the system.

Response time is the primary metric for assessing responsiveness under controlled workloads \[5\], \[6\], and in relational aggregation pipelines it is directly shaped by the number and cost of database operations, making query count an equally important diagnostic indicator that isolates latency at the infrastructure level \[7\]. Memory utilization captures the cost of server-side object materialization from unrestricted ORM column retrieval \[8\], \[9\]. Cache hit ratio measures the share of requests served from cache; because the baseline relies on a database-backed snapshot while the optimized configuration uses Redis, cache hit ratio is interpreted together with SQL query count to check whether caching genuinely reduces database access \[10\], \[11\]

Direct inspection of the Koladi codebase, validated with Laravel Telescope, revealed two related sources of inefficiency. First, the baseline SnapshotManager stores DSS metrics in a relational table, so repeated requests still trigger SQL queries against workspace\_performance\_snapshots; the modal-data endpoint additionally shows a *partial cache hit*, in which only the primary metrics are snapshotted while urgent tasks and workload distribution are recomputed on every call. Second, three ORM inefficiencies were found: getTasksInPeriod() eagerly loads the entire assignedUsers relationship although workload calculations need only the id column (an Unnecessary Data Retrieval anti-pattern, \[7\], which increases memory use, \[9\]); the checklists relationship is lazy-loaded, triggering an N+1 query for every task inside getProgressPercentage() \[12\] and getWorkloadDistribution() runs a Task::whereHas() query inside a loop over workspace members, so query count grows with member count.

Two coordinated interventions address these issues. First, the database-backed cache is replaced with a Redis Cache-Aside pattern: Redis is checked first, and only on a miss does the system run the DSS pipeline and populate the cache. Second, ORM optimization targets both bottlenecks: the eager-loading directive is refactored from \-\>with(\['assignedUsers'\]) to \-\>with(\['assignedUsers:id', 'checklists:id,task\_id,is\_done'\]), cutting column over-fetching and eliminating the checklists N+1 pattern, while getWorkloadDistribution() is refactored from a query-in-loop into batch retrieval with in-memory aggregation. Both interventions are evaluated together because they interact over shared server resources: ORM efficiency determines the cost of a cache miss, while cache effectiveness determines how often the ORM layer runs at all. Studies integrating caching and ORM optimization within a single experimental framework remain limited \[5\]. To address this gap, this study is guided by the following research question: To what extent do response time, SQL query count, memory utilization, and cache hit ratio change on the GET /statistik/suggestions and GET /statistik/modal-data endpoints following the simultaneous implementation of Redis Cache-Aside and ORM query optimization?"

**Related Work**  
In-memory caching to reduce database latency is among the most consistently validated web-systems optimizations. A Redis-backed caching implementation in Spring Boot achieved up to a tenfold reduction in query time (197.5 ms to 30 ms), replacing direct database retrieval \[6\]. In Laravel specifically, average response time was cut from 800 ms to 150 ms with a 95% cache hit ratio on a 100,000-user dataset \[9\], while intercepting reads with Redis in a high-concurrency Node.js API reduced response time from 30.32 s to 1.66 s and raised write throughput by 50% \[13\]. The theoretical explanation is straightforward: memory access is roughly four orders of magnitude faster than disk retrieval \[14\]. At the cache-management layer, Monotonic Consistent Caching middleware was shown to reduce database reads by 39.09% and raise throughput by 77.15% versus conventional LRU \[10\]; the same work also showed that cache hit ratio alone is an unreliable proxy for system cost when the caching layer still performs backend validation, so it should be read together with query count.

A separate body of work addresses ORM-layer inefficiency. Unnecessary Data Retrieval has been formalized as a measurable anti-pattern, and eliminating it has been shown to cut response time by up to 71% \[7\]. Automated N+1 refactoring has been shown to reduce queries from 11 to 2, yielding up to a 38.58-fold performance gain as data scaled \[12\]. Lazy loading in Doctrine ORM has been found to generate 12,748 queries for a single request, cut to 2 through eager loading, with execution time falling from 2,999 ms to 1,348 ms \[15\]. Selective field loading has been reported to cut memory by 35–40%, while eager loading reduced queries from 15 to 2 (1,200 ms to 300 ms) \[9\]. Unrestricted ORM loading has been found to use two to three times more memory than raw SQL (977 KB vs. 357 KB) \[8\]. Most of these studies, however, evaluate query optimization independently of caching, leaving unclear how ORM efficiency shapes the cost of cache misses and how cache effectiveness shapes ORM execution frequency over time.

The combined interaction of caching and ORM optimization remains comparatively underexplored, a gap explicitly named in prior work \[5\]. The closest related work is \[9\], who combined Redis caching, eager loading, and selective field loading in an e-commerce system while measuring all four metrics used here, cutting response time from 2.4 s to 680 ms, query count from 47 to 5, and memory from 450 MB to 120 MB. However, that system was a generic product listing rather than a DSS pipeline performing staged aggregation, and DSS literature more broadly emphasizes decision accuracy over infrastructure responsiveness \[2\], \[4\] leaving optimization of aggregation-intensive DSS pipelines relatively unexplored.

Methodologically, before-and-after experimental designs are standard for evaluating technical interventions under controlled conditions \[5\], \[10\] typically using synthetic seeded/factory datasets for reproducibility \[8\], \[16\] and warm-up runs with repeated measurement to reduce cold-start bias \[6\], \[17\], \[18\]. A paired-sample t-test at 95% confidence has been used for exactly this kind of paired before/after comparison \[19\], and Docker Compose is a common reproducibility tool in recent studies \[8\], \[9\]. In sum, existing studies largely treat caching and ORM optimization as isolated interventions, rarely measuring response time, query count, memory, and cache hit ratio together, particularly for web-based DSS applications running relational aggregation pipelines. No identified study evaluates infrastructure optimization for a DSS module performing staged aggregation over relational team-activity data. This study addresses that gap by simultaneously implementing Redis Cache-Aside and selective eager loading within Koladi's DSS execution path, evaluated across all four metrics under controlled conditions computing 22 productivity metrics per workspace.

## **Methodology**

### ***Research Design***

This study follows Experimental Software Engineering framework\[20\], adopting a controlled before-and-after comparative experiment: the software artifact is Koladi's DSS module, with independent variables (caching mechanism, ORM implementation) deliberately manipulated and dependent variables (response time, query count, memory, cache hit ratio) measured precisely. This design enables direct causal attribution between each intervention and observed outcomes \[6\], \[12\]. The interventions target the GET /statistik/suggestions and GET /statistik/modal-data endpoints: replacing the database-backed cache with Redis Cache-Aside, and applying selective eager loading in getTasksInPeriod() plus refactoring getWorkloadDistribution() from a per-member query loop into batch retrieval with in-memory aggregation. Both are evaluated simultaneously since they interact over shared server resources: ORM efficiency shapes cache-miss cost, and cache effectiveness shapes how often the ORM layer runs \[5\]. Metrics are collected under identical data volumes, endpoints, cache conditions, and request parameters for both configurations, isolating the effect of the optimizations from confounding variation \[5\], \[6\].

The overall research workflow is illustrated in Figure 1 (Research Process Flow), which summarizes the sequential phases of the experimental procedure from baseline profiling through statistical evaluation. The study follows a multi-phase pipeline rather than a single monolithic test run, so that bottleneck identification, intervention, and evaluation remain analytically separable. Baseline Profiling identifies initial bottlenecks, namely excessive queries, database-backed caching, memory overhead, and N+1 patterns. Implementation then integrates Redis Cache-Aside and the two ORM refactors described above. Controlled Performance Measurement follows, running sequential (non-concurrent) requests against both endpoints under both configurations, which constitutes isolation/benchmarking rather than load or stress testing \[10\], \[21\]. Measurement and Data Collection subsequently records all four metrics across every combination of data volume, endpoint, configuration, and cache condition, with ten repetitions each after warm-up, for 240 total measurements (3 volumes × 2 endpoints × 2 configurations × 2 cache conditions × 10 repetitions); warm-up and repetition reduce cold-start bias \[6\], \[17\], \[18\]. Finally, Statistical Evaluation applies a paired-sample t-test (α \= 0.05) plus percentage-improvement calculations \[6\], \[12\], \[19\], so that both statistical significance and practical magnitude of change are captured together rather than relying on significance testing alone.  
![][image1]  
**Figure 1\.** Research Process Flow.

***System Environment***

The evaluation is scoped to the DSS execution path of the performance-statistics module (DSSService, SnapshotManager, MetricsCalculator, TrendAnalyzer, RecommendationEngine, plus getUrgentTasks() and getWorkloadDistribution() for modal-data), not the whole Koladi application. The stack runs in Docker Compose on an isolated network (koladi\_net) for configuration consistency \[6\], \[8\]. Baseline uses CACHE\_STORE=database with Redis disabled; the optimized configuration deploys Redis 7.x in the same network. Redis was chosen for its RAM-based retrieval and demonstrated ability to reduce database reads and shift I/O from PostgreSQL to memory \[6\], \[9\], \[10\], \[11\]. Table 1 summarizes the complete experimental environment, covering the platform, database, ORM, cache configuration for both conditions, deployment topology, measurement tooling, hardware, and repetition scheme used throughout the study.

**Table 1\.** Experimental Environment Specifications.

| Component | Description |
| :---- | :---- |
| Platform | Koladi, Laravel 12.x / PHP ^8.2 |
| Database | PostgreSQL 15 |
| ORM | Eloquent ORM |
| Cache \- Baseline | Database-backed snapshot using CACHE\_STORE=database |
| Cache \- Optimized | Redis 7.x (redis:7-alpine, CACHE\_STORE=redis, phpredis client) |
| Redis Client | phpredis using REDIS\_CLIENT=phpredis |
| Deployment | Docker Compose (koladi\_net): Nginx, PHP-FPM, PostgreSQL, Redis |
| Measurement Tools | Laravel middleware instrumentation, query listener, memory\_get\_peak\_usage(true), Redis INFO STATS, Node.js benchmark runner, CSV validator, Laravel Telescope (profiling only) |
| Hardware / OS | Apple M2 Pro, 16 GB RAM, 994.66 GB SSD, macOS Tahoe |
| Repetitions | 10 valid per combination \+ 3 excluded warm-ups; 120 valid measurements per configuration  |

 

***Baseline Profiling***

As shown in Figure 2, Telescope profiling confirmed that the snapshot mechanism is not a true database bypass. The suggestions endpoint still issues several queries against workspace\_performance\_snapshots on repeated identical requests, and the modal-data endpoint shows a partial cache hit, where only the primary metrics are snapshotted while urgent tasks and workload distribution are recomputed live. As a result, repeated modal-data requests still run a substantial number of SQL queries even when the snapshot itself is successfully retrieved. This behavior contrasts with Redis, which is designed to bypass the database entirely and access data directly from RAM rather than relying on a relational round trip for every repeated request \[6\].

![][image2]  
**Figure 2**. Repeated Snapshot Queries in the DSS Execution Path.

As illustrated in Figure 3, profiling on the ORM side confirmed three inefficiencies described earlier in the Introduction. The unrestricted assignedUsers eager load, classified as an Unnecessary Data Retrieval anti-pattern \[7\], has been shown to raise memory consumption by up to 40% in Laravel applications \[9\]. In addition, the checklists relationship is lazy-loaded, so every invocation of getProgressPercentage() during a single request triggers one additional query per task, producing a query count that scales directly with the number of tasks processed. Meanwhile, getWorkloadDistribution() executes a Task::whereHas() query inside a loop over workspace members, so its query count instead scales with the number of active members. These are two distinct linear-growth problems originating from different execution paths within the same DSS pipeline.

![][image3]

**Figure 3**. ORM Query Inefficiencies in the modal-data Endpoint.

***Implementation of Proposed Optimizations***

As illustrated in Figure 4, the database-backed cache was replaced with Redis via the Laravel Cache facade using the Cache-Aside pattern. On each request, Redis is checked first; on a hit, the response is returned directly without touching PostgreSQL; on a miss, the DSS pipeline is executed, the generated results are stored in Redis, and the response is returned to the client \[6\], \[22\]. For the modal-data endpoint specifically, caching now covers the entire response structure, including the primary metrics, the urgent task list, and the workload distribution, which turns the baseline's partial cache hit behavior into a full response cache for repeated requests with identical parameters.  
![][image4]

**Figure 4\.** Redis Cache-Aside Architecture for the DSS Execution Path.

As summarized in Figure 5, ORM-level changes were applied at two bottlenecks. The eager-loading directive in getTasksInPeriod() was refactored from \-\>with(\['assignedUsers'\]) to \-\>with(\['assignedUsers:id', 'checklists:id,task\_id,is\_done'\]), which restricts the assignedUsers relationship to the required column and eagerly loads the checklists relationship, eliminating the associated N+1 query pattern \[7\], \[12\]. Separately, getWorkloadDistribution() was refactored to retrieve task and assignedUsers data in a single batch rather than issuing one Task::whereHas() query per workspace member, after which workload distribution is computed entirely through in-memory aggregation.

![][image5]

**Figure 5\.** ORM Optimizations in getTasksInPeriod() and getWorkloadDistribution().

***Controlled Before-and-After Performance Testing***

Both endpoints were tested with synthetic data generated via Laravel Seeders/Factories (not production data), giving precise, deterministic control over volume, status distribution, and task-member density for fair comparison \[8\], \[16\]. Task status distribution was fixed at 40% completed / 35% in progress / 25% to-do, with a subset overdue to exercise time-dependent metrics. Three dataset sizes were used: Small (50 tasks, 5 members), Medium (200 tasks, 15 members), Large (500 tasks, 30 members), representing workload levels for Koladi rather than replications of prior studies, chosen so that increasing tasks exercises the checklists N+1 pattern and increasing members exercises the workload-distribution query loop.

Each endpoint was tested under 2 configurations × 2 cache conditions. Under cache miss, the relevant cache was cleared before every request; under cache hit, an excluded initialization request first populated the cache. The full factorial design (dataset size × endpoint × configuration × cache condition × repetition) yielded 240 valid measurements after excluding 3 warm-up requests per combination \[17\], \[18\], \[23\]. Table 2 lays out this full experimental design matrix, listing every factor and its variations alongside the resulting count of valid and excluded observations, so that the relationship between the factorial structure and the final 240 measurements is fully traceable. Structuring the design this way ensures that every combination of dataset size, endpoint, configuration, and cache condition is represented by the same number of repetitions, which is a precondition for the paired-sample comparisons reported later in the Results section.

**Table 2\.** Experimental Design Matrix.

| Factor | Variations | Number of Variations | Description |
| :---- | :---- | :---- | :---- |
| Dataset size | Small, Medium, Large | 3 | 50/5, 200/15, 500/30 tasks/members |
|  DSS endpoint  |  suggestions, modal-data  | 2 | Two DSS execution paths |
| System configuration | Baseline, Optimized | 2 | DB snapshot vs. Redis \+ ORM optimization |
| Cache condition | Miss, Hit | 2 | Full execution cost vs. cache retrieval efficiency |
| Valid repetitions | 10 per combination | 10 | Recorded after warm-up |
| Warm-up | 3 requests per combination | — | Excluded from analysis |
| Total valid measurements | 3 x 2 x 2 x 2 x 10 | 240 | 120 baseline \+ 120 optimized  |

## ***Measurement Instruments***

A custom instrumentation suite recorded every request automatically, consistent with established practice for controlled web-performance evaluation \[9\], \[16\], \[21\]. It comprises a Laravel middleware (enabled only in local/testing, activated only via a dedicated benchmark header) and a Node.js runner executing requests sequentially, chosen over concurrent execution because the goal is controlled benchmarking rather than load testing. Telescope was used only for baseline profiling; all experimental data came from the instrumentation suite. Response time is the runner's end-to-end HTTP duration; query count comes from Laravel's query listener; memory is peak PHP usage via memory\_get\_peak\_usage(true), both returned through debug response headers. Cache hit ratio in the optimized configuration comes from the change in Redis keyspace\_hits/keyspace\_misses around each request; in baseline, a hit is recorded when SnapshotManager successfully retrieves a snapshot, though this is not interpreted as a full database bypass, since PostgreSQL is still involved and modal-data may still run extra queries for urgent tasks and workload distribution. All measurements were stored as CSV and validated for completeness before analysis.

## ***Measurement and Data Collection***

Only one identified study has measured all four metrics simultaneously, though in a generic e-commerce rather than DSS context \[9\]. Response time (ms) is the primary indicator of web application performance \[5\], recorded per request without simulated concurrent load, following isolation/configuration testing principles \[10\], \[21\], \[24\]. SQL query count reflects reduced database round-trips from caching and the eliminated checklists N+1 pattern \[9\]. A comparable 15-to-2 reduction from similar refactoring has been reported \[12\]. Memory utilization (peak RAM via memory\_get\_peak\_usage()) captures the effect of restricting assignedUsers to the id column while eager-loading checklists; selective loading has been reported to reduce memory by 35–40% \[9\], while unrestricted ORM retrieval uses two to three times more memory than column-restricted queries \[8\]. Cache hit ratio, in the optimized configuration, is computed using Equation (1), following \[11\].

| Cache Hit Ratio (%) \= (keyspace\_hits / (keyspace\_hits \+ keyspace\_misses)) × 100%  |  (1) |
| :---- | :---- |

Because baseline cache hits may still involve SQL queries against workspace\_performance\_snapshots, cache hit ratio is always interpreted together with query count, consistent with \[10\], who showed database read operations better represent system cost than cache hit rate alone for certain caching mechanisms. This caveat matters specifically because Equation (1) treats a Redis keyspace hit as binary, without distinguishing whether the underlying response still required any residual database access, so the ratio it produces can look identical across configurations that in fact behave very differently at the infrastructure level. Table 3 summarizes the definition, unit, source, and interpretation of all four metrics used throughout this study, including the query-count caveat associated with Equation (1), and is intended as a single reference point for how each metric should be read during the Results and Discussion sections that follow.

**Table 3\.** Measurement Parameters,

| Metric | Definition | Unit | Source | Interpretation |
| :---- | :---- | :---- | :---- | :---- |
| Response time | HTTP request-to-response duration | ms | Node.js runner | Database round-trip cost |
| SQL Query count | Queries executed per request | Query/request | Laravel query listener | Database round-trip cost |
| Memory utilization | Peak PHP memory per request | MB | memory\_get\_peak\_usage(true) | PHP object materialization overhead |
| Cache hit ratio | Share of cache accesses served from cache | % | Redis keyspace stats (optimized) / snapshot retrieval (baseline)  | Must be read together with query count |

## **Statistical Evaluation**

Each combination was repeated ten times after a warm-up phase \[6\] and summarized by mean (primary comparison measure), with minimum, maximum, and standard deviation indicating stability; significance was assessed separately via paired-sample t-test \[8\], \[23\]. A paired-sample t-test (α \= 0.05) was chosen because each baseline observation has a directly corresponding optimized observation under identical dataset size, endpoint, cache condition, and repetition order \[19\]. For deterministic metrics like query count, near-zero variance in paired differences can produce extreme statistics and very small p-values \[18\], \[19\]; such results are reported as *p* \< 0.001 rather than *p* \= 0, following \[19\] and \[18\], and interpreted alongside percentage improvement rather than significance alone. Percentage change, calculated for response time, query count, and memory utilization, is reported as a practical-significance measure \[5\], \[9\]; changes below 3% are interpreted cautiously as possible measurement noise \[23\]). Cache hit ratio was excluded from the t-test itself, since it is measured differently across configurations (Redis stats vs. snapshot retrieval), and is instead read jointly with query count: a baseline hit may still involve PostgreSQL queries, while an optimized hit is corroborated by a much lower query count \[10\]. Final conclusions rest on the combination of statistical significance, percentage improvement, and consistency across all four metrics.

**Result**

## ***Baseline Configuration Results***

**Table 4**. Summary of Baseline Measurement Results.

| Dataset Size | Endpoint | Cache Condition | n | Response Time Mean ± SD (ms) | Query Count Mean ± SD | Memory Mean ± SD (MB) | Cache Hit Ratio Mean ± SD (%) |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| Small | suggestions | Cache miss | 10 | 122.49 ± 24.75 | 63.00 ± 0.00 | 4.00 ± 0.00 | 0.00 ± 0.00 |
| Small | suggestions | Cache hit | 10 | 57.64 ± 25.73 | 8.00 ± 0.00 | 4.00 ± 0.00 | 100.00 ± 0.00 |
| Small | modal-data | Cache miss | 10 | 149.85 ± 25.49 | 87.00 ± 0.00 | 4.00 ± 0.00 | 0.00 ± 0.00 |
| Small | modal-data | Cache hit | 10 | 84.07 ± 26.98 | 32.00 ± 0.00 | 4.00 ± 0.00 | 100.00 ± 0.00 |
| Medium | suggestions | Cache miss | 10 | 276.17 ± 28.74 | 213.00 ± 0.00 | 8.00 ± 0.00 | 0.00 ± 0.00 |
| Medium | suggestions | Cache hit | 10 | 63.96 ± 27.56 | 8.00 ± 0.00 | 4.40 ± 0.84 | 100.00 ± 0.00 |
| Medium | modal-data | Cache miss | 10 | 330.61 ± 23.86 | 292.00 ± 0.00 | 8.00 ± 0.00 | 0.00 ± 0.00 |
| Medium | modal-data | Cache hit | 10 | 159.56 ± 40.48 | 87.00 ± 0.00 | 6.00 ± 0.00 | 100.00 ± 0.00 |
| Large | suggestions | Cache miss | 10 | 519.98 ± 32.79 | 513.00 ± 0.00 | 14.00 ± 0.00 | 0.00 ± 0.00 |
| Large | suggestions | Cache hit | 10 | 60.45 ± 22.98 | 8.00 ± 0.00 | 5.80 ± 1.75 | 100.00 ± 0.00 |
| Large | modal-data | Cache miss | 10 | 678.95 ± 30.52 | 696.00 ± 0.00 | 14.00 ± 0.00 | 0.00 ± 0.00 |
| Large | modal-data | Cache hit | 10 | 259.86 ± 59.28 | 191.00 ± 0.00 | 10.20 ± 0.63 | 100.00 ± 0.00 |

As shown in Table 4, under cache miss query count grew sharply with dataset size for both endpoints (suggestions: 63→213→513; modal-data: 87→292→696), and response time grew correspondingly (suggestions: 122.49→519.98 ms; modal-data: 149.85→678.95 ms). Under cache hit, query count for suggestions stayed flat at 8, but modal-data still grew from 32 to 191, direct evidence of the partial cache hit problem, since a true bypass would not scale with data volume. Memory utilization under cache miss rose from \~4 MB (Small) to 14 MB (Large). Cache hit ratio was 0% under all miss scenarios and 100% under all hit scenarios, but this cannot be read as full database bypass given the concurrent query counts above; this apparent contradiction between a nominally perfect hit ratio and continued database load is precisely the partial-cache-hit issue described earlier and is examined further in Table 6\.

### ***Optimized Configuration Results***

**Table 5**. Summary of Optimized Measurement Results.

| Dataset Size | Endpoint | Cache Condition | n | Response Time Mean ± SD (ms) | Query Count Mean ± SD | Memory Mean ± SD (MB) | Cache Hit Ratio Mean ± SD (%) |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| Small | suggestions | Cache miss | 10 | 62.84 ± 25.53 | 10.00 ± 0.00 | 4.00 ± 0.00 | 50.00 ± 0.00 |
| Small | suggestions | Cache hit | 10 | 53.21 ± 11.32 | 6.00 ± 0.00 | 4.00 ± 0.00 | 100.00 ± 0.00 |
| Small | modal-data | Cache miss | 10 | 72.99 ± 29.38 | 22.00 ± 0.00 | 4.00 ± 0.00 | 0.00 ± 0.00 |
| Small | modal-data | Cache hit | 10 | 61.46 ± 23.49 | 6.00 ± 0.00 | 4.00 ± 0.00 | 100.00 ± 0.00 |
| Medium | suggestions | Cache miss | 10 | 63.83 ± 30.55 | 10.00 ± 0.00 | 4.00 ± 0.00 | 50.00 ± 0.00 |
| Medium | suggestions | Cache hit | 10 | 64.34 ± 24.28 | 6.00 ± 0.00 | 4.40 ± 0.84 | 100.00 ± 0.00 |
| Medium | modal-data | Cache miss | 10 | 69.48 ± 32.35 | 15.00 ± 0.00 | 4.00 ± 0.00 | 0.00 ± 0.00 |
| Medium | modal-data | Cache hit | 10 | 60.96 ± 24.84 | 6.00 ± 0.00 | 4.00 ± 0.00 | 100.00 ± 0.00 |
| Large | suggestions | Cache miss | 10 | 64.93 ± 29.13 | 10.00 ± 0.00 | 4.00 ± 0.00 | 50.00 ± 0.00 |
| Large | suggestions | Cache hit | 10 | 65.85 ± 24.09 | 6.00 ± 0.00 | 4.00 ± 0.00 | 100.00 ± 0.00 |
| Large | modal-data | Cache miss | 10  | 68.02 ± 28.45 | 15.00 ± 0.00 | 4.00 ± 0.00 | 0.00 ± 0.00 |
| Large | modal-data | Cache hit | 10 | 54.72 ± 28.86 | 6.00 ± 0.00 | 4.00 ± 0.00 | 100.00 ± 0.00 |

As shown in Table 5, query count and response time now stay essentially flat as data volume increases: suggestions holds at 10 queries (miss) / 6 (hit) across all sizes, with response time in a narrow 62.84–65.85 ms band; modal-data holds at 15 queries (miss, Medium/Large) / 6 (hit) with response time in a 54.72–72.99 ms band. Memory utilization is flat at 4.00 MB in nearly every combination, unlike the baseline's growth to 14 MB. The suggestions endpoint's 50% cache-miss hit ratio reflects one internal cache-miss-then-hit sequence within a single request (the suggestions data is invoked twice), not degraded caching. Overall, Redis Cache-Aside plus ORM optimization stabilized both response time and query count regardless of dataset size, and modal-data's cache-hit query count now approaches a genuine bypass at a constant 6 queries.

### ***Comparison and Statistical Significance***

**Table 6**. Performance Comparison Between the Baseline and Optimized Configurations.

| Dataset | Endpoint | Cache Condition | Response Time Reduction (%) | SQL Query Count Reduction (%) | Memory Utilization Reduction (%) | Cache Hit Ratio (Baseline → Optimized) |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| Small | suggestions | Cache miss | 48.7 | 84.1 | 0.0 | 0% → 50% |
| Small | suggestions | Cache hit | 7.7 | 25.0 | 0.0 | 100% → 100% |
| Small | modal-data | Cache miss | 51.3 | 74.7 | 0.0 | 0% → 0% |
| Small | modal-data | Cache hit | 26.9 | 81.2 | 0.0 | 100% → 100% |
| Medium | suggestions | Cache miss | 76.9 | 95.3 | 50.0 | 0% → 50% |
| Medium | suggestions | Cache hit | \-0.6 | 25.0 | 9.1 | 100% → 100% |
| Medium | modal-data | Cache miss | 79.0 | 94.9 | 50.0 | 0% → 0% |
| Medium | modal-data | Cache hit | 61.8 | 93.1 | 33.3 | 100% → 100% |
| Large | suggestions | Cache miss | 87.5 | 98.1 | 71.4 | 0% → 50% |
| Large | suggestions | Cache hit | \-8.9 | 25.0 | 31.0 | 100% → 100% |
| Large | modal-data | Cache miss | 90.0 | 97.8 | 71.4 | 0% → 0% |
| Large | modal-data | Cache hit | 78.9 | 96.9 | 60.8 | 100% → 100% |

As presented in Table 6, improvements grow with dataset size, especially under cache miss: response-time reduction on suggestions rises from 48.7% (Small) to 87.5% (Large), and on modal-data from 51.3% to 90.0%. Query-count reduction is large and fairly stable across sizes (74–98%), reflecting the elimination of dataset-scaling query growth. Memory reduction is negligible for Small (both configurations already sit near 4 MB) but reaches 71.4% for Large. Under cache hit, suggestions shows small or even slightly negative response-time change (-0.6% to \-8.9%) because both configurations are already fast, but query count still drops 25%, and modal-data's cache-hit query count falls 81–97%, indicating a much closer approach to true database bypass than the baseline snapshot mechanism.

**Table 7**. Paired-Sample t-Test Results.

| Dataset Size | Endpoint | Cache Condition | n | Response Time (p-value) | SQL Query Count (p-value) | Memory Utilization (p-value) |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| Small | suggestions | Cache miss | 10 | \<0,001 | \<0,001 | 1,000000 |
| Small | suggestions | Cache hit | 10 | 0,209235 | \<0,001 | 1,000000 |
| Small | modal-data | Cache miss | 10 | \<0,001 | \<0,001 | 1,000000 |
| Small | modal-data | Cache hit | 10 | 0,524301 | \<0,001 | 1,000000 |
| Medium | suggestions | Cache miss | 10 | \<0,001 | \<0,001 | \<0,001 |
| Medium | suggestions | Cache hit | 10 | 0,118266 | \<0,001 | 0,167851 |
| Medium | modal-data | Cache miss | 10 | \<0,001 | \<0,001 | \<0,001 |
| Medium | modal-data | Cache hit | 10 | \<0,001 | \<0,001 | \<0,001 |
| Large | suggestions | Cache miss | 10 | \<0,001 | \<0,001 | \<0,001 |
| Large | suggestions | Cache hit | 10 | 0,523675 | \<0,001 | 0,009991 |
| Large | modal-data | Cache miss | 10 | \<0,001 | \<0,001 | \<0,001 |
| Large | modal-data | Cache hit | 10 | \<0,001 | \<0,001 | \<0,001 |

As reported in Table 7, query count differences were significant (*p* \< 0.001) in every combination, the strongest and most consistent result in the study. Response time was significant under cache miss for all sizes and both endpoints, consistent with the descriptive stabilization shown above; under cache hit, suggestions showed no significant difference at any size (both configurations already fast), whereas modal-data was significant for Medium and Large, reflecting the baseline's partial-cache-hit overhead being eliminated. Memory differences were non-significant for Small (*p* \= 1.000000, both configurations near-identical) but significant for most Medium/Large combinations, matching the growing baseline memory footprint observed earlier. Overall, the largest and most consistent statistical gains are in query count and cache-miss response time, concentrated on the modal-data endpoint, which previously carried higher costs from partial cache hits and additional response components.

**Discussion** 

This study was designed to examine the extent to which the simultaneous implementation of Redis Cache-Aside and ORM query optimization affects response time, query count, memory utilization, and cache hit ratio across the two Koladi DSS endpoints, GET /statistik/suggestions and GET /statistik/modal-data. Based on the experimental results, this research question has been addressed. The optimized configuration consistently outperformed the baseline, primarily through lower query counts, more stable response times, and more efficient memory utilization.

The findings support the initial assumption that the performance bottlenecks in the Koladi DSS originated from the combined effects of the baseline caching mechanism and inefficient ORM query patterns. Under the baseline configuration, the database-backed snapshot mechanism did not function as a true database bypass, as SQL queries were still executed during cache hits, particularly for the modal-data endpoint, which exhibited a partial cache hit behavior. After implementing Redis Cache-Aside, cache retrieval no longer depended on the workspace\_performance\_snapshotstable, allowing cache hits in the optimized configuration to more closely approximate a true database bypass. This finding is consistent with \[10\], who argued that cache effectiveness should be evaluated not only by cache hit rate but also in conjunction with database read operations.

Regarding response time, the optimized configuration demonstrated greater stability as data volume increased. Under the baseline configuration, cache miss response time increased substantially from the Small to the Large dataset, whereas the optimized configuration maintained response times within a much narrower range. This indicates that the proposed optimizations effectively mitigated the growth in DSS execution cost as the numbers of tasks and workspace members increased. These findings are consistent with previous studies that identify response time as the primary indicator of web application responsiveness under controlled performance evaluations \[5\], \[6\].

The most substantial improvement was observed in query count. Under the baseline configuration, the number of SQL queries increased with data volume because of N+1 queries in the checklists relationship and loop-based queries within getWorkloadDistribution(). Following the ORM optimizations, query counts became significantly lower and remained stable across different data volumes. These results are consistent with \[12\], who demonstrated that eliminating N+1 queries can substantially improve performance, particularly as the number of processed entities increases. In the context of Koladi, query count provides strong technical evidence that the proposed optimizations not only reduced request execution time but also structurally decreased round-trips to PostgreSQL.

From the perspective of memory utilization, the optimized configuration also produced more controlled results. Memory consumption increased with larger datasets under the baseline configuration, whereas it remained constant under the optimized configuration. This finding supports previous studies suggesting that unrestricted ORM loading and unnecessary column retrieval increase PHP object materialization overhead \[8\], \[9\]. By restricting the assignedUsers relationship to only the required columns and eagerly loading the checklistsrelationship, the system reduced object materialization costs without altering the functional behavior of the DSS endpoints.

The cache hit ratio findings further emphasize the importance of interpreting performance metrics collectively rather than independently. Although both configurations achieved a 100% cache hit ratio under cache hit conditions, their performance implications differed substantially. Under the baseline configuration, cache hits could still involve PostgreSQL queries, whereas cache hits under the optimized configuration were accompanied by significantly lower query counts. Furthermore, the 50% cache hit ratio observed during cache miss conditions for the suggestions endpoint does not indicate poor cache performance. Instead, it reflects the execution flow within a single request, where the first access resulted in a cache miss and populated Redis, while the subsequent access successfully retrieved the newly cached data. This behavior demonstrates that cache hit ratio should be interpreted alongside query count rather than as a standalone performance metric.

From a research contribution perspective, this study provides a simultaneous evaluation of Redis Cache-Aside and ORM query optimization within the execution path of a Decision Support System (DSS). This approach addresses the research gap identified by \[5\], who noted the limited number of studies evaluating the integration of caching strategies and query optimization techniques. The two optimization approaches complement one another: ORM optimization reduces computational and memory costs during cache misses, whereas Redis Cache-Aside decreases the frequency of ORM pipeline execution by serving requests directly from the cache. Consequently, evaluating response time, query count, memory utilization, and cache hit ratio together provides a more comprehensive assessment of DSS performance improvements than evaluating each optimization independently.

The use of synthetic datasets was also an integral component of the controlled experimental design. Datasets generated through Laravel Seeder and Factory enabled consistent control over data volume, status distribution, and task-member relationship density across both baseline and optimized configurations. This approach is consistent with \[16\] and \[8\], who employed synthetic data to ensure experimental control and reproducibility. As a result, the observed performance differences can be more confidently attributed to the proposed optimizations rather than variations in the underlying data. Nevertheless, the interpretation of these findings should remain limited to the controlled experimental environment.

The practical implication of these findings is that Redis Cache-Aside and ORM query optimization should be considered for DSS execution paths involving repeated relational aggregation. In systems such as Koladi, replacing database-backed snapshots with in-memory caching can reduce dependency on PostgreSQL during repeated requests, while improvements in eager loading and batch retrieval can mitigate query growth as the numbers of tasks and workspace members increase. Therefore, DSS performance optimization should not focus solely on either the caching layer or the ORM layer but should instead consider the interaction between both components.

This study has several limitations. The experiments were conducted sequentially under a controlled benchmarking environment rather than through load testing, stress testing, concurrency testing, throughput testing, or P95 latency evaluation. Furthermore, the evaluation was limited to two DSS endpoints on the performance statistics page rather than the entire Koladi application. Accordingly, the findings should be interpreted as evidence of improved DSS request execution efficiency under controlled experimental conditions.

Future research may extend this work beyond controlled benchmarking by evaluating system performance under parallel workloads, including load testing, stress testing, concurrency testing, throughput testing, and P95 latency analysis. Such evaluations are necessary to determine whether the performance improvements observed under sequential request execution can be maintained when the system processes multiple concurrent requests. This would provide a more comprehensive assessment of the effectiveness of Redis Cache-Aside and ORM query optimization under large-scale operational workloads.

## **Conclusion**

Simultaneously implementing Redis Cache-Aside and ORM query optimization improved Koladi's DSS execution path across all four measured metrics compared to baseline: more stable response times, substantially lower query counts, more controlled memory utilization, and more effective caching once read alongside query count. The largest improvement was in query count, which had grown with data volume under baseline due to the database-backed snapshot, modal-data's partial cache hits, the checklists N+1 pattern, and the workload-distribution query loop; after optimization it fell and stayed stable across all volumes, endpoints, and cache conditions, with the reduction confirmed statistically significant (*p* \< 0.001). Response time was likewise stabilized, especially under cache miss, and memory utilization became more consistent through selective loading, eager loading, and batch retrieval. Cache hit ratio, meanwhile, should always be read together with query count, since baseline cache hits did not necessarily represent a full database bypass. The research question is thus answered: Redis Cache-Aside combined with ORM query optimization delivers measurable performance improvements for these DSS endpoints, though the conclusions are bounded to sequential, controlled benchmarking of two endpoints and should not be generalized to load testing, concurrency testing, or Koladi's overall application performance.

**Funding**  
This research received no specific grant from any funding agency in the public, commercial, or not-for-profit sectors.

**Data Availability Statement**  
The benchmark measurement data supporting the findings of this study are available within the article and its supplementary materials. The source code implementing the Redis Cache-Aside and ORM optimizations is available from the corresponding author upon reasonable request.

**Declaration of Generative AI Use**  
In preparing this manuscript, the author used Claude Sonnet 5 (Anthropic) solely as a language-assistance tool to refine grammar, sentence clarity, and readability of the written text. No generative AI tool was involved in designing the experiment, collecting or analyzing the benchmark data, or forming the study's scientific conclusions. The author reviewed and verified all AI-assisted edits and takes full responsibility for the accuracy, originality, and content of this manuscript.

**Conflict or Interest statement**  
No potential conflict of interest was reported by the author.

**References**

\[1\]	F. J. Froese, T. Blay, C. B. Gibson, M. A. Shaffer, and J. Benitez, “Global virtual work: a review, integrative framework, and future research opportunities,” Aug. 01, 2025, *Palgrave Macmillan*. doi: 10.1057/s41267-025-00775-1.  
\[2\]	H. Ryu *et al.*, “A web-based decision support system (DSS) for hydrogen refueling station location and supply chain optimization,” *Int. J. Hydrogen Energy*, vol. 48, no. 93, pp. 36223–36239, Dec. 2023, doi: 10.1016/j.ijhydene.2023.06.064.  
\[3\]	C. Cai and C. Chen, “Optimization of Human Resource File Information Decision Support System Based on Cloud Computing,” *Complexity*, vol. 2021, 2021, doi: 10.1155/2021/8919625.  
\[4\]	H. Koornneef, W. J. C. Verhagen, and R. Curran, “A decision support framework and prototype for aircraft dispatch assessment,” *Decis. Support Syst.*, vol. 135, Aug. 2020, doi: 10.1016/j.dss.2020.113338.  
\[5\]	F. P. E. Putra, A. Zulfikri, A. Rohman, and R. Alim, “Analysis Comparative of Performance Optimization Techniques for PHP Framework Testing: Laravel, CodeIgniter, Symfony,” *Brilliance: Research of Artificial Intelligence*, vol. 5, no. 1, pp. 242–248, Jun. 2025, doi: 10.47709/brilliance.v5i1.5989.  
\[6\]	M. V. Privalov and M. V. Stupina, “Improving web-oriented information systems efficiency using Redis caching mechanisms,” *Indonesian Journal of Electrical Engineering and Computer Science*, vol. 33, no. 3, pp. 1667–1675, Mar. 2024, doi: 10.11591/ijeecs.v33.i3.pp1667-1675.  
\[7\]	B. Alshemaimri, R. Elmasri, T. Alsahfi, and M. Almotairi, “A survey of problematic database code fragments in software systems,” Oct. 01, 2021, *John Wiley and Sons Inc*. doi: 10.1002/eng2.12441.  
\[8\]	J. C. Yusmita, R. Arya, J. M. Wijaya, K. M. Suryaningrum, and R. R. Siswanto, “Optimizing Database Access Strategy: A Performance Analysis Comparison of Raw SQL and Prisma ORM,” in *Procedia Computer Science*, Elsevier B.V., 2025, pp. 1201–1210. doi: 10.1016/j.procs.2025.09.061.  
\[9\]	N. Makwana and Kundan, “A Comprehensive Study on Performance Optimization Techniques Database Queries and API Response Time in Laravel-Based Web Applications,” 2025\.  
\[10\]	S. An and Y. Cao, “Making Cache Monotonic and Consistent,” *Proceedings of the VLDB Endowment*, vol. 16, no. 4, pp. 891–904, 2022, doi: 10.14778/3574245.3574271.  
\[11\]	Z. Yang, K. Ma, X. Zhang, L. Cui, and B. Yang, “RSCVC: Row-based semantic cache with incremental versioning consistency,” *Concurr. Comput.*, vol. 32, no. 17, Sep. 2020, doi: 10.1002/cpe.5672.  
\[12\]	A. Turcotte, M. W. Aldrich, and F. Tip, “Reformulator: Automated Refactoring of the N+1 Problem in Database-Backed Applications,” in *ACM International Conference Proceeding Series*, Association for Computing Machinery, Sep. 2022\. doi: 10.1145/3551349.3556911.  
\[13\]	M. D. I. Sakti, D. Dirgantara, A. K. Ramadhan, and M. A. Ibrahim, “Optimizing Asynchronous Performance in Node.js with Express and PostgreSQL,” in *Procedia Computer Science*, Elsevier B.V., 2025, pp. 172–181. doi: 10.1016/j.procs.2025.08.270.  
\[14\]	T. Taipalus, “Database management system performance comparisons: A systematic literature review,” *Journal of Systems and Software*, vol. 208, Feb. 2024, doi: 10.1016/j.jss.2023.111872.  
\[15\]	F. Majerik and M. Borkovcova, “Design of Data Access Architecture Using ORM Framework.”  
\[16\]	Sasmoko, Y. Indrianti, S. R. Manalu, and J. Danaristo, “Analyzing Database Optimization Strategies in Laravel for an Enhanced Learning Management,” in *Procedia Computer Science*, Elsevier B.V., 2024, pp. 799–804. doi: 10.1016/j.procs.2024.10.306.  
\[17\]	S. Henning and W. Hasselbring, “A configurable method for benchmarking scalability of cloud-native applications,” *Empir. Softw. Eng.*, vol. 27, no. 6, Nov. 2022, doi: 10.1007/s10664-022-10162-1.  
\[18\]	L. Traini, V. Cortellessa, D. Di Pompeo, and M. Tucci, “Towards effective assessment of steady state performance in Java software: are we there yet?,” *Empir. Softw. Eng.*, vol. 28, no. 1, Jan. 2023, doi: 10.1007/s10664-022-10247-x.  
\[19\]	S. Uddin and H. Lu, “Confirming the statistically significant superiority of tree-based machine learning algorithms over their counterparts for tabular data,” *PLoS One*, vol. 19, no. 4 April, Apr. 2024, doi: 10.1371/journal.pone.0301541.  
\[20\]	C. Wohlin, P. Runeson, M. Höst, M. C. Ohlsson, B. Regnell, and A. Wesslén, “Experimentation in Software Engineering,” Berlin, 2000\.  
\[21\]	H. Alnuhait, W. Alzyadat, A. Althunibat, H. Kahtan, B. Zaqaibeh, and H. A. Al-Khawaja, “Web application performance assessment: A study of responsiveness, throughput, and scalability,” *International Journal of Advanced and Applied Sciences*, vol. 11, no. 9, pp. 214–226, Sep. 2024, doi: 10.21833/ijaas.2024.09.023.  
\[22\]	Z. Mao, R. Iyer, S. Shenker, and I. Stoica, “Revisiting Cache Freshness for Emerging Real-Time Applications,” in *HOTNETS 2024 \- Proceedings of the 2024 3rd ACM Workshop on Hot Topics in Networks*, Association for Computing Machinery, Inc, Nov. 2024, pp. 335–342. doi: 10.1145/3696348.3696858.  
\[23\]	N. Japke, M. Grambow, C. Laaber, and D. Bermbach, “µOpTime: Statically Reducing the Execution Time of Microbenchmark Suites Using Stability Metrics,” *ACM Transactions on Software Engineering and Methodology*, vol. 34, no. 8, Oct. 2025, doi: 10.1145/3715322.  
\[24\]	C. Gómez-Pantoja, D. Baeza-Rocha, and A. Inostrosa-Psijas, “A Behavior-Aware Caching Architecture for Web Applications Using Static, Dynamic, and Burst Segmentation,” *Future Internet*, vol. 17, no. 9, Sep. 2025, doi: 10.3390/fi17090429.  
 

# **APPENDIX** 

The supplementary materials supporting this study, including the complete raw, per-repetition benchmark measurement results for both the Baseline and Optimized configurations across all data volumes, endpoints, and cache conditions, have been submitted as a separate Supplementary Material file.  

