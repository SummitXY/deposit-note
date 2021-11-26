# PingCAP

- [PingCAP](#pingcap)
  - [Tutorial](#tutorial)
    - [TiDB Internal (I) - Data Storage](#tidb-internal-i---data-storage)

## Tutorial

### TiDB Internal (I) - Data Storage
> https://en.pingcap.com/blog/tidb-internal-data-storage

RAID(Redundant Array of Independent Disks) 备份磁盘，用于防止一个磁盘挂了而导致的数据丢失

TiKV can be seen one huge Map where Key and Value are the original Byte array.

And In the huge Map, the key-value paires are ordered according to the Key's binary sequence.

TiKV is based on the standalone KV storage engine - `RocksDB`, and uses `RAFT` to replicate data to 
multiple machines in case of machine failure.

![](img/raft-rocksdb.png)

For a KV storage, there are two typical solutions to distribute data among multiple machines:
1. Create `Hash` and select `region/node` according to the Hash value
2. Use `Range` and store a segmentof serial Key in a storage node