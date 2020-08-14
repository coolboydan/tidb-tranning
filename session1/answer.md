



### 作业

>分值：200
>
>题目描述：
>
>本地下载 TiDB，TiKV，PD 源代码，改写源码并编译部署以下环境：
>
>* 1 TiDB
>* 1 PD
>* 3 TiKV 
>  改写后：使得 TiDB 启动事务时，能打印出一个 “hello transaction” 的 日志
>
>输出：一篇文章介绍以上过程
>
>截止时间：本周日 24:00:00（逾期不给分）
>作业提交方式：提交至个人 github ，将链接发送给 talent-plan@tidb.io



### 解题思路

- 源码并且编译产生执行文件。
- 使用精简配置让tidb，pd，tikv跑起来。
- 找出TiDB启动事务代码段添加日志。
- 启动了之后执行sql生成日志。



### 编译源码

#### 一、编译TiKV

##### 1、下载TiKV，使用idea的rust插件进行查看。

```
https://github.com/tikv/tikv.git
```



##### 2、安装rust，得用mac os或者linux不然基本跑不动

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```



##### 3、找到`CONTRIBUTING.md`文件下有编译安装步骤

```
安装组件
rustup component add rustfmt
rustup component add clippy
```



##### 4、编译文件

```
make build
```



##### 6、遇到了错误,缺少cmac

```
Caused by:
  process didn't exit successfully: `/Users/zhengjunbo/work/fcbox_work/tikv/target/debug/build/snappy-sys-860531c249ec2e1a/build-script-build` (exit code: 101)
--- stdout
running: "cmake" "/Users/zhengjunbo/.cargo/git/checkouts/rust-snappy-0ed33e4b7b96fc57/8c12738/snappy-sys/snappy" "-DCMAKE_INSTALL_PREFIX=/Users/zhengjunbo/work/fcbox_work/tikv/target/debug/build/snappy-sys-7294ca1483d9b509/out" "-DCMAKE_C_FLAGS= -ffunction-sections -fdata-sections -fPIC -m64" "-DCMAKE_C_COMPILER=/usr/bin/cc" "-DCMAKE_CXX_FLAGS= -ffunction-sections -fdata-sections -fPIC -m64" "-DCMAKE_CXX_COMPILER=/usr/bin/c++" "-DCMAKE_BUILD_TYPE=Debug"

--- stderr
thread 'main' panicked at '
failed to execute command: No such file or directory (os error 2)
is `cmake` not installed?

build script failed, must exit now', /Users/zhengjunbo/.cargo/registry/src/github.com-1ecc6299db9ec823/cmake-0.1.42/src/lib.rs:861:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

warning: build failed, waiting for other jobs to finish...
error: build failed
make: *** [build] Error 101
```



###### 发现需要以下权限

> To build TiKV you'll need to at least have the following installed:
>
> * `git` - Version control
> * [`rustup`](https://rustup.rs/) - Rust installer and toolchain manager
> * `make` - Build tool (run common workflows)
> * `cmake` - Build tool (required for gRPC)
> * `awk` - Pattern scanning/processing language





###### 检查发现只缺少cmake,macos安装`https://cmake.org/download/`

```
https://github.com/Kitware/CMake/releases/download/v3.18.1/cmake-3.18.1-Darwin-x86_64.dmg
```



###### 将命令加入系统中

```
sudo "/Applications/CMake.app/Contents/bin/cmake-gui" --install=/usr/local/bin
```



###### 再次`make build`执行编译文件得到结果,编译过程还是挺吃机器的。

