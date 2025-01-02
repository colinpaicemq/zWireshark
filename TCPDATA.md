# Capture AT-TLS encrypted text in clear text
##  Capturing a data trace of encrypted data
The TCPDATA module uses a documented TCPIP interface to collect data.  It can collect encrypted data in clear text

You can run it as a batch job or as a started task.
The userid running the program needs access to a SERVAUTH profile.  When I ran it without permission I got
```  
   ICH408I USER(START1  ) GROUP(SYS1    ) NAME(####################)
     EZB.TRCCTL.S0W1.TCPIP.OPEN CL(SERVAUTH)                        
     INSUFFICIENT ACCESS AUTHORITY                                  
     FROM EZB.TRCCTL.*.*.* (G)                                      
     ACCESS INTENT(READ   )  ACCESS ALLOWED(NONE   ) 
```

I used the following definitions 

```
permit  EZB.TRCSEC.*.*.ATTLS            - 
   CL(SERVAUTH) id(ADCDB) access(READ) 
permit  EZB.TRCCTL.S0W1.TCPIP.DATTRACE  - 
   CL(SERVAUTH) id(ADCDB) access(READ) 
permit EZB.TRCCTL.S0W1.TCPIP.OPEN  - 
   CL(SERVAUTH) id(ADCDB) access(READ) 
permit EZB.TRCCTL.*.*.* CL(SERVAUTH) id(ADCDB) access(READ) 
SETROPTS RACLIST(SERVAUTH) refresh 
```

See [SAF resource names for NMI resources](https://www.ibm.com/docs/en/zos/3.1.0?topic=enablement-saf-resource-names-nmi-resources).
## JCL to execute the program

```
//COLINC5    JOB 1,MSGCLASS=H,COND=(4,LE) 
// SET LOADLIB=COLIN.ZWIRESHA.LOAD 
//RUN      EXEC PGM=TCPDATA,REGION=0M,PARMDD=MYPARMS 
//STEPLIB  DD DISP=SHR,DSN=&LOADLIB 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=* 
//SYSERR   DD SYSOUT=* 
//* MYPARMS needs a / to split LE parms from program parms 
//*         needs a blank before each parm, because trailing blanks removed 
//MYPARMS DD * 
/
 --IP   10.1.0.2 
 --WAIT 60 
 --PAYLOAD 1024 
 --DEBUG 0 
 --PRINT A
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

--PAYLOAD
: This is the number of bytes the program processes

--IP
:  which IP V4 address to trace, in format 10.1.1.2

--IPV6
:  which IPV6 interface to trace, in format 2001:db8:8::9

--PORT
: which port value to trace

--COUNT
:  The number of records to trace before ending

--DEBUG
: If the value is greater than 0, it display additional information

--PRINT 
:  This takes a value
    . X : this prints the data in Hexadecimal. 
    . A : cconverts the data from ASCII to EBCDIC and displays it.  This is useful to see web browser raffic
    . E : displays the data in EBCDIC


You should specify one of  --IP, --IPV6, --PORT.


### Output file
The output file is written to the file in //SYSPRINT.




## Stopping the program
The program sits in a loop waiting for trace records, or until it times out.
You can stop it before it times out using the operator command
```
p COLINC5
```
The program listens for this, and shuts down.

## Example output

```
Data trace.  Data length 171.  ATTLS Clear Text.                                
>GPMSERVE  09:58:24.093466 Src 10.1.0.2 Port 41416 Dst 10.1.1.2 Port 8803       
GET /gpm/perform.xml?resource=",ADCDPL,SYSPLEX"&id=8D0EA0 HTTP/1.1              
Host: 10.1.1.2:8803                                                             
Authorization: Basic YWRjZGY6TkVXNDNEQVk=                                       
User-Agent: curl/8.5.0                                                          
Accept: */*                                                                     
```
Where

* Data Trace - this identifies the type of trace ( packet trace or data trace)
* Data length - the amount of data processed.   This will be less than or equal to the --PAYLOAD value
* ATTLS Clear Text - the data was encrypted with AT-TTLS   
* > Input (< is output )
* GPMSERVE - is the name of the job executing                                                                          