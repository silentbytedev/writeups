# Stratoshark - CSCG CTF 2025
- Challenge Name: __Stratoshark__
- Challenge category: __forensics__
- Challenge Difficulty: __easy__
- Challenge points: __138__
- Challenge Author: __dontrash__
- CTF Year and Date: __2025-03-01 - 2025-05-01__
- Author: __nebuc42__
- Date: __2025-05-12__

## Challenge Description


# Information Gathering
I first opened the file with wireshark and was very confused. While browsing, I could not imagine, that the <spärliche> information presented could lead to any flag. I ended up googling for “SysDig” [1]() and eventually found the “stratoshark”-tool (2)
![assets/stratoshark_01.png]
 
After installing and opening the provided pcapng – File with stratoshark there was much more information available. However, I was quickly overwhelmed with the amount of information available and had no idea of where to start after looking at the entries for a few minutes. Event
 
## Rabbit-Hole (1)
I hoped, some more googling will help and ended up analyzing the sysdig using the sysdig-tool
<user>
<examples of sysdig commands and>
 
## <einen Überblick verschaffen>
While the sysdig excursion did not yield any quick results, I turned back to stratoshark. Filtering did not yet make sense to me, because, I did not know, what to look for. Because stratoshark is a wireshark sibling, it also offers some statistic analysis of the file. The statistics of the current pcap yielded some interesting information
 
<screenshot of process-statistics>
 
Weitere statistiken?
 
## System
Es scheint sich um ein Linux system zu handeln, weil x,y processe vorhanden
 
## Process
I decided to have a quick look at some processes applying the display filters in stratoshark
Node `proc.name == node` The node process seems to run the vs code development process. This makes sense, as the challenge description yields that this is in fact the ‘development server’
Ssh `proc.name== ssh`: There were sessions from different src addresses (). Because there is no data nor user information available in the pcap, I quickly abandoned this path. Möglicherweise the users are connecting to the server using ssh and are tunneling the communication with the development server through ssh.
top `proc.name ==  top`. A this time of the analysis, it the presence of the top process made no sense to me. I assume, that one of the users logged in one of the ssh-sessions was monitoring the server, because they had <festgestellt> some unusual behaviour
 
## sharky process
The sharky process <zog> mein interesse auf sich. Eine google-suche erweckte den Eindruck, dass dies weder standard process noch ein bekannter prozess ist. Let’s look at it by filtering for this process `proc.name == sharky`
 
<screenshot sharky startup>
<screenshot connect and key>
<screenshot tmp file written>
<screenshot flag written>
<screenshot heartbeat>
 
Whois of he c2 IP address à XEROX
 
## Rabit Hole (2)
Filtering too much: Only looking at a single process war ein sackgasse, weil dadurch die Zusammenhänge und das zusammenspiel verloren geht. Das betrachten der Aktionen des sharky prozesses alleine führte nicht weiter.
 
## Active debugging session
After giving up digging further into this rabbit hole, I decided to ditch the display filter (proc.name == sharky) o be able to look at the arguments just before the sharky process has been started. It took me some time & scrolling before I suddenly realized a debugging session has been started by the sharky process
 
<screenshot PTRACE_ATTACH>
 
Decode the data injected into the process-memory of the top-process
<screenshot PTRACE_POKE>
 
<screenshot raw address/Bytes>
<decode script>
<decoded string>
 
## IO/operations
Now it seemed like a good idea, to do a deep investigation of the top process after its memory has been tampered with
`proc.name == top` 
 
## summary of reconnaissance
Aus dem vorliegenden pcap können wir folgendes herauslesen
sysdig is running on a linux system being used as a vs code service
mehrere ssh sessions zu verschiedenen IP addressen (möglicherweise ssh-tunnels)
ungewöhnlich ist ein laufender top-prozess
A suspiciuous `sharky` prozess, welcher im zweiten drittel (@ timestamp/id) gestartet wird. Nach dem Start verbindet er sich offensichtlich zu einem c2 server, wo er einen encryption key abholt und den erfolg seiner Mission (flag written) zurückmeldet, bevor er in einen Schlafmodus übergeht, in welchem er regelmässig (alle x sekunden) einen hartbeat sendet.
The memory of the top process has been altered by a debugging session started from within the sharky process (PTRACE_POKE) after the sharky process writes 40 bytes to a file in the tmp directory `/tmp/`
After the debugging session has been closed, the top process reads a library file and then writes an
 
# Solution
 
 
# Conclusion
Consider the hints of the challenge authors: “stratoshark” and the “wireshark guys”
When presented with a caputer file (netcap, sysdig) It might be good starting point to look at the statistics
Filtering the write action quickly yields interesting insights of io operations (files, networks) and possible zusammenhänge zwischen den überwchten prozesen
 
# References

1. <a name="1"></a>[SysDig](https://sysdig.com/)
2. <a name="2"></a>What is [Stratoshark](https://sysdig.com/learn-cloud-native/?what-is-stratoshark/)
3. [Fishing for Hackers](https://sysdig.com/blog/fishing-for-hackers/)
4. Sysdig chisel
3. Library injection
4. Debuging-operations
5. XOR