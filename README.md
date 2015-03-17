# BGE-netsync
A simple way to create a client/server system on the Blender game engine.



I everyone ! I'm currently on an open source game project and I want to make it multiplayer : http://blenderartists.org/forum/showthread.php?362226-Tron-R-reboot-reloaded-An-open-source-openworld-of-tron

my network module : https://github.com/jimy-byerley/BGE-netsync
I use python sockets and threading to create a server and a client.

file basic_client.py :
- class for basic client (implements a queue for receiving separated packets)
- class for basic server (implements the same queue, manage all client sockets)

file client.py
- uses the bge module
- class client, inheriting from client and thread
- thread method for receiving server requests and synchronizing bge datas (as object properties and locations)

file server.py
- don't use bge (server can run on a small machine without blender installed
- class server, inheriting from basic server and thread
- thread method to synchronize all datas and answer to clients requests

I run:
[CODE]
$ python3.2  server-test.py
$ blenderplayer  client-test.blend
[/CODE]

----- server output -----
[CODE][('192.168.1.10', 45619)]
{}
[('192.168.1.10', 45619)]
{}
[('192.168.1.10', 45619)]
{}
getpacket: 'registerloc\x00Cube'
getpacket: 'registerrot\x00Cube'
getpacket: 'registerprop\x00Cube\x00prop'
send 'setloc\x000\x000\x000\x00Cube'
send 'setrot\x000\x000\x000\x00Cube'
send 'setprop\x00Cube\x00prop\x00None'
send 'setloc\x000\x000\x000\x00Cube'
send 'setrot\x000\x000\x000\x00Cube'
send 'setprop\x00Cube\x00prop\x00None'
send 'setloc\x000\x000\x000\x00Cube'
Exception in thread Thread-1:
Traceback (most recent call last):
  File "/usr/lib/python3.2/threading.py", line 740, in _bootstrap_inner
    self.run()
  File "/home/jimy/tron-reboot/server/BGE-bug-socket/server.py", line 140, in run
    if data.position : self.sendall('setloc\0%d\0%d\0%d\0%s' % (data.position[0], data.position[1], data.position[2], obj))
  File "/home/jimy/tron-reboot/server/BGE-bug-socket/basic_connection.py", line 206, in sendall
    self.send(i, data)
  File "/home/jimy/tron-reboot/server/BGE-bug-socket/basic_connection.py", line 202, in send
    self.tunnels[index].send('{}{}{}'.format(START_MARK, str(data), END_MARK).encode())
socket.error: [Errno 32] Broken pipe

[('192.168.1.10', 45619)]
{'Cube': {pos=(0, 0, 0),         rot=(0, 0, 0),  parent=False,   properties={'prop': None}}}
[('192.168.1.10', 45619)]
{'Cube': {pos=(0, 0, 0),         rot=(0, 0, 0),  parent=False,   properties={'prop': None}}}
^CTraceback (most recent call last):
  File "server-test.py", line 12, in <module>
    sleep(1)
KeyboardInterrupt
[/CODE]


----- client output ----
[CODE]
Blender Game Engine Started
send queued
<------------------------------ time elapsed : 15s, the BG has been stopped by 'esc' in the 3d view
Blender Game Engine Finished
queue
send 'registerloc\x00Cube'
queue
send 'registerrot\x00Cube'
queue
send 'registerprop\x00Cube\x00prop'
receiving
getpacket
no packet
send queued
receiving
getpacket
getpacket: 'setloc\x000\x000\x000\x00Cube'
confirmed
Writing: /tmp/client-test.crash.txt
/usr/local/bin/blender : ligne 2 : 17951 Erreur de segmentation  /usr/local/share/blender/blender "$@"

[/CODE]

Blender is crashing because the script is trying to access datas outside the GE, but it is not the problem.
The problem is as you can see, When I start the game engine the script is staying on the line 46 from client.py just after printed 'send queued'
At line 48, the script shoud print 'queue' (printed, when the BG is released)

[CODE]
45:    # send all methods requests
46:    print('send queued')
47:    for i in range(len(self.queue)):
48:        print('queue') ## THE PROGRAM DOESN'T EXECUTE THIS LINE BEFORE THE GAME HAS CRASHED
49:        self.send(self.queue[0])
[/CODE]

The script so continues only after the game ended !


