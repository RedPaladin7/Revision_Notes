# Cloud Interview — Weekly Revision Guide

There was no long cloud notes file in this repo (unlike `os_notes.db` / `Production_GenAI_Systems_Guide.md`). This is the same **weekly recitation pass**: every critical interview fact, examples compressed not dropped. AWS names are the default dialect; the *ideas* are portable to GCP/Azure.

**How to use (one full pass per week, ~40 min/day):**

| Day | Cover | Recite until you can say it without looking |
|-----|--------|---------------------------------------------|
| **Mon** | Cloud primitives + shared responsibility | IaaS/PaaS/SaaS, region/AZ, blast radius, shared-resp table |
| **Tue** | Compute | EC2 vs Lambda vs containers, ASG, cold start, spot |
| **Wed** | Storage | Object/block/file, S3 11 nines, classes, EBS vs EFS |
| **Thu** | Networking | VPC diagram, SG vs NACL, ALB vs NLB, Route53, CDN |
| **Fri** | IAM + security | Identity vs resource policy, roles, KMS envelope, secrets |
| **Sat** | Data + messaging | RDS vs Dynamo, SQS vis-timeout + DLQ, SNS vs EventBridge |
| **Sun** | HA / IaC / ops + cheat sheet | RPO/RTO, multi-AZ vs multi-region, Terraform, cost |

Each day: **Recite** the boxed facts → **Scan** the section → **Answer** the self-quiz at the end of that day. Sunday: run the full cheat sheet out loud.

---

# DAY 1 — Cloud primitives

## What “the cloud” actually is

Someone else’s data centers, APIs, and control plane. You rent **API-shaped** compute, storage, and network instead of buying racks. The win is not “no servers” — it is **elasticity, global reach, and shifting undifferentiated heavy lifting**.

| Model | You manage | Provider manages | Example |
|---|---|---|---|
| **On-prem** | Everything | Nothing | Your DC |
| **IaaS** | OS, runtime, app, data | Hardware, hypervisor, facility | EC2, GCE, Azure VM |
| **PaaS** | App + data | OS, runtime, scaling plumbing | Elastic Beanstalk, Cloud Run, App Service |
| **FaaS** | Function + event | Servers, idle capacity | Lambda, Cloud Functions |
| **SaaS** | Config / users | The product | Gmail, Salesforce |

Interview move: pick the **highest** abstraction that still gives you the control you need. Don’t run Kubernetes to host a cron that could be a function.

## Region, AZ, Edge

- **Region** = geographic cluster of DCs. Data residency, latency to users, product availability (not every service is in every region).
- **Availability Zone (AZ)** = one or more DCs in a region with **independent power / network / flood plain**. A well-built app survives **one AZ dying**.
- **Edge / PoP** = CloudFront, Anycast DNS, Global Accelerator. Cache and terminate TLS close to the user. Not where you put the database of record.

**Blast radius:** one AZ failure should not take you down. One region failure (earthquake, config nuke) *will* unless you designed **multi-region**. Multi-region is a product decision (RPO/RTO, Day 7), not a checkbox.

```
Users → Edge (CDN / DNS)
      → Region (e.g. ap-south-1)
           AZ-a   AZ-b   AZ-c
           (independent failure domains)
```

**Latency intuition:** same AZ ~0.1 ms, cross-AZ ~1–2 ms, cross-region ~20–200 ms + egress **money**. Chatty sync RPC across regions is how you invent an outage.

## Shared responsibility

| | You | Provider |
|---|---|---|
| **IaaS (EC2)** | Guest OS, patches, IAM *inside* the box, app, data, SG rules you wrote | Physical, hypervisor, disk destruction |
| **PaaS / RDS** | Schema, queries, users, *which* AZ/multi-AZ you chose | OS, engine patching (mostly), backups *if you enabled them* |
| **SaaS** | Who you invite, data you put in | Almost everything else |
| **Always you** | **Identity, encryption keys you manage, what you log, what you put in the bucket, network policy you configured** | |
| **Always them** | Buildings, hardware, hypervisor, global network | |

