# gmmpool

A multi level memory pool for Golang:


![](https://ws1.sinaimg.cn/large/44cd29dagy1fociejthjoj20n40ckgm0.jpg)

The design thoughts are detailed in [golang multi-level memory pool design and implemention](https://liudanking.com/arch/golang-multi-level-memory-pool-design-implementation/)(in Chinese🇨🇳).

## Installation

`go get github.com/liudanking/gmmpool`

## Usage


```go
package main

import (
	"bytes"
	"log"

	"github.com/liudanking/gmmpool"
)

func main() {
	pool := gmmpool.NewMultiLevelPool([]gmmpool.PoolOpt{
		gmmpool.PoolOpt{Num: 10, Size: 1024},     // level 0
		gmmpool.PoolOpt{Num: 10, Size: 1024 * 2}, // level 1
	})

	buf := pool.Get(1025)
	defer pool.Put(buf)
	data, err := buf.ReadAll(bytes.NewReader(make([]byte, 8)))
	if err != nil {
		log.Fatal(err)
	}
	log.Print(data)
}


```

## Benchmark (compared with ioutil.ReadAll, x19 speed up)

```
BenchmarkStdReadAll-4             200000          5969 ns/op
BenchmarkMultiLevelPool-4        5000000           311 ns/op
```


## Credit

[goim](https://github.com/Terry-Mao/goim/)

### 疑问：
预分配大Node slice，Node里有pre和next，这样搞array内链表。释放时没有做全部解引用，只在Node slice首尾两端置nil。这个搞法会导致你说的这个情况。Node slice在应用层看是独立的单元，在runtime看，就是一个object，object内部的引用，也会计入当前分配内存，这样gc阈值被放大了。
arr=[a,b,c,d] a->b->c->d，这样有三个引用关系，gc那边统计就是 len(arr)*3。 如果是简单的a->b->c->d，gc那边统计的就是3

