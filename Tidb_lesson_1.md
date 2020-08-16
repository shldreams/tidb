# Tidb 第一课

TiDB 的安装有多种方式,既可以通过源码编译生成二进制文件,也可以通过 tiup 直接进行安装,在通过源码编译时 tikv 需要提前安装好 **cmake** 和 **cargo** 这样才能正常的编译 tikv,否则会失败,tidb,pd 直接在对应目录下 **make** 即可编译.

在编译完成后我们先启动 pd 和 tikv:

启动 pd:

```shell
bin/pd-server --name=pd-10.101.1.115-3379 --client-urls=http://0.0.0.0:3379 --advertise-client-urls=http://10.101.1.115:3379 --peer-urls=http://10.101.1.115:3380 --advertise-peer-urls=http://10.101.1.115:3380 --data-dir=/data2/pd-3379/pd_data --initial-cluster=pd-10.101.1.115-3379=http://10.101.1.115:3380,pd-10.101.1.118-3379=http://10.101.1.118:3380,pd-10.101.1.116-3379=http://10.101.1.116:3380 --config=conf/pd.toml --log-file=/data2/pd-3379/log/pd.log
bin/pd-server --name=pd-10.101.1.118-3379 --client-urls=http://0.0.0.0:3379 --advertise-client-urls=http://10.101.1.118:3379 --peer-urls=http://10.101.1.118:3380 --advertise-peer-urls=http://10.101.1.118:3380 --data-dir=/data2/pd-3379/pd_data --initial-cluster=pd-10.101.1.115-3379=http://10.101.1.115:3380,pd-10.101.1.118-3379=http://10.101.1.118:3380,pd-10.101.1.116-3379=http://10.101.1.116:3380 --config=conf/pd.toml --log-file=/data2/pd-3379/log/pd.log
bin/pd-server --name=pd-10.101.1.116-3379 --client-urls=http://0.0.0.0:3379 --advertise-client-urls=http://10.101.1.116:3379 --peer-urls=http://10.101.1.116:3380 --advertise-peer-urls=http://10.101.1.116:3380 --data-dir=/data2/pd-3379/pd_data --initial-cluster=pd-10.101.1.115-3379=http://10.101.1.115:3380,pd-10.101.1.118-3379=http://10.101.1.118:3380,pd-10.101.1.116-3379=http://10.101.1.116:3380 --config=conf/pd.toml --log-file=/data2/pd-3379/log/pd.log
```

启动 tikv:

```shell
bin/tikv-server --addr 0.0.0.0:20161 --advertise-addr 10.101.1.114:20161 --status-addr 10.101.1.114:20181 --pd 10.101.1.115:3379,10.101.1.118:3379,10.101.1.116:3379 --data-dir /data2/tidb_20161/data --config conf/tikv.toml --log-file /data2/tidb_20161/log/tikv.log
bin/tikv-server --addr 0.0.0.0:20162 --advertise-addr 10.101.1.114:20162 --status-addr 10.101.1.114:20182 --pd 10.101.1.115:3379,10.101.1.118:3379,10.101.1.116:3379 --data-dir /data3/tidb_20162/data --config conf/tikv.toml --log-file /data3/tidb_20162/log/tikv.log
bin/tikv-server --addr 0.0.0.0:20163 --advertise-addr 10.101.1.114:20163 --status-addr 10.101.1.114:20183 --pd 10.101.1.115:3379,10.101.1.118:3379,10.101.1.116:3379 --data-dir /data4/tidb_20163/data --config conf/tikv.toml --log-file /data4/tidb_20163/log/tikv.log
```

启动 tidb:

```shell
./tidb-server -L info -P 4000 --status=10080 --host=0.0.0.0 --advertise-address=10.13.50.14 --store=tikv --path=10.101.1.115:3379,10.101.1.118:3379,10.101.1.116:3379 --log-slow-query=/tmp/tidb_slow_query.log --log-file=/tmp/tidb.log
```

根据以往对事物理解,数据库事务以 **BEGIN** 或者 **START TRANSACTION** 为起始,我们在 tidb 代码中进行遍历可以直接追溯到:

https://github.com/shldreams/tidb/blob/9bdc5cedaf8723f4de188e402af37af36f1d94d1/kv/kv.go#L383

```go
type Storage interface {
	// Begin transaction
	Begin() (Transaction, error)
	// BeginWithStartTS begins transaction with startTS.
	BeginWithStartTS(startTS uint64) (Transaction, error)
	// GetSnapshot gets a snapshot that is able to read any data which data is <= ver.
```

