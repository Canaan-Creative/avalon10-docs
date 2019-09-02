# Avalon A10 API manual v0.1



### Release Notes

| Version | Date of modify | Notes            |
| ------- | -------------- | ---------------- |
| v0.1    | 2019-8-31      | Create document. |
|         |                |                  |
|         |                |                  |
|         |                |                  |



 

# 1.   summarize

- The API command uses TCP short connections to send and receive. Each command is sent and returned in one TCP connection.
- The API needs to be executed by single thread and does not support concurrency.
- You must wait for the API command finish in order to start the next API communication.
- API communication port is 4028.

# 2.   API Instructions

## 2.1.  Query controller version

| Note                   | Query control   board version information.                   |
| ---------------------- | ------------------------------------------------------------ |
| Format                 | version\|                                                    |
| Example(linux   socat) | echo -n "version\|"   \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none && echo |

 

## 2.2.  Reboot miner

| Note                   | Reboot miner.                                                |
| ---------------------- | ------------------------------------------------------------ |
| Format                 | ascset\|0,reboot,0                                           |
| Example(linux   socat) | echo -n   "ascset\|0,reboot,0" \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none   && echo |

 

## 2.3.  LED control

| Note                   | Turn on/off the   white LED on the miner panel.              |
| ---------------------- | ------------------------------------------------------------ |
| Format                 | ascset\|0,led,1-0              turn off   ascset\|0,led,1-1              turn on   ascset\|0,led,1-255             query state |
| example(linux   socat) | echo -n   "ascset\|0,led,1-255" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

 

## 2.4.  Set work mode

| Note                   | Configure or   query the working mode of the miner (effective after restart) |
| ---------------------- | ------------------------------------------------------------ |
| Format                 | ascset\|0,workmode,0   31T normal mode   ascset\|0,workmode,1      37Tperformance mode   ascset\|0,workmode,255    query current mode |
| Example(linux   socat) | echo -n   "ascset\|0,workmode,1" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

 

 

## 2.5.  Set pool information

| Note                   | Set pool information(effective   after restart).             |
| ---------------------- | ------------------------------------------------------------ |
| Format                 | ascset\|0,setpool,<username>,<password>,<pooladdr>,<worker>,<workerpasswd> |
| Arguments              | <username>  miner username(same   as web login)   <password>  miner password(same   as web login)   <pooladdr>  pool url address or IP address   <worker>   worker name   <workerpasswd>  worker password |
| Example(linux   socat) | Use pool :stratum+[**tcp://btc.ss.poolin.com:443**](tcp://btc.ss.poolin.com:443)   Worker name:    cctrix.001   Worker password   :  123   We assume both   user name and password are root.   echo -n   "ascset\|0,setpool,root,root,stratum+[**tcp://btc.ss.poolin.com:443,cctrix.001,123**](tcp://btc.ss.poolin.com:443,cctrix.001,123)" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

 

## 2.6.  Shut down HASH power

| Note                   | Shut down the   HASH power supply to make the miner enter the low power state. To return to   the working state, use the reboot API command. |
| ---------------------- | ------------------------------------------------------------ |
| Format                 | ascset\|0,hashpower,0                                        |
| Example(linux   socat) | echo -n "ascset\|0,hashpower,0"   \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none && echo |

## 2.7.  Query HASH power state

| Note                   | Read Hash power   status.                                    |
| ---------------------- | ------------------------------------------------------------ |
| JFormat                | ascset\|0,hashpower                                          |
| Result                 | We get reply as   below：   STATUS=I,When=1052,Code=118,Msg=ASC   0 set info: **PS[0 1220 1314 140   1839 1308]**,Description=cgminer   4.11.1\|   For this example   , each argument ：   0：errcode ，no-err.(For   more information, see Appendix "Power supply error Code Table")   1220：voltage to controller，12.2V.   1314：voltage to HASH boards，13.14V   140：output current 140A   1839：output power 1839W   1308：the setting voltage to HASH boards. |
| example(linux   socat) | echo -n   "ascset\|0,hashpower,0" \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none   && echo |

 

# 3.   Appendix :  Power supply error Code Table

| **Value** | **Definition** | **Notes**           |
| --------- | -------------- | ------------------- |
| 0         | OK             | All is OK.          |
| 1         | Input_UV       | Input voltage low.  |
| 2         | OT1            | Over hot            |
| 4         | OT2            | Over hot            |
| 8         | OT3            | Over hot            |
| 16        | OC_Pri         | Input over current  |
| 32        | UV_out         | Output voltage low  |
| 64        | OC_out         | Output over current |
| 128       | CS_error       | N/A                 |
| 256       | OC_IOSA        | N/A                 |
| 512       | OC_IOSB        | N/A                 |
| 1024      | OC_IOSC        | N/A                 |
| 2048      | FAN_error      | Fan failed          |

 

 