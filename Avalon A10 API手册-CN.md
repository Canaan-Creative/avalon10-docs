# Avalon A10 矿机 API说明手册 v0.1





###                                          版本说明

| 版本号 | 修改日期  | 说明       |
| ------ | --------- | ---------- |
| v0.1   | 2019-8-31 | 创建文档。 |
|        |           |            |


 

# 1.   综述

- API命令使用TCP短连接发送和接收。一次命令发送和返回在一个TCP连接中完成。
- API的发送和接收需要单线程执行，不支持并发。
- 必须等待上一条API命令返回结果，才能发起下一个API通信。
- API通信端口为4028。

# 2.   命令说明

## 2.1.  查询版本

| 命令说明                | 查询控制板版本相关信息。                                     |
| ----------------------- | ------------------------------------------------------------ |
| 命令格式                | version\|                                                    |
| 举例（使用linux socat） | echo -n "version\|"   \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none && echo |

 

## 2.2.  重启矿机

| 命令说明                | 无条件重启矿机。                                             |
| ----------------------- | ------------------------------------------------------------ |
| 命令格式                | ascset\|0,reboot,0                                           |
| 举例（使用linux socat） | echo -n   "ascset\|0,reboot,0" \| socat -t 30 stdio tcp:192.168.189.135:4028,shut-none   && echo |

 

## 2.3.  LED灯控制

| 命令说明                | 点亮/关闭矿机面板上白色LED。                                 |
| ----------------------- | ------------------------------------------------------------ |
| 命令格式                | ascset\|0,led,1-0              关灯   ascset\|0,led,1-1              开灯   ascset\|0,led,1-255             返回当前LED灯的状态。 |
| 举例（使用linux socat） | echo -n   "ascset\|0,led,1-255" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

 

## 2.4.  配置工作模式

| 命令说明                | 配置或查询矿机的工作模式（重启后生效）。                     |
| ----------------------- | ------------------------------------------------------------ |
| 命令格式                | ascset\|0,workmode,0   31T 普通模式   ascset\|0,workmode,1      37T高性能模式   ascset\|0,workmode,255    查询当前工作模式 |
| 举例（使用linux socat） | echo -n   "ascset\|0,workmode,1" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

 

## 2.5.  配置矿池信息

| 命令说明                | 配置矿机的矿池（重启后生效）。                               |
| ----------------------- | ------------------------------------------------------------ |
| 命令格式                | ascset\|0,setpool,<username>,<password>,<pooladdr>,<worker>,<workerpasswd> |
| 参数说明                | <username>矿机登录用户名（与web登录相同）   <password>矿机登录密码（与web登录相同）   <pooladdr>配置的矿池地址   <worker>配置的矿工名   <workerpasswd>矿工密码 |
| 举例（使用linux socat） | 设置矿机使用矿池stratum+[**tcp://btc.ss.poolin.com:443**](tcp://btc.ss.poolin.com:443)   使用矿工名cctrix.001并且密码为123   假设矿机的登录名和密码都是默认的root，root。   echo -n "ascset\|0,setpool,root,root,stratum+[**tcp://btc.ss.poolin.com:443,cctrix.001,123**](tcp://btc.ss.poolin.com:443,cctrix.001,123)" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

 

## 2.6.  关闭HASH电源

| 命令说明                | 关闭HASH电源使矿机进入关机状态，要回到工作状态请使用重启矿机API命令。 |
| ----------------------- | ------------------------------------------------------------ |
| 命令格式                | ascset\|0,hashpower,0                                        |
| 举例（使用linux socat） | echo -n   "ascset\|0,hashpower,0" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

## 2.7.  查询HASH电源状态

| 命令说明                | 读取电源状态。                                               |
| ----------------------- | ------------------------------------------------------------ |
| 命令格式                | ascset\|0,hashpower                                          |
| 结果解析举例            | 发送API得到如下返回：   STATUS=I,When=1052,Code=118,Msg=ASC   0 set info: **PS[0 1220 1314 140   1839 1308]**,Description=cgminer   4.11.1\|   对于PS字段解析如下：   0：错误码 ，当前无错误。（详见附录”电源错误码表“）   1220：提供给控制板的电压，12.2V。   1314：提供给HASH板的电压，13.14V   140：输出电流 140A   1839：输出功率 1839W   1308：对HASH板输出电源的设定值。 |
| 举例（使用linux socat） | echo -n   "ascset\|0,hashpower,0" \| socat -t 30 stdio   tcp:192.168.189.135:4028,shut-none && echo |

 

# 3.   附录  电源返回值表

| **Value** | **Definition** | **Notes**              |
| --------- | -------------- | ---------------------- |
| 0         | OK             | 一切正常               |
| 1         | Input_UV       | 输入欠压（AV电不正常） |
| 2         | OT1            | 过温保护               |
| 4         | OT2            | 过温保护               |
| 8         | OT3            | 过温保护               |
| 16        | OC_Pri         | 原边过流               |
| 32        | UV_out         | 输出欠压               |
| 64        | OC_out         | 输出过流               |
| 128       | CS_error       | N/A                    |
| 256       | OC_IOSA        | N/A                    |
| 512       | OC_IOSB        | N/A                    |
| 1024      | OC_IOSC        | N/A                    |
| 2048      | FAN_error      | 风扇故障               |

 

 