**“Secure *of* the cloud vs *in* the cloud.”** They don’t patch your `npm` or your open `0.0.0.0/0` SSH.

## Well-Architected six (say the names)

Operational excellence · Security · Reliability · Performance efficiency · Cost optimization · **Sustainability**.

When they ask “how would you design X,” map the answer onto these instead of dumping services.

## Cloud vs on-prem tradeoffs (honest)

**Cloud wins:** capex → opex, minutes to a new region, managed data stores, commodity IAM/KMS, you buy back engineer time.

**Cloud costs you:** noisy-neighbor / quota surprises, **egress bills**, weaker “I control the metal,” compliance paperwork (you still own it), easy to **leave idle spend running**.

**Hybrid:** on-prem + cloud (Direct Connect / Interconnect). **Multi-cloud:** two providers for leverage or anti-lock-in. Real multi-cloud *active-active* is rare and expensive; most “multi-cloud” is **different tool for different job** (GCP BigQuery + AWS prod).

### Day 1 self-quiz
1. IaaS vs PaaS vs FaaS vs SaaS — one example each. When do you refuse Kubernetes?
2. Region vs AZ vs Edge. What survives an AZ death vs a region death?
3. Same-AZ / cross-AZ / cross-region latency + why egress matters.
4. Shared-responsibility: what is *always* yours? EC2 vs RDS split.
5. Six Well-Architected pillars. Hybrid vs multi-cloud in one sentence each.

---

# DAY 2 — Compute

## EC2 (a VM you can API)

You pick: vCPU/RAM, network, **instance family** (general `m`, compute `c`, memory `x`/`r`, GPU `p`/`g`, burstable `t`).

- **User data:** script on first boot (or every boot). Bake an **AMI** instead if startup must be fast/repeatable.
- **IMDS:** instance metadata (IAM role creds live here). IMDSv2 (session token) — IMDSv1 was SSRF candy.
- **Placement groups:** cluster (low latency, same rack — AZ risk), spread (distinct hardware), partition (Hadoop/Kafka).
- **Hibernate / stop / terminate:** stop keeps EBS, drops instance-store. Terminate can delete the root volume — know the flag.

## Scaling the VM layer

**Auto Scaling Group (ASG):** min / desired / max across AZs. Scale on CPU, ALB request count, custom metric, or schedule.

Launch template → AMI + instance type + SG + IAM role + user data.

Health check fail → replace. That is how you get cattle, not pets.

## Purchase options

| | When | Risk |
|---|---|---|
| **On-demand** | Spiky, unknown | Full price |
| **Reserved / Savings Plans** | Steady 1–3 yr | You’re paying even if unused |
| **Spot** | Interruptible (stateless workers, batch) | **2-minute** (AWS) interruption notice |
| **Dedicated / hosts** | License / isolation | Expensive |

Never put a single-node stateful primary on Spot.

## Lambda / functions

Upload code. Provider runs it on an **event**: HTTP (API Gateway / Function URL), queue, timer, S3 put, stream.

**Hard limits to recite (AWS-shaped, idea is universal):**
- Timeout cap (Lambda **15 min**) — not for hour-long jobs
- Memory setting also buys CPU
- **Cold start:** new execution environment; worse with VPC ENI attach + big deps + custom runtimes
- **Reserved / provisioned concurrency** to cap noisy neighbors or pre-warm
- **One request per execution** (unless you opted into a multi-concurrency runtime). Scale = more instances, not threads you invented
- Disk: `/tmp` is ephemeral and small unless you configure more
- **Idempotency required** — at-least-once events are normal

**When Lambda wins:** spiky, event-driven, low/idle traffic, glue between services.

**When it loses:** long jobs, huge fan-out of chatty VPC calls, tight p99, custom kernels, huge local disk.

## Containers

**Image** = app + libs + runtime. **Container** = running instance. **Orchestrator** = place them, restart them, network them.

| | You want | You don’t want |
|---|---|---|
| **ECS on Fargate** | AWS-native containers, no nodes | Fine-grained node control |
| **ECS on EC2** | GPU / special instances, cheaper at scale | Node patching |
| **EKS / GKE / AKS** | Portability, huge ecosystem, CRDs | Control-plane + YAML tax |
| **Cloud Run / App Runner** | Request-scaled containers, almost FaaS | Complex service mesh |

