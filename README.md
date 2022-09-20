# ffikdb-golang

Calling golang code from kdb+/q through KxSystems/ffi

## kdb+/q and KxSystem/ffi installation

First of all, double check that your local [kdb+/q installation](https://code.kx.com/q/learn/install/) got that [KxSystem/ffi](https://code.kx.com/q/interfaces/ffi/) up & running. 

For example, on my Archlinux daily driver, **ffikdb.so** was copied to *$QHOME/l64* and **ffi.q** to *$QHOME*. However, my installed libffi.so was not the expected version by ffikdb.so; a humble softlink did the trick but it's not a good example to follow. Your milage might vary.

	ldd $QHOME/l64/ffikdb.so
	  linux-vdso.so.1 => linux-vdso.so.1 (0x00007ffe99786000)
	  libdl.so.2 => /usr/lib/libdl.so.2 (0x00007f8d0ca1a000)
	  libffi.so.6 => /usr/lib/libffi.so.6 (0x00007f8d0ca0e000)
	  libc.so.6 => /usr/lib/libc.so.6 (0x00007f8d0c619000)
	  /usr/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007f8d0ca5d000)

Then, just double check is really working with that [KxSystem/ffi example](https://code.kx.com/q/interfaces/ffi/examples/):

	q)\l ffi.q
	q)buffer: 80#"\000"
	q)args:(buffer; "%s %.4f %hd\000"; "example\000"; 3.16f; 144000h; ::)
	q)n:.ffi.callFunction[("i"; `sprintf)] args
	q)buffer til n
	"example 3.1600 32767"

## basic golang dynamic library

Build your [source code](basic/main.go) into a dynamic shared library by invoking *make*:

	go build -buildmode=c-shared -o libgolang.so . 

That should create your basic libgolang.so library. Do not forget to add its path to *LD_LIBRARY_PATH* before calling *q* prompt:

	q)\l ffi.q
	q).golang.cgoCurrentMillis:.ffi.bind[`libgolang.so`cgoCurrentMillis; "i"; "j"]

Previous command *binds* your **golang** function to a **kdb+** one:

	q).golang.cgoCurrentMillis (0; ::)
	1663701692
	q).golang.cgoCurrentMillis (0; ::)
	1663701694
	q).golang.cgoCurrentMillis (0; ::)
	1663701695
	q).golang.cgoCurrentMillis (0; ::)
	1663701696

Pay attention to the fact that your calling your *kdb+* function with at least one argument, that fake integer at .ffi.bind.,  although your *golang* function doesn't have any. More details at [ffi kdb+ reference](https://code.kx.com/q/interfaces/ffi/reference/#ffibind)

