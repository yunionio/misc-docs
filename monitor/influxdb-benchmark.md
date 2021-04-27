# Influxdb Bechmark

- Tools: https://github.com/influxdata/influxdb-comparisons

## 安装测试工具

```bash
go get \
    github.com/influxdata/influxdb-comparisons/cmd/bulk_data_gen \
    github.com/influxdata/influxdb-comparisons/cmd/bulk_load_influx \
    github.com/influxdata/influxdb-comparisons/cmd/bulk_query_gen \
    github.com/influxdata/influxdb-comparisons/cmd/query_benchmarker_influxdb
```

## 加载数据

```bash
$GOPATH/bin/bulk_data_gen | $GOPATH/bin/bulk_load_influx -urls http://10.127.40.252:30086
# local path
loaded 77760 items in 5.912244sec with 1 workers (mean point rate 13152.366966/sec, mean value rate 147598.784846/s, 5.42MB/sec from stdin)

# minio s3fs
loaded 77760 items in 19.037617sec with 1 workers (mean point rate 4084.544762/sec, mean value rate 45837.668994/s, 1.68MB/sec from stdin)
```

## 查询数据

```bash
$GOPATH/bin/bulk_query_gen -query-type "1-host-1-hr" | $GOPATH/bin/query_benchmarker_influxdb -urls http://10.127.40.252:30086
# local path
run complete after 1000 queries with 1 workers:
InfluxDB (InfluxQL) max cpu, rand    1 hosts, rand 1h0m0s by 1m : min:     9.33ms ( 107.23/sec), mean:    12.37ms (  80.85/sec), max:  240.13ms (  4.16/sec), count:     1000, sum:  12.4sec
all queries                                                     : min:     9.33ms ( 107.23/sec), mean:    12.37ms (  80.85/sec), max:  240.13ms (  4.16/sec), count:     1000, sum:  12.4sec
wall clock time: 12.396242sec

# minio s3fs
run complete after 1000 queries with 1 workers:
InfluxDB (InfluxQL) max cpu, rand    1 hosts, rand 1h0m0s by 1m : min:     8.80ms ( 113.68/sec), mean:    11.99ms (  83.37/sec), max:   60.77ms ( 16.45/sec), count:     1000, sum:  12.0sec
all queries                                                     : min:     8.80ms ( 113.68/sec), mean:    11.99ms (  83.37/sec), max:   60.77ms ( 16.45/sec), count:     1000, sum:  12.0sec
wall clock time: 12.021002sec
```

## 使用 inch 读写数据

```bash
git clone https://github.com/influxdata/inch && cd insh
go build cmd/inch -o inch
```

### 写 100M points

```bash
./inch -host http://10.127.40.252:30086 -t 100,10,10 -p 10000

# local path
Total time: 3013.7 seconds

# minio s3fs
Total time: 9259.2 seconds
```

## 使用 influx-stress 测试

https://github.com/influxdata/influx-stress

```bash
go get -v github.com/influxdata/influx-stress/cmd/...

# 测试 60s
$GOPATH/bin/influx-stress insert --host http://10.127.40.252:30086 --runtime 60s
# local path
Write Throughput: 117084
Points Written: 7260000

# s3fs
Write Throughput: 83479
Points Written: 5260000
```

## fio 测试

1. 大文件顺序读