**Fargate / Cloud Run = “container as a function of capacity.”** You still build images; you don’t SSH boxes.

## The decision table (interview gold)

| Need | Pick |
|---|---|
| Lift-and-shift VM, special kernel, Windows | EC2 |
| Stateless HTTP, spiky, team doesn’t want cluster | Lambda or Cloud Run |
| Long-running workers, sidecars, many services | ECS/EKS |
| GPU training | EC2/ECS + GPU family, not Lambda |
| Cron every 5 min, 30 seconds of work | Scheduled function |
| Always-on WebSocket with sticky state | Containers or EC2 + sticky / externalize state |

**Externalize state.** Compute is cattle. Session in Redis, files in S3, identity in the token.

### Day 2 self-quiz
1. Instance families in one word each. Why IMDSv2?
2. ASG: min/desired/max, two scale signals, what a failed health check does.
3. On-demand vs Savings Plan vs Spot. Why not a Spot primary?
4. Lambda: timeout, cold start causes, why idempotency.
5. ECS+Fargate vs EKS vs Cloud Run — one line each.
6. Recite the decision table. “Externalize state” in one sentence.

---

# DAY 3 — Storage

## Three shapes

| | API | Use | Example |
|---|---|---|---|
| **Object** | HTTP PUT/GET, key → blob | Unstructured, huge, cheap | **S3**, GCS, Azure Blob |
| **Block** | Disk attached to one VM | Boot vol, DB data dir | **EBS**, Persistent Disk |
| **File** | NFS/SMB, many VMs | Shared home, lift-and-shift | **EFS**, Filestore, Azure Files |
| **Instance store** | Local NVMe | Cache, scratch | Dies with the instance |

Don’t put a database data directory on object storage. Don’t share an EBS volume across AZs (you can’t, attached to one AZ).

## S3 — the interview bucket

**Durability 11 nines (99.999999999%).** That is “won’t lose the object.” It is **not** availability. A region or API outage can still make you unable to *read* a perfectly durable object. Multi-AZ inside a region is how S3 is built; **cross-region** is your job (CRR) if the region is gone.

**Consistency:** S3 is now **strongly consistent** for overwrite PUT and DELETE (used to be eventual — old blog posts lie).

**Classes (recite the ladder):**

| Class | When |
|---|---|
| Standard | Hot |
| Standard-IA / One Zone-IA | Infrequent; One Zone = cheaper, **AZ risk** |
| Glacier Instant / Flexible / Deep Archive | Archive; Deep = hours to restore, pennies |
| Intelligent-Tiering | Unknown access pattern; small monitor fee |

**Lifecycle rules** move or expire. That is how you stop a logs bucket from becoming a second mortgage.

**Versioning:** overwrite becomes a new version; delete is a marker. Pair with **MFA delete** / Object Lock (WORM, compliance).

**Encryption:**
- SSE-S3 (AWS-managed)
- SSE-KMS (you see key use in CloudTrail — **default for anything with a compliance story**)
- SSE-C (you bring the key; lose it = data gone)
- Client-side

**Access:**
- Bucket policy (resource) + IAM (identity) — **both** must allow, plus no explicit deny
- Block Public Access master switch — leave it ON unless the bucket is a website
- **Presigned URL** — temporary GET/PUT without making the bucket public
- VPC endpoint so traffic never hits the public internet

**Multipart upload** for big objects. **S3 Select / Inventory** exist; don’t pretend you query S3 like SQL for OLTP.

## EBS vs EFS vs instance store

| | EBS | EFS | Instance store |
|---|---|---|---|
| Scope | **One AZ**, attach to one instance (io2 Block Express / multi-attach is special) | Regional, **many** NFS clients | That instance only |
| Persist after stop | Yes (until you delete) | Yes | **No** |
| Latency | Low, disk-like | Higher, shared file | Lowest |
| Good for | DB volumes, boot | CMS shared uploads, home dirs | Cache, scratch |
| Snapshot | To S3, can copy regions | Back up differently | N/A |

