# go-dotnet

[![GoDoc](https://godoc.org/github.com/matiasinsaurralde/go-dotnet?status.svg)](https://godoc.org/github.com/matiasinsaurralde/go-dotnet)
[![MIT License][license-image]][license-url]
[![Build status][travis-build-image]][travis-build-status]

_Warning: This is just a fork, tweaked for my own purposes; you ought to [go to the source](https://github.com/matiasinsaurralde/go-dotnet) to get the original source code instead._

***

This is a PoC Go wrapper for the .NET Core Runtime, this project uses ```cgo``` and has been tested under OSX. It covers two basic use cases provided by the [CLR Hosting API](https://blogs.msdn.microsoft.com/msdnforum/2010/07/09/use-clr4-hosting-api-to-invoke-net-assembly-from-native-c/):

* Load and run an .exe, using its default entrypoint, just like [corerun](https://github.com/dotnet/coreclr/blob/master/src/coreclr/hosts/unixcorerun/corerun.cpp) and [coreconsole](https://github.com/dotnet/coreclr/blob/master/src/coreclr/hosts/unixcoreconsole/coreconsole.cpp) do, check ```ExecuteManagedAssembly```.

* Load a .dll, setup [delegates](http://www.fancy-development.net/hosting-net-core-clr-in-your-own-process) and call them from your Go functions.

I've tried calling both C# and VB.NET methods, of course you need to generate the assembly first, check below for more details!

**Note: After some tweaks it seems to work fine under Linux! Remember to install the SDK first :)**

![Capture][capture]


## An example

```go
package main

import (
	"github.com/matiasinsaurralde/go-dotnet"

	"fmt"
	"os"
)

func main() {
	fmt.Println("Hi, I'll initialize the .NET runtime.")

	/*
		If you don't set the TRUSTED_PLATFORM_ASSEMBLIES, it will use the default tpaList value.
		APP_PATHS & NATIVE_DLL_SEARCH_DIRECTORIES use the path of the current program,
		this makes it easier to load an assembly, just put the DLL in the same folder as your Go binary!
		You're free to override them to fit your needs.
	*/

	properties := map[string]string{
		// "TRUSTED_PLATFORM_ASSEMBLIES": "/usr/local/share/dotnet/shared/Microsoft.NETCore.App/1.0.0/mscorlib.ni.dll:/usr/local/share/dotnet/shared/Microsoft.NETCore.App/1.0.0/System.Private.CoreLib.ni.dll",
		// "APP_PATHS":                     "/Users/matias/dev/dotnet/lib/HelloWorld",
		// "NATIVE_DLL_SEARCH_DIRECTORIES": "/Users/matias/dev/dotnet/lib/HelloWorld",
	}

	/*
		CLRFilesAbsolutePath sets the SDK path.
		In case you don't set this parameter, this package will try to find the SDK using a list of common paths.
		It seems to find the right paths under Linux & OSX, feel free to override this setting (like the commented line).
	*/

	runtime, err := dotnet.NewRuntime(dotnet.RuntimeParams{
		Properties:                  properties,
		// CLRFilesAbsolutePath: "/usr/share/dotnet/shared/Microsoft.NETCore.App/1.0.0"
	})
	defer runtime.Shutdown()

	if err != nil {
		fmt.Println("Something bad happened! :(")
		os.Exit(1)
	}

	fmt.Println("Runtime loaded.")

	SayHello := runtime.CreateDelegate("HelloWorld", "HelloWorld.HelloWorld", "Hello")

    // this will call HelloWorld.HelloWorld.Hello :)
	SayHello()
}
```

## Preparing your code (C#)

I've used ```dmcs``` (from Mono) to generate an assembly file, the original code was something like:

```c#
using System;

namespace HelloWorld {

	public class HelloWorld {
    	public static void Hello() {
      		Console.WriteLine("Hello from .NET");
    	}
	}

}
```

And the command:

```
dmcs -t:library HelloWorld.cs
```

## Preparing your code (VisualBasic)

I did a quick test with [this program](https://github.com/matiasinsaurralde/go-dotnet/blob/master/examples/HelloWorld.vb), using the VB.NET compiler from Mono:

```
vbnc -t:library HelloWorld.vb
```

I'm not sure about the status of [Roslyn](https://github.com/dotnet/roslyn) but it could be interesting to try it.

## Setup

Coming soon!

## Ideas

* Run some benchmarks.
* Add/enhance ```net/http``` samples, like [this](https://github.com/matiasinsaurralde/go-dotnet/blob/master/examples/http.go).
* Provide useful callbacks.
* Support blittable types.
* CSharpScript support.
* Code generation tool (with `go generate`), a few notes [here](https://github.com/matiasinsaurralde/go-dotnet/blob/master/code_generation.md).
* **Add tests.**

I'm open to PRs, Go/.NET swag, suggestions, etc.

## Additional resources

* [Hosting .NET Core](https://github.com/dotnet/docs/blob/master/docs/core/tutorials/netcore-hosting.md)

Build Status
------------

Linux x64 / Go 1.9/1.10 / .NET Core 1.0/2.0 - OS X / Go 1.9/1.10 - .NET Core 1.0/2.0

[![Linux and OS X build status][travis-build-image]][travis-build-status]

Windows / Go 1.9 / .NET Core 2.x

[![Windows build status][windows-build-image]][windows-build-status]

## License

[MIT](LICENSE)

[license-url]: LICENSE

[license-image]: http://img.shields.io/badge/license-MIT-blue.svg?style=flat

[capture]: capture.png

[travis-build-image]: https://travis-ci.org/matiasinsaurralde/go-dotnet.svg?branch=master
[travis-build-status]: https://travis-ci.org/matiasinsaurralde/go-dotnet

[windows-build-image]: https://ci.appveyor.com/api/projects/status/wc6hi7h89c6f83it/branch/windows-support?svg=true
[windows-build-status]: https://ci.appveyor.com/project/matiasinsaurralde/go-dotnet/branch/windows-support
