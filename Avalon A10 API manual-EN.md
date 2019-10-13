# Avalon A10 API manual v0.2



### Release Notes

| Version | Date of modify | Notes                                                        |
| ------- | -------------- | ------------------------------------------------------------ |
| v0.1    | 2019-8-31      | Create document.                                             |
| v0.2    | 2019-10-12     | Add a new API description and add an example of the usage scenario description and return data. At the same time, the description of common CGMiner API  is added. |
|         |                |                                                              |
|         |                |                                                              |



 

# 1.   summarize

- The API command uses TCP short connections to send and receive. Each command is sent and returned in one TCP connection.
- The API needs to be executed by single thread and does not support concurrency.
- You must wait for the API command finish in order to start the next API communication.
- API communication port is 4028.
- All the examples in this article use the socat command under linux. See the appendix 2 for the installation method of the socat tool.

# 2.   API Instructions

## 2.1.  Query controller version

| Note                    | Query control   board version information.                   |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | version                                                      |
| Example(linux   socat)  | echo -n "version"   \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none && echo |
| Scenes                  | You can call this API to query miner firmware version. For example, you need to confirm the original mine firmware version or want to know the version before the remote upgrade operation.  Use this interface to get the software and hardware version and the CGMiner version of the miner's control board. <br />The following is an introduction to the return value.<br />**VERSION** is the firmware software version.<br />**HWTYPE** is the hardware type.<br />**DNA** is the global unique number.<br />**MAC** is the MAC address. |
| Example of return value | STATUS=S,When=18633,Code=22,Msg=CGMiner versions,Description=cgminer 4.11.1\| VERSION,CGMiner=4.11.1,API=3.7,PROD=AvalonMiner 10xx,MODEL=10xx,HWTYPE=MM3v2,SWTYPE=MM320,VERSION=19090589_789f8e9t_965ebdct,DNA=0000000000000000,MAC=aaa5836a0f14,UPAPI=2 |

 

## 2.2.  Reboot miner

| Note                   | Reboot miner.                                                |
| ---------------------- | ------------------------------------------------------------ |
| Format                 | ascset\|0,reboot,0                                           |
| Example(linux   socat) | echo -n   "ascset\|0,reboot,0" \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none   && echo |
| Scenes                 | 1. You should reboot the miner when pool or network setting finished to make the configuration take effect.<br /> 2. If the hash boards have powered off remotely, it is necessary to reboot the miner when you want to restart mining. <br /> 3. When the firmware upgrading crashed and need to be retried, you need to reboot the miner before upgrading. |

 

## 2.3.  LED control

| Note                    | Turn on/off the   white LED on the miner panel.              |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | ascset\|0,led,1-\<cmd>              turn off   ascset\|0,led,1-1              turn on   ascset\|0,led,1-255             query state |
| Parameter Description   | \<cmd\>    0:  turn off         1:  turn on       255:  query state。 |
| example(linux   socat)  | echo -n   "ascset\|0,led,1-255" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |
| Scenes                  | When you need to pick out one or several miners of specific IP in many miners, you can use LED control API to make them different. Turn on the LED will turn the LED to white, and turning off the LED will restore the previous LED color . |
| Example of return value | **When you are querying the status of the LED，return value would be**:<br />STATUS=I,When=18798,Code=118,Msg=ASC 0 set info: LED[0],Description=cgminer 4.11.1<br />**"LED[0]" means the white LED is off，"LED[1]" means the white LED is on.**<br /><br />**When you are setting the LED status，For example,Turn on the LED，the return value would be**：<br />STATUS=S,When=18984,Code=119,Msg=ASC 0 set OK,Description=cgminer 4.11.1 |

 

## 2.4.  Set work mode