**gp3** is the default general SSD (you set IOPS/throughput independently). **io2** for high-IOPS databases. Throughput-optimized `st1` for big sequential.

**Resize:** you can grow EBS live; shrinking is a migration.

## The “where do I put this?” table

| Data | Where |
|---|---|
| User uploads, data lake, backups, static site | S3 |
| Postgres data directory | EBS (Multi-AZ RDS hides this) |
| Six app nodes need the same files | EFS, or better: **don’t** — put objects in S3 |
| Session cache | ElastiCache, not disk |
| Secrets | Secrets Manager, not a file on the AMI |

### Day 3 self-quiz
1. Object vs block vs file vs instance store. One “don’t” each.
2. 11 nines ≠ availability. S3 consistency today.
3. Recite the storage-class ladder + lifecycle. Versioning + Object Lock.
4. SSE-S3 vs SSE-KMS vs SSE-C. Presigned URL vs public bucket.
5. EBS vs EFS vs instance store table.
6. Where do uploads / PG data / shared files / secrets live?

---

# DAY 4 — Networking

## Draw a VPC every time

```
VPC 10.0.0.0/16
  Public subnet AZ-a 10.0.1.0/24   ─ IGW ─ Internet
  Public subnet AZ-b 10.0.2.0/24   ─ IGW
  Private subnet AZ-a 10.0.11.0/24 ─ NAT (in public a)
  Private subnet AZ-b 10.0.12.0/24 ─ NAT (in public b)   ← one NAT per AZ if you care
  DB subnet AZ-a 10.0.21.0/24      no NAT, no IGW
  DB subnet AZ-b 10.0.22.0/24
```

- **IGW:** bidirectional internet for resources with a **public IP**.
- **NAT gateway:** private subnet **egress only** (patches, APIs). Stateful, billed per GB. Put one **per AZ** or that AZ’s egress dies with the other AZ’s NAT.
- **Public IP ≠ public subnet.** Public subnet = route table has `0.0.0.0/0 → IGW`.
- **VPC endpoints (Gateway for S3/Dynamo, Interface/PrivateLink for the rest):** private IPs to AWS APIs, no internet, smaller blast + no NAT $ for S3.

## CIDR sanity

`/16` = 65,536 addresses. `/24` = 256. AWS reserves 5 per subnet. You cannot resize a VPC CIDR casually — **plan or peer**.

Overlapping CIDRs = you cannot VPC-peer or connect via Transit Gateway cleanly. Classic M&A wound.

## SG vs NACL

| | Security Group | NACL |
|---|---|---|
| Attached to | **ENI** (instance / LB / RDS) | **Subnet** |
| State | **Stateful** — return traffic allowed automatically | **Stateless** — inbound *and* outbound rules |
| Default | Deny inbound, allow outbound | Allow all (custom VPC: you often tighten) |
| Rules | Allow only | Allow **and deny** (eval by rule number) |
| Use | Primary app firewall | Coarse subnet guardrail, explicit deny |

Interview: “I use **SGs as the real control**. NACLs for an extra explicit deny or to fence a whole subnet.”

## Load balancers

| | ALB (L7) | NLB (L4) | Gateway LB |
|---|---|---|---|
| Sees | HTTP/HTTPS, path, host, header | TCP/UDP/TLS, IP | Packets to appliances |
| Target | Instance, IP, **Lambda** | Instance, IP, ALB | GWLBE |
| Sticky | Cookie | 5-tuple | — |
| Use | HTTP microservices, WAF, OIDC | Extreme PPS, static IP, non-HTTP | Firewalls |

**CLB (classic)** is legacy. Don’t propose it.

ALB + target group health checks + ASG = the standard public HTTP shape.

**Cross-zone load balancing:** each LB node can hit targets in any AZ. On by default for ALB; know the NLB cost/behavior.

## DNS — Route 53 policies

| Policy | Meaning |
|---|---|
| Simple | One answer |
| Weighted | % split — canaries |
| Latency | Closest region |
| Failover | Primary health-check → secondary (active-passive DR) |
| Geolocation / geoproximity | Compliance / bias |
| Multi-value | Up to 8 healthy random — poor man’s RR |

