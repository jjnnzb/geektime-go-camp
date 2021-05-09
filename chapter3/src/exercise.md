基于 errgroup 实现一个 http server 的启动和关闭 ，以及 linux signal 信号的注册和处理，要保证能够一个退出，全部注销退出。
```go
package main

import (
	"context"
	"errors"
	"fmt"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"golang.org/x/sync/errgroup"
)

func main() {
	g, ctx := errgroup.WithContext(context.Background())

	handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		_, err := fmt.Fprintln(w, "hello")
		if err != nil {
			return 
		}
	})
	serv := http.Server{
		Addr:    ":8080",
		Handler: handler,
	}

	g.Go(func() error {
		fmt.Println("Starting http server...")
		go func() {
			<-ctx.Done()
			fmt.Println("ctx is done...")
			err := serv.Shutdown(ctx)
			if err != nil {
				return 
			}
		}()
		return serv.ListenAndServe()
	})
	
	g.Go(func() error {
		fmt.Println("Expecting error in 10 seconds...")
		time.Sleep(10 * time.Second)
		return errors.New("error occurred")
	})
	
	g.Go(func() error {
		signals := make(chan os.Signal, 1)
		signal.Notify(signals, syscall.SIGINT, syscall.SIGTERM, os.Interrupt)

		select {
		case <-ctx.Done():
			return ctx.Err()
		case <-signals:
			return fmt.Errorf("terminating signal: %v", signals)
		}
	})

	err := g.Wait()
	fmt.Println(err)
}
```