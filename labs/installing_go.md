# Installing Go

## Install a Go binary distribution
 
```
wget https://go.googlecode.com/files/go1.2.linux-amd64.tar.gz
tar -xvf go1.2.linux-amd64.tar.gz -C /usr/local
```

## Setup the Go workspace

`~/.bashrc`

```
export GOPATH=~/go
export PATH=/usr/local/go/bin:${GOPATH}/bin:$PATH
```

## Load the environment

```
source ~/.bashrc
```

## Create The Go workspace directories

- $HOME/go/bin
- $HOME/go/pkg
- $HOME/go/src/github.com/username


## Create the directories on Unix

```
mkdir -p ~/go/{src,pkg,bin}
mkdir -p ~/go/src/github.com/username
```

## Check the Go environment

```
go env
```

```
GOARCH="amd64"
GOBIN=""
GOCHAR="6"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/kelseyhightower/go"
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
TERM="dumb"
CC="clang"
GOGCCFLAGS="-g -O2 -fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fno-common"
CXX="clang++"
CGO_ENABLED="1"
```