```
   Compiling encryption v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/encryption)
   Compiling into_other v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/into_other)
   Compiling resolved_ts v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/resolved_ts)
   Compiling tidb_query_datatype v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/tidb_query_datatype)
   Compiling engine_rocks v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/engine_rocks)
   Compiling security v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/security)
   Compiling pd_client v0.1.0 (/Users/zhengjunbo/work/fcbox_work/tikv/components/pd_client)
   Compiling sst_importer v0.1.0 (/Users/zhengjunbo/work/fcbox_work/tikv/components/sst_importer)
   Compiling tidb_query_shared_expr v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/tidb_query_shared_expr)
   Compiling raftstore v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/raftstore)
   Compiling tidb_query_vec_expr v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/tidb_query_vec_expr)
   Compiling tidb_query_normal_expr v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/tidb_query_normal_expr)
   Compiling tidb_query_normal_executors v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/tidb_query_normal_executors)
   Compiling tidb_query_vec_aggr v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/tidb_query_vec_aggr)
   Compiling tidb_query_vec_executors v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/tidb_query_vec_executors)
   Compiling tikv v4.1.0-alpha (/Users/zhengjunbo/work/fcbox_work/tikv)
   Compiling backup v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/backup)
   Compiling cdc v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/components/cdc)
   Compiling cmd v0.0.1 (/Users/zhengjunbo/work/fcbox_work/tikv/cmd)
    Finished dev [unoptimized + debuginfo] target(s) in 7m 23s
```



#### 二、编译TiDB

##### 1、下载tidb的git到本地

```
https://github.com/pingcap/tidb.git
```



##### 2、找到文档中的构建文件查找到构建方法

```
tidb/docs/QUICKSTART.md
```



##### 3、安装go语言（略）



##### 4、编译源码

###### 使用代理

```
export GOPROXY=https://mirrors.aliyun.com/goproxy/
```

###### 执行编译程序

```
make
```

###### 完成之后可以在找到执行文件

```
bin/tidb-server
```



#### 三、编译PD

##### 1、下载PD到本地

```
https://github.com/pingcap/pd.git
```

##### 2、到文档中查找到构建的方法

```
docs/development.md
```

##### 3、编译源码

###### 使用代理

```
export GOPROXY=https://mirrors.aliyun.com/goproxy/
```

###### 执行编译程序

```
make
```

###### 完成之后可以在找到执行文件

```
bin/pd-recover
bin/pd-ctl
bin/pd-server
```



### 组合配置让系统运行起来



#### 一、启动顺序

启动是否有顺序

1. 先启动一个PD

2. 再启动三个TiKV

3. 最后启动TiDB



#### 二、启动过程

##### 1、直接启动PD,

```
./pd-server
```



##### 2、启动TiKV

###### 直接启动TiKV的时候发现报错

```
[2020/08/14 10:48:18.600 +08:00] [FATAL] [server.rs:919] ["the maximum number of open file descriptors is too small, got 10240, expect greater or equal to 82920"]

```

###### 调整系统的句柄数

```
launchctl limit maxfiles 99999 99999
ulimit -n 99999
```



###### 启动第一台TiKV

```
./tikv-server --addr 127.0.0.1:10927 --status-addr 127.0.0.1:20181  --labels host="tikv1" 
```

###### 启动第二台TiKV

```
./tikv-server --addr 127.0.0.1:10928 --status-addr 127.0.0.1:20182  --labels host="tikv2" --data-dir=db2
```

###### 启动第三台TiKV

```
./tikv-server --addr 127.0.0.1:10929 --status-addr 127.0.0.1:20183  --labels host="tikv3" --data-dir=db3
```



###### 启动完成之后发现TiKV已经加入到系统中了