| Note                    | Configure or   query the working mode of the miner (effective after restart) |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | ascset\|0,workmode,\<mode\><br />0   31T normal mode   ascset\|0,workmode,1      37Tperformance mode   ascset\|0,workmode,255    query current mode |
| Parameter Description   | \<mode\>   0：normal mode，   1：high performance mode      255： query current work mode |
| Example(linux   socat)  | echo -n   "ascset\|0,workmode,1" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |
| Scenes                  | Switch the working mode of the mining machine, for example, when it is necessary to save power, configure it to the normal mode. Configure high performance mode when high Hash rate is required. Note that after switching modes, you need to reboot the miner to make the new working mode take effect. |
| Example of return value | If you set the miner to High performance mode, the return value would be：<br />STATUS=I,When=19189,Code=118,Msg=ASC 0 set info: WORKMODE[1],Description=cgminer 4.11.1<br />**"WORKMODE[1]"means high performance mode，if you set the miner to normal mode，here would be "WORKMODE[0]".<br />**<br />When you query the work mode, the response data as follow：<br />STATUS=I,When=19313,Code=118,Msg=ASC 0 set info: WORKMODE[1],Description=cgminer 4.11.1<br /> |

 

 

## 2.5.  Set pool information

| Note                    | Set pool information(effective   after restart).             |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | ascset\|0,setpool,\<username\>,\<password\>,\<pooladdr\>,\<worker\>,\<workerpasswd\> |
| Arguments               | \<username\>  miner username(same   as web login)   <br />\<password\>  miner password(same   as web login)   <br />\<pooladdr\>  pool url address or IP address   <br />\<worker\>   worker name   <br />\<workerpasswd\>  worker password |
| Example(linux   socat)  | Use pool :stratum+[**tcp://btc.ss.poolin.com:443**](tcp://btc.ss.poolin.com:443)   Worker name:    cctrix.001   Worker password   :  123   We assume both   user name and password are root.   <br />echo -n   "ascset\|0,setpool,root,root,stratum+[**tcp://btc.ss.poolin.com:443,cctrix.001,123**](tcp://btc.ss.poolin.com:443,cctrix.001,123)" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |
| Scenes                  | When you need to modify the mine, miner, and miner passwords for the miner, you need to call this API . <br />Note: The configuration will take effect after reboot. |
| Example of return value | STATUS=I,When=8337,Code=118,Msg=ASC 0 set info:<br/>pool success set to stratum+tcp://btc.ss.poolin.com:443<br />worker is cctrix.001<br />workerpassword is 123<br />Please reboot miner to make config work.<br />,Description=cgminer 4.11.1 |

 

## 2.6.  Shut down HASH power

| Note                    | Shut down the   HASH power supply to make the miner enter the low power state. To return to   the working state, use the reboot API command. |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | ascset\|0,hashpower,0                                        |
| Example(linux   socat)  | echo -n "ascset\|0,hashpower,0"   \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none && echo |
| Scenes                  | If you need to shut down the mining machine remotely, you can use this API. Note that using this interface only turns off the power output to the HASH board. The power to the control board and fans will not be cut off. If you need to restart to hash , you need to call the reboot API . |
| Example of return value | STATUS=I,When=20159,Code=118,Msg=ASC 0 set info: Power off hash now ...,Description=cgminer 4.11.1 |

## 2.7.  Query HASH power state

| Note                    | Read Hash power   status.                                    |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | ascset\|0,hashpower                                          |
| Result                  | We get reply as   below：   <br />STATUS=I,When=1052,Code=118,Msg=ASC   0 set info: **PS[0 1220 1314 140   1839 1308]**,Description=cgminer   4.11.1\|   <br />For this example   , each argument ：   <br />0：errcode ，no-err.(For   more information, see Appendix "Power supply error Code Table")   <br />1220：voltage to controller，12.2V.   <br />1314：voltage to HASH boards，13.14V   <br />140：output current 140A  (Just the current to hash boards) <br />1839：output power 1839W   (Just the power to hash boards)<br />1308：the setting voltage to HASH boards. |
| example(linux   socat)  | echo -n   "ascset\|0,hashpower" \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none   && echo |
| Scenes                  | This interface is called when the power state needs to be queried. The output voltage, current, power, and power state of the power supply (normal or overcurrent, etc.) can be read. |
| Example of return value | STATUS=I,When=20136,Code=118,Msg=ASC 0 set info: PS[0 1196 1284 230 2953 1284],Description=cgminer 4.11.1 |

 

# 3.  Description of common CGMiner APIs 

## 3.1.  Query current pool

| Note                    | Query current pool information                               |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | pools                                                        |
| example(linux   socat)  | echo -n "pools“ \| socat -t 300 stdio tcp:192.168.193.226:4028,shut-none && echo |
| Scenes                  | This interface is called when you need to query the information for the current working of the miner. You can get information about the pool address, miner name, current difficulty, rejection rate, etc. See the example of the results returned below. |
| Example of return value | STATUS=S,When=17592,Code=7,Msg=1 Pool(s),Description=cgminer 4.11.1\|POOL=0,URL=stratum+tcp://btc.ss.poolin.com:443,Status=Alive,Priority=0,Quota=1,Long Poll=N,Getworks=654,Accepted=1701,Rejected=2,Works=204480,Discarded=0,Stale=0,Get Failures=0,Remote Failures=0,User=cctrix.001,Last Share Time=17590,Diff1 Shares=209377280,Proxy Type=,Proxy=,Difficulty Accepted=206045184.00000000,Difficulty Rejected=196608.00000000,Difficulty Stale=0.00000000,Last Share Difficulty=131072.00000000,Work Difficulty=131072.00000000,Has Stratum=true,Stratum Active=true,Stratum URL=btc.ss.poolin.com,Stratum Difficulty=131072.00000000,Has Vmask=true,Has GBT=false,Best Share=459832747,Pool Rejected%=0.0953,Pool Stale%=0.0000,Bad Work=0,Current Block Height=599045,Current Block Version=536870912 |

## 3.2.  Query the summary information

| Note                    | Query the summary information                                |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | summary                                                      |
| example(linux   socat)  | echo -n "summary"   \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none && echo |
| Scenes                  | This interface is called when you need to query the work information summary. You can get the miner's running time (Elapsed), average hash rate in 1 minute, 5 minutes, 15 minutes, the number of works accepted and rejected by the pool, etc. See the return information example below for details. |
| Example of return value | SUMMARY,Elapsed=17994,MHS av=51278810.20,MHS 30s=51423478.12,MHS 1m=51209334.97,MHS 5m=51088640.16,MHS 15m=51312058.99,Found Blocks=0,Getworks=669,Accepted=1740,Rejected=2,Hardware Errors=10,Utility=5.80,Discarded=0,Stale=0,Get Failures=0,Local Work=419458,Remote Failures=0,Network Blocks=24,Total MH=921456687985.0000,Work Utility=716041.36,Difficulty Accepted=211156992.00000000,Difficulty Rejected=196608.00000000,Difficulty Stale=0.00000000,Best Share=459832747,Device Hardware%=0.0000,Device Rejected%=0.0916,Pool Rejected%=0.0930,Pool Stale%=0.0000,Last getwork=0 |

##  3.3.  Query detail information

| Note                    | Query miner's detail information                             |
| ----------------------- | ------------------------------------------------------------ |
| Format                  | estats                                                       |
| example(linux   socat)  | echo -n "estats"   \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none && echo |
| Scenes                  | This interface is called when the detailed working status needs to be queried. Can get the running time (Elapsed), working status and the number of working HASH boards (SYSTEMSTATU), theoretical hash rate power (GHSmm), 1 hour average hash rate (GHSavg), ambient temperature (Temp), maximum chip temperature (TMax), average chip temperature (TAvg), fan speed (Fan1, Fan2), fan speed percentage (FanR), average chip voltage (Vo), frequency configuration (SF0, SF1) per HASH board, The temperature of each chip on the HASH board (PVT_T0, PVT_T1), the voltage of each chip on each HASH board (PVT_V0, PVT_V1), etc. See the return information example for details. |
| Example of return value | STATUS=S,When=6397,Code=70,Msg=CGMiner stats,Description=cgminer 4.11.1\|STATS=0,ID=AVA100,Elapsed=6354,Calls=0,Wait=0.000000,Max=0.000000,Min=99999999.000000,MM ID0=Ver[1066-19101244_fe411a8_71edeca] DNA[020100002c2af618] NETFAIL[0 0 0 0 0 0 0 0] SYSTEMSTATU[Work: In Work, Hash Board: 3 ] Elapsed[6355] MW[1443554 1443536 1443615] LW[4330705] MH[0 2 0] HW[2] DH[0.736%] Temp[25] TMax[74] TAvg[66] Fan1[3671] Fan2[3635] Fan3[3564] Fan4[3623] FanR[55%] Vo[336] PS[0 1217 1279 221 2834 1280] PLL0[510 1002 2072 3142] PLL1[1353 2097 2323 953] PLL2[628 1079 2043 2976] GHSmm[51062.40] GHSavg[50291.83] WU[702568.70] Freq[632.65] Led[0] MGHS[16645.36 16761.64 16884.83] MTmax[74 72 74] MTavg[65 63 66] TA[342] ECHU[512 512 512] ECMM[0] SF0[575 600 625 650] SF1[575 600 625 650] SF2[575 600 625 650] PVT_T0[ 59  58  55  56  60  62  63  62  58  59  62  64  64  62  59  62  66  67  67  67  63  61  64  66  66  66  62  61  65  66  66  64  61  61  65  66  66  63  61  59  63  65  65  63  59  58  62  64  62  61  57  56  60  59  64  67  67  68  68  66  67  69  68  69  70  67  67  70  70  72  71  69  70  73  74  71  73  69  70  72  72  72  72  69  70  72  72  71  72  71  70  71  72  69  71  67  67  70  69  68  70  66  65  68  68  66  66  64  63  66  65  64  63  61] PVT_T1[ 57  55  52  53  58  59  61  60  56  56  60  61  63  62  58  61  63  65  64  63  60  58  63  64  65  62  59  59  62  64  65  63  60  61  65  66  64  63  58  59  62  62  62  61  59  57  61  61  60  61  56  54  57  59  63  63  64  65  65  63  64  68  68  67  68  66  65  68  68  71  70  68  67  72  72  71  71  67  66  69  70  69  69  67  66  69  69  69  69  66  68  69  71  69  68  64  64  66  67  66  67  64  62  65  65  64  63  61  61  64  64  64  64  61] PVT_T2[ 61  59  56  56  61  63  65  63  59  60  64  66  67  65  62  62  67  68  67  66  61  60  66  66  67  65  61  61  65  67  67  66  62  64  67  68  67  67  63  62  67  67  66  65  63  60  64  66  64  63  60  56  58  61  65  67  69  69  69  66  69  71  71  72  71  70  70  73  72  72  73  70  71  73  73  72  72  70  70  73  73  74  73  70  70  73  74  72  73  69  70  72  72  71  71  69  70  72  70  71  70  68  67  69  70  68  67  65  65  68  67  70  68  64] PVT_V0[333 335 336 330 332 331 329 330 331 327 330 330 328 327 327 327 327 327 326 325 328 325 326 326 323 323 324 327 327 327 330 329 331 322 323 322 324 324 325 324 327 325 325 325 327 328 328 329 330 331 331 328 331 333 329 326 325 324 327 328 321 321 324 318 323 323 321 317 321 319 320 318 320 319 320 316 317 319 320 319 322 319 321 320 316 318 319 319 320 322 319 318 320 318 320 319 323 321 324 318 321 322 322 322 323 322 324 324 321 322 323 322 326 328] PVT_V1[328 330 331 330 330 330 326 327 326 327 326 327 330 329 333 323 326 325 326 325 327 325 324 326 328 328 329 325 327 326 325 324 324 330 329 330 329 329 331 327 325 327 324 325 324 327 328 326 333 332 333 334 335 337 333 328 329 323 328 329 328 327 327 319 321 320 319 318 316 311 312 312 315 317 321 314 317 320 323 322 327 320 324 324 325 325 324 323 323 324 324 324 322 319 323 323 316 315 318 321 324 325 327 324 326 322 324 324 325 325 328 323 326 326] PVT_V2[339 340 342 327 328 328 328 329 331 326 326 325 334 334 333 326 325 325 327 327 328 330 330 331 325 325 327 325 324 324 322 321 322 324 323 325 324 325 326 324 325 324 326 326 325 329 330 327 332 331 331 328 329 332 330 327 324 321 323 323 324 322 318 319 318 317 319 319 320 321 321 321 316 317 317 320 322 322 320 318 321 316 320 323 320 323 326 316 317 317 322 321 320 316 317 318 320 322 324 328 328 328 325 322 321 324 324 324 322 322 323 319 322 324] MW0[198 209 215 214 248 223 222 230 211 231 219 230 199 218 188 218 203 226 215 218 226 217 213 246 229 231 208 208 208 199 222 230 226 196 208 193 214 212 210 201 206 219 229 217 197 208 200 214 193 245 215 207 203 212 221 190 209 238 194 240 209 219 213 197 192 186 187 207 216 215 213 241 225 186 209 201 200 226 197 201 235 202 221 201 209 183 218 213 184 196 197 194 187 203 186 229 228 201 196 189 201 210 211 187 215 202 226 208 219 197 217 213 215 230] MW1[212 219 229 202 219 191 204 203 232 214 204 191 200 229 192 234 215 230 207 210 190 200 222 178 219 230 227 226 223 231 232 204 244 221 209 217 245 229 234 207 225 218 197 202 231 214 220 235 230 232 216 206 202 233 215 218 244 224 224 181 191 243 199 213 223 223 195 199 177 185 191 206 182 181 204 197 216 214 208 212 228 226 218 211 218 219 229 213 217 181 221 211 195 200 210 209 197 211 195 243 210 205 203 237 206 216 209 202 213 214 207 213 187 194] MW2[238 217 226 195 218 220 216 226 184 193 208 172 212 207 231 203 207 245 212 238 246 205 218 220 246 222 210 191 197 202 192 237 179 203 195 209 197 223 201 229 217 210 211 218 200 245 229 217 225 190 221 206 216 238 216 244 222 212 197 222 208 209 208 231 211 198 184 243 210 234 209 231 200 192 196 219 241 192 216 232 212 221 209 225 192 221 186 234 212 218 211 212 202 204 205 207 209 208 227 214 231 234 191 227 240 194 243 230 205 203 226 197 224 213] CRC[0 1 0] POW_I2C_CONN[SUCCES] HS_RCD0[Tenv:23 Tavg:65 Fan1:3407 Fan2:3407 V12:1217 Vout:1277 P_I:220 P_P:2809] HS_RCD1[Tenv:23 Tavg:65 Fan1:3272 Fan2:3320 V12:1217 Vout:1277 P_I:220 P_P:2809] HS_RCD2[Tenv:23 Tavg:65 Fan1:3280 Fan2:3316 V12:1217 Vout:1277 P_I:220 P_P:2813] HS_RCD3[Tenv:23 Tavg:66 Fan1:3369 Fan2:3273 V12:1217 Vout:1277 P_I:221 P_P:2826] HS_RCD4[Tenv:24 Tavg:66 Fan1:3448 Fan2:3340 V12:1217 Vout:1279 P_I:221 P_P:2834] MEMFREE[269880 B] FACOPTS0[] FACOPTS1[] WORKMODE[1],MM Count=1,Smart Speed=1,Connection Overloaded=false,Voltage Level Offset=0,Nonce Mask=25 |

## 



# Appendix  1 :  Power supply error Code Table

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

 

##  Appendix  2 :  Socat tool installation method under Linux

### 1. On Linux distributions such as RedHat, Fedora, and CentOS

​         Make sure your linux system can connect to the internet. If you are logged in to the terminal with a non-root user, you need to make sure your users are in the sudo authorization list (you can contact the operating system administrator for confirmation).

​       Enter "sudo yum install socat" in the terminal and press Enter. Then enter the login password as prompted. In the subsequent prompts, select "yes" or enter "y" to complete the installation.

### 2. On Linux distributions such as Debian，Ubuntu，and Kali

​      Make sure your linux system can connect to the internet. If you are logged in to the terminal with a non-root user, you need to make sure your users are in the sudo authorization list (you can contact the operating system administrator for confirmation).

​        Enter "sudo apt install socat" in the terminal and press Enter. Then enter the login password as prompted. In the subsequent prompts, select "yes" or enter "y" to complete the installation.