```
## s3fs
fio --name=big-file-sequential-read --directory=/s3fs --rw=read --refill_buffers --bs=256k --size=4G
big-file-sequential-read: (g=0): rw=read, bs=(R) 256KiB-256KiB, (W) 256KiB-256KiB, (T) 256KiB-256KiB, ioengine=psync, iodepth=1
fio-3.16
Starting 1 process
big-file-sequential-read: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [R(1)][93.6%][r=202MiB/s][r=808 IOPS][eta 00m:06s]
big-file-sequential-read: (groupid=0, jobs=1): err= 0: pid=876: Fri Apr 23 14:33:06 2021
  read: IOPS=185, BW=46.4MiB/s (48.7MB/s)(4096MiB/88217msec)
    clat (usec): min=78, max=945780, avg=5377.11, stdev=28865.86
     lat (usec): min=78, max=945780, avg=5377.68, stdev=28865.84
    clat percentiles (usec):
     | 1.00th=[   121],  5.00th=[   180], 10.00th=[   200], 20.00th=[   223],
     | 30.00th=[   243], 40.00th=[   285], 50.00th=[   429], 60.00th=[   766],
     | 70.00th=[  1090], 80.00th=[  2671], 90.00th=[  4178], 95.00th=[  9110],
     | 99.00th=[145753], 99.50th=[217056], 99.90th=[358613], 99.95th=[459277],
     | 99.99th=[608175]
   bw (  KiB/s): min= 8159, max=361322, per=98.36%, avg=46764.49, stdev=40435.98, samples=175
   iops        : min=   31, max= 1411, avg=182.48, stdev=157.91, samples=175
  lat (usec)   : 100=0.02%, 250=32.90%, 500=19.52%, 750=7.15%, 1000=8.26%
  lat (msec)   : 2=9.12%, 4=11.90%, 10=6.40%, 20=1.25%, 50=1.10%
  lat (msec)   : 100=0.76%, 250=1.29%, 500=0.30%, 750=0.01%, 1000=0.01%
  cpu          : usr=0.23%, sys=4.33%, ctx=11156, majf=0, minf=94
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=16384,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=46.4MiB/s (48.7MB/s), 46.4MiB/s-46.4MiB/s (48.7MB/s-48.7MB/s), io=4096MiB (4295MB), run=88217-88217msec

## local
fio --name=big-file-sequential-read --directory=/local --rw=read --refill_buffers --bs=256k --size=4G
big-file-sequential-read: (g=0): rw=read, bs=(R) 256KiB-256KiB, (W) 256KiB-256KiB, (T) 256KiB-256KiB, ioengine=psync, iodepth=1
fio-3.16
Starting 1 process
big-file-sequential-read: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [R(1)][100.0%][r=103MiB/s][r=410 IOPS][eta 00m:00s]
big-file-sequential-read: (groupid=0, jobs=1): err= 0: pid=889: Fri Apr 23 15:12:36 2021
  read: IOPS=345, BW=86.4MiB/s (90.6MB/s)(4096MiB/47399msec)
    clat (usec): min=45, max=450634, avg=2889.26, stdev=13612.31
     lat (usec): min=45, max=450634, avg=2889.60, stdev=13612.30
    clat percentiles (usec):
     | 1.00th=[    53],  5.00th=[    60], 10.00th=[    67], 20.00th=[    94],
     | 30.00th=[    99], 40.00th=[   104], 50.00th=[   113], 60.00th=[   131],
     | 70.00th=[   153], 80.00th=[  1369], 90.00th=[  4178], 95.00th=[ 10028],
     | 99.00th=[ 64226], 99.50th=[ 98042], 99.90th=[170918], 99.95th=[219153],
     | 99.99th=[354419]
   bw (  KiB/s): min=14336, max=141312, per=99.51%, avg=88052.00, stdev=26309.08, samples=94
   iops        : min=   56, max=  552, avg=343.85, stdev=102.82, samples=94
  lat (usec)   : 50=0.19%, 100=31.34%, 250=46.17%, 500=0.72%, 750=0.37%
  lat (usec)   : 1000=0.79%
  lat (msec)   : 2=1.61%, 4=7.62%, 10=6.18%, 20=1.57%, 50=2.04%
  lat (msec)   : 100=0.96%, 250=0.43%, 500=0.02%
  cpu          : usr=0.21%, sys=7.58%, ctx=3819, majf=0, minf=95
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=16384,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=86.4MiB/s (90.6MB/s), 86.4MiB/s-86.4MiB/s (90.6MB/s-90.6MB/s), io=4096MiB (4295MB), run=47399-47399msec
```

2. 大文件顺序写

