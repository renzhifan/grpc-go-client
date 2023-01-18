# grpc-go-client

## 运行
>首选确定你本地已经具备了运行go grpc服务的环境

```go
 go run client.go
 ```
 
 ## 拦截器(非全局，只是指定单个方法)

 ```go
 package main

import (
	"context"
	"flag"
	"log"
	"time"

	pb "grpc-go-client/pb"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

/*type PerRPCCredentials interface {
	// GetRequestMetadata gets the current request metadata, refreshing tokens
	// if required. This should be called by the transport layer on each
	// request, and the data should be populated in headers or other
	// context. If a status code is returned, it will be used as the status for
	// the RPC (restricted to an allowable set of codes as defined by gRFC
	// A54). uri is the URI of the entry point for the request.  When supported
	// by the underlying implementation, ctx can be used for timeout and
	// cancellation. Additionally, RequestInfo data will be available via ctx
	// to this call.  TODO(zhaoq): Define the set of the qualified keys instead
	// of leaving it as an arbitrary string.
	GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error)
	// RequireTransportSecurity indicates whether the credentials requires
	// transport security.
	RequireTransportSecurity() bool
}*/
type ClientTokenAuth struct {
}

func (c ClientTokenAuth) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {

	return map[string]string{
		"authorization": "some-secret-token",
	}, nil
}
func (c ClientTokenAuth) RequireTransportSecurity() bool {
	return false
}

const (
	defaultName = "world"
)

var (
	addr = flag.String("addr", "localhost:50051", "the address to connect to")
	name = flag.String("name", defaultName, "Name to greet")
)

func main() {
	flag.Parse()
	var opts []grpc.DialOption
	opts = append(opts, grpc.WithTransportCredentials(insecure.NewCredentials()))
	opts = append(opts, grpc.WithPerRPCCredentials(new(ClientTokenAuth)))
	// Set up a connection to the server.
	conn, err := grpc.Dial(*addr, opts...)
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewGreeterClient(conn)

	// Contact the server and print out its response.
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()
	r, err := c.SayHello(ctx, &pb.HelloRequest{Name: *name})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.GetMessage())
}
```