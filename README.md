# Bolt Stub

Bolt Stub is a scriptable Neo4j server simulator designed as a testing resource for client software.
Scripts can be created against which unit tests can be run without the need for a full Neo4j server.

Synopsis:
```
$ bolt-stub --help
usage: bolt-stub [-h] [-l LISTEN_ADDR] [-t TIMEOUT] [-v] script [script ...]

Run a Bolt stub server. The stub server process listens for an incoming client 
connection and will attempt to play through a pre-scripted exchange with that 
client. Any deviation from that script will result in a non-zero exit code. 
This utility is primarily useful for Bolt client integration testing.

positional arguments:
  script

optional arguments:
  -h, --help            show this help message and exit
  -l LISTEN_ADDR, --listen-addr LISTEN_ADDR
                        The base address on which to listen for incoming 
                        connections in INTERFACE:PORT format, where INTERFACE 
                        may be omitted for 'localhost'. Each script (which 
                        doesn't specify an explicit port number) will use 
                        subsequent ports. If completely omitted, this defaults 
                        to ':17687'. The BOLT_LISTEN_ADDR environment variable 
                        may be used as an alternative to this option. Scripts
                        may also specify their own explicit port numbers.
  -t TIMEOUT, --timeout TIMEOUT
                        The number of seconds for which the stub server will 
                        run before automatically terminating. If unspecified, 
                        the server will wait for 30 seconds.
  -v, --verbose         Show more detail about the client-server exchange.
```

A server script describes a conversation between client and server in terms of the messages
exchanged. An example can be seen in the [`test/scripts/count.bolt`](test/scripts/count.bolt) file.

When the server receives a client message, it will attempt to match that against the next client message in the script; if found, this line of the script will be consumed.
Then, any server messages that follow will also be consumed and sent back.
When the client closes its connection, the server will shut down.
If any script lines remain, the server will exit with an error status; if none remain it will exit successfully.
After 30 seconds of inactivity, the server will time out and shut down with an error status.

#### Scripting 

Scripts generally consist of alternating client (`C:`) and server (`S:`) messages.
Each message line contains the message name followed by its fields, in JSON format.

Some messages, such as `RESET`, can be automatically (successfully) consumed if they are not relevant to the current test.
For this use a script line such as `!: AUTO RESET`.

An example:
```
!: BOLT 3
!: AUTO HELLO
!: AUTO RESET

C: RUN "RETURN {x}" {"x": 1} {}
   PULL_ALL
S: SUCCESS {"fields": ["x"]}
   RECORD [1]
   SUCCESS {}
```
