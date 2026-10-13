# What tiny-aws is missing

tiny-aws is an educational platform, not a production cloud. These are the known gaps — things that exist in AWS but are deliberately absent, partially implemented, or architecturally impossible in tiny-aws.

The gaps are not bugs. They are deliberate scope decisions. Each one points to a real engineering problem that AWS solved with significant investment. Understanding what is missing and why it is hard to add is as valuable as understanding what is present.

---

## The most important thing to know

**Do not expose any port to the internet.** All service-to-service communication is plain HTTP. Bearer tokens travel in cleartext. There is no TLS anywhere.

tiny-aws is safe to run on a single machine for learning. It is not safe for production workloads.

---

## Security

- **No TLS.** All HTTP. All tokens in cleartext. Local use only.
- **Shared kernel.** Instances share the host kernel. A kernel exploit breaks all isolation instantly. AWS uses Nitro hardware VMs to prevent this at the silicon level. tiny-aws cannot offer that guarantee.
- **No encryption at rest.** Object store writes raw bytes to disk. SQLite databases are unencrypted files.
- **IAM is two roles.** `admin` and `readonly`. No resource-level permissions, no policy language, no cross-account access. Real IAM is a distributed system evaluated on every API call.
- **No secret rotation.** API keys are static strings.
- **seccomp is a denylist, not an allowlist.** The profile blocks known-dangerous syscalls but permits everything else by default. Production runtimes use allowlists (~150 reviewed syscalls). New dangerous syscalls are automatically permitted in tiny-aws until manually added to the list.

---

## Networking

- **VPC is metadata, not networking.** CIDRs, subnets, and route tables are strings in SQLite. There is no real IP allocation, no DHCP, and no routing enforcement between instances. Two instances on the same machine can reach each other regardless of security group rules.
- **iptables rules are host-level only.** Security group enforcement applies to traffic entering and leaving the host, not between co-located instances.
- **No DNS.** No Route 53 equivalent. Service addresses are IPs and ports.
- **No IPv6.**
- **No VPC peering, Transit Gateway, Direct Connect, or VPN.**
- **Load balancer has no SSL termination.**

---

## Storage

- **Single-node durability.** Object store writes to one disk. No replication unless you configure `REPLICATION_FACTOR`, and even then there is no quorum or automatic conflict resolution.
- **No versioning.** Overwriting an object destroys the previous version.
- **No multipart upload.** Large uploads are a single PUT.
- **No server-side encryption.**
- **No bucket policies or ACLs.** One API key controls everything.

---

## Compute

- **No persistent volumes.** Instance filesystems are ephemeral overlayfs. All data is lost on termination. No EBS equivalent.
- **No live migration.** Instances cannot move between nodes without stopping.
- **No auto-scaling.** Instance count is always manual.
- **No snapshots.**
- **Windows agents run without sandbox.** Namespace isolation and cgroup limits require Linux.

---

## Scheduling

- **Round-robin only.** No bin-packing, no affinity rules, no capacity-aware placement.
- **Retry is once.** No exponential backoff.
- **No priority queues.** All jobs are equal.

---

## Messaging

- **SQS has no FIFO mode.** No ordering guarantees. No exactly-once delivery.
- **SQS visibility timeout is hardcoded at 30 seconds.**
- **SNS has no retry.** If a subscriber endpoint returns an error, the notification is dropped.
- **SNS supports HTTP subscribers only.** No email, SMS, SQS fan-out, or mobile push.

---

## Lambda

- **Always cold.** Every invocation downloads the zip and starts fresh. No warm container pool.
- **Python 3 and Node 20 only.** No other runtimes.
- **No sandbox.** Lambda functions run with full agent privileges.
- **No SQS → Lambda triggers.** Only S3 ObjectCreated is wired.

---

## Observability

- **No metrics.** No CloudWatch equivalent.
- **No distributed tracing.** Request IDs are not propagated across services.
- **No alerting.**

---

## Operations

- **No rolling deploys.** Updating a service means downtime.
- **No high availability.** If the registry goes down, no new work can be scheduled.
- **No rate limiting on any API.**
- **No multi-region.** There is one region: your machine.