Alias to ALB/CloudFront/S3 — no extra DNS charge, no CNAME-at-apex pain.

## CDN (CloudFront)

Cache at edge. Origin = S3 or ALB. **Origin Access Control** so the bucket is not public. Invalidate on deploy (or versioned filenames — better).

TLS at edge, optionally **AWS WAF**. Signed cookies/URLs for private content.

## Connectivity extras (name them)

- **Peering:** two VPCs, non-transitive
- **Transit Gateway:** hub-and-spoke, transitive
- **Direct Connect / Interconnect:** private pipe to the DC; not “the internet but lucky”
- **VPN:** cheaper, higher latency, encrypted over internet
- **PrivateLink:** expose a service into another VPC as a local endpoint — no shared CIDR, no IGW

### Day 4 self-quiz
1. Draw the 6-subnet VPC. IGW vs NAT. Why NAT per AZ?
2. What makes a subnet public? Gateway vs Interface endpoint.
3. SG vs NACL table. Stateful in one sentence.
4. ALB vs NLB: OSI layer, one use each. Why not CLB.
5. Six Route 53 policies in one word each.
6. CloudFront + OAC. Peering vs TGW vs PrivateLink.

---

# DAY 5 — IAM + Security

## Identity vs resource

**Identity policy** (on a user/role): “what this principal may do.”

**Resource policy** (on a bucket, queue, KMS key, account): “who may touch *me*.”

Final decision: explicit **Deny** wins. Then both identity **and** resource must **Allow** (plus SCPs in Organizations, plus session policies, plus permissions boundaries). If you blank: **Deny > SCP > identity Allow ∩ resource Allow**.

## Users, groups, roles, STS

- **User** = long-lived person or (please don’t) service password. Console + access keys.
- **Group** = permission bundle for humans.
- **Role** = **assumed**, short-lived creds via **STS**. This is how EC2/Lambda/EKS get rights — **instance/task role**, no keys on disk.
- **AssumeRole** across accounts = the multi-account pattern.

**Least privilege:** start deny, add specific `Action` + `Resource` + `Condition` (source VPC, MFA, prefix). `*` on production is a confession.

**No long-lived keys on apps.** If you see `AWS_ACCESS_KEY_ID` in a Lambda env var, that is the bug.

## Multi-account

One account per **env / blast radius** (prod ≠ sandbox). **Organizations + SCPs** = “nobody in this OU can disable CloudTrail / leave the region.” Shared services account, log-archive account (immutable).

## KMS — envelope encryption (recite)

You do **not** encrypt 5 GB with the KMS RSA/AES key over the API (throttled, expensive).

```
1. Generate a data key (plaintext + ciphertext) from KMS
2. Encrypt the file with the plaintext data key (local AES)
3. Throw away plaintext; store ciphertext blob + encrypted data key
4. Decrypt: KMS decrypts the data key, you decrypt the file
```

**CMK** rotation, grants, CloudTrail of every `Decrypt`. Key policy is a resource policy — this is how you share a key across accounts.

Encryption **at rest** (disk/S3/RDS) and **in transit** (TLS). Both. “We use KMS” without TLS to the client is half a story.

## Secrets

| | Secrets Manager | SSM Parameter Store |
|---|---|---|
| Rotation | Built-in (RDS passwords) | You build it |
| Cost | Per secret | Standard params cheap |
| Use | DB creds, third-party API keys | Config, feature flags, non-rotating |

Inject at runtime (role can `GetSecretValue`). Never bake into AMI or git.

## Network + app guards

- **WAF** on ALB/CloudFront: SQLi, XSS, rate, bot (L7)
- **Shield Standard** free volumetric; **Shield Advanced** for serious DDoS + cost protection
- **GuardDuty:** threat findings from logs/VPC flow
- **Security Hub / CSPM:** score the account
- **Macie:** PII in S3

## The ten commandments (recite)

1. No root user except break-glass, MFA on it
2. MFA on humans
3. Roles for machines, no static keys
4. SCPs on the org
5. S3 Block Public Access
6. Encrypt with KMS, log key use
7. Private subnets for compute; endpoints for AWS APIs
8. SG default-deny inbound
9. CloudTrail + access logs in a **separate** account, retention
10. Secrets manager, not `.env` in the image