```
[2020/08/14 11:08:20.052 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=split-check]
[2020/08/14 11:08:20.054 +08:00] [INFO] [node.rs:173] ["put store to PD"] [store="id: 7 address: \"127.0.0.1:10929\" labels { key: \"host\" value: \"tikv3\" } version: \"4.1.0-alpha\" status_address: \"127.0.0.1:20183\" git_hash: \"df2fab45b19b5d067ff55ff5074819c50eef3ea2\" start_timestamp: 1597374500 deploy_path: \"/Users/zhengjunbo/work/fcbox_work/tikv/target/debug\""]
[2020/08/14 11:08:20.055 +08:00] [INFO] [node.rs:239] ["initializing replication mode"] [store_id=7] [status=Some()]
[2020/08/14 11:08:20.056 +08:00] [INFO] [replication_mode.rs:51] ["associated store labels"] [labels="[key: \"host\" value: \"tikv3\"]"] [store_id=7]
[2020/08/14 11:08:20.056 +08:00] [INFO] [replication_mode.rs:51] ["associated store labels"] [labels="[key: \"host\" value: \"tikv1\"]"] [store_id=1]
[2020/08/14 11:08:20.056 +08:00] [INFO] [replication_mode.rs:51] ["associated store labels"] [labels="[key: \"host\" value: \"tikv2\"]"] [store_id=5]
[2020/08/14 11:08:20.056 +08:00] [INFO] [node.rs:386] ["start raft store thread"] [store_id=7]
[2020/08/14 11:08:20.056 +08:00] [INFO] [store.rs:942] ["start store"] [takes=109.632µs] [merge_count=0] [applying_count=0] [tombstone_count=0] [region_count=0] [store_id=7]
[2020/08/14 11:08:20.057 +08:00] [INFO] [store.rs:993] ["cleans up garbage data"] [takes=176.864µs] [garbage_range_count=1] [store_id=7]
[2020/08/14 11:08:20.057 +08:00] [INFO] [snap.rs:1132] ["Initializing SnapManager, encryption is enabled: false"]
[2020/08/14 11:08:20.059 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=snapshot-worker]
[2020/08/14 11:08:20.059 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=raft-gc-worker]
[2020/08/14 11:08:20.059 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=cleanup-worker]
[2020/08/14 11:08:20.059 +08:00] [INFO] [future.rs:138] ["starting working thread"] [worker=pd-worker]
[2020/08/14 11:08:20.060 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=consistency-check]
[2020/08/14 11:08:20.060 +08:00] [INFO] [compaction_filter.rs:66] ["initialize GC context for compaction filter"]
[2020/08/14 11:08:20.065 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=cdc]
[2020/08/14 11:08:20.068 +08:00] [INFO] [future.rs:138] ["starting working thread"] [worker=waiter-manager]
[2020/08/14 11:08:20.068 +08:00] [INFO] [future.rs:138] ["starting working thread"] [worker=deadlock-detector]
[2020/08/14 11:08:20.069 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=backup-endpoint]
[2020/08/14 11:08:20.071 +08:00] [INFO] [mod.rs:334] ["starting working thread"] [worker=snap-handler]
[2020/08/14 11:08:20.071 +08:00] [INFO] [server.rs:219] ["listening on addr"] [addr=127.0.0.1:10929]
[2020/08/14 11:08:20.081 +08:00] [INFO] [server.rs:244] ["TiKV is ready to serve"]
[2020/08/14 11:08:30.063 +08:00] [INFO] [client.rs:519] ["set cluster version to 4.1.0-alpha"]
[2020/08/14 11:08:30.129 +08:00] [INFO] [kv.rs:615] ["batch_raft RPC is called, new gRPC stream established"]
[2020/08/14 11:08:30.130 +08:00] [INFO] [peer.rs:219] ["replicate peer"] [peer_id=8] [region_id=2]
[2020/08/14 11:08:30.130 +08:00] [INFO] [raft.rs:894] ["became follower at term 0"] [term=0] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.130 +08:00] [INFO] [raft.rs:289] [newRaft] [peers="[]"] ["last term"=0] ["last index"=0] [applied=0] [commit=0] [term=0] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.131 +08:00] [INFO] [raw_node.rs:223] ["RawNode created with id 8."] [id=8] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.131 +08:00] [INFO] [raft.rs:1114] ["received a message with higher term from 3"] ["msg type"=MsgHeartbeat] [message_term=8] [term=0] [from=3] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.131 +08:00] [INFO] [raft.rs:894] ["became follower at term 8"] [term=8] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.133 +08:00] [INFO] [transport.rs:145] ["resolve store address ok"] [addr=127.0.0.1:10927] [store_id=1]
[2020/08/14 11:08:30.133 +08:00] [INFO] [raft_client.rs:49] ["server: new connection with tikv endpoint"] [addr=127.0.0.1:10927]
[2020/08/14 11:08:30.134 +08:00] [INFO] [<unknown>] ["New connected subchannel at 0x7f9ee814fd70 for subchannel 0x7f9ee814e010"]
[2020/08/14 11:08:30.155 +08:00] [INFO] [snap.rs:224] ["saving snapshot file"] [file=/Users/zhengjunbo/work/fcbox_work/tikv/target/debug/db3/snap/rev_2_8_11_(default|lock|write).sst] [snap_key=2_8_11]
[2020/08/14 11:08:30.169 +08:00] [INFO] [raft.rs:2079] ["[commit: 0, lastindex: 0, lastterm: 0] starts to restore snapshot [index: 11, term: 8]"] [snapshot_term=8] [snapshot_index=11] [last_term=0] [last_index=0] [commit=0] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.169 +08:00] [INFO] [raft_log.rs:487] ["log [committed=0, applied=0, unstable.offset=1, unstable.entries.len()=0] starts to restore snapshot [index: 11, term: 8]"] [snapshot_term=8] [snapshot_index=11] [log="committed=0, applied=0, unstable.offset=1, unstable.entries.len()=0"] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.169 +08:00] [INFO] [raft.rs:2015] ["[commit: 11, term: 8] restored snapshot [index: 11, term: 8]"] [snapshot_term=8] [snapshot_index=11] [commit=11] [term=8] [raft_id=8] [region_id=2]
[2020/08/14 11:08:30.169 +08:00] [INFO] [peer_storage.rs:1079] ["begin to apply snapshot"] [peer_id=8] [region_id=2]
[2020/08/14 11:08:30.169 +08:00] [INFO] [peer_storage.rs:1122] ["apply snapshot with state ok"] [state="applied_index: 11 truncated_state { index: 11 term: 8 }"] [region="id: 2 region_epoch { conf_ver: 4 version: 1 } peers { id: 3 store_id: 1 } peers { id: 6 store_id: 5 } peers { id: 8 store_id: 7 is_learner: true }"] [peer_id=8] [region_id=2]
[2020/08/14 11:08:30.170 +08:00] [INFO] [peer.rs:2766] ["snapshot is applied"] [region="id: 2 region_epoch { conf_ver: 4 version: 1 } peers { id: 3 store_id: 1 } peers { id: 6 store_id: 5 } peers { id: 8 store_id: 7 is_learner: true }"] [peer_id=8] [region_id=2]
[2020/08/14 11:08:30.171 +08:00] [INFO] [region.rs:300] ["begin apply snap data"] [region_id=2]
[2020/08/14 11:08:30.172 +08:00] [INFO] [region.rs:368] ["apply new data"] [time_takes=717.784µs] [region_id=2]
[2020/08/14 11:08:30.882 +08:00] [INFO] [apply.rs:1261] ["execute admin command"] [command="cmd_type: ChangePeer change_peer { peer { id: 8 store_id: 7 } }"] [index=12] [term=8] [peer_id=8] [region_id=2]
[2020/08/14 11:08:30.882 +08:00] [INFO] [apply.rs:1593] ["exec ConfChange"] [epoch="conf_ver: 4 version: 1"] [type=AddNode] [peer_id=8] [region_id=2]
[2020/08/14 11:08:30.882 +08:00] [INFO] [apply.rs:1648] ["add peer successfully"] [region="id: 2 region_epoch { conf_ver: 4 version: 1 } peers { id: 3 store_id: 1 } peers { id: 6 store_id: 5 } peers { id: 8 store_id: 7 is_learner: true }"] [peer="id: 8 store_id: 7"] [peer_id=8] [region_id=2]

```



