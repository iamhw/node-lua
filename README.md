# node-lua

## description
1.	Node-lua is a node complementation of lua which supports sync and async rpc, and task-multiplexing support in multi-threads without harmless wakeup.
2.	It can be used as a simple script engine or complex server engine which supports a massive of independent lua contexts (or named services) running on multi-threads which restricted to the cpu core count.
3.	The lua context will suspend itself when it calls a sync and async rpc using lua coroutine inside the core c codes.
4.	The rpc can be called within the lua coroutine where the user creates and it won't impact the normal coroutine procedure.
5.	A context starts with a lua file as the entry. The process will exit automaticly when all contexts ends or terminates and a context will exit automaticly when all its running and suspending sync and async remote procedure call ends or terminates.
6.	A optimized task scheduling is used with a thread based context queue which reduces the thread race condition and work-stealing algorithm is used in the task scheduling.
7.  A more friendly tcp api is embeded in this engine with sync and async implementation which is much more convenient to build a tcp server.
8.  A shared buffer api is much more efficient for sending a broadcast packet.

## build & install

For windows, just open node-lua.sln and build the whole solution. For linux or other unix-like system where centos, macos, freebsd and ubuntu has been successfully supported and tested, run the flowing shell scripts:

    git clone https://github.com/socoding/node-lua.git
    cd node-lua
    make [release]  #add release to build for release versions
	make install

## usefull api
### tcp api
1.	`result, listen_socket = tcp.listen(addr, port[, backlog, [listen_callback(result, listen_socket, addr, port[, backlog])]])` 
	*`--listen on a ipv4 address.`* 
    *`--listen_callback is a once callback, blocking if listen_callback is nil.`*
	
2.	`result, listen_socket = tcp.listens(sock_name[, listen_callback(result, listen_socket, sock_name)])`  
    *`--listen on a windows named pipe or unix domain socket.`*
    *`--listen_callback is a once callback, blocking if listen_callback is nil.`*

3.	`result, listen_socket = tcp.listen6(addr, port[, backlog, [listen_callback(result, listen_socket, addr, port[, backlog])]])`  
	*`--listen on a ipv6 address.`*
    *`--listen_callback is a once callback, blocking if listen_callback is nil.`*
	
4.	`result, accept_socket = tcp.accept(listen_socket[, timeout])`  
	*`--accept on a listen socket in blocking mode, timeout is activated if timeout is set.`*

5.	`result, accept_socket = tcp.accept(listen_socket[, accept_callback(result, accept_socket, listen_socket)])`  
    *`--accept_callback is a continues callback, blocking if accept_callback is nil.`*
	
6. 	`result, connect_socket = tcp.connect(host_addr, host_port[, timeout [, connect_callback(result, connect_socket, host_addr, host_port[, timeout])]])`  
    *`--connect_callback is a once callback, blocking if connect_callback is nil.`*
	
7. 	`result, connect_socket = tcp.connects(sock_name[, timeout [, connect_callback(result, connect_socket, host_addr, host_port[, timeout])]])`  
	*`--connects to a windows named pipe or unix domain socket.`*
    *`--connect_callback is a once callback, blocking if connect_callback is nil.`*
	
8. 	`result, connect_socket = tcp.connect6(host_addr, host_port[, timeout [, connect_callback(result, connect_socket, host_addr, host_port[, timeout])]])`  
    *`--connect_callback is a once callback, blocking if connect_callback is nil.`*
	
9.  `result, buffer = tcp.read(socket[, timeout])`  
	
10. `result, buffer = tcp.read(socket[, read_callback(result, buffer, socket)])`  
    *`--read_callback is a continues callback, blocking if read_callback is nil.`*
	
11. `result, error = tcp.write(socket/fd, buffer_or_lstring[, send_callback(result, error, socket, buffer_or_lstring)/bool safety])`  
	*`--write a tcp socket or directly on a tcp fd(tcp.fd()) where the socket must enable shared write in advance.`*  
    *`--send_callback is a once callback and is safety assurance, blocking until buffer_or_lstring is sent only if safety is true.`*
	
12. `tcp.set_rwopt(socket, option_table)`  
    *`--set tcp_socket read and write options --e.g. { "read_head_endian" = "L", "read_head_bytes" = 2, "read_head_max" = 65535, "write_head_endian" = "L", "write_head_bytes" = 2, "write_head_max" = 65535,}`*
	
