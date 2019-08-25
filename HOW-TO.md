# Introduction

The purpose of this document is to provide you with minimal knowledge of Go and Hyperledger Fabric (Fabric for short) infrastructure to enable you to start writing and debugging chaincode. This is not an extensive lession but you will have gained sufficient knowledge to move on to developing real-world chaincode.

This document is intended for anyone with programming experience but having **no** or **very little experience** of Go and chaincode development. However, you are expected to be sufficiently familiar with the concept of compilation and packaging, as Go is a compiled, not scripting, langauge.

In this document, you will learn to:

* [Setup for chaincode development](#setupDevEnv)

* [Minimal Go for chaincode development](#learnGo)

* [Write chaincodes](#goForChaincode)

# <a name="setupDevEnv">Setup for chaincode development</a>

Setting up a development environment for chaincode projects is no different from setting up for other non chaincode Go projects. 

For a basic (terminal and command-line) environment for chaincode development, please follow the following steps:

1. Install [Go tools](http://golang.org/dl).

    * for macOS, we recommend installing via [homebrew](http://brew.sh/);

    * for other platforms please refer to [installation guide](https://golang.org/doc/install).

    Additional notes for this setp:

    * Please also ensure that you also install C++ compiler. Refer to your respective platform documentation for instructions.

    * On Ubuntu you may also need to install a library call `ltdl` (please refer to `apt-get install ltdl-dev`).

2. Set the environmental variable `GOPATH` to a reference a directory to host your Go source codes and binaries (i.e. Go workspace). For example,
    
    ```
    export GOPATH=$HOME/go-projects
    ```

3. Navigate to the `$GOPATH` directory and install a Go application call [Govendor](https://github.com/kardianos/govendor) by executing this command:
    
    ```
    go get -u github.com/kardianos/govendor
    ```

    At the completion of the command, you will find in `$GOPATH` three directories:

    ```
    drwxr-xr-x  3 <userid>  <groupid>  102  3 Feb 15:44 bin
    drwxr-xr-x  3 <userid>  <groupid>  102  3 Feb 15:44 pkg
    drwxr-xr-x  3 <userid>  <groupid>  102  3 Feb 15:44 src
    ```

    This structure is dictated by Go tooling and will be your primary workspace for organising your chaincodes and and other dependencies such as third parties codes, tooling extensions, etc.

    In the context of chaincode development, you will be working mainly with Go sources. Hence, you only need to concern yourself with organising stuff within `src` directory.

    Additional notes for this step:

    * This step is not strictly needed. You could have create the workspace directories manually.

    * [Govendor](https://github.com/kardianos/govendor) is a package or dependency management tool. It is one of many tools you can use to manage Go dependencies. The choice of `Govendor` is purely based on familarity. You could elect to install [other tools](https://github.com/golang/go/wiki/PackageManagementTools)).

4. Add the `$GOPATH/bin` to your `PATH` environmental variable. For example:

    ```
    export PATH=$GOPATH/bin:$PATH
    ```

    `$GOPATH/bin` is a directory for binaries generated from Go compilations. Some of these binaries may be used to extend the functionalities of Go tooling or any other support tools. If you are using [Visual Studio Code](https://code.visualstudio.com/), you will find extensions to the editor such as code completion or syntax highlighting, served from this directory.

5. Get the Fabric dependencies (the framework to support your chaincode developmemnt) by issuing the following commands:

    ```
    go get -d github.com/hyperledger/fabric
    ```

    At the completion of this command, you will see this message:

    ```
    package github.com/hyperledger/fabric: no buildable Go source files in /$GOPATH/src/github.com/hyperledger/fabric
    ```
    
    There is no need to worry. Go tooling typically pull source code and then tries to build a binary but in this case the hyperledger fabric dependencies have nothing to be built.

    If you wish to ensure that the dependencies have been pulled down, simply navigate to `$GOPATH/src/github.com` and if you see the directory `hyperledger` it means that dependencies have been downloaded. 

# <a name="learnGoLang">Minimal Go for Chaincode</a>

If you are already and experience Go programmer you can skip this section.

To learn about Go progranmning language, please refer to these resources:

* [Go playground](https://play.golang.org/) - This a a web-base development environment where you can learn to code in Go without the need to setup a local development environment.

* [Go by example](https://gobyexample.com/) - This is a series of code snippets demonstrating features of Go by theme.

To create a minimal chaincode focus your learning on these areas 

* [data types](https://gobyexample.com/variables);

* [functions](https://gobyexample.com/functions);

* [structs](https://gobyexample.com/structs);

* [interfaces](https://gobyexample.com/interfaces).

You will also need to be aware that all your Go codes (and chaincodes) have to be organised around `$GOPATH/src` directory. For example, here is a hypothetical structure:

```
    $GOPATH/src
        git.ng.bluemix.net/project/repo
            cmd
                main.go
            helper
                math.go
        github/spf13/corbra // Third parties code
            ....
```

Organise your code and dependencies to reflect the way codes would be stored in a typical Git-like repository. Please refer to the official documentation about [code organisation](https://golang.org/doc/code.html#Organization).

# <a name="goForChaincode">Writing chaincode</a> 

In this section you will learn to:

* [write the smallest unit of executable chaincode](#smallestchaincode);

* [organise your chaincode project](#organiseChaincode).

### <a name="smallestchaincode">Smallest unit of executable chaincode</a>

A minimal executable Go code is this:

```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}

```

To compile and execute this code all you need to do is to issue the command `go run`. It will run in your macOS, Linux, Windows or any compatible platform.

In the case of chaincode, the [smallest unit of executable code](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html) is this:

```
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    pb "github.com/hyperledger/fabric/protos/peer" 
)

type SimpleChaincode struct{}

func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
    return shim.Success([]byte("Init called"))
}

func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
    return shim.Success([]byte("Invoke called"))
}

func main() {
    err := shim.Start(new(SimpleChaincode))
    if err != nil {
        fmt.Printf("Error: %s", err)
    }
}
```

Place your code in the file `chaincode.go` under the appropriate part of your Go workspace for example, `$GOPATH/src/github.com/user/repo/chaincode.go`.

To get a sense of whether the code is workable, navigate to the directory containing your main chaincode file and execute `go run` command. For example:

```
cd $GOPATH/src/github.com/user/repo/
go run chaincode.go
```

You will see the following output:

```
<Date> <Timestamp> <Timezone> [shim] SetupChaincodeLogging -> INFO 001 Chaincode log level not provided; defaulting to: INFO
<Date> <Timestamp> <Timezone> [shim] getPeerAddress -> CRIT 002 peer.address not configured, can't connect to peer
exit status 1
```

This simply indicates that the chaincode has been successfully compiled and your code has been executed. However, there is no running Fabric infastructure to interact with so you see this error message.

Unlike normal Go program, you can't just compile and run chaincode in macOS, Linux, Windows, etc. Instead you will need to bundle the code and deploy it to a running Fabric platform known as a **Fabric peer** (see [architecture](http://hyperledger-fabric.readthedocs.io/en/latest/arch-deep-dive.html#system-architecture) for detailed explanation). 

A Fabric peer typically runs in a Docker container. You could "natively" deploy a peer as part of a component of macOS, Linux, Windows, etc., but is beyond the scope of this document to discuss these types of configuration. We'll only focus on the Docker deployment here.

<hr>

**Note**

* In the `import` clause of your chaincode, you'll see this `github.com/hyperledger/fabric/core/chaincode/shim`, which is a Fabric component (i.e. think library if you are C++ programmer).

* The `import` is derived from `$GOPATH/src/github.com/hyperledger/fabric`.

<hr>

### <a name="organiseChaincode">Organising your chaincode project</a>

It is extremely unusual to create a chaincode from a single file. 

You may want to re-use aspects of a Go code developed by third parties and/or separate out your functional from non-functional (e.g. string formatter) dependencies. Hence, you may want to distribute chaincode in different files and/or directories.

If you were working on a Go project you would organise your dependencies this way:

```
$GOPATH/
    src/
        github.com/anotheruser/repo/
            anothersrc.go
        github.com/user/repo/
            mypkg/
                mysrc1.go
                mysrc2.go
            cmd/mycmd/
                main.go
```

Your Go project `github.com/user/repo` is dependent on `github.com/anotheruser/repo` to provide a service. This is a sufficient structure to compile and run code on macOS, Linux, Windows, etc.

In the case of chaincode development, that structure will not work. You will need to organise all your dependencies under one root directory and then deploy the root directory to a Fabric peer. 

Here is an example of a hypothetical chaincode project with dependencies in a vendor folder:

```
$GOPATH/
    src/
        github.com/user/repo/
            mychaincode/
                util/
                    mymaths.go
                chaincode.go
                vendor/
                    github.com/anotheruser/repo
                        anothersrc1.go
                        anothersrc2.go
                    vendor.json
```
In this example:

* `mychaincode` is the root directory encapsulating your entire chaincode artifacts including dependencies;

* `util` is an example custom directory created by you to support your chaincode;

* `vendor` is a special directory (with a file `vendor.json`) typically to package dependencies not located at the chaincode root or dependencies from third parties (see detailed explanations of the use of [vendor folder](https://blog.gopheracademy.com/advent-2015/vendor-folder/));

* `github.com/anotheruser/repo` is a dependency that is referenced by `chaincode.go`, which would have existed outside you chaincode root folder but it is now in the vendor folder.

You can manually create and provision the `vendor` directory but using tools makes it easier. As per the [setup step](#setupDevEnv), let's use `Govendor` to `vendor` your dependencies: 

1. Navigate to `$GOPATH/src/github.com/user/repo/mychaincode` and execute this command:

    ```
    govendor init
    ```

   You should see a directory call `vendor`.

1. In the directory `vendor` add the following line to `vendor.json`:

    ```
    "ignore": "test github.com/hyperledger/fabric"
    ```

    This line tells `govendor` not to include `github.com/hyperledger/fabric` and test dependencies. You don't need to include hyperledger fabric dependency because it is part of the fabric peer infrastructure.

1. We are going to `vendor` a third party dependency `github.com/anotheruser/repo` by issuing this command:

    ```
    govendor fetch github.com/anotheruser/repo
    ```

    If no error you will see the dependencies stored in `vendor` directory.

<hr>

**Note:**

* Go (and chaincode) tooling typically search for dependencies from $GOPATH but the presence of `vendor` will enable Go tool to search there first.

* What if I wish to re-use another project that is outside the chaincode root directory in the Go workspace but not yet in say github.com repo?

* For example, this is my code structure:

    ```
    $GOPATH/
        src/
            github.com/user/another-repo/
                support/
                    superduper-support.go
                superduper-algo.go
            github.com/user/repo/
                mychaincode/
                    chaincode.go
                    util/
                        mymaths.go
                    vendor/
                        github.com/user/another-repo
                            support/
                                superduper-support.go
                            superduper-algo.go
                        vendor.json
    ```

    I wish to vendor `github.com/user/another-repo/`, but it does not yet exists in my `github.com` repo, into `mychaincode` directory.

    In this case run the command `govendor add github.com/user/another-repo`. You could also use this command `govendor add +external`. This will pull **ALL** the artefacts in `$GOPATH` into `vendor` folder. 
    
    It is beyond the scope of this document to discuss all use cases pertaining to `govendor`. Please refer to [Govendor documentation](https://github.com/kardianos/govendor) for details.

# Disclaimer

The methodologies discussed in this document and artefacts in this repository are intended only to illustrate concepts and are for educational purpose.

There is no guarantee that these artefacts are free from defects and are **NOT** intended for used in any mission critical, corporate or regulated projects. Should you choose to use them for these types of projects, you do so at your own risk.

Unless otherwise specified, the artefacts in this repository are distributed under Apache 2 license. In particular, the chaincodes are provided on "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
