go-nflog [![GoDoc](https://godoc.org/github.com/rcsiberia/go-nflog?status.svg)](https://godoc.org/github.com/rcsiberia/go-nflog) [![Build Status](https://travis-ci.org/rcsiberia/go-nflog.svg?branch=master)](https://travis-ci.org/rcsiberia/go-nflog) [![Go Report Card](https://goreportcard.com/badge/github.com/rcsiberia/go-nflog)](https://goreportcard.com/report/github.com/rcsiberia/go-nflog)
============

This is `go-nflog` and it is written in [golang](https://golang.org/). It provides a [C](https://en.wikipedia.org/wiki/C_(programming_language))-binding free API to the netfilter based log subsystem of the [Linux kernel](https://www.kernel.org).

Example
-------

```golang
func main() {
	// Send outgoing pings to nflog group 100
	// # sudo iptables -I OUTPUT -p icmp -j NFLOG --nflog-group 100

	//Set configuration parameters
	config := nflog.Config{
		Group:       100,
		Copymode:    nflog.NfUlnlCopyPacket,
		ReadTimeout: 10 * time.Millisecond,
	}

	nf, err := nflog.Open(&config)
	if err != nil {
		fmt.Println("could not open nflog socket:", err)
		return
	}
	defer nf.Close()

	ctx, _ := context.WithTimeout(context.Background(), 10*time.Second)

	fn := func(attrs nflog.Attribute) int {
		fmt.Printf("%v\n", attrs.Payload)
		return 0
	}

	// Register your function to listen on nflog group 100
	err = nf.Register(ctx, fn)
	if err != nil {
		fmt.Println(err)
		return
	}

	// Block till the context expires
	<-ctx.Done()
}
```

Privileges
----------

This package processes information directly from the kernel and therefore it requires special privileges. You
can provide this privileges by adjusting the `CAP_NET_ADMIN` capabilities.
```
	setcap 'cap_net_admin=+ep' /your/executable
```

For documentation and more examples please take a look at [![GoDoc](https://godoc.org/github.com/rcsiberia/go-nflog?status.svg)](https://godoc.org/github.com/rcsiberia/go-nflog)
