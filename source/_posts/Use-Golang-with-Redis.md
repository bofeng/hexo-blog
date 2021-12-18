---
title: Use Golang with Redis
typora-root-url: ../../source
date: 2021-12-17 23:05:35
tags: [golang, redis]
---



1, Install Package:

```bash
$ go get github.com/go-redis/redis/v8 
```



2, Golang code with Get/Set/Exists example:

```go
package main

import (
	"context"
	"log"
	"time"

	"github.com/go-redis/redis/v8"
)

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	rcli := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       3,  // use default DB
	})
	err := rcli.Ping(ctx).Err()
	if err != nil {
		log.Fatalf("Failed to connect to redis: %v\n", err)
	}

	// set
	err = rcli.Set(ctx, "name", "John Doe", 0).Err()
	if err != nil {
		log.Fatalf("Failed to call Set: %v\n", err)
	}

	// check exists
	exists, err := rcli.Exists(ctx, "name").Result()
	if err != nil {
		log.Printf("Failed to call Exists: %v\n", err)
	} else {
		if exists == 1 {
			log.Println("Key 'name' exists")
		} else {
			log.Println("Key 'name' doesn't exist")
		}
	}

	// get value
	val, err := rcli.Get(ctx, "name").Result()
	if err != nil {
		if err == redis.Nil {
			log.Println("Key 'name' doesn't exist")
		} else {
			log.Printf("Failed to call Get: %v\n", err)
		}
	} else {
		log.Printf("Value: %s\n", val)
	}

	// get list
	list, err := rcli.LRange(ctx, "scores", 0, -1).Result()
	if err != nil {
		log.Printf("Failed to call LRange: %v\n", err)
	} else {
		// list will be an empty list [] if key doesn't exist
		log.Printf("List: %v\n", list)
	}

	// push list
	err = rcli.LPush(ctx, "scores", 42).Err()
	if err != nil {
		log.Printf("Failed to call LPush: %v\n", err)
	}

	// get list again
	list, err = rcli.LRange(ctx, "scores", 0, -1).Result()
	// if the key doesn't exist, it will return an empty list
	if err != nil {
		log.Printf("Failed to call LRange: %v\n", err)
	} else {
		log.Printf("List: %v\n", list)
	}
}
```



Run it:

```bash
$ go run main.go
2021/12/17 23:03:10 Key 'name' exists
2021/12/17 23:03:10 Value: John Doe
2021/12/17 23:03:10 List: []
2021/12/17 23:03:10 List: [42]
```