```
# s3fs
fio --name=big-file-sequential-write --directory=/s3fs --rw=write --refill_buffers --bs=256k --size=4G
big-file-sequential-write: (g=0): rw=write, bs=(R) 256KiB-256KiB, (W) 256KiB-256KiB, (T) 256KiB-256KiB, ioengine=psync, iodepth=1
fio-3.16
Starting 1 process
big-file-sequential-write: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [f(1)][100.0%][eta 00m:00s]
big-file-sequential-write: (groupid=0, jobs=1): err= 0: pid=898: Fri Apr 23 15:27:34 2021
  write: IOPS=87, BW=21.8MiB/s (22.8MB/s)(4096MiB/188153msec); 0 zone resets
    clat (usec): min=522, max=7785.2k, avg=11376.92, stdev=88501.82
     lat (usec): min=522, max=7785.2k, avg=11377.52, stdev=88501.81
    clat percentiles (usec):
     |  1.00th=[    553],  5.00th=[    578], 10.00th=[    594],
     | 20.00th=[    627], 30.00th=[    668], 40.00th=[    717],
     | 50.00th=[    816], 60.00th=[   1139], 70.00th=[   1401],
     | 80.00th=[   5473], 90.00th=[  53740], 95.00th=[  58459],
     | 99.00th=[  66847], 99.50th=[  85459], 99.90th=[ 154141],
     | 99.95th=[ 212861], 99.99th=[6207570]
   bw (  KiB/s): min=  510, max=206023, per=100.00%, avg=24617.00, stdev=24586.75, samples=340
   iops        : min=    1, max=  804, avg=96.06, stdev=96.00, samples=340
  lat (usec)   : 750=44.94%, 1000=9.39%
  lat (msec)   : 2=19.78%, 4=4.57%, 10=3.47%, 20=1.15%, 50=4.80%
  lat (msec)   : 100=11.58%, 250=0.29%, 750=0.01%, 1000=0.01%, 2000=0.01%
  lat (msec)   : >=2000=0.02%
  cpu          : usr=0.71%, sys=1.66%, ctx=33083, majf=0, minf=28
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,16384,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=21.8MiB/s (22.8MB/s), 21.8MiB/s-21.8MiB/s (22.8MB/s-22.8MB/s), io=4096MiB (4295MB), run=188153-188153msec


fio --name=big-file-sequential-write --directory=/local --rw=write --refill_buffers --bs=256k --size=4G
big-file-sequential-write: (g=0): rw=write, bs=(R) 256KiB-256KiB, (W) 256KiB-256KiB, (T) 256KiB-256KiB, ioengine=psync, iodepth=1
fio-3.16
Starting 1 process
big-file-sequential-write: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=0): [f(1)][100.0%][eta 00m:00s]
big-file-sequential-write: (groupid=0, jobs=1): err= 0: pid=1316: Fri Apr 23 15:58:16 2021
  write: IOPS=352, BW=88.2MiB/s (92.4MB/s)(4096MiB/46461msec); 0 zone resets
    clat (usec): min=162, max=198027, avg=2725.46, stdev=6187.41
     lat (usec): min=162, max=198027, avg=2726.17, stdev=6187.40
    clat percentiles (usec):
     |  1.00th=[  188],  5.00th=[  219], 10.00th=[  231], 20.00th=[  269],
     | 30.00th=[  363], 40.00th=[  404], 50.00th=[  437], 60.00th=[  469],
     | 70.00th=[  537], 80.00th=[ 6783], 90.00th=[ 7898], 95.00th=[ 9110],
     | 99.00th=[27657], 99.50th=[36439], 99.90th=[68682], 99.95th=[70779],
     | 99.99th=[74974]
   bw (  KiB/s): min=27136, max=604672, per=100.00%, avg=90281.86, stdev=65724.29, samples=92
   iops        : min=  106, max= 2362, avg=352.59, stdev=256.71, samples=92
  lat (usec)   : 250=16.79%, 500=48.97%, 750=10.08%, 1000=0.76%
  lat (msec)   : 2=0.31%, 4=0.10%, 10=18.44%, 20=2.42%, 50=1.76%
  lat (msec)   : 100=0.35%, 250=0.01%
  cpu          : usr=3.89%, sys=13.29%, ctx=5485, majf=0, minf=28
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,16384,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=88.2MiB/s (92.4MB/s), 88.2MiB/s-88.2MiB/s (92.4MB/s-92.4MB/s), io=4096MiB (4295MB), run=46461-46461msec
```

## 结果对比

| 存储类型   | Write            | Read    |
|------------|------------------|---------|
| Local      | 13152 points/sec | 12.37ms |
| Minio S3FS | 4084 points/sec  | 11.99ms |

在单机环境下，分别使用本地盘和 minio 的 S3FS 作为存储后端。

在写数据的时候性能贴别多，本地盘 (13152 points/sec) 与 S3FS (4084 points/sec) 相比，S3FS 慢 4 倍左右。

读数据基本没有区别，感觉像是直接从内存中读的数据。
