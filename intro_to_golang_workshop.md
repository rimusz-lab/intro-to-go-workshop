

## Installing Go


Install a Go binary distribution
 

	wget https://go.googlecode.com/files/go1.2.linux-amd64.tar.gz
	tar -xvf go1.2.linux-amd64.tar.gz -C /usr/local



Setup the Go workspace

`~/.bashrc`

	export GOPATH=~/go
	export PATH=/usr/local/go/bin:${GOPATH}/bin:$PATH

Load the environment

	source ~/.bashrc


Create The Go workspace directories

	$HOME/go/bin
	$HOME/go/pkg
	$HOME/go/src/github.com/username


Create the directories on Unix

	mkdir -p ~/go/{src,pkg,bin}
	mkdir -p ~/go/src/github.com/username


Check the Go environment

    go env

## Hello World

    mkdir ~/go/src/github.com/username/hello


Edit `~/go/src/github.com/username/hello/main.go`


	package main

	import (
		"fmt"
	)

	func main() {
		fmt.Println("Hello World")
	}


Build

    go build .


Run

    ./hello


## Package to convert CSV to JSON - csv2json

### Write the code

Create ~/go/src/github.com/username/csv2json

Edit ~/go/src/github.com/username/csv2json/csv2json.go


	package csv2json

	import (
		"encoding/csv"
		"encoding/json"
		"io"
	)

	func Convert(r io.Reader, columns []string) ([]byte, error) {
		rows := make([]map[string]string, 0)
		csvReader := csv.NewReader(r)
		csvReader.TrimLeadingSpace = true
		for {
			record, err := csvReader.Read()
			if err == io.EOF {
				break
			}
			if err != nil {
				return nil, err
			}
			row := make(map[string]string)
			for i, n := range columns {
				row[n] = record[i]
			}
			rows = append(rows, row)
		}
		data, err := json.MarshalIndent(&rows, "", "  ")
		if err != nil {
			return nil, err
		}
		return data, nil
	}

### Build

    go build .


### Writing unit tests

Edit: ~/go/src/github.com/username/csv2json/csv2json_test.go

	package csv2json

	import (
		"bytes"
		"io/ioutil"
		"os"
		"testing"
	)

	var columns = []string{"name", "email", "phone"}
	var want = `[
	  {
		"email": "ken@google.com",
		"name": "Ken Thompson",
		"phone": "555-5550"
	  },
	  {
		"email": "rob@google.com",
		"name": "Rob Pike",
		"phone": "555-5551"
	  },
	  {
		"email": "robert@google.com",
		"name": "Robert Griesemer",
		"phone": "555-5552"
	  }
	]`

	var input = `
	Ken Thompson, ken@google.com, 555-5550
	Rob Pike, rob@google.com, 555-5551
	Robert Griesemer, robert@google.com, 555-5552
	`

	func TestConvertWithBuffer(t *testing.T) {
		var b bytes.Buffer
		_, err := b.WriteString(input)
		if err != nil {
			t.Error(err)
		}
		got, err := Convert(bytes.NewReader(b.Bytes()), columns)
		if err != nil {
			t.Error(err)
		}
		if string(got) != want {
			t.Errorf("TestConvertWithBuffer(f, %s) = %s; want %s",
				columns, string(got), want)
		}
	}

### Running the test suite

Run 

    go test


Run with the -v flag for verbose output:

    go test -v


### Exercise: Write an additional test

Create an additional test named TestConvertWithFile.

Hint

	func TestConvertWithFile(t *testing.T) {
		tf, err := ioutil.TempFile("", "")
		if err != nil {
			t.Error(err)
		}
		defer os.Remove(tf.Name())

		err = ioutil.WriteFile(tf.Name(), []byte(input), 0644)
		if err != nil {
			t.Error(err)
		}

		f, err := os.Open(tf.Name())
		if err != nil {
			t.Error(err)
		}
		defer f.Close()

		// Test your results
	}

## CLI tool to convert CSV to JSON - csv2json-cli

Convert CSV files into JSON.

### Write the code

Create ~/go/src/github.com/username/csv2json-cli

Edit ~/go/src/github.com/username/csv2json-cli/main.go


	package main

	import (
		"flag"
		"fmt"
		"log"
		"os"

		"github.com/kelseyhightower/csv2json"
	)

	var (
		infile string
		columns = []string{"Name","Date","Title"}
	)

	func init() {
		flag.StringVar(&infile, "infile", "", "input file")
	}

	func main() {
		flag.Parse()
		if infile == "" {
			log.Fatal("Input file required")
		}
		f, err := os.Open(infile)
		if err != nil {
			log.Fatal(err)
		}
		defer f.Close()
		jsonData, err := csv2json.Convert(f, columns)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println(string(jsonData))
	}


Edit ~/go/src/github.com/username/famous-gophers.csv

	Mac, 1947, The Goofy Gophers
	Tosh, 1947, The Goofy Gophers
	Samuel J. Gopher, 1966, Winnie the Pooh
	Chief Running Board, 1968, Go Go Gophers
	Ruffled Feathers, 1968, Go Go Gophers


Build

    go build .


Run

    ./csv2json-cli -infile famous-gophers.csv



Managing dependencies with godep

Run

	go list -json
	go get github.com/kr/godep
	godep save



Exercise: Better error messages

Improve the error message when err != nil

Hint 

import "error"


Exercise: Add comments

//

/* */


### Exercise: Generate Go docs

    godoc -http=":8080"


## HTTP API to convert CSV to JSON - csv2json-http

### Write the code

Create ~/go/src/github.com/username/csv2json-http

Edit ~/go/src/github.com/username/csv2json-http/main.go


	package main

	import (
		"log"
		"net/http"

		"github.com/kelseyhightower/csv2json"
	)

	var (
		columns = []string{"Name", "Date", "Title"}
	)

	func csv2JsonServer(w http.ResponseWriter, req *http.Request) {
		jsonData, err := csv2json.Convert(req.Body, columns)
		defer req.Body.Close()
		if err != nil {
			http.Error(w, "Could not convert csv to json", 503)
		}
		print(string(jsonData))
		w.Write(jsonData)
	}

	func main() {
		http.HandleFunc("/csv2json", csv2JsonServer)
		err := http.ListenAndServe(":8080", nil)
		if err != nil {
			log.Fatal("ListenAndServe: ", err)
		}
	}


Build

    go build .



Run

    ./csv2json-http

### Exercise: Make the listen port configurable via the environment

Hint

    import "os"
    os.Getenv("PORT")


### Exercise: Add more logging

Hint

    import "log"

## Cross Compiling

Build the toolchain

	GOOS
	GOARCH
	CGO_ENABLED

Run

	go env
	cd /usr/local/go/pkg
	ls
	linux_amd64

	Run

	apt-get install gcc
	cd /usr/local/go/src
	GOOS=linux GOARCH=386 CGO_ENABLED=0 ./make.bash —no-clean

	cd /usr/local/go/pkg
	ls
	linux_386 linux_amd64 

	GOOS=darwin GOARCH=amd64 CGO_ENABLED=0 ./make.bash —no-clean

	cd /usr/local/go/pkg
	ls
	darwin_amd64 linux_386 linux_amd64


Cross compile for Window or OS X

	cd to a project
	GOOS=darwin go build .