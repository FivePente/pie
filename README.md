# pie [![GoDoc](https://godoc.org/github.com/natefinch/pie?status.png)](https://godoc.org/github.com/natefinch/pie) [![Build Status](https://drone.io/github.com/natefinch/pie/status.png)](https://drone.io/github.com/natefinch/pie/latest)

    import "github.com/natefinch/pie"

package pie provides a toolkit for creating plugins for Go applications.

![pie](https://cloud.githubusercontent.com/assets/3185864/7804533/6f2d70b8-0332-11e5-9a53-574aba44ea69.png)

**Why is it called pie?**

Because if you pronounce API like "a pie", then all this consuming and serving
of APIs becomes a lot more palatable.  Also, pies are the ultimate pluggable
interface - depending on what's inside, you can get dinner, dessert, a snack, or
even breakfast.  Plus, then I get to say that plugins in Go are as easy as pie.

## About Pie

Plugins using this toolkit and the applications managing those plugins
communicate via RPC over the plugin application's Stdin and Stdout.

Functions in this package with the prefix `New` are intended to be used by the
plugin to set up its end of the communication.  Functions in this package
with the prefix `Start` are intended to be used by the main application to set
up its end of the communication and start a plugin executable.

This package provides two conceptually different types of plugins, based on
which side of the communication is the server and which is the client.
Plugins which provide an API server for the main application to call are
called Providers.  Plugins which consume an API provided by the main
application are called Consumers.

The default codec for RPC for this package is Go's gob encoding, however you
may provide your own codec, such as JSON-RPC provided by net/rpc/jsonrpc.

There is no requirement that plugins for applications using this toolkit be
written in Go. As long as the plugin application can consume or provide an
RPC API of the correct codec, it can interoperate with main applications
using this process.  For example, if your main application uses JSON-RPC,
many languages are capable of producing an executable that can provide a
JSON-RPC API for your application to use.

Included in this repo are a couple simple examples of plugins.  The basic plugin
that provides an API for the master process can be seen in the example\_master
and example\_plugin folders.  example\_master expects example_plugin to be in
the same directory.  You can just go install both of them, and it'll work
correctly.

To see an example of a plugin that consumes an API from the host process, look
in the example\_host and example\_consumer folders.


## func NewConsumer
``` go
func NewConsumer() *rpc.Client
```
NewConsumer returns an rpc.Client that will consume an API from the host
process over this application's Stdin and Stdout using gob encoding.


## func NewConsumerCodec
``` go
func NewConsumerCodec(f func(io.ReadWriteCloser) rpc.ClientCodec) *rpc.Client
```
NewConsumerCodec returns an rpc.Client that will consume an API from the host
process over this application's Stdin and Stdout using the ClientCodec
returned by f.


## func StartProvider
``` go
func StartProvider(output io.Writer, path string, args ...string) (*rpc.Client, error)
```
StartProvider start a plugin application at the given path and args, and
returns an RPC client that communicates with the plugin using gob encoding
over the plugin's Stdin and Stdout.  The writer passed to output will receive
output from the plugin's stderr.  Closing the RPC client returned from this
function will shut down the plugin application.


## func StartProviderCodec
``` go
func StartProviderCodec(
	f func(io.ReadWriteCloser) rpc.ClientCodec, 
	output io.Writer, 
	path string, 
	args ...string,
) (*rpc.Client, error)
```
StartProviderCodec starts a plugin application at the given path and args,
and returns an RPC client that communicates with the plugin using the
ClientCodec returned by f over the plugin's Stdin and Stdout. The writer
passed to output will receive output from the plugin's stderr.  Closing the
RPC client returned from this function will shut down the plugin application.


## type Provider
``` go
type Provider struct {
    // contains filtered or unexported fields
}
```
Provider is a type that will allow you to register types for the API of a
plugin and then serve those types over RPC.  It encompasses the functionality
to talk to a master process.


### func NewProvider
``` go
func NewProvider() Provider
```
NewProvider returns a plugin provider that will serve RPC over this
application's Stdin and Stdout.  This method is intended to be run by the
plugin application.


### func StartConsumer
``` go
func StartConsumer(output io.Writer, path string, args ...string) (Provider, error)
```
StartConsumer starts a plugin application with the given path and args,
writing its stderr to output.  The plugin consumes an API this application
provides.


### func (Provider) Register
``` go
func (p Provider) Register(rcvr interface{}) error
```
Register publishes in the provider the set of methods of the receiver value
that satisfy the following conditions:


	- exported method
	- two arguments, both of exported type
	- the second argument is a pointer
	- one return value, of type error

It returns an error if the receiver is not an exported type or has no
suitable methods. It also logs the error using package log. The client
accesses each method using a string of the form "Type.Method", where Type is
the receiver's concrete type.



### func (Provider) RegisterName
``` go
func (p Provider) RegisterName(name string, rcvr interface{}) error
```
RegisterName is like Register but uses the provided name for the type
instead of the receiver's concrete type.



### func (Provider) Serve
``` go
func (p Provider) Serve()
```
Serve starts the plugin's RPC server, serving via gob encoding.  This call
will block until the client hangs up.



### func (Provider) ServeCodec
``` go
func (p Provider) ServeCodec(f func(io.ReadWriteCloser) rpc.ServerCodec)
```
ServeCodec starts the plugin's RPC server, serving via the encoding returned by f.
This call will block until the client hangs up.