### Day 5 self-quiz
1. Identity vs resource policy. What wins? Where do SCPs sit?
2. Why roles not users for EC2/Lambda? What is STS?
3. Recite envelope encryption. Why not encrypt the file *at* KMS?
4. Secrets Manager vs Parameter Store.
5. WAF vs Shield vs GuardDuty in one line each.
6. Recite the ten commandments.

---

# DAY 6 — Data + Messaging

## Managed databases — pick on access pattern

| | RDS / Aurora (SQL) | DynamoDB (key-value) | ElastiCache | Redshift / BigQuery / Snowflake |
|---|---|---|---|---|
| Shape | Relational, joins, tx | Item by PK / GSI | Hot cache | Columnar analytics |
| Scale | Vertical + read replicas; Aurora storage scales | Horizontal, partitions | Memory | Warehouse |
| Consistency | Strong (primary) | Strong on PK by default; GSI eventual | Evict | Batch / eventual-ish |
| When | Multi-row invariants | Known keys, huge QPS, schemaless items | Sessions, computed | OLAP, not request path |

**Aurora:** storage replicated across 6 copies / 3 AZs. Failover seconds. Aurora Serverless for spiky.

**Dynamo:** you design **partition key** so traffic is even. Hot partition = you invented a queue. Single-table vs multi-table is a religion — know *why* you pick.

**RDS Multi-AZ:** sync standby in another AZ — **failover**, not extra read QPS. **Read replica:** async, extra reads, can promote (replica lag is real).

## Queues — SQS

Decouple producer from consumer. Pull model.

| | Standard | FIFO |
|---|---|---|
| Order | Best-effort | Per message-group |
| Delivery | **At-least-once**, rare dupes | Exactly-once *processing* if you use the dedup id well |
| Throughput | Nearly unlimited | 300/s (or 3000 with batching) |

**Visibility timeout:** after a receive, the message is invisible. If you crash before delete, it **reappears**. Set vis-timeout > worst processing time (or heartbeat-extend). Too short = two workers do the same job.

**DLQ:** after N receives, park it. Without a DLQ, poison messages **infinite-loop** your workers.

**Idempotency:** at-least-once means your handler must tolerate a second delivery (dedup key in DB, conditional write).

## Pub/sub and buses

| | SNS | EventBridge | Kinesis / Kafka |
|---|---|---|---|
| Model | Push topics → many subscribers | Event bus + **rules** | Ordered stream, replay |
| Fan-out | Email, SQS, Lambda, HTTP | SaaS events, schedule, archive/replay | Many consumers, each with a cursor |
| Use | “Tell everyone this happened” | App-wide event routing, Schema Registry | Clickstream, logs, exactly-once-ish pipelines |

**SNS → SQS → Lambda** is the classic durable fan-out (SNS alone to HTTP can drop).

**SQS vs Kinesis:** SQS = competing consumers, delete when done. Kinesis = **retain** (hours–days), many independent readers, partition-key order.

## Outbox / idempotency / exactly-once

The cloud does **at-least-once**. You fake exactly-once:

1. Write business row + **outbox** row in one DB transaction
2. Publisher ships the outbox to SNS/SQS
3. Consumer **idempotency key** (order_id) unique constraint

“Exactly-once SQS” in an interview without this story is a red flag.

## API front door

API Gateway / ALB → auth (Cognito/JWT) → throttle → Lambda/ECS. **WAF** in front. Stage variables + usage plans for partners.

### Day 6 self-quiz
1. RDS vs Dynamo vs cache vs warehouse — one sentence each.
2. Multi-AZ vs read replica. Aurora 6/3.
3. SQS standard vs FIFO. Visibility timeout. Why DLQ.
4. SNS vs EventBridge vs Kinesis. Why SNS→SQS not SNS→HTTP for work.
5. Recite the transactional outbox. Why “exactly-once” is a lie.

---

# DAY 7 — Reliability, IaC, Ops

