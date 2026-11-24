
# EXPERIMENT 13 – Distributed File System and Loading Data into HDFS

## AIM:

To simulate how HDFS splits a file into blocks, replicates them across DataNodes and recovers from a node failure.

## QUESTION:

A smart-city project stores sensor logs in the Hadoop Distributed File System (HDFS). Write a Python simulation of a six-node, two-rack HDFS cluster with a NameNode and DataNodes. Load (put) a log file, inspect block placement, read it back, and show how HDFS keeps the file available when a DataNode fails.

The program must:

1. Split the file into fixed-size blocks and assign a block ID to each.
2. Place three replicas of every block using rack-aware placement.
3. Maintain NameNode metadata (file to blocks, block to DataNodes) and a DataNode storage report.
4. Read the file back and verify it with a checksum.
5. Simulate a DataNode failure, re-replicate the lost blocks and confirm the file is still healthy.

## ALGORITHM:

1. Import hashlib and itertools modules.
2. Define the block size and replication factor.
3. Create a DataNode class to store blocks, rack information and node status.
4. Create a NameNode class to maintain file and block metadata.
5. Implement rack-aware replica placement across the two racks.
6. Split the sensor log into fixed-size blocks and assign unique block IDs.
7. Store three replicas of each block across the DataNodes.
8. Display NameNode metadata and DataNode storage information.
9. Read the stored file and verify it using an MD5 checksum.
10. Simulate DN1 failure and re-replicate the missing blocks.
11. Read the file again and verify its checksum and replication health.

## CODE:

```python
import hashlib
import itertools

BLOCK_SIZE = 64
REPLICATION = 3


class DataNode:
    def __init__(self, name, rack):
        self.name, self.rack, self.alive = name, rack, True
        self.blocks = {}


class NameNode:
    """Keeps only metadata: file -> blocks, block -> DataNodes."""

    def __init__(self, datanodes):
        self.datanodes = datanodes
        self.namespace = {}
        self.block_map = {}
        self._rr = itertools.cycle(range(len(datanodes)))
        self._next_id = 1

    def choose_targets(self):
        """Rack-aware: 1st replica on one rack, 2nd and 3rd on another rack."""
        live = [d for d in self.datanodes if d.alive]
        turn = next(self._rr)
        first = live[turn % len(live)]
        others = [d for d in live if d.rack != first.rack]
        second = others[turn % len(others)]
        third = [
            d for d in others if d.rack == second.rack and d is not second
        ][0]
        return [first, second, third]

    def put(self, path, data):
        ids = []
        for start in range(0, len(data), BLOCK_SIZE):
            chunk = data[start : start + BLOCK_SIZE]
            bid = f"blk_{self._next_id:04d}"
            self._next_id += 1
            targets = self.choose_targets()

            for dn in targets:
                dn.blocks[bid] = chunk

            self.block_map[bid] = [dn.name for dn in targets]
            ids.append(bid)

        self.namespace[path] = ids
        return ids

    def get(self, path):
        out = []
        for bid in self.namespace[path]:
            for name in self.block_map[bid]:
                dn = self.node(name)
                if dn.alive:
                    out.append(dn.blocks[bid])
                    break
        return "".join(out)

    def node(self, name):
        return next(d for d in self.datanodes if d.name == name)

    def handle_failure(self, name):
        failed = self.node(name)
        failed.alive = False

        repaired = 0
        for bid, holders in self.block_map.items():
            if name in holders:
                holders.remove(name)
                source = self.node(holders[0])
                spare = [
                    d for d in self.datanodes
                    if d.alive and d.name not in holders
                ][0]
                spare.blocks[bid] = source.blocks[bid]
                holders.append(spare.name)
                repaired += 1

        return repaired


def checksum(text):
    return hashlib.md5(text.encode()).hexdigest()[:8]


cluster = [
    DataNode("DN1", "Rack-1"),
    DataNode("DN2", "Rack-1"),
    DataNode("DN3", "Rack-1"),
    DataNode("DN4", "Rack-2"),
    DataNode("DN5", "Rack-2"),
    DataNode("DN6", "Rack-2"),
]

nn = NameNode(cluster)

log = "\n".join(
    f"2026-03-0{d} 10:{m:02d} sensor={s} temp={30 + (d * m + s) % 9}"
    for d in range(1, 3)
    for m in (5, 20)
    for s in (1, 2)
)

print("Distributed File System Simulation (HDFS)")
print("=" * 45)
print(f"Block size = {BLOCK_SIZE} bytes | Replication factor = {REPLICATION}")
print("Cluster    :", ", ".join(f"{d.name}({d.rack})" for d in cluster))

print("\n$ hdfs dfs -put sensor_log.txt /data/sensor_log.txt")
blocks = nn.put("/data/sensor_log.txt", log)
print(f"File size = {len(log)} bytes -> {len(blocks)} blocks")

print("\nNameNode metadata (block -> DataNodes):")
for bid in blocks:
    racks = {nn.node(n).rack for n in nn.block_map[bid]}
    print(f" {bid} {nn.block_map[bid]} racks={sorted(racks)}")

print("\nDataNode storage report:")
for d in cluster:
    print(f" {d.name} blocks={sorted(d.blocks)}")

print("\n$ hdfs dfs -cat /data/sensor_log.txt | head -3")
data = nn.get("/data/sensor_log.txt")
print("\n".join(data.splitlines()[:3]))
print("Checksum original =", checksum(log), "| read back =", checksum(data))

print("\n*** DataNode DN1 fails (heartbeat lost) ***")
fixed = nn.handle_failure("DN1")
print("Under-replicated blocks re-replicated:", fixed)

for bid in blocks:
    print(f" {bid} {nn.block_map[bid]}")

data2 = nn.get("/data/sensor_log.txt")
print(
    "File readable after failure:", data2 == log,
    "| checksum =", checksum(data2)
)

print("\n$ hdfs fsck /data/sensor_log.txt -files -blocks")
healthy = all(len(v) == REPLICATION for v in nn.block_map.values())

print(
    f"Total blocks: {len(blocks)} | Replication: {REPLICATION} | "
    f"Status: {'HEALTHY' if healthy else 'CORRUPT'}"
)
```

