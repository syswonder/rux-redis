
# How to run?
- Run:
  
  - for aarch64(without 9pfs)
  
     ```
     make A=apps/c/redis/ LOG=error NET=y BLK=y ARCH=aarch64 SMP=4 ARGS="./redis-server,--bind,0.0.0.0,--port,5555,--save,\"\",--appendonly,no,--protected-mode,no,--ignore-warnings,ARM64-COW-BUG" run
     ```
  
  - for x86_64(without 9pfs)
  
     ```
     make A=apps/c/redis/ LOG=error NET=y BLK=y ARCH=x86_64 SMP=4 ARGS="./redis-server,--bind,0.0.0.0,--port,5555,--save,\"\",--appendonly,no,--protected-mode,no" run
     ```

  - for aarch64(using 9pfs)
     ```
     make A=apps/c/redis/ LOG=error NET=y V9P=y BLK=y FEATURES=virtio-9p V9P_PATH=apps/c/redis ARCH=aarch64 SMP=4 ARGS="./redis-server,/v9fs/redis.conf" run
     ```  

  - for x86_64(using 9pfs)
     ```
     make A=apps/c/redis/ LOG=error NET=y V9P=y BLK=y FEATURES=virtio-9p V9P_PATH=apps/c/redis ARCH=x86_64 SMP=4 ARGS="./redis-server,/v9fs/redis.conf" run
     ```  

# How to test?
- Use `redis-cli -p 5555` to connect to redis-server, and enjoy Ruxos-Redis world!
- Use `redis-benchmark -p 5555` and other optional parameters to run the benchmark.
    - Like: `redis-benchmark -p 5555 -n 5 -q -c 10`, this command issues 5 requests for each commands (like `set`, `get`, etc.), with 10 concurrency.
- It is recommended to **use pipeline**: `redis-benchmark -p 5555 -n 100000 -q -c 30 -P 16 -t set`
    - Use `-t` to specify what to test: set, get, incr, lpush, rpush, lpop, rpop, sadd, hset, spop, zadd, mset

# Some test result
## AARCH64
### SET and GET(without pipiline):
- Here test `SET` and `GET` throughput.(30 conns, 100k requests like `unikraft`)
- command:
  - `redis-benchmark -p 5555 -n 100000 -q -t set -c 30`
  - `redis-benchmark -p 5555 -n 100000 -q -t get -c 30`

- 0714(Update net implementation, maximum: 2.9w(set))

| Operation | Op number | Concurrency | Round | Result(request per seconds) |
|-|-|-|-|-|
| SET | 100K | 30 | 1 | 25713.55 |
| | | | 2 | 25246.15 |
| | | | 3 | 24968.79 |
| | | | 4 | 25018.76 |
| | | | 5 | 25348.54 |
| GET | 100K | 30 | 1 | 27917.37 |
| | | | 2 | 28360.75 |
| | | | 3 | 27525.46 |
| | | | 4 | 27901.79 |
| | | | 5 | 27495.19 |

### Other tests, one round(without pipiline)

- 0714
```
PING_INLINE: 28768.70 requests per second
PING_BULK: 31347.96 requests per second
SET: 23185.72 requests per second
GET: 25700.33 requests per second
INCR: 25746.65 requests per second
LPUSH: 20416.50 requests per second
RPUSH: 20868.12 requests per second
LPOP: 20370.75 requests per second
RPOP: 19956.10 requests per second
SADD: 25361.40 requests per second
HSET: 21431.63 requests per second
SPOP: 25438.82 requests per second
ZADD: 23820.87 requests per second
ZPOPMIN: 26954.18 requests per second
LPUSH (needed to benchmark LRANGE): 26385.22 requests per second
LRANGE_100 (first 100 elements): 23912.00 requests per second
LRANGE_300 (first 300 elements): 22665.46 requests per second
LRANGE_500 (first 450 elements): 23369.95 requests per second
LRANGE_600 (first 600 elements): 22256.84 requests per second
MSET (10 keys): 18460.40 requests per second
```

## X86_64
### SET and GET(without pipiline)
- command:
  - `redis-benchmark -p 5555 -n 100000 -q -t set -c 30`
  - `redis-benchmark -p 5555 -n 100000 -q -t get -c 30`

- 0714

| Operation | Op number | Concurrency | Round | Result(request per seconds) |
|-|-|-|-|-|
| SET | 100K | 30 | 1 | 105263.16 |
| | | | 2 | 105263.16 |
| | | | 3 | 103950.10 |
| | | | 4 | 107758.62 |
| | | | 5 | 105820.11 |
| GET | 100K | 30 | 1 | 103199.18 |
| | | | 2 | 104058.27 |
| | | | 3 | 99502.48 |
| | | | 4 | 106951.88 |
| | | | 5 | 105263.16 |

### Other tests(without pipiline)

