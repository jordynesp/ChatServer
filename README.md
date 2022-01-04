# ChatServer
An operating systems class assignment that uses a server and sender/receiver clients to communicate using TCP. Built from https://beej.us/guide/bgnet/ examples.

TO RUN:

- type 'make'
- set up server by typing 'server'

- set up sender(s) by typing 'sender <hostname> <port>'
    - you can use port 30420 (as specified by sender port in server)

- set up receiver(s) by typing 'receiver <hostname> <port>'
    - you can use port 30421 (as specified by receiver port in server)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

OVERVIEW:

server.c:
- we use ports 30420 and 30421 (specific ports allowed by our uni)
- we create a global buffer with a lock and condition variable
- we create two threads to do the bulk of the work: one for setting up 
sender connections and one for setting up receiver connections

- the sender connection thread will spawn a new detached thread for 
every new sender that connects to the server
    - this detached thread will wait to receive a message from its sender 
    connection, and if it gets one, it will acquire the buffer lock, update 
    the buffer, unlock, and then broadcast to all receiver detached threads 
    that a message has arrived
    - this will repeat until the sender disconnects from the server

- the receiver connection thread will spawn a new detached thread for every 
new receiver that connects to the server
    - this detached thread will begin a loop, acquire the buffer lock at 
    the beginning of every iteration, and then wait on a condition, which 
    happens when a message is received by a sender detached thread
    - once it is awoken and has the lock again, it proceeds to send a message 
    to its receiver client, unlock, and then begin the loop again
    - this will repeat until the receiver disconnects from the server

- the two main threads are initialized and called in main, and then everything 
is destroyed in main when those threads return

sender.c:
- takes a hostname and port as command line arguments
- makes a connection to that port
- begins a loop of reading a line from standard input and sending each line 
to the socket
    - if the user ctrl+c program or types "exit", the program terminates

receiver.c: 
- takes a hostname and a port as command line arguments
- makes a connection to that port
- begins a loop of receiving bytes from the server and printing each line to 
output
    - if the connection is lost, the program terminates
    - the program can also terminate with ctrl+c

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LIMITATIONS:

- we can only send/receive/process messages with a size equal to the BUFSIZ 
macro (which is machine dependent)
- if a receiver client disconnects, the server will not know about it until 
after it receives two more messages (a result of the TCP connection)
- if the server terminates, the receivers will terminate, but the senders 
will only terminate after trying to send two more messages (a result of the 
TCP connection)
- the server can successfully process most normal standard input (including 
piping), HOWEVER, it cannot handle having a file redirected as standard input
- the BACKLOG macro means there will be only a max of 10 pending connections 
in the queue
- if the ports defined as macros in the server are in use, connections are not 
possible until they are manually changed within the server to free ports
