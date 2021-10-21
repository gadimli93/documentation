# Chapter 9. Command Pipelining

Command pipelining is a function that delivers(pipelines) multiple collection commands to the cache server through
the "pipe" keyword and the cache server does not send each command's response to the client immediately
after processing it, but rather it stores the responses in the _replay queue_ and once processing the last command 
it sends the entire response stored in the replay queue to the client.
Compared to the previously delivered _N_ time requests-responses,
command pipelining has the advantage of significantly reducing network costs and overall latency
by sending request stream and response stream at once.

Command pipelining is currently only possible for some of the collection commands. Available command pipelining 
commands are as follows, and this applies only commands with simple response strings. Beware that the maximum 
number of commands that can be pipelined at once is limited to 500.

- lop commands - lop insert/delete
- sop commands - sop insert/delete/exist
- mop commands - mop insert/delete/update
- bop commands - bop insert/upsert/delete/update/incr/decr

As an example of performing command pipelining, if you want to add 10 elements toward the tail of a specific list, 
send `lop insert` commands continuously as shown below to the cache server. From the first command to the last command, all must be
connected using the "pipe" factor, and in the last command, the "pipe" factor must be omitted to express that it is the end of the pipelining.

```
 lop insert lkey -1 6 pipe\r\ndatum0\r\n
 lop insert lkey -1 6 pipe\r\ndatum1\r\n
 lop insert lkey -1 6 pipe\r\ndatum2\r\n
 lop insert lkey -1 6 pipe\r\ndatum3\r\n
 lop insert lkey -1 6 pipe\r\ndatum4\r\n
 lop insert lkey -1 6 pipe\r\ndatum5\r\n
 lop insert lkey -1 6 pipe\r\ndatum6\r\n
 lop insert lkey -1 6 pipe\r\ndatum7\r\n
 lop insert lkey -1 6 pipe\r\ndatum8\r\n
 lop insert lkey -1 6\r\ndatum9\r\n
```

Response string of command pipelining is as follows.

```
RESPONSE <count>\r\n
<STATUS of the 1st pipelined command>\r\n
<STATUS of the 2nd pipelined command>\r\n
...
<STATUS of the last pipelined command>\r\n
END|PIPE_ERROR <error_string>\r\n
```

In the RESPONSE line, \<count\> indicates the total number of results, and then the following lines represent the execution results
of each command respectively. Since the results are different for each command, refer to each one's description.
The last line indicates the state of pipelining performance and which will be one of the following.

- "END" - pipelining operation successfully executed.
- "PIPE_ERROR command overflow" - maximum number of pipelining commands exceeded 500. In this case, only up to 500 commands are processed
  as a single command pipelining and returned as a single response stream. Exceeded subsequent commands are not processed.
- "PIPE_ERROR memory overflow" - indicates a memory space insufficiency for pipelining process inside the ARCUS Cache Server.
  ARCUS Cache Server is unlikely to cause this error because space to contain results of 500 commands secured in advance.
  However, this error has been added in the case of unintended reasons. If this error occurs, command pipelining will be stopped at 
  that point and immediately, sends the response stream to the client. In this case, the response string of a last performed command
  is omitted from the response stream and, subsequent commands are not processed.
- "PIPE_ERROR bad error" - An important error message starting with "CLIENT_ERROR" and "SERVER_ERROR" occurs while executing
  a command with pipelining. As well as in this case, the command pipelining is stopped immediately and
  the response stream up to this point send to the client. Subsequent commands are not processed.