- 0714

```
PING_INLINE: 111607.14 requests per second
PING_BULK: 102880.66 requests per second
SET: 80971.66 requests per second
GET: 103519.66 requests per second
INCR: 98425.20 requests per second
LPUSH: 70274.07 requests per second
RPUSH: 108108.11 requests per second
LPOP: 53561.86 requests per second
RPOP: 100200.40 requests per second
SADD: 62150.41 requests per second
HSET: 99009.90 requests per second
SPOP: 104712.05 requests per second
ZADD: 105263.16 requests per second
ZPOPMIN: 110497.24 requests per second
LPUSH (needed to benchmark LRANGE): 74682.60 requests per second
LRANGE_100 (first 100 elements): 62305.30 requests per second
LRANGE_300 (first 300 elements): 8822.23 requests per second
LRANGE_500 (first 450 elements): 22446.69 requests per second
LRANGE_600 (first 600 elements): 17280.11 requests per second
MSET (10 keys): 92081.03 requests per second
```

- Comparing to local Redis server
```
PING_INLINE: 176056.33 requests per second
PING_BULK: 173611.12 requests per second
SET: 175131.36 requests per second
GET: 174825.17 requests per second
INCR: 177304.97 requests per second
LPUSH: 176678.45 requests per second
RPUSH: 176056.33 requests per second
LPOP: 178253.12 requests per second
RPOP: 176678.45 requests per second
SADD: 175746.92 requests per second
HSET: 176991.16 requests per second
SPOP: 176991.16 requests per second
ZADD: 177619.89 requests per second
ZPOPMIN: 176056.33 requests per second
LPUSH (needed to benchmark LRANGE): 178253.12 requests per second
LRANGE_100 (first 100 elements): 113895.21 requests per second
LRANGE_300 (first 300 elements): 50942.43 requests per second
LRANGE_500 (first 450 elements): 35186.49 requests per second
LRANGE_600 (first 600 elements): 28320.59 requests per second
MSET (10 keys): 183150.19 requests per second
```

# Run RuxOS-Redis On PC

## Notification

- Comment out `spt_init()`(Already patched).
- It will be nicer to comment out `pthread_cond_wait` as well.

## Compile and Run

- `make A=apps/c/redis LOG=error PLATFORM=x86_64-pc-oslab SMP=4 FEATURES=driver-ixgbe,driver-ramdisk IP=10.2.2.2 GW=10.2.2.1`
- Copy `redis_x86_64-pc-oslab.elf` to `/boot`, then reboot.
- Enter `grub` then boot the PC by Ruxos Redis.
- Connect to Ruxos-Redis server by:
  - `redis-cli -p 5555 -h 10.2.2.2`
  - `redis-benchmark -p 5555 -h 10.2.2.2`

## Test Result(without pipiline)

### Qemu

- 0801

```
PING_INLINE: 54171.18 requests per second
PING_BULK: 69156.30 requests per second
SET: 70274.07 requests per second
GET: 71479.62 requests per second
INCR: 65876.16 requests per second
LPUSH: 53850.30 requests per second
RPUSH: 53908.36 requests per second
LPOP: 67704.80 requests per second
RPOP: 69832.40 requests per second
SADD: 69444.45 requests per second
HSET: 69541.03 requests per second
SPOP: 69492.70 requests per second
ZADD: 53879.31 requests per second
ZPOPMIN: 53908.36 requests per second
LPUSH (needed to benchmark LRANGE): 37216.23 requests per second
LRANGE_100 (first 100 elements): 48123.20 requests per second
LRANGE_300 (first 300 elements): 7577.48 requests per second
LRANGE_500 (first 450 elements): 15288.18 requests per second
LRANGE_600 (first 600 elements): 12371.64 requests per second
MSET (10 keys): 54318.30 requests per second
```

### Real Machine(without pipiline)

- 0801

```
PING_INLINE: 71377.59 requests per second
PING_BULK: 74571.22 requests per second
SET: 75642.96 requests per second
GET: 75872.54 requests per second
INCR: 76394.20 requests per second
LPUSH: 75815.01 requests per second
RPUSH: 75930.14 requests per second
LPOP: 75528.70 requests per second
RPOP: 75585.79 requests per second
SADD: 75815.01 requests per second
HSET: 75757.57 requests per second
SPOP: 75930.14 requests per second
ZADD: 75757.57 requests per second
ZPOPMIN: 75757.57 requests per second
LPUSH (needed to benchmark LRANGE): 75471.70 requests per second
LRANGE_100 (first 100 elements): 52438.39 requests per second
LRANGE_300 (first 300 elements): 677.74 requests per second
LRANGE_500 (first 450 elements): 16485.33 requests per second
LRANGE_600 (first 600 elements): 13159.63 requests per second
MSET (10 keys): 72780.20 requests per second
```
