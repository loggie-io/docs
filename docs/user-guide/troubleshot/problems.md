# 问题案例

## net.cgoLookupIP 导致segmentation violation

**现象**

可能在使用了比如unix sock source的时候，Loggie启动后crash，并有如下的日志：

```
fatal error: unexpected signal during runtime execution
[signal SIGSEGV: segmentation violation code=0x1 addr=0x47 pc=0x7f59b4528360]

runtime stack:
runtime.throw({0x213b1a1, 0x7f59b423b640})
	/usr/local/go/src/runtime/panic.go:1198 +0x71
runtime.sigpanic()
	/usr/local/go/src/runtime/signal_unix.go:719 +0x396

goroutine 86 [syscall]:
runtime.cgocall(0x1a39d30, 0xc000510d90)
	/usr/local/go/src/runtime/cgocall.go:156 +0x5c fp=0xc000510d68 sp=0xc000510d30 pc=0x40565c
net._C2func_getaddrinfo(0xc00030bc40, 0x0, 0xc000724fc0, 0xc0005bd990)
	_cgo_gotypes.go:91 +0x56 fp=0xc000510d90 sp=0xc000510d68 pc=0x5c7bb6
net.cgoLookupIPCNAME.func1({0xc00030bc40, 0xc0004d29c0, 0x4}, 0xc00030bb80, 0xc000510e50)
	/usr/local/go/src/net/cgo_unix.go:163 +0x9f fp=0xc000510de8 sp=0xc000510d90 pc=0x5c98ff
net.cgoLookupIPCNAME({0x20f751c, 0x3}, {0xc00030bb80, 0xc00022b5e0})
	/usr/local/go/src/net/cgo_unix.go:163 +0x16d fp=0xc000510f38 sp=0xc000510de8 pc=0x5c914d
net.cgoIPLookup(0x358aed0, {0x20f751c, 0xc00030bbc0}, {0xc00030bb80, 0xc000510fb8})
	/usr/local/go/src/net/cgo_unix.go:220 +0x3b fp=0xc000510fa8 sp=0xc000510f38 pc=0x5c99bb
net.cgoLookupIP·dwrap·25()
	/usr/local/go/src/net/cgo_unix.go:230 +0x36 fp=0xc000510fe0 sp=0xc000510fa8 pc=0x5c9e36
runtime.goexit()
	/usr/local/go/src/runtime/asm_amd64.s:1581 +0x1 fp=0xc000510fe8 sp=0xc000510fe0 pc=0x46ae81
created by net.cgoLookupIP
	/usr/local/go/src/net/cgo_unix.go:230 +0x125
```

**原因**

具体原因请参考：[go net](https://go.dev/src/net/net.go)。

```
Name Resolution

The method for resolving domain names, whether indirectly with functions like Dial
or directly with functions like LookupHost and LookupAddr, varies by operating system.

On Unix systems, the resolver has two options for resolving names.
It can use a pure Go resolver that sends DNS requests directly to the servers
listed in /etc/resolv.conf, or it can use a cgo-based resolver that calls C
library routines such as getaddrinfo and getnameinfo.

By default the pure Go resolver is used, because a blocked DNS request consumes
only a goroutine, while a blocked C call consumes an operating system thread.
When cgo is available, the cgo-based resolver is used instead under a variety of
conditions: on systems that do not let programs make direct DNS requests (OS X),
when the LOCALDOMAIN environment variable is present (even if empty),
when the RES_OPTIONS or HOSTALIASES environment variable is non-empty,
when the ASR_CONFIG environment variable is non-empty (OpenBSD only),
when /etc/resolv.conf or /etc/nsswitch.conf specify the use of features that the
Go resolver does not implement, and when the name being looked up ends in .local
or is an mDNS name.

The resolver decision can be overridden by setting the netdns value of the
GODEBUG environment variable (see package runtime) to go or cgo, as in:

	export GODEBUG=netdns=go    # force pure Go resolver
	export GODEBUG=netdns=cgo   # force cgo resolver

The decision can also be forced while building the Go source tree
by setting the netgo or netcgo build tag.

```

**解决办法**

在Loggie 部署脚本里增加环境变量：

```yaml
  env:
    - name: GODEBUG
      value: netdns=go
```