##### 3、启动TiDB

###### 启动的过程中必须直接配置path到pd地址中才能正常连上已经配置好的系统。

```
./tidb-server --advertise-address=10.204.246.196  --store=tikv --path=127.0.0.1:2379
```

###### 进入TiDB

```
mysql -P 4000 -u root
```



##### 4、检查集群

###### 使用PD-CTL检查

使用进入交互模式

```
./pd-ctl -i 
```

```
» store
{
  "count": 3,
  "stores": [
    {
      "store": {
        "id": 1,
        "address": "127.0.0.1:10927",
        "labels": [
          {
            "key": "host",
            "value": "tikv1"
          }
        ],
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20181",
        "git_hash": "df2fab45b19b5d067ff55ff5074819c50eef3ea2",
        "start_timestamp": 1597374262,
        "deploy_path": "/Users/zhengjunbo/work/fcbox_work/tikv/target/debug",
        "last_heartbeat": 1597377193093351000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "465.6GiB",
        "available": "389.4GiB",
        "used_size": "29.85MiB",
        "leader_count": 13,
        "leader_weight": 1,
        "leader_score": 13,
        "leader_size": 13,
        "region_count": 21,
        "region_weight": 1,
        "region_score": 21,
        "region_size": 21,
        "start_ts": "2020-08-14T11:04:22+08:00",
        "last_heartbeat_ts": "2020-08-14T11:53:13.093351+08:00",
        "uptime": "48m51.093351s"
      }
    },
    {
      "store": {
        "id": 5,
        "address": "127.0.0.1:10928",
        "labels": [
          {
            "key": "host",
            "value": "tikv2"
          }
        ],
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20182",
        "git_hash": "df2fab45b19b5d067ff55ff5074819c50eef3ea2",
        "start_timestamp": 1597374458,
        "deploy_path": "/Users/zhengjunbo/work/fcbox_work/tikv/target/debug",
        "last_heartbeat": 1597377189638921000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "465.6GiB",
        "available": "389.4GiB",
        "used_size": "29.84MiB",
        "leader_count": 4,
        "leader_weight": 1,
        "leader_score": 4,
        "leader_size": 4,
        "region_count": 21,
        "region_weight": 1,
        "region_score": 21,
        "region_size": 21,
        "start_ts": "2020-08-14T11:07:38+08:00",
        "last_heartbeat_ts": "2020-08-14T11:53:09.638921+08:00",
        "uptime": "45m31.638921s"
      }
    },
    {
      "store": {
        "id": 7,
        "address": "127.0.0.1:10929",
        "labels": [
          {
            "key": "host",
            "value": "tikv3"
          }
        ],
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20183",
        "git_hash": "df2fab45b19b5d067ff55ff5074819c50eef3ea2",
        "start_timestamp": 1597374500,
        "deploy_path": "/Users/zhengjunbo/work/fcbox_work/tikv/target/debug",
        "last_heartbeat": 1597377190908202000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "465.6GiB",
        "available": "389.4GiB",
        "used_size": "29.84MiB",
        "leader_count": 4,
        "leader_weight": 1,
        "leader_score": 4,
        "leader_size": 4,
        "region_count": 21,
        "region_weight": 1,
        "region_score": 21,
        "region_size": 21,
        "start_ts": "2020-08-14T11:08:20+08:00",
        "last_heartbeat_ts": "2020-08-14T11:53:10.908202+08:00",
        "uptime": "44m50.908202s"
      }
    }
  ]
}
```



