# Kubernetes ETCD - Comprehensive Guide

## Overview

ETCD is a distributed, reliable, and highly available key-value store that serves as the backing store for all cluster data in Kubernetes. It is simple, secure, and fast, providing the single source of truth for the entire Kubernetes cluster.

**Definition**: ETCD is a distributed key-value store written in Go that uses the Raft consensus algorithm to ensure data consistency and fault tolerance.

---

## Table of Contents
1. [ETCD Features](#etcd-features)
2. [Why ETCD in Kubernetes](#why-etcd-in-kubernetes)
3. [Architecture](#architecture)
4. [ETCD Fundamentals](#etcd-fundamentals)
5. [Installation & Setup](#installation--setup)
6. [ETCD Operations & API](#etcd-operations--api)
7. [Clustering](#clustering)
8. [Performance & Optimization](#performance--optimization)
9. [Backup & Restore](#backup--restore)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)
12. [Security](#security)
13. [Examples](#examples)

---

## ETCD Features

### 1. **Simple Interface**
- Read and write values using standard HTTP tools (e.g., curl)
- RESTful API based on gRPC
- Easy integration with applications
- Language-agnostic clients available

![Simple Interface](/images/interface.svg)

### 2. **Key-Value Storage** 
- Data stored in hierarchically organized directories, like a filesystem
- Flat key-value model with directory-like organization
- Support for binary keys and values
- Efficient prefix searches

![Key Value storage](/images/Key_value_strorage.svg)

### 3. **Watch for Changes**
- Watch specific keys or directories for changes
- React to changes in values in real-time
- Enables event-driven architecture
- Long-polling and gRPC streaming support

![Watch for Changes](/images/watch.svg)

### 4. **Distributed & Reliable**
- Replicates data across multiple machines
- Automatic failover when nodes fail
- Strong consistency guarantee (linearizable reads)
- Raft consensus algorithm for data safety

### 5. **Secure**
- TLS/SSL encryption for communication
- Authentication and authorization
- Certificate-based client verification
- Audit logging capabilities

### 6. **Fast**
- Optimized for read-heavy workloads
- Sub-millisecond response times
- In-memory data structures
- Efficient network utilization

### 7. **Version Control**
- Multi-version concurrency control (MVCC)
- Maintains history of all changes
- Point-in-time recovery possible
- Revision tracking for all modifications

---

## Why ETCD in Kubernetes

### Critical Role
ETCD is the **single source of truth** for a Kubernetes cluster. It stores:
- All cluster configuration data
- Pod and node status
- Service definitions and endpoints
- Secrets and ConfigMaps
- Persistent volume claims and volumes
- NetworkPolicies and RBAC rules
- Deployments, StatefulSets, DaemonSets
- All other Kubernetes objects

### Key Reasons for ETCD Selection

| Reason | Benefit |
|--------|---------|
| **Strong Consistency** | All replicas see the same data at the same time |
| **Distributed** | High availability through clustering |
| **Raft Consensus** | Reliable leader election and data synchronization |
| **High Performance** | Handles thousands of writes per second |
| **Fault Tolerance** | Survives node failures without data loss |
| **Compaction** | Can clean up old revisions to save space |
| **Watch Mechanism** | Real-time change notifications for controllers |

### Impact on Cluster Operations

```
┌─────────────────────────────────────┐
│     All Kubernetes Components       │
│                                     │
│  ├─ API Server (reads/writes)      │
│  ├─ Controllers (watch changes)    │
│  ├─ Scheduler (reads capabilities) │
│  ├─ Kubelet (reads config)         │
│  └─ All other components            │
└────────────┬────────────────────────┘
             │
             ▼
    ┌────────────────────┐
    │    ETCD Cluster    │
    │  ├─ Member 1       │
    │  ├─ Member 2       │
    │  └─ Member 3       │
    └────────────────────┘
             ▲
    Single Source of Truth
    All Cluster Data
```

---

## Architecture

### ETCD Server Components

```
┌──────────────────────────────────────┐
│         ETCD Node                    │
├──────────────────────────────────────┤
│  ┌──────────────────────────────┐   │
│  │   HTTP/gRPC API Server       │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │   Raft Protocol Engine       │   │
│  │  (Consensus & Replication)   │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │   Logical Store              │   │
│  │  (In-Memory B+Tree)          │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │   Physical Store             │   │
│  │  (Persistent Disk Storage)   │   │
│  └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

### Data Storage Structure

```
ETCD Cluster
├── /registry (Kubernetes objects stored here)
│   ├── /api.openshift.io/v1/
│   ├── /apps/v1/
│   ├── /batch/v1/
│   ├── /configmaps/
│   ├── /secrets/
│   ├── /persistentvolumes/
│   ├── /persistentvolumeclaims/
│   ├── /roles/
│   ├── /rolebindings/
│   └── ...all other resources
├── /kubernetes.io/
├── /events/
└── /masterleases/ (leader election)
```

### Raft Consensus Algorithm

ETCD uses Raft to ensure all members have the same data:

```
        Term 1:     Term 2:       Term 3:
          
State:  FOLLOWER -> CANDIDATE -> LEADER
          
Events: Timeout    Win Election  Accept Commands
        Election   (majority votes) From Clients
        Starts     Gained
```

**Key Raft Concepts:**
- **Term**: Logical clock for cluster membership
- **Quorum**: Majority of members (e.g., 3 of 5)
- **Leader**: Single node handling all write operations
- **Followers**: Replicate log entries from leader
- **Log Entry**: Represents a state change

---

## ETCD Fundamentals

### Key-Value Model

Unlike relational databases (tables with rows and columns), ETCD uses a simple key-value model:

**Relational Database:**
```
User Table:
| ID | Name  | Email          |
|----|-------|----------------|
| 1  | John  | john@mail.com |
| 2  | Jane  | jane@mail.com |
```

**ETCD Key-Value Store:**
```
/users/1 → {"name": "John", "email": "john@mail.com"}
/users/2 → {"name": "Jane", "email": "jane@mail.com"}
```

### Kubernetes Object Storage Example

```
Key: /registry/deployments/default/nginx-deployment
Value: (JSON encoded Deployment object)

{
  "apiVersion": "apps/v1",
  "kind": "Deployment",
  "metadata": {
    "name": "nginx-deployment",
    "namespace": "default"
  },
  "spec": {
    "replicas": 3,
    ...
  }
}
```

### Revisions & History

Every write operation increments the revision number:

```
Revision 100: /nginx/replicas = "3"
Revision 101: /nginx/replicas = "5"  ← Current
Revision 102: /nginx/replicas = "2"  ← Latest

Can retrieve any previous revision for point-in-time recovery
```

### MVCC (Multi-Version Concurrency Control)

ETCD maintains multiple versions of keys:

```
Key: /app/version

Version 1: "v1.0.0"
Version 2: "v1.1.0"
Version 3: "v1.2.0" ← Current
Version 4: "v2.0.0" ← Latest

Clients can read any version without blocking writers
```

---

## Installation & Setup

### System Requirements

**Compute:**
- CPU: Minimum 1 core, recommended 2+ cores
- Memory: Minimum 512MB, recommended 2GB+
- Disk: Fast SSD recommended, minimum 20GB

**Network:**
- Peer communication: Port 2380 (Raft)
- Client communication: Port 2379 (API)
- Low latency between cluster members

**OS Support:**
- Linux (recommended)
- macOS
- Windows

### Single Node Installation

```bash
# Download binary
ETCD_VERSION=v3.5.9
wget https://github.com/etcd-io/etcd/releases/download/${ETCD_VERSION}/etcd-${ETCD_VERSION}-linux-amd64.tar.gz

# Extract
tar -xzf etcd-${ETCD_VERSION}-linux-amd64.tar.gz
cd etcd-${ETCD_VERSION}-linux-amd64

# Run standalone ETCD
./etcd \
  --name=etcd-single \
  --data-dir=/var/lib/etcd \
  --listen-client-urls=http://localhost:2379 \
  --advertise-client-urls=http://localhost:2379
```

### Kubernetes Cluster ETCD

In a Kubernetes cluster, ETCD typically runs as a static pod:

```bash
# View ETCD pod in kube-system namespace
kubectl get pods -n kube-system | grep etcd

# Check ETCD pod details
kubectl describe pod -n kube-system etcd-master

# View ETCD logs
kubectl logs -n kube-system etcd-master --tail=50
```

### Docker Container Installation

```bash
docker run \
  -d \
  --name etcd \
  -p 2379:2379 \
  -p 2380:2380 \
  -e ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379 \
  -e ETCD_ADVERTISE_CLIENT_URLS=http://etcd:2379 \
  quay.io/coreos/etcd:v3.5.9
```

---

## ETCD Operations & API

### Common Operations

#### 1. **PUT (Write)**

```bash
# Simple put
etcdctl put /nginx/version "1.20"

# Put with flags
etcdctl put /nginx/version "1.21" \
  --lease=<lease-id> \
  --ignore-value \
  --ignore-lease
```

#### 2. **GET (Read)**

```bash
# Get single key
etcdctl get /nginx/version

# Get with prefix
etcdctl get /nginx --prefix

# Get all keys in range
etcdctl get /nginx 0  # from /nginx to end

# Get with output format
etcdctl get /nginx --write-out=json

# Get specific revision
etcdctl get /nginx --rev=102
```

#### 3. **DEL (Delete)**

```bash
# Delete single key
etcdctl del /nginx/version

# Delete with prefix
etcdctl del /nginx --prefix

# Delete range
etcdctl del /app /app0  # from /app to /app0

# Get delete count
etcdctl del /nginx --prefix --prev-kv
```

#### 4. **WATCH (Monitor Changes)**

```bash
# Watch single key
etcdctl watch /nginx/replicas

# Watch with prefix
etcdctl watch /nginx --prefix

# Watch from revision
etcdctl watch /nginx --rev=100

# Watch multiple keys
etcdctl watch /nginx /app
```

#### 5. **CAS (Compare-And-Swap)**

```bash
# Only update if current value matches
etcdctl put /counter "100"
etcdctl put /counter "101" --prev-kv  # Shows previous value

# With comparison
etcdctl txn <<EOF
compare: value("/counter") = "100"
success: put /counter "101"
failure: get /counter
```

#### 6. **TTL/Lease**

```bash
# Grant lease with TTL
lease_id=$(etcdctl lease grant 60 | awk '{print $2}')

# Associate key with lease
etcdctl put /session/user1 "active" --lease=$lease_id

# Keep lease alive
etcdctl lease keep-alive $lease_id

# Revoke lease (expires immediately)
etcdctl lease revoke $lease_id

# Check lease TTL
etcdctl lease timetolive $lease_id
```

### API Examples

#### Using curl

```bash
# GET with curl
curl http://localhost:2379/v3/kv/range \
  -X POST \
  -d '{"key":"L25naW54"}'  # base64 encoded "/nginx"

# PUT with curl
curl http://localhost:2379/v3/kv/put \
  -X POST \
  -d '{"key":"L25naW54","value":"MSAuMjA="}'

# DELETE with curl
curl http://localhost:2379/v3/kv/deleterange \
  -X POST \
  -d '{"key":"L25naW54"}'
```

#### Using Client Libraries

**Go Client:**
```go
package main

import (
    "context"
    clientv3 "go.etcd.io/etcd/client/v3"
)

func main() {
    cli, _ := clientv3.New(clientv3.Config{
        Endpoints: []string{"localhost:2379"},
    })
    defer cli.Close()

    // Put
    cli.Put(context.Background(), "/app/version", "1.0")

    // Get
    resp, _ := cli.Get(context.Background(), "/app/version")

    // Watch
    watchChan := cli.Watch(context.Background(), "/app/version")
}
```

**Python Client:**
```python
import etcd3

etcd = etcd3.client(host='localhost', port=2379)

# Put
etcd.put('/app/version', '1.0')

# Get
value = etcd.get('/app/version')

# Watch
watch_iter = etcd.watch('/app/version')
for event in watch_iter:
    print(event)
```

---

## Clustering

### Cluster Architecture

A production Kubernetes cluster typically uses a 3, 5, or 7-member ETCD cluster:

```
        Kubernetes Master/Control Plane
        
    ┌──────────────────────────────┐
    │  Master Node 1               │
    │  ├─ API Server               │
    │  ├─ Controller Manager        │
    │  ├─ Scheduler                │
    │  └─ ETCD Member 1 ⬤ LEADER  │
    └──────────────────────────────┘
    
    ┌──────────────────────────────┐
    │  Master Node 2               │
    │  ├─ API Server               │
    │  ├─ Controller Manager        │
    │  ├─ Scheduler                │
    │  └─ ETCD Member 2            │
    └──────────────────────────────┘
    
    ┌──────────────────────────────┐
    │  Master Node 3               │
    │  ├─ API Server               │
    │  ├─ Controller Manager        │
    │  ├─ Scheduler                │
    │  └─ ETCD Member 3            │
    └──────────────────────────────┘
    
    Raft Consensus (Port 2380)
    ────────────────────────────
```

### Member Management

```bash
# List cluster members
etcdctl member list

# Output:
# 2b8c092c66e0e3f7: name=etcd-1 peerURLs=http://10.0.0.1:2380 clientURLs=http://10.0.0.1:2379 isLeader=true
# 3e2c8c1a2f8b5d7f: name=etcd-2 peerURLs=http://10.0.0.2:2380 clientURLs=http://10.0.0.2:2379 isLeader=false
# 4f3d9d2b3g9c6e8g: name=etcd-3 peerURLs=http://10.0.0.3:2380 clientURLs=http://10.0.0.3:2379 isLeader=false

# Add new member
etcdctl member add etcd-4 --peer-urls=http://10.0.0.4:2380

# Remove member
etcdctl member remove <member-id>

# Update member
etcdctl member update <member-id> --peer-urls=http://10.0.0.5:2380
```

### Cluster Formation

**Static Bootstrap:**
```bash
etcd \
  --name=etcd-1 \
  --initial-advertise-peer-urls=http://10.0.0.1:2380 \
  --listen-peer-urls=http://10.0.0.1:2380 \
  --listen-client-urls=http://10.0.0.1:2379 \
  --advertise-client-urls=http://10.0.0.1:2379 \
  --initial-cluster=etcd-1=http://10.0.0.1:2380,etcd-2=http://10.0.0.2:2380,etcd-3=http://10.0.0.3:2380 \
  --initial-cluster-state=new
```

**Dynamic Discovery:**
```bash
etcd \
  --name=etcd-1 \
  --discovery=https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2a27 \
  --initial-advertise-peer-urls=http://10.0.0.1:2380 \
  --listen-peer-urls=http://10.0.0.1:2380 \
  --listen-client-urls=http://10.0.0.1:2379 \
  --advertise-client-urls=http://10.0.0.1:2379
```

### Quorum & Fault Tolerance

| Cluster Size | Quorum | Fault Tolerance |
|--------------|--------|-----------------|
| 1 member | 1 | 0 |
| 3 members | 2 | 1 |
| 5 members | 3 | 2 |
| 7 members | 4 | 3 |

**Rule:** With N members, you can tolerate (N-1)/2 failures

---

## Performance & Optimization

### Monitoring

```bash
# Check ETCD endpoint health
etcdctl endpoint health
# Output: 127.0.0.1:2379, healthy, took = 2.973972ms, error =

# Check endpoint status
etcdctl endpoint status
# Output: 127.0.0.1:2379, 3ad937344e6b66ab, 3.5.9, 12 MB, true, 7, 7

# Check leader
etcdctl member list | grep isLeader=true
```

### Performance Metrics

Access ETCD metrics on port 2379 at `/metrics`:

```bash
curl http://localhost:2379/metrics

# Key metrics:
# etcd_server_has_leader (binary - 0 or 1)
# etcd_server_leader_changes_seen_total
# etcd_disk_backend_commit_duration_seconds
# etcd_mvcc_put_total
# etcd_mvcc_range_total
# etcd_mvcc_delete_total
# etcd_disk_backend_defrag_duration_seconds
```

### Defragmentation

Over time, ETCD database size increases even after deletes. Defragmentation reclaims space:

```bash
# Check before defrag
etcdctl endpoint status

# Defragment
etcdctl defrag

# Check after defrag
etcdctl endpoint status
```

### Compaction

Remove historical revisions to free space:

```bash
# Get current revision
etcdctl endpoint status --write-out=json | jq '.header.revision'

# Compact old revisions (keep last 100000)
latest_rev=$(etcdctl endpoint status --write-out=json | jq '.header.revision')
etcdctl compact $((latest_rev - 100000))

# Defrag after compaction
etcdctl defrag
```

### Tuning Parameters

| Parameter | Effect | Value |
|-----------|--------|-------|
| `--quota-backend-bytes` | Max database size | Default: 2GB |
| `--heartbeat-interval` | Raft heartbeat | Default: 100ms |
| `--election-timeout` | Leader election timeout | Default: 1000ms |
| `--snapshot-count` | Snapshot frequency | Default: 100000 |
| `--max-request-bytes` | Max request size | Default: 2MB |

```bash
# Example with tuning
etcd \
  --quota-backend-bytes=8589934592 \
  --heartbeat-interval=100 \
  --election-timeout=1000 \
  --snapshot-count=50000
```

---

## Backup & Restore

### Backup Strategies

#### 1. **Snapshot Backup** (Recommended for Kubernetes)

```bash
# Create snapshot
sudo etcdctl snapshot save /backup/etcd-backup-$(date +%Y%m%d-%H%M%S).db

# Verify snapshot
sudo etcdctl snapshot status /backup/etcd-backup.db
```

#### 2. **Automated Backup Script**

```bash
#!/bin/bash
BACKUP_DIR="/backups/etcd"
RETENTION_DAYS=30
ETCDCTL_API=3

# Create backup
mkdir -p $BACKUP_DIR
timestamp=$(date +%Y%m%d-%H%M%S)
backup_file="$BACKUP_DIR/etcd-backup-$timestamp.db"

etcdctl snapshot save $backup_file

# Remove old backups
find $BACKUP_DIR -name "etcd-backup-*.db" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: $backup_file"
```

#### 3. **Kubernetes Native Backup**

```bash
# For kubeadm-managed ETCD
sudo cp -r /etc/kubernetes/pki/etcd /backup/etcd-pki-$(date +%Y%m%d-%H%M%S)

# Backup ETCD data directory
sudo cp -r /var/lib/etcd /backup/etcd-data-$(date +%Y%m%d)
```

### Restore Procedures

#### 1. **From Snapshot**

```bash
# Stop ETCD (if running)
sudo systemctl stop etcd

# Restore snapshot
sudo etcdctl snapshot restore /backup/etcd-backup.db \
  --data-dir=/var/lib/etcd-restored

# Verify restoration
sudo etcdctl --endpoints=http://localhost:2379 member list

# Start ETCD
sudo systemctl start etcd
```

#### 2. **Full Cluster Restore**

```bash
# On each ETCD member
sudo systemctl stop etcd

# Restore data directory
sudo rm -rf /var/lib/etcd
sudo cp -r /backup/etcd-data /var/lib/etcd
sudo chown -R etcd:etcd /var/lib/etcd

# Restore PKI if needed
sudo cp -r /backup/etcd-pki/* /etc/kubernetes/pki/etcd/

# Start ETCD on all members
sudo systemctl start etcd
```

#### 3. **Disaster Recovery**

```bash
# Restore single member to new cluster
etcdctl snapshot restore /backup/etcd-backup.db \
  --data-dir=/var/lib/etcd-new \
  --name=etcd-recovery \
  --initial-cluster=etcd-recovery=http://10.0.0.100:2380 \
  --initial-cluster-token=etcd-cluster-1 \
  --initial-advertise-peer-urls=http://10.0.0.100:2380

# Start recovery cluster with specified configuration
etcd \
  --name=etcd-recovery \
  --data-dir=/var/lib/etcd-new \
  --listen-peer-urls=http://10.0.0.100:2380 \
  --listen-client-urls=http://10.0.0.100:2379 \
  --advertise-client-urls=http://10.0.0.100:2379
```

---

## Troubleshooting

### Common Issues

#### 1. **ETCD Pod Not Running**

**Symptoms:**
```bash
$ kubectl get pods -n kube-system | grep etcd
# No output or CrashLoopBackOff status
```

**Diagnosis:**
```bash
# Check pod status
kubectl describe pod -n kube-system etcd-master

# Check logs
kubectl logs -n kube-system etcd-master --tail=50

# Check for common errors:
# - "address already in use" → Port conflict
# - "bad certificate file" → Certificate issue
# - "Raft quorum lost" → Cluster unanimous problem
```

**Solutions:**
```bash
# Free port
sudo lsof -i :2379
sudo kill <pid>

# Verify certificates
sudo openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text

# Reset ETCD (loses data!)
sudo rm -rf /var/lib/etcd
```

#### 2. **ETCD Cluster Unhealthy**

**Symptoms:**
```bash
$ etcdctl endpoint health
127.0.0.1:2379, unhealthy, took = 5.000028ms, error = context deadline exceeded
```

**Diagnosis:**
```bash
# Check member status
etcdctl member list

# Check connectivity between members
# From member 1, try reaching member 2
nc -zv member2-ip 2380

# Check ETCD logs
journalctl -u etcd -n 100
kubectl logs -n kube-system etcd-master
```

**Solutions:**
```bash
# Restart ETCD
sudo systemctl restart etcd

# If member is unresponsive, remove and re-add
etcdctl member remove <member-id>
# Reconfigure member and restart

# Check network policies
sudo iptables -L | grep 2380
sudo firewall-cmd --list-all
```

#### 3. **High Disk Usage**

**Symptoms:**
```bash
$ df -h /var/lib/etcd
Filesystem      Size  Used Avail Use%
/dev/sda1       100G   95G  5G   95%
```

**Diagnosis:**
```bash
# Check database size
etcdctl endpoint status --write-out=json | jq '.dbSize'

# Check for old revisions
etcdctl endpoint status

# Check backend usage
# Get current revision
current_rev=$(etcdctl endpoint status --write-out=json | jq '.header.revision')
echo "Current revision: $current_rev"
```

**Solutions:**
```bash
# Compact old revisions
latest_rev=$(etcdctl endpoint status --write-out=json | jq '.header.revision')
etcdctl compact $((latest_rev - 50000))

# Defragment
etcdctl defrag

# Increase quota if needed
# Restart ETCD with larger quota:
# --quota-backend-bytes=8589934592  # 8GB
```

#### 4. **Watch Connections Not Working**

**Symptoms:**
```bash
$ etcdctl watch /key
# No response, hangs indefinitely
```

**Diagnosis:**
```bash
# Check if API server is responding
etcdctl get /key

# Check network connectivity
netstat -an | grep 2379

# Check gRPC connectivity
grpcurl -plaintext localhost:2379 list
```

**Solutions:**
```bash
# Restart ETCD
sudo systemctl restart etcd

# Check firewall for bidirectional connectivity
sudo firewall-cmd --permanent --add-port=2379/tcp
sudo firewall-cmd --reload

# Rebuild client connection
```

#### 5. **Leader Election Failures**

**Symptoms:**
```bash
$ etcdctl member list
# Shows no leader or multiple leaders (split brain)
```

**Diagnosis:**
```bash
# Check current leader
etcdctl member list | grep leader=true

# Check election timeout in logs
journalctl -u etcd | grep "became leader\|election"

# Check cluster health
etcdctl endpoint health -w table
```

**Solutions:**
```bash
# Check network connectivity between members
for member in etcd1 etcd2 etcd3; do
  ssh $member "nc -zv etcd2 2380"
done

# Increase election timeout if network is unstable
# Restart with: --election-timeout=2000

# Force leader election by restarting leader
sudo systemctl restart etcd  # on leader node
```

#### 6. **Connection Timeout**

**Symptoms:**
```bash
$ kubectl get pods
error: unable to connect to the server: dial tcp i/o timeout
```

**Diagnosis:**
```bash
# Check API server status
kubectl get componentstatus

# Test ETCD directly
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

**Solutions:**
```bash
# Check API server logs
kubectl logs -n kube-system kube-apiserver

# Verify ETCD certificates
kubectl describe secret -n kube-system etcd-server

# Restart API server
kubectl delete pod -n kube-system kube-apiserver-master
```

---

## Best Practices

### 1. **High Availability Setup**

✅ **Always** use 3-node minimum ETCD cluster
✅ **Use** odd-numbered clusters (3, 5, 7)
✅ **Deploy** on dedicated hosts or separate nodes
✅ **Use** SSD storage for best performance
✅ **Monitor** cluster health continuously

```bash
# Recommended 3-node production setup
etcdctl member list
# 3 healthy members with 1 leader

# Verify quorum
# 3 members: majority = 2
# If any 2 are healthy, cluster operates
```

### 2. **Backup & Disaster Recovery**

✅ **Backup** ETCD daily
✅ **Test** restore procedures regularly
✅ **Keep** encrypted encrypted backups offsite
✅ **Document** recovery procedures
✅ **Set** RTO/RPO targets

```bash
# Suggested backup schedule
0 2 * * * /usr/local/bin/etcd-backup.sh  # Daily at 2 AM
0 0 * * 0 /usr/local/bin/etcd-prune.sh   # Weekly cleanup
```

### 3. **Security**

✅ **Use** TLS/SSL for all communication
✅ **Enable** authentication
✅ **Restrict** network access
✅ **Use** RBAC for API access
✅ **Audit** all changes

```bash
# TLS configuration
etcd \
  --cert-file=/etc/kubernetes/pki/etcd/server.crt \
  --key-file=/etc/kubernetes/pki/etcd/server.key \
  --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt \
  --client-cert-auth \
  --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt \
  --peer-key-file=/etc/kubernetes/pki/etcd/peer.key \
  --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt \
  --peer-client-cert-auth
```

### 4. **Performance Tuning**

✅ **Set** appropriate quota limits
✅ **Enable** auto-compaction
✅ **Monitor** latency metrics
✅ **Use** SSD for storage
✅ **Tune** Raft parameters for network conditions

```bash
# Auto-compaction configuration
etcd \
  --auto-compaction-mode=revision \
  --auto-compaction-retention=100000  # Keep last 100k revisions
  
# Or time-based
# --auto-compaction-mode=periodic \
# --auto-compaction-retention=1h
```

### 5. **Monitoring & Alerting**

✅ **Monitor** cluster health
✅ **Alert** on leader changes
✅ **Track** disk usage
✅ **Watch** request latency
✅ **Monitor** database size

```bash
# Key metrics to monitor
- etcd_server_has_leader
- etcd_server_leader_changes_seen_total
- etcd_disk_backend_commit_duration_seconds_bucket
- etcd_mvcc_db_total_size_in_bytes
- etcd_network_peer_round_trip_time_seconds
- process_resident_memory_bytes
```

### 6. **Operational Procedures**

✅ **Document** cluster topology
✅ **Maintain** playbooks for common failures
✅ **Practice** disaster recovery
✅ **Plan** upgrades carefully
✅ **Version** control configuration

```bash
# Operational checklist
- Daily health check: etcdctl endpoint health
- Weekly backup verification
- Monthly defragment: etcdctl defrag
- Quarterly disaster recovery drill
- Annual cluster review and optimization
```

---

## Security

### TLS Certificates

ETCD requires certificates for:
- **Server Certificate**: Authenticates ETCD server to clients
- **Peer Certificate**: Authenticates members to each other
- **Client Certificate**: Authenticates clients to ETCD

```bash
# View certificate details
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout

# Check certificate expiration
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt \
  -noout -dates

# Rotate certificates (planned maintenance)
kubeadm certs renew all
```

### Authentication & Authorization

```bash
# Enable authentication
etcdctl user add root
etcdctl user grant-role root root

# Create application user
etcdctl user add app-user
etcdctl role create app-role
etcdctl role grant-permission app-role readwrite /app
etcdctl user grant-role app-user app-role

# Enable auth
etcdctl auth enable

# Use authentication in commands
etcdctl --user=app-user get /app
```

### Encryption at Rest

By default, ETCD stores data unencrypted. Use encryption provider:

```yaml
# encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE64-ENCODED-32-BYTE-KEY>
      - identity: {}
```

### Network Policies

Restrict ETCD access:

```bash
# UFW firewall (Ubuntu)
sudo ufw allow from 10.0.0.1 to any port 2379  # API
sudo ufw allow from 10.0.0.2 to any port 2379
sudo ufw allow from 10.0.0.1 to any port 2380  # Peer
sudo ufw allow from 10.0.0.2 to any port 2380

# iptables
sudo iptables -A INPUT -p tcp --dport 2379 -s 10.0.0.1 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 2380 -s 10.0.0.1 -j ACCEPT
```

---

## Examples

### Example 1: Deploying a StatefulSet with ETCD Persistence

In Kubernetes, when you create a StatefulSet, the definitions are stored in ETCD:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
```

**What gets stored in ETCD:**
```
/registry/statefulsets/default/mysql
/registry/controllerrevisions/default/mysql-<revision>
/registry/persistentvolumeclaims/default/data-mysql-0
/registry/persistentvolumeclaims/default/data-mysql-1
/registry/persistentvolumeclaims/default/data-mysql-2
```

### Example 2: Monitoring ETCD from Kubernetes

```bash
# Port-forward to ETCD pod
kubectl port-forward -n kube-system etcd-master 2379:2379

# In another terminal, query ETCD
etcdctl --endpoints=http://localhost:2379 member list

# Check specific namespace data
etcdctl --endpoints=http://localhost:2379 \
  get /registry/namespaces/default --prefix

# Find all pods in default namespace
etcdctl --endpoints=http://localhost:2379 \
  get /registry/pods/default --prefix
```

### Example 3: Implementing Watch for ConfigMap Changes

Developers can watch for ConfigMap updates:

```python
import etcd3

etcd = etcd3.client(host='localhost', port=2379)

# Watch for changes to app-config
watch_iter = etcd.watch_prefix('/registry/configmaps/default/app-config')

for event in watch_iter:
    if isinstance(event, etcd3.events.PutEvent):
        print(f"ConfigMap updated: {event.key}")
        print(f"New value: {event.value}")
    elif isinstance(event, etcd3.events.DeleteEvent):
        print(f"ConfigMap deleted: {event.key}")
```

### Example 4: Capacity Planning for ETCD

```bash
# Check current usage
etcdctl endpoint status --write-out=json | jq '.dbSize'
# Output: 1234567890 (1.2 GB)

# Estimate growth
# Assume: 1000 pods, each stores ~2KB
# Total: 1000 * 2KB = 2MB per namespace state
# With 50 namespaces: 100MB data + overhead = 500MB optimal
# Plan quota for 2x growth: 1GB quota recommended

# Set quota
etcd --quota-backend-bytes=1073741824  # 1GB
```

---

## Quick Reference

### ETCD Ports

| Port | Purpose |
|------|---------|
| 2379 | Client API communication |
| 2380 | Peer-to-peer communication |

### Key ETCD Commands

```bash
# Health & Status
etcdctl endpoint health
etcdctl endpoint status
etcdctl member list

# Data Operations
etcdctl put /key value
etcdctl get /key
etcdctl del /key
etcdctl watch /key

# Maintenance
etcdctl snapshot save backup.db
etcdctl snapshot restore backup.db
etcdctl defrag
etcdctl compact <revision>

# Cluster
etcdctl member add <name> --peer-urls=<urls>
etcdctl member remove <id>
etcdctl member update <id> --peer-urls=<urls>
```

### Kubernetes ETCD Locations

| Item | Path |
|------|------|
| Deployment | `/registry/deployments/<namespace>/<name>` |
| StatefulSet | `/registry/statefulsets/<namespace>/<name>` |
| Pod | `/registry/pods/<namespace>/<name>` |
| ConfigMap | `/registry/configmaps/<namespace>/<name>` |
| Secret | `/registry/secrets/<namespace>/<name>` |
| Service | `/registry/services/specs/<namespace>/<name>` |

---

## Conclusion

ETCD is the critical backbone of Kubernetes, storing all cluster state and configuration. Understanding its architecture, operations, backup/restore procedures, and troubleshooting is essential for maintaining healthy, reliable Kubernetes clusters. Always follow best practices for security, backup, and monitoring to ensure your cluster's data safety and availability.