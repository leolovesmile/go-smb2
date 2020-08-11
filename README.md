smb2
====

[![Build Status](https://travis-ci.org/hirochachacha/go-smb2.svg?branch=master)](https://travis-ci.org/hirochachacha/go-smb2)
[![GoDoc](https://godoc.org/github.com/hirochachacha/go-smb2?status.svg)](http://godoc.org/github.com/hirochachacha/go-smb2)

Description
-----------

SMB2/3 client implementation.

Installation
------------

`go get github.com/hirochachacha/go-smb2`

Documentation
-------------

http://godoc.org/github.com/hirochachacha/go-smb2

Examples
--------

### List share names ###

```go
package main

import (
	"fmt"
	"net"

	"github.com/hirochachacha/go-smb2"
)

func main() {
	conn, err := net.Dial("tcp", "SERVERNAME:445")
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	d := &smb2.Dialer{
		Initiator: &smb2.NTLMInitiator{
			User:     "USERNAME",
			Password: "PASSWORD",
		},
	}

	s, err := d.Dial(conn)
	if err != nil {
		panic(err)
	}
	defer s.Logoff()

	names, err := s.ListShareNames()
	if err != nil {
		panic(err)
	}

	for _, name := range names {
		fmt.Println(name)
	}
}
```

### File manipulation ###

```go
package main

import (
	"io"
	"io/ioutil"
	"net"
	"os"

	"github.com/hirochachacha/go-smb2"
)

func main() {
	conn, err := net.Dial("tcp", "SERVERNAME:445")
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	d := &smb2.Dialer{
		Initiator: &smb2.NTLMInitiator{
			User:     "USERNAME",
			Password: "PASSWORD",
		},
	}

	s, err := d.Dial(conn)
	if err != nil {
		panic(err)
	}
	defer s.Logoff()

	fs, err := s.Mount("SHARENAME")
	if err != nil {
		panic(err)
	}
	defer fs.Umount()

	f, err := fs.Create("hello.txt")
	if err != nil {
		panic(err)
	}
	defer fs.Remove("hello.txt")
	defer f.Close()

	_, err = f.Write([]byte("Hello world!"))
	if err != nil {
		panic(err)
	}

	_, err = f.Seek(0, io.SeekStart)
	if err != nil {
		panic(err)
	}

	bs, err := ioutil.ReadAll(f)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(bs))
}
```