## HA vs DR

**HA:** survive *expected* failures (one AZ, one node) with little/no user pain. Multi-AZ, ASG, ALB, RDS Multi-AZ.

**DR:** survive a **region** or a “we deleted prod.” Needs a story.

| Strategy | RPO (data loss) | RTO (time to up) | Cost |
|---|---|---|---|
| Backup / restore | Hours (backup interval) | Hours | $ |
| Pilot light | Minutes–hours | Tens of minutes | $$ |
| Warm standby | Seconds–minutes | Minutes | $$$ |
| Active-active multi-region | ~0 | ~0 | $$$$ |

**RPO** = how much data you can lose. **RTO** = how long you can be down. Get these from the business *before* you buy a second region.

Multi-AZ is **not** multi-region. Recite that.

## Failure-mode habits

- Timeouts on every RPC
- **Retries with jittered exponential backoff**
- **Idempotent** writes
- Circuit breaker so a dead dependency doesn’t thread-starve you
- Bulkhead (pools per dependency)
- Load shed (429) before you melt
- Health checks that test **deps**, not just `200 OK` from a process that is wedged

## IaC

If it isn’t in git, it doesn’t exist.

| | Terraform / OpenTofu | CloudFormation / CDK | Pulumi |
|---|---|---|---|
| Scope | Multi-cloud | AWS-native | Real languages |
| State | You (S3+Dynamo lock) or cloud | AWS manages | You / service |

**Plan then apply.** State lock. Modules. Don’t click the console except to **read**. Drift happens; detect it.

**Immutable deploys:** new AMI / new task definition / new Lambda version + alias. Roll forward. Blue/green or canary via weighted Route 53 or CodeDeploy / Flag.

## Observability (cloud edition)

- **Metrics:** CloudWatch / Prometheus — RED (Rate, Errors, Duration) + USE (Utilization, Saturation, Errors)
- **Logs:** structured JSON, one request id everywhere, central store (CloudWatch Logs, Loki). Retention + a **deny-delete** archive account
- **Traces:** X-Ray / OTel — follow one request across ALB → ECS → RDS
- **Alarms:** on SLOs, not “CPU > 80 forever.” Page on user pain, ticket on capacity

## Cost (they will ask)

- Right-size; turn off non-prod at night
- Savings Plans on the steady baseline; Spot on the batch
- S3 lifecycle, log retention, unused IPs/NAT/unattached EBS (the silent bill)
- **Egress** and NAT GB are the surprise
- Allocate tags (`team`, `env`, `service`) or you cannot attribute

FinOps one-liner: **cost is an architecture signal.** A $12k NAT bill means you needed a VPC endpoint yesterday.

## Twelve-factor, cloud-shaped

1. Codebase in git  
2. Config from env / Parameter Store, not baked  
3. Backing services attached by URL  
4. Stateless processes  
5. Bind port, LB in front  
6. Concurrency = more processes  
7. Disposable (fast boot, SIGTERM drain)  
8. Dev/prod parity  
9. Logs to stdout → agent  
10. Admin tasks as one-off jobs  

### Day 7 self-quiz
1. HA vs DR. Multi-AZ ≠ multi-region.
2. Recite the four DR strategies with RPO/RTO/cost.
3. Timeout, jittered retry, idempotency, circuit breaker, bulkhead, load shed.
4. Terraform state + lock. Immutable deploy in one sentence.
5. RED vs USE. What do you page on?
6. Three silent cloud bills. Recite 12-factor 2, 4, 6, 7, 9.

---

# SUNDAY CLOSE — Full cheat sheet (recite out loud)

**Abstractions:** IaaS VM, PaaS runtime, FaaS event, SaaS product. Pick the highest that still fits.

**Region / AZ / Edge.** Survive one AZ. Region death needs a DR product decision. Cross-AZ ~1–2 ms; cross-region is latency + egress $.

**Shared resp:** they own the building and hypervisor. You own identity, your config, your data, your `0.0.0.0/0`. Always.

**Compute:** EC2 cattle + ASG. Spot for interruptible. Lambda: 15 min, cold start, idempotent. Fargate if you want containers without nodes. EKS if you need the ecosystem. Externalize state.