13. `option_table = tcp.get_rwopt(socket)`  
    *`--get tcp_socket read and write options --e.g. { "read_head_endian" = "L", "read_head_bytes" = 2, "read_head_max" = 65535, "write_head_endian" = "L", "write_head_bytes" = 2, "write_head_max" = 65535,}`*
	
14. `tcp.set_nodelay(socket, enable)`  
    *`--change tcp socket write 'nodelay' option. Enable nodelay if 'enable' is true or disable it if 'enable' is false.`*

15. `tcp.set_wshared(socket, enable)`  
    *`--change tcp socket write shared option. Enable write shared if 'enable' is true or disable it if 'enable' is false. tcp.set_rwopt will be choked or invalid if write shared is enabled. So if you want to call both tcp.set_wshared and tcp.set_rwopt, call tcp.set_rwopt first!`*

16. `local_addr = tcp.local_addr(socket)`  
	*`--socket can't be a listen socket.`*

17. `remote_addr = tcp.remote_addr(socket)`  
	*`--socket can't be a listen socket.`*

18. `local_port = tcp.local_port(socket)`  
	*`--socket can't be a listen socket`*

19. `remote_port = tcp.remote_port(socket)`  
	*`--socket can't be a listen socket.`*

20. `tcp.close(socket)` 
	*`--close the tcp socket directly.`*

21. `is_closed = tcp.is_closed(socket)`  
	*`--check whether the tcp socket is closed.`*

22. `fd = tcp.fd(socket)`  
	*`--get tcp socket lua fd`*

### context api
1.	`result, error = context.send(handle, data1[, ...])`  
    *`--send data1, data2, ... directy to context specified by handle noblocking.`*

2.	`result, query_data1, ... = context.query(handle, data1[, ... [, query_callback(result, query_data1[, ...])]])`  
    *`--query context specified by handle with data1, data2, ... and query_data1, query_data2, ... is the queried datas.`*  
	*`--query_callback is a once callback, blocking if query_callback is nil.`*

3.	`result, query_data1, ... = context.timed_query(handle, timeout, data1[, ... [, query_callback(result, query_data1[, ...])]])`  
    *`--query context specified by handle with data1, data2, ... in timeout seconds`*
	*`--query_callback is a once callback, blocking if query_callback is nil.`*
	
4.	`result, error = context.reply(handle, session, data1[, ...])`  
    *`--reply session(received by context.recv) context specified by handle with data1, data2, ...`*

5.	`result, recv_handle, session, recv_data1, ... = context.recv(handle[, timeout])`  
    *`--receive data from context specified by handle(receive data from all contexts if handle equals 0).`*  
    *`--recv_handle specifies the source context id. It's a query action if session >= 0, where you'd better reply this query action.`*
	
5.	`result, recv_handle, session, recv_data1, ... = context.recv(handle[, recv_callback(result, recv_handle, session, recv_data1, ...)])`  
    *`--receive data from context specified by handle(receive data from all contexts if handle equals 0).`*  
    *`--recv_handle specifies the source context id. It's a query action if session >= 0, where you'd better reply this query action.`*  
    *`--recv_callback is a continues callback, blocking if recv_callback is nil.`*  
	
6.	`result, error = context.wait(handle[, timeout[, callback(result, error, handle[, timeout])]])`  
    *`--wait context to quit or to be destroyed in blocking or nonblocking mode.`*  
    *`--callback is a once callback, blocking if callback is nil.`*

7.	`error = context.strerror(errno)`  
    *`--convert error number to error string. Error number is always the next argument after result in most apis.`*

8.	`handle = context.create(file_name[, arg1[, ...]])`  
    *`--create a new context with file_name as the context entry. arg1, arg2, ... will be the argument for the context.`*

9.	`handle = context.destroy([handle[, message]])`  
    *`--destroy a context specified by handle with a string message. You'll kill the context itself if handle is nil and message is a optional argument.`*

10.	`thread = context.thread()`  
    *`--return the running thread index.`*

11.	`context.winos`  
    *`--whether the running system is windows.`*

12.	`context.self`  
    *`--the running context id.`*

13.	`context.parent`  
    *`--the running context parent id.`*

### timer api	
1.	`timer.sleep(seconds)`  
    *`--blocking for seconds.`*

2.	`timer.timeout(seconds, ..., callback(...))`  
    *`--make a tiimeout callback in seconds.`*

3.	`timer.loop(interval, repeat_time, ..., callback(...))`  
    *`--make a repeated callback. The first time callback will be triggered in interval seconds and then repeated in repeat_time`*

### buffer api