###### 使用sql检查

检查系统版本是正确的

```
zhengjunbo@zhengjunbodeMacBook-Pro ~ % mysql -h 10.204.246.196 -P 4000 -u root
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.7.25-TiDB-v4.0.0-beta.2-955-g030eab2bc TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE information_schema;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT * FROM cluster_info;
+------+---------------------+----------------------+--------------+------------------------------------------+---------------------------+-----------------+
| TYPE | INSTANCE            | STATUS_ADDRESS       | VERSION      | GIT_HASH                                 | START_TIME                | UPTIME          |
+------+---------------------+----------------------+--------------+------------------------------------------+---------------------------+-----------------+
| tidb | 10.204.246.196:4000 | 10.204.246.196:10080 | 4.0.0-beta.2 | 030eab2bca2980245af8386d622b8852273bee7e | 2020-08-14T12:25:46+08:00 | 35.008767s      |
| pd   | 127.0.0.1:2379      | 127.0.0.1:2379       | 4.1.0-alpha  | 68d2ba6b0e82f17231f5af8700b822a5f7eb4c98 | 2020-08-14T10:46:04+08:00 | 1h40m17.008771s |
| tikv | 127.0.0.1:10927     | 127.0.0.1:20181      | 4.1.0-alpha  | df2fab45b19b5d067ff55ff5074819c50eef3ea2 | 2020-08-14T11:04:22+08:00 | 1h21m59.008772s |
| tikv | 127.0.0.1:10928     | 127.0.0.1:20182      | 4.1.0-alpha  | df2fab45b19b5d067ff55ff5074819c50eef3ea2 | 2020-08-14T11:07:38+08:00 | 1h18m43.008773s |
| tikv | 127.0.0.1:10929     | 127.0.0.1:20183      | 4.1.0-alpha  | df2fab45b19b5d067ff55ff5074819c50eef3ea2 | 2020-08-14T11:08:20+08:00 | 1h18m1.008775s  |
+------+---------------------+----------------------+--------------+------------------------------------------+---------------------------+-----------------+
5 rows in set (0.00 sec)
```