但是当我们查看其定义,并在其下面添加日志输出标记为是事务起点时,log 中午任何反应.这样根据以往直接经验来判断事务起点就不是很成功.

那就换个方向,一般事务由用户发起,用户发起连接,创建 session 后 sql 经过解析后开始执行事务,那这样我们就可以大体的知道事务开始的文件是位于 session 下面,在在大体的阅读了 session.go 的代码后发现,所有在执行事务时会去调用 store.Begin() 即我们上面定位到的,事务开始地方,我们在下面添加 logutil.Logger(ctx).Info("hello transaction") 后进行测试

```go
func (s *session) NewTxn(ctx context.Context) error {
	if s.txn.Valid() {
		txnID := s.txn.StartTS()
		err := s.CommitTxn(ctx)
		if err != nil {
			return err
		}
		vars := s.GetSessionVars()
		logutil.Logger(ctx).Info("NewTxn() inside a transaction auto commit",
			zap.Int64("schemaVersion", vars.TxnCtx.SchemaVersion),
			zap.Uint64("txnStartTS", txnID))
	}

	txn, err := s.store.Begin()
	logutil.Logger(ctx).Info("hello transaction")
	if err != nil {
		return err
```

发现只有在 DDL 操作的时候才会触发输出 "hello transaction":

```
[2020/08/16 13:50:03.934 +08:00] [INFO] [session.go:2131] ["CRUCIAL OPERATION"] [conn=1] [schemaVersion=1417] [cur_db=] [sql="create database test"] [user=root@127.0.0.1]
[2020/08/16 13:50:03.964 +08:00] [INFO] [session.go:1421] ["hello transaction"] [conn=1]
[2020/08/16 13:50:04.137 +08:00] [INFO] [ddl_worker.go:261] ["[ddl] add DDL jobs"] ["batch count"=1] [jobs="ID:1730, Type:create schema, State:none, SchemaState:none, SchemaID:1729, TableID:0, RowCount:0, ArgLen:1, start time: 2020-08-16 13:50:03.973 +0800 CST, Err:<nil>, ErrCount:0, SnapshotVersion:0; "]
[2020/08/16 13:50:04.137 +08:00] [INFO] [ddl.go:477] ["[ddl] start DDL job"] [job="ID:1730, Type:create schema, State:none, SchemaState:none, SchemaID:1729, TableID:0, RowCount:0, ArgLen:1, start time: 2020-08-16 13:50:03.973 +0800 CST, Err:<nil>, ErrCount:0, SnapshotVersion:0"] [query="create database test"]
```

这说明我们只是找对了一部分,说明 tidb 的 DDL 事务和 DML 事务是分开定义的.

继续想下阅读代码,我们会发现在 session/txn.go 下面有定义的 wait,在获取了 startTS 后就开始执行事务并没有调用 store.Begin,而是在失败后才回去调用 store.Begin(),那我们同样尝试在下面添加:logutil.BgLogger().Info("hello transaction")

```go
func (tf *txnFuture) wait() (kv.Transaction, error) {
	startTS, err := tf.future.Wait()
	if err == nil {
		logutil.BgLogger().Info("hello transaction")
		return tf.store.BeginWithStartTS(startTS)
	} else if config.GetGlobalConfig().Store == "mocktikv" {
		return nil, err
	}
	logutil.BgLogger().Warn("wait tso failed", zap.Error(err))
	// It would retry get timestamp.
	return tf.store.Begin()
```

最终在做 DML 操作时可以正常的打印"hello transaction",

```
[2020/08/16 13:58:29.047 +08:00] [INFO] [txn.go:499] ["hello transaction"]
[2020/08/16 13:58:29.059 +08:00] [INFO] [txn.go:499] ["hello transaction"]
[2020/08/16 13:58:30.024 +08:00] [INFO] [txn.go:499] ["hello transaction"]
[2020/08/16 13:58:31.242 +08:00] [INFO] [txn.go:499] ["hello transaction"]
[2020/08/16 13:58:31.799 +08:00] [INFO] [txn.go:499] ["hello transaction"]
[2020/08/16 13:58:31.845 +08:00] [INFO] [txn.go:499] ["hello transaction"]
[2020/08/16 13:58:31.894 +08:00] [INFO] [txn.go:499] ["hello transaction"]
```

而且 tidb 本身也会经常性的获取 startTS 来进行一些事务,具体进行的是什么操作还有为什么 DDL 和 DML 会有不同的设计,这个需要更加详细的通读代码,后续进行.