**Storage:** object S3 (11 nines durability ≠ availability; strong consistency; lifecycle; KMS; Block Public Access; presigned). Block EBS one AZ. File EFS many clients. Instance store dies.

**Network:** public = route to IGW. NAT for private egress, one per AZ. Endpoints for S3. SG stateful on ENI; NACL stateless on subnet. ALB L7, NLB L4. Route53: weighted / latency / failover. CloudFront + OAC. TGW transitive; peering not; PrivateLink for exposing a service.

**IAM:** deny wins. Identity ∩ resource. Roles + STS, no static keys on apps. Envelope: data key locally, KMS unwraps. Secrets Manager rotates. Ten commandments.

**Data:** SQL for joins/invariants; Dynamo for known keys + huge QPS (watch hot partitions); cache for hot; warehouse for OLAP. Multi-AZ = failover; replica = reads + lag.

**Messaging:** SQS vis-timeout + DLQ + idempotent handler. Standard = at-least-once. SNS fan-out via SQS. EventBridge for rules. Kinesis/Kafka to retain and replay. Outbox for “exactly-once.”

**Ops:** RPO/RTO first. Backup < pilot < warm < active-active. IaC + lock. Immutable + canary. RED/USE, traces, SLO pages. Cost: tags, lifecycle, endpoints not NAT, Savings + Spot.

---

# Interview lines (say these verbatim)

1. **AZ vs region:** “I build for an AZ to disappear. A region disappearing is a DR conversation about RPO and RTO, not a second checkbox on the VPC wizard.”
2. **Shared responsibility:** “AWS secures the cloud. I secure what I put in it — IAM, data, the security groups I opened, the packages on the guest.”
3. **Cattle:** “ASG replaces unhealthy nodes. State lives in RDS, Redis, or S3 — never on the instance.”
4. **Lambda:** “Great for bursty event work. I design for cold starts and at-least-once. If it needs more than 15 minutes or a special kernel, it isn’t a function.”
5. **S3 11 nines:** “Durability is not availability. The object can be safe and still unreachable if I only have one region.”
6. **SG vs NACL:** “Security groups are stateful and sit on the ENI — that’s my real firewall. NACLs are stateless subnet guards for coarse deny.”
7. **ALB vs NLB:** “ALB if I need HTTP routing, WAF, or Lambda targets. NLB if I need millions of packets, static IPs, or raw TCP.”
8. **Roles:** “Machines assume roles. If I see access keys in a Lambda environment variable, that’s the incident.”
9. **Envelope:** “KMS gives me a data key. I encrypt locally. I store the wrapped key next to the blob. I never push a 5 GB file through the KMS API.”
10. **SQS:** “Visibility timeout must exceed processing or two workers grab the same message. A DLQ is how I stop a poison pill from spinning forever. Handlers are idempotent because delivery is at-least-once.”
11. **Outbox:** “The database transaction writes the business row and the event. A publisher drains the outbox. That’s how I don’t lose the message or emit it without the write.”
12. **Cost:** “The NAT gateway and egress are where surprise bills hide. A VPC endpoint for S3 is both a security win and a cost win.”

---

# Weekly 15-minute emergency pass (if you only have one sitting)

Recite in this order, eyes closed:

1. IaaS/PaaS/FaaS/SaaS → region/AZ/edge → shared-resp “always you”
2. EC2+ASG cattle → Spot vs reserved → Lambda limits + cold start → Fargate vs EKS
3. Object/block/file → S3 11 nines ≠ availability → classes + lifecycle → KMS + Block Public Access + presigned
4. Draw VPC → IGW vs NAT-per-AZ → SG vs NACL → ALB vs NLB → Route53 policies → OAC
5. Identity ∩ resource, Deny wins → roles/STS → envelope encryption → Secrets vs SSM → ten commandments
6. RDS vs Dynamo vs cache vs warehouse → Multi-AZ ≠ replica → SQS vis + DLQ → SNS→SQS → outbox
7. RPO/RTO + four DR shapes → jittered retry + breaker → Terraform lock → RED/USE → egress/NAT cost
