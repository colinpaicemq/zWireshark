# zWireshark
Produces a network trace file from TCPIP on z/OS which Wireshark can use.
For example 
![example of a ping to z/OS](https://colinpaiceblog.files.wordpress.com/2024/02/zwireshark-1.jpg?w=999)

New Function:  Produces a clear text data trace from AT-TLS encrypted sessions.  See documentation [here](./TCPDATA.md).



# Network packet trace
##  Capturing a trace
The TCPTRACE modules uses a documented TCPIP interface to collect packet trace data.  It writes it to a file which can be downloaded and used as input to wireshark.

You can run it as a batchjob or as a started task.
The userid running the program needs access to a SERVAUTH profile.  When I ran it without permission I got
```  
   ICH408I USER(START1  ) GROUP(SYS1    ) NAME(####################)
     EZB.TRCCTL.S0W1.TCPIP.OPEN CL(SERVAUTH)                        
     INSUFFICIENT ACCESS AUTHORITY                                  
     FROM EZB.TRCCTL.*.*.* (G)                                      
     ACCESS INTENT(READ   )  ACCESS ALLOWED(NONE   ) 
```

You need to define a specify profile or a generic profile EZB.TRCCTL.*.*.* in class SERVAUTH           and give the userid read access to it.
See [SAF resource names for NMI resources](https://www.ibm.com/docs/en/zos/3.1.0?topic=enablement-saf-resource-names-nmi-resources).
## JCL to execute the program

```
//COLINC5    JOB 1,MSGCLASS=H,COND=(4,LE) RESTART=RUN 
// SET LOADLIB=COLIN.ZWIRESHA.LOAD 
//RUN      EXEC PGM=TCPTRACE,REGION=0M,PARMDD=MYPARMS 
//STEPLIB  DD DISP=SHR,DSN=&LOADLIB 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
//PCAP     DD PATH='/tmp/x.pcap', 
//            PATHOPTS=(OWRONLY,OCREAT,OTRUNC), 
//            PATHMODE=SIRWXU 
//MYPARMS DD * 
/
 --WAIT 60 
 --INTERFACE TAP0 
 --PAYLOAD 1024 
 --DEBUG 0 
/* 
 --COUNT 1000 
 --IP   10.1.1.2 
 --IPV6 2001:db8::22b1:8b3:5051:d8db 
 --PORT 80 
// 
```

Where the parameters are (in upper case)


--WAIT 
: wait for the specified number of seconds before ending

--INTERFACE
:  which TCPIP interface to trace

--IP
:  which IP V4 address to trace, in format 10.1.1.2

--IPV6
:  which IPV6 interface to trace, in format 2001:db8:8::9

--PORT
: which local port value to trace

--COUNT
:  The number of records to trace before ending

--DEBUG
: If the value is greater than 0, it display additional information

You should specify one of --INTERFACE, --IP, --IPV6, --PORT.


### Output file
The output file is written to the file in //PCAP.

Download it to your work station in binary.


## Stopping the program
The program sits in a loop waiting for trace records, or until it times out.
You can stop it before it times out using the operator command
```
p COLINC5
```
The program listens for this, and shuts down.


## Installation
FTP the load/COLIN.ZWIRESHA.LOAD.XMIT to z/OS.  For example using
```
site quote  cyl pri=1 recfm=fb blksize=800 lrecl=80
bin
put COLIN.ZWIRESHA.LOAD.XMIT 'myid.ZWIRESHA.LOAD.XMIT'
quit 
```
then use
```
tso receive indsn('myid.ZWIRESHA.LOAD.XMIT')
``` 

# Change history
-  2024 Feb 24: Version 1.1 Add interface name to the printed output
-  2024 Feb 24: Version 1.1 Add copyright and version to the load module 
-  2024 Feb 25: Version 1.1 Fix problem in printhex caused by invalid length being used.  Simplify the write code. 
-  2024 July 6: Fix documentation problems
-  2024 Oct 8:  Used Locate mode instead of Move mode to allow for big buffers.
-  2024 Oct 8:  Fixed documentation problems with the wrong command options specified.  
-  2025 Jan 2:  Add support for TCPDATA - capturing and formatting of AT-TLS encrypted
-  2025 Jan 2:  Minor formatting enhancements