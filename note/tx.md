# tx Transaction

bbolt 的事务是串行执行。
db 对象有一个 rwlock 表示每次只会有一个写事务存在。

在创建事务的时候，会给数据库对象加锁 db.go#L632,之后在tx对象初始化的时候，tx 对象会持有 db 对象。在事务关闭的时候，会释放锁。tx.go#L303。

事务开始的时候会创建一个空的 Bucket 对象。事务在创建或者获取打开一个 Bucket 的时候，会用这个 Bucket 对象去打开。

读取不到另一个事务写的数据的是因为每次 tx cursor 读取数据都是从持久化的文件中直接读取新的 page 生成 node.

### 隔离级别的测试代码
```go
package main

import (
	"fmt"
	"log"

	bolt "go.etcd.io/bbolt"
)

func main() {
	// Open the my.db data file in your current directory.
	// It will be created if it doesn't exist.
	db, err := bolt.Open("my.db", 0600, nil)
	if err != nil {
		log.Fatal(err)
	}

	bucketName := []byte("b1")
	key := []byte("key")

	// init bucket
	{
		wTx, err := db.Begin(true)
		if err != nil {
			log.Fatal(err)
		}
		bucket, err := wTx.CreateBucketIfNotExists(bucketName)
		if err != nil {
			log.Fatal(err)
		}
		if err := bucket.Put(key, []byte("init")); err != nil {
			log.Fatal(err)
		}
		if err := wTx.Commit(); err != nil {
			log.Fatal(err)
		}
	}

	wTx, err := db.Begin(true)
	if err != nil {
		log.Fatal(err)
	}

	{
		bucket := wTx.Bucket(bucketName)
		if err != nil {
			log.Fatal(err)
		}
		if err := bucket.Put(key, []byte("change")); err != nil {
			log.Fatal(err)
		}
	}

	rTx, err := db.Begin(false)
	if err != nil {
		log.Fatal(err)
	}

	{
		bucket := rTx.Bucket(bucketName)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println("read transaction get", string(bucket.Get(key)))
	}

	if err := wTx.Commit(); err != nil {
		log.Fatal(err)
	}
	{
		rTx2, err := db.Begin(false)
		if err != nil {
			log.Fatal(err)
		}
		bucket := rTx2.Bucket(bucketName)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println("read transaction after commit get", string(bucket.Get(key)))
	}
}
```


# bucket

tx.root 的那个 bucket 是 metadata 里面的 bucket
