---
title: Proxy UDP server to websocket with Golang fiber framework
typora-root-url: ../../source
date: 2023-01-24 15:40:17
tags: [golang, websocket, fiber]
---



## Problem

Proxy UDP server to Websocket server with Golang.



## Solution

### 1) UDP Server

First let's make a UDP server. It is a simple server that just echoes things back.

```go
package main

import (
	"log"
	"net"
)

func main() {
	addr := "127.0.0.1:1053"
	udpServer, err := net.ListenPacket("udp", addr)
	log.Println("Server is running at", addr)
	if err != nil {
		log.Fatal(err)
	}
	defer udpServer.Close()

	for {
		buf := make([]byte, 1024)
		n, addr, err := udpServer.ReadFrom(buf)
		if err != nil {
			continue
		}
		go udpServer.WriteTo(buf[:n], addr)
	}
}
```

We can run this server: `go run main.go`, it will listen on "127.0.0.1:1053"



### 2) Websocket proxy server

Note: Here we use the [go fiber](https://gofiber.io) framework, but you don't have to. With Golang's standard `net` package, you still can achieve this.

```bash
go get github.com/gofiber/fiber/v2
```

Now with the code:

```go
package main

// this is to test if fiber proxy middleware can correctly proxy websocket

import (
	"flag"
	"log"
	"net"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/logger"
	"github.com/gofiber/websocket/v2"
)

const (
	localKeyBackendURL = "localKeyBackendURL"
)

func main() {
	addrPtr := flag.String("backend", "", "backend addr")
	flag.Parse()
	if addrPtr == nil || *addrPtr == "" {
		log.Fatalln("Missing backend parameter. Use -h to help")
	}
	log.Println("Proxy to:", *addrPtr)

	app := fiber.New(fiber.Config{
		Immutable: true,
	})
	app.Use(logger.New())
	app.Get("/ws/*", wsCheckMiddleware(*addrPtr), websocket.New(wsHandler))
	app.Listen(":10001")
}

func wsCheckMiddleware(backendURL string) fiber.Handler {
	return func(c *fiber.Ctx) error {
		if !websocket.IsWebSocketUpgrade(c) {
			return fiber.ErrUpgradeRequired
		}
		c.Locals(localKeyBackendURL, backendURL)
		return c.Next()
	}
}

func wsHandler(c *websocket.Conn) {
	url := c.Locals(localKeyBackendURL).(string)
	defer c.Close()

	udpServer, err := net.ResolveUDPAddr("udp", url)
	if err != nil {
		log.Fatalln(err)
	}
	udpConn, err := net.DialUDP("udp", nil, udpServer)
	if err != nil {
		log.Fatalln(err)
	}
	defer udpConn.Close()

	clientErrChan := make(chan error, 1)
	backendErrChan := make(chan error, 1)

	go forwardWS2UDP(c, udpConn, clientErrChan)
	go forwardUDP2WS(udpConn, c, backendErrChan)

	var msg string

	select {
	case err = <-clientErrChan:
		msg = "forward client to backend server error"
	case err = <-backendErrChan:
		msg = "forward backend to client server error"
	}

	if websocket.IsUnexpectedCloseError(
		err,
		websocket.CloseGoingAway,
		websocket.CloseNoStatusReceived) {
		log.Println(msg, "error:", err)
	}
}

func forwardWS2UDP(
	wsConn *websocket.Conn,
	udpConn *net.UDPConn,
	errChan chan error,
) {
	for {
		_, msg, err := wsConn.ReadMessage()
		if err != nil {
			errChan <- err
			break
		}

		_, err = udpConn.Write(msg)
		if err != nil {
			errChan <- err
			break
		}
	}
}

func forwardUDP2WS(
	udpConn *net.UDPConn,
	wsConn *websocket.Conn,
	errChan chan error,
) {
	for {
		buf := make([]byte, 1024)
		n, err := udpConn.Read(buf)
		if err != nil {
			errChan <- err
			break
		}

		err = wsConn.WriteMessage(websocket.TextMessage, buf[:n])
		if err != nil {
			errChan <- err
			break
		}
	}
}
```

Our websocket server is running at port 10001. Run it:

```bash
go run main.go -backend=127.0.0.1:1053
```



Now let's test it with `wscat`:

```bash
❯ wscat --connect ws://127.0.0.1:10001/ws
Connected (press CTRL+C to quit)
> hello hello how are you
< hello hello how are you
> can you read this
< can you read this
> echo
< echo
```

Please notice we are setting the websocket message type to be "TextMessage". If the data from backend UDP server is binary, then you need to change the type to "BinaryMessage".

## Reference:

* https://www.golinuxcloud.com/golang-udp-server-client/