###  找出对应代码打上日志并且执行语句



##### 1、TiDB项目中的事务开启在`kv/kv.go:427`中找到begin的相关实现

```go
type Storage interface {
	// Begin transaction
	Begin() (Transaction, error)
	// BeginWithStartTS begins transaction with startTS.
	BeginWithStartTS(startTS uint64) (Transaction, error)
	// GetSnapshot gets a snapshot that is able to read any data which data is <= ver.
	// if ver is MaxVersion or > current max committed version, we will use current version for this snapshot.
	GetSnapshot(ver Version) (Snapshot, error)
	// GetClient gets a client instance.
	GetClient() Client
	// Close store
	Close() error
	// UUID return a unique ID which represents a Storage.
	UUID() string
	// CurrentVersion returns current max committed version.
	CurrentVersion() (Version, error)
	// GetOracle gets a timestamp oracle client.
	GetOracle() oracle.Oracle
	// SupportDeleteRange gets the storage support delete range or not.
	SupportDeleteRange() (supported bool)
	// Name gets the name of the storage engine
	Name() string
	// Describe returns of brief introduction of the storage
	Describe() string
	// ShowStatus returns the specified status of the storage
	ShowStatus(ctx context.Context, key string) (interface{}, error)
}
```



##### 2、在BeginWithStartTS中添加日志

```go
func (s *tikvStore) BeginWithStartTS(startTS uint64) (kv.Transaction, error) {
	logutil.BgLogger().Info("hello transaction")
	txn, err := newTikvTxnWithStartTS(s, startTS, s.nextReplicaReadSeed())
	if err != nil {
		return nil, errors.Trace(err)
	}
	return txn, nil
}
```



##### 3、启动成功之后就疯狂打日志了

```
[2020/08/14 14:00:58.211 +08:00] [INFO] [domain.go:146] ["full load InfoSchema success"] [usedSchemaVersion=0] [neededSchemaVersion=23] ["start time"=10.287221ms]
[2020/08/14 14:00:58.211 +08:00] [INFO] [domain.go:377] ["full load and reset schema validator"]
[2020/08/14 14:00:58.212 +08:00] [INFO] [kv.go:291] ["hello transaction"]
[2020/08/14 14:00:58.229 +08:00] [INFO] [kv.go:291] ["hello transaction"]
[2020/08/14 14:00:58.234 +08:00] [INFO] [kv.go:291] ["hello transaction"]
[2020/08/14 14:00:58.239 +08:00] [INFO] [kv.go:291] ["hello transaction"]
```