## EQUIVALENT HDFS COMMANDS ON A REAL CLUSTER:

```bash
hdfs dfs -mkdir -p /data
hdfs dfs -put sensor_log.txt /data/
hdfs dfs -ls /data
hdfs dfs -cat /data/sensor_log.txt | head -3
hdfs fsck /data/sensor_log.txt -files -blocks -locations
hdfs dfs -get /data/sensor_log.txt ./copy.txt
```

## OUTPUT:

```text
Distributed File System Simulation (HDFS)
=============================================

Block size = 64 bytes | Replication factor = 3
Cluster    : DN1(Rack-1), DN2(Rack-1), DN3(Rack-1), DN4(Rack-2), DN5(Rack-2), DN6(Rack-2)

$ hdfs dfs -put sensor_log.txt /data/sensor_log.txt
File size = 271 bytes -> 5 blocks

NameNode metadata (block -> DataNodes):
 blk_0001 ['DN1', 'DN4', 'DN5'] racks=['Rack-1', 'Rack-2']
 blk_0002 ['DN2', 'DN5', 'DN4'] racks=['Rack-1', 'Rack-2']
 blk_0003 ['DN3', 'DN6', 'DN4'] racks=['Rack-1', 'Rack-2']
 blk_0004 ['DN4', 'DN1', 'DN2'] racks=['Rack-1', 'Rack-2']
 blk_0005 ['DN5', 'DN2', 'DN1'] racks=['Rack-1', 'Rack-2']

DataNode storage report:
 DN1 blocks=['blk_0001', 'blk_0004', 'blk_0005']
 DN2 blocks=['blk_0002', 'blk_0004', 'blk_0005']
 DN3 blocks=['blk_0003']
 DN4 blocks=['blk_0001', 'blk_0002', 'blk_0003', 'blk_0004']
 DN5 blocks=['blk_0001', 'blk_0002', 'blk_0005']
 DN6 blocks=['blk_0003']

$ hdfs dfs -cat /data/sensor_log.txt | head -3
2026-03-01 10:05 sensor=1 temp=36
2026-03-01 10:05 sensor=2 temp=37
2026-03-01 10:20 sensor=1 temp=33

Checksum original = 48e3c478 | read back = 48e3c478

*** DataNode DN1 fails (heartbeat lost) ***
Under-replicated blocks re-replicated: 3
 blk_0001 ['DN4', 'DN5', 'DN2']
 blk_0002 ['DN2', 'DN5', 'DN4']
 blk_0003 ['DN3', 'DN6', 'DN4']
 blk_0004 ['DN4', 'DN2', 'DN3']
 blk_0005 ['DN5', 'DN2', 'DN3']

File readable after failure: True | checksum = 48e3c478

$ hdfs fsck /data/sensor_log.txt -files -blocks
Total blocks: 5 | Replication: 3 | Status: HEALTHY
```

## RESULT:

Thus, an HDFS cluster was successfully simulated using Python. The sensor log was split into five blocks, with three replicas distributed across two racks. The file was successfully read and verified using a checksum. After DN1 failed, three affected blocks were re-replicated, and the file remained readable with a HEALTHY status.
```
