# 魔豆路由器 Web Services API 规格文档

## 基本说明

- 系统操作处于锁的状态下返回 code=－1
- 身份鉴权基于 cookies
- 未登录状态下访问需要 auth 的 API 返回 403 状态码
- 单位：流量的单位（kbps），磁盘容量单位（MB），时间单位（s）
- 所有 POST 的请求的返回值的格式都为 JSON

基础返回格式：

```
{
  "code":number,         // 返回代码，0为成功，其他失败
  "msg":string             // 可能存在的出错消息
}
```

## 登录

* /api/auth/login

 post 需要登录验证：否

```
{
  "password":string
}
```

## 版本升级

### 获取当前版本信息

* api/system/get_version_info
  
  get 不需要auth

  return
  <pre>
    {
      "version1": "1.0.5 Build 130608",  // 当前软件版本
      "version2": "WR841N 10.0"          // 当前硬件版本
    }
  </pre>


### 检查是否有新版本

* api/system/check_remote_version_upgrade
  
  get

  return
  <pre>
    {
      "code": 0, // 0->有新版本，3->read json faild, 4->已经是最新版
      "msg": "",
      "	": [0-9]*,
      "filename": ***.bin,
      "version": ****, 
      "releasenote": "release note<br>release note<br>release note<br>release note<br>release note<br>release note"
    }
  </pre>


### 下载新固件

* api/system/download_version_upgrade
  get

  return
  <pre>
    {
      "code": 0,
      "msg": "",
    }
  </pre>

### 查看下载进度

* api/system/check_download_progress(jsondata)

  post  jsondata[filename]
  			jsondata[filesize]

  return
  <pre>
    {
      "code": 0|1|2,        // (0 -> success, 1-> running ,2->没有mount /data, 3->读取latestversion失败 4-> 解析json失败 5->存储空间不足 6-> 下载失败 7->md5校验失败 8->link创建失败 9->自动升级正在下载 10->正在升级中不能下载 11->更名失效)
      "msg": "",
      "percent": "xx",
      "stage": 0        // (0->pre check, 1-> download, 2->post check)
    }
  </pre>

### 取消下载

* api/system/cancel_download

  get

  return
  <pre>
    {
      "code": 0|1,
      "msg": ""
    }
  </pre>


### 进行版本升级

* api/system/upgrade_version

  get

  return
  <pre>
    {
      "code": 0,  // 0|1(0,成功；１,失败)
      "msg": "" // 
    }
  </pre>

### 获取当前升级百分比
  
* api/system/check_upgrade_progress

  get
  
  return
  <pre>
  {
  "code": 0|1|2,          //(0-> success,
                                          2->faild, upgraded nothing, Error
                                          3->faild, upgraded safe only, Error
                                          4->faild, upgraded vm only, Error
                                          5->faild, upgraded sys only, Error
                                          6->faild, upgraded safe and vm, Error
                                          7->faild, upgraded safe and sys, Error
                                          8->faild. 自动下载正在进行
      9->faild, 数字签名出错
                                          -1->progress)
  "percent": 10           // number 表示升级进度10% (0-100)
  "msg":"xx",
  "stage": 0               // (0->uboot, 1->check image, 2->safe, 3->vm, 4->sys, 5->sys check, -1->wait)
}
  </pre>



## 重启

### 正常重启

* api/system/reboot

  get

  return
  <pre>
    {
      "code": 0,
      "msg": ""
    }
  </pre>

### 重启进安全模式

* api/system/safe_reboot

  get

  return
  <pre>
    {
      "code": 0,
      "msg": ""
    }
  </pre>

  
### 恢复出厂设置

*api/system/reset_config
get  需要auth 


## 背光控制 

### 锁定背光，保持常亮

*api/system/lock_backlight
 
### 解锁背光，停止保持

*api/system/release_backlight

### 唤醒背光

*api/system/wakeup_backlight

## 防蹭网

* api/security/get_config
method: get
return 
{
  “code”: 0,
   “enabled”:true             (type: boolean)        // 防蹭网是否开启
}

* api/security/set_config
method: post
post data
{
  “enabled”: true            (type:boolean)   // 是否开启防蹭网
}

post 需要登录验证：否
* api/security/request_permission         // 请求上网权限
method: post
post data:
{
  “username”: “aaa”    (type:string)    // 用户名字
}
post 需要登录验证：否
* api/security/check_permission      // 检查上网权限
method: get

return : 
{
  “code”: 0,                  （type:number）         // 0 -> 允许上网，1->不允许上网,2->等待主人处理 -1 ->系统内部错误
}



### WAN 口设置


* api/wan/get_info  (获取当前wan设置)

 get 需要登录验证：是
<pre>
{
  "type":"STATIC", // 当前连接方式( DHCP, PPPOE, STATIC, wireless-repeater)
  "ip":"192.168.1.12",
  "mask":"182.168.1.1",
  "gateway":"255.255.255.0",
  "dns1":"8.8.8.8",
  "dns2":"8.8.4.4",
  "mtu":2,
  "stp": true,
  "account":"account",    // 如果当前是PPPOE
  "password":"password",   // 如果当前是PPPOE
 "macCloneEnabled":true, //是否开启Macclone
 "macCloneMac":"40:6c:8f:2d:6c:3b" ,//MAC CLONE mac
"uptime":"22486" 路由器运行时间
}
</pre>

* /api/wan/clientmacaddr
   获取client macaddr,专门给mac clone feature 使用 
   {"macaddr":"40:6C:8F:2D:6C:3A"}

* api/wan/get_info/dhcp  (获取dhcp设置)

 get 需要登录验证：是
<pre>
{
  "dns1":"8.8.8.8",
  "dns2":"8.8.4.4",
  "mtu":2,
  "stp": true
}
</pre>

* api/wan/get_info/pppoe  (获取pppoe设置)

 get 需要登录验证：是
<pre>
{
  "account":"account",  
  "password":"password",
  "pppoe_method": "KeepAlive" // 连接模式(KeepAlive, OnDemand, Manual)
  "pedial_period": 60         // 连接断开xx秒后尝试重拨,单位(秒) 当前KeepAlive
  "idle_time": 5              // 无流量时xx分钟后断开,单位(分) 当前OnDemand
  "status": -1/0/1/2/3/4/5/6/7// -1: PPPoE暂时无状态；
                              // 0: 连接已成功；
                              // 1: 用户名／密码错误；
                              // 2: 连接已断开; 
                              // 3: 不允许本帐户在此时间登录; 
                              // 4: 帐户已禁用;
                              // 5: 密码已过期;
                              // 6: 帐户没有远程访问权限;
                              // 7: 未知错误.
}
</pre>

* api/wan/get_info/static  (获取static设置)

 get 需要登录验证：是
<pre>
{
  "ip":"192.168.1.12",
  "mask":"182.168.1.1",
  "gateway":"255.255.255.0",
  "dns1":"8.8.8.8",
  "dns2":"8.8.4.4",
  "mtu":2,
  "stp": true
}
</pre>



* api/wan/set_config  (设置wan口)

  post 需要登录验证：是
  <pre>
{
  "type":"STATIC", // 当前连接方式( DHCP, PPPOE, STATIC, wireless-repeater)
  "ip":"192.168.1.12",
  "mask":"255.255.255.0",
  "gateway":"192.168.1.1",
  "dns1":"8.8.8.8",
  "dns2":"8.8.4.4",
  "mtu":2,
  "stp": true,
  "account":"account",    // 如果当前是PPPOE
  "password":"password"   // 如果当前是PPPOE
  "pppoe_method": "KeepAlive" // 连接模式(KeepAlive, OnDemand, Manual)
  "pedial_period": 60         // 连接断开xx秒后尝试重拨,单位(秒) 当前KeepAlive
  "idle_time": 5              // 无流量时xx分钟后断开,单位(分) 当前OnDemand
 "macCloneEnabled":true, //是否开启Macclone
  "macCloneMac":"40:6c:8f:2d:6c:3b" //MAC CLONE mac
}
return
{
“code”: 0,          // (0->设置成功，1-> 正在设置，-1 ->已有全局设置锁)
“msg”: “xx”
}
</pre>

* api/wan/is_internet_available

 get : 需要登录验证：是
<pre>
{
  "code":0                // 取得外网是否正常可用(0->网通， 1->（不通）不能解析域名，２->（不通）不能到达网关， -1->等待)

}
</pre>

* api/wan/get_traffics

  get : 需要登录验证：是
<pre>
{
  "up":number,               // 取得自系统启动以来，上行数据的总量(单位字节）
  "up_str":"number",             // up 值的字符串形式 例如: "12345678"
  "down":number              // 取得自系统启动依赖，下行数据的总量（单位字节）
  "down_str":"number"              //  down 值的字符串形式 例如: "12345678"
"code":0, 成功
"tx_rate":0, 发丢包率
"rx_dropped":0, 收丢包

"rx_packets":270649, 收包
"tx_packets":282499, 发包
"rx_rate":0, 收丢包率
"tx_dropped":0 发丢包


</pre>


### wifi

* api/wifi/get_config 取得当前wifi的配置信息

  get 需要登录验证:是
  <pre>
{
  "2g": {
    "enabled":true,                                     // 2.4g开关    RadioOff
    "ssid": "ssid1",                                    // 名称 SSID1(长度1-32字符)
    “broadcastssid”:true,                                    // 是否广播SSID
    "security_mode": "WPAPSKWPA2PSK"      // Security Mode
     "encrypt": "TKIP",                          // WPA Algorithms(TKIP,AES ,TKIPAES)  EncrypType
    "password": "123456kskdajksdasdasdasdasdasdasda",   // 密码   AuthMode（长度8-64字符）
    "power": 20,                                        // 无线信号功率	TXPower
    "channel": 6,                                       // 信道                         Channel
    "net_type": 9,       (0,1,4,6,9,)                          // 网络模式              WirelessMode
    "band_width_mode":1,           (0|1)// 频道带宽         HT_BW
    "mac":"28-2c-b2-97-82-39",                          // mac地址        命令行ifconfig
    "beacon": 40,                    (20~1024)                    // Beacon时槽       BeaconPeriod
    "apsd_enabled": true,                                       // APSD开关	      APSDCapable
    "ap_enabled":true,                                          // AP隔离开关	      NoForwarding
    "shortgi_enabled": true,                                    // short GI开关    HT_GI
    "wmm_enabled": true                                         // 多媒体优先WMM开关            WmmCapable
  },
  "5g": {
    "enabled":true,                                     // 5g开关
    "ssid": "ssid1",                                    // 名称
    “broadcastssid”:true,                                    // 是否广播SSID
    "security_mode": "WPAPSKWPA2PSK"                    // Security Mode
     "encrypt": "TKIP",                                 // WPA Algorithms(TKIP,AES ,TKIPAES)					EncrypType
    "password": "123456kskdajksdasdasdasdasdasdasda",   // 密码
    "power": 20,                                        // 无线信号功率
    "channel": 14,                                      // 信道
    "net_type": 14           (2,8,14,15)                     // 网络模式
    "band_width_mode":1       (0|1)            // 频道带宽
    "mac":"28-2c-b2-97-82-39",                          // mac地址
    "beacon": 40,                                       // Beacon时槽
    "apsd_enabled": true,                                       // APSD开关
    "ap_enabled":true,                                          // AP隔离开关
    "shortgi_enabled": true,                                    // short GI开关
    "wmm_enabled": true                                         // 多媒体优先WMM开关
    “same_as_2g”: true                                       // 使用与2.4g相同的设置（包含：无线名称，加密方式，加密算法，密码，传输功率，Beacon时槽，APSD，AP隔离，Short GI，多媒体优先WMM，无线广播）
  }
}
</pre>

* api/wifi/set_config                    // 设置wifi 非阻塞
  
  post 需要登录验证：否
<pre>
{
  "2g": {
    "enabled":true,                    (true|false)          // 2.4g开关
    "ssid": "ssid1",                    (any string)        // 名称(长度1-32字符)
    “broadcastssid”:true,                                    // 是否广播SSID
    "security_mode": "Disable" (Disable,WPAPSK,WPA2PSK,WPAPSKWPA2PSK) //Security Mode
     "encrypt": "TKIP",   (NONE<>Disable,   TKIP<>WPA(2)PSK,    AES<>WPA(2)PSK ,   TKIPAES<>WPA(2)PSK  )  // WPA Algorithms EncrypType
    "password": "123456kskdajksdasdasdasdasdasdasda",  (any string)   // 密码(长度8-64字符)
    "power": 20,           (100,90,60,30,15,0)            // 无线信号功率
   "channel": 0,           (0)            // 哪个信道
      //
                {'name': '自动选择', 'value': 0},
    {'name': '2412MHz (Channel 1)', 'value': 1},
    {'name': '2417MHz (Channel 2)', 'value': 2},
    {'name': '2422MHz (Channel 3)', 'value': 3},
    {'name': '2427MHz (Channel 4)', 'value': 4},
    {'name': '2432MHz (Channel 5)', 'value': 5},
    {'name': '2437MHz (Channel 6)', 'value': 6},
    {'name': '2442MHz (Channel 7)', 'value': 7},
    {'name': '2447MHz (Channel 8)', 'value': 8},
    {'name': '2452MHz (Channel 9)', 'value': 9},
    {'name': '2457MHz (Channel 10)', 'value': 10},
    {'name': '2462MHz (Channel 11)', 'value': 11},
    {'name': '2467MHz (Channel 12)', 'value': 12},
    {'name': '2472MHz (Channel 13)', 'value': 13}
 
  "net_type": 9,        (0,1,4,6,9,)                     // 网络模式
2G: 9
0: legacy 11b/g mixed
1: legacy 11B only
4: legacy 11G only
6: 11N only
9: 11BGN mixed

    "band_width_mode":1,      (0|1|2)                 // 频道带宽(20Mhz->0, 20Mhz/40Mhz->1,强制40Mhz->2)
    "beacon": 40,                       (20~1024)        // Beacon时槽
    "apsd_enabled": true,                         (true|flase)      // APSD开关
    "ap_enabled":true,                              (true|flase)     // AP隔离开关
    "shortgi_enabled": true,                       (true|flase)      // short GI开关
    "wmm_enabled": true                          (true|flase)        // 多媒体优先WMM开关
  },
  "5g": {
    "enabled":true,                              // 5g开关
    "ssid": "ssid1",                              // 名称(长度1-32字符)
    “broadcastssid”:true,                    // 是否广播SSID
    "security_mode": "Disable"    // Security Mode(Disable,WPAPSK,WPA2PSK,WPAPSKWPA2PSK)
     "encrypt": "TKIP",    (NONE<>Disable,   TKIP<>WPA(2)PSK,    AES<>WPA(2)PSK ,   TKIPAES<>WPA(2)PSK  )   // WPA Algorithms(TKIP,AES ,TKIPAES)
    "password": "123456kskdajksdasdasdasdasdasdasda",    // 密码(长度8-64字符)
    "power": 20,      (100,90,60,30,15,0)        // 无线信号功率
    "channel": 0,                          // 哪个信道
 //
      {'name': '自动选择', 'value': 0},
    {'name': '5180MHz (Channel 36)', 'value': 36},
    {'name': '5200MHz (Channel 40)', 'value': 40},
    {'name': '5220MHz (Channel 44)', 'value': 44},
    {'name': '5240MHz (Channel 48)', 'value': 48},
    {'name': '5260MHz (Channel 52)', 'value': 52},
    {'name': '5280MHz (Channel 56)', 'value': 56},
    {'name': '5300MHz (Channel 60)', 'value': 60},
    {'name': '5320MHz (Channel 64)', 'value': 64},
    {'name': '5745MHz (Channel 149)', 'value': 149},
    {'name': '5765MHz (Channel 153)', 'value': 153},
    {'name': '5785MHz (Channel 157)', 'value': 157},
    {'name': '5805MHz (Channel 161)', 'value': 161},
    {'name': '5825MHz (Channel 165)', 'value': 165}
    

    "net_type": 14,                (2,8,14,15)         // 网络模式
5G 14
2: legacy 11A only
8: 11AN mixed
14: 11A/AN/AC mixed 5G band only
15: 11 AN/AC mixed 5G band only

    "band_width_mode":1       (0|1) ,                   // 频道带宽
    "beacon": 40,                               // Beacon时槽
    "apsd_enabled": true,                               // APSD开关
    "ap_enabled":true,                                  // AP隔离开关
    "shortgi_enabled": true,                            // short GI开关
    "wmm_enabled": true                                 // 多媒体优先WMM开关
 “same_as_2g”: true                                       // 使用与2.4g相同的设置（包含：无线名称，加密方式，加密算法，密码，传输功率，Beacon时槽，APSD，AP隔离，Short GI，多媒体优先WMM，无线广播）
  }
}
</pre>

* api/wifi/check_set
gett  需要登录验证：是
return
{
“code”: 0,          // (0->设置成功，1-> 正在设置，-1 ->已有全局设置锁)
“msg”: “xx”
}

* api/wifi/is_enabled

  get  需要登录验证：是
  <pre>
{
  "is_enabled":bool,              // 是否已经打开了wifi
}
</pre>



* api/wifi/ssid_get_config  (TODO)
  
  get 需要登录验证：是
  return 
 <pre>
[
  {
    "enable": true,                                       // 是否启用
    "name": "ssid1",                                      // 名称
    "encrypt": "WAP/WAP2",                               // 无线加密方式
    "password": "123456kskdajksdasdasdasdasdasdasda",    // 密码
    "rate": "5G",                                        // 无线频段
    "double_security": true                              // 是否开启双重安全认证
  },
  {
    "enable": false,
    "name": "ssid2idsijdoajosjdaojsijdao",
    "encrypt": "不加密",
    "password": "",
    "rate": "2.4G",
    "double_security": false
  }
]
</pre>

* api/wifi/ssid_set_config   (TODO)
  
  post 需要登录验证：是

 <pre>
[
  {
    "enable": true,                                       // 是否启用
    "name": "ssid1",                                      // 名称
    "encrypt": "WAP/WAP2",                               // 无线加密方式
    "password": "123456kskdajksdasdasdasdasdasdasda",    // 密码
    "rate": "5G",                                        // 无线频段
    "double_security": true                              // 是否开启双重安全认证
  },
  {
    "enable": false,
    "name": "ssid2idsijdoajosjdaojsijdao",
    "encrypt": "不加密",
    "password": "",
    "rate": "2.4G",
    "double_security": false
  }
]
</pre>

### lan口

* api/lan/get_lan_config

  get 需要登录验证：是
<pre>
{
  "ip": "192.168.1.2",                        // ip
  "net_mask": "255.255.255.0",                // 子网掩码
  "dhcp_enabled": true,                       // dhcp开关
  "ipaddr_start": "4.4.4.4",                  // IP地址开始段
  "ipaddr_end": "4.4.4.4",                    // IP地址结束段
  "gateway": "192.168.1.0",                   // 网关
  "dhcp_net_mask": "255.255.255.0",           // dhcp子网掩码
  "dns1Method": "手动",                       // dns1设置方式（手动，自动）
  "dns2Method": "自动",                        // dns2设置方式（手动，自动）
  "dns1": "8.8.8.8",                          // 首选dns
  "dns2": "",                                 // 备用dns
  "time": 381,                                // 地址租期
  "mac":"28-2c-b2-97-82-39"                   // mac地址
}
</pre>

* api/lan/set_lan_config


  post 需要登录验证：是
<pre>
{
  "ip": "192.168.1.2",                        // ip
  "net_mask": "255.255.255.0",                // 子网掩码
  "dhcp_enabled": true,                       // dhcp开关
  "ipaddr_start": "4.4.4.4",                  // IP地址开始段
  "ipaddr_end": "4.4.4.4",                    // IP地址结束段
  "gateway": "192.168.1.0",                   // 网关
  "dhcp_net_mask": "255.255.255.0",           // dhcp子网掩码
  "dns1Method": "手动",                       // dns1设置方式（手动，自动）
  "dns2Method": "自动",                       // dns2设置方式（手动，自动）
  "dns1": "8.8.8.8",                          // 首选dns
  "dns2": "",                                 // 备用dns
  "time": 381,                                // 地址租期
  "mac":"28-2c-b2-97-82-39"                   // mac地址
}
</pre>
* api/lan/check_set
gett  需要登录验证：是
return
{
“code”: 0,          // (0->设置成功，1-> 正在设置，2-> 设置失败)
“msg”: “xx”
}




* api/system/get_cable_connection

 get 需要登录验证:是
<pre>
{
  "wan":true   // wan口是否有物理连接
  "lan1":true   // wan口是否有物理连接
  “lan2":true   // wan口是否有物理连接
  “usb”:true   // usb是否有连接
}
</pre>


*api/system/get_time  
get 需要登录验证:是

return json 
{
  “time”: 2361632818231,       //路由器时间（单位秒）
  “time_type”: 0|1   (type:number)                 //时间格式，0->12小时制式,1->24小时制式
}

*api/system/set_time
post  需要登录验证:是

post data:
{
 “time_type”: 0|1   (type:number)                 //时间格式，0->12小时制式,1->24小时制式
}

### devices manager

* api/devices/disk(硬盘设备信息)

  get
  
return:
<pre>
{
  "totalSize": 1,             // 总大小(KB)
  "left": 0.23,               // 剩余空间(KB)
  "name": "disk1",            // 硬盘的名字
  "time": 5677                // 连接时间
}
</pre>

* api/devices/disk_uninstall(卸载硬盘)
  
get

return
<pre>
{
  "code": 0,
  "msg": ""
}
</pre>
  
* api/devices/ddr2_flash                   // 内存 DDR2,闪存 NAND FLASH信息
method:get
return:
{
  "ddr2_total_size": 64,            //ddr2总大小（KB）
  "ddr2_remain": 28,              // ddr2剩余（KB）
  "flash_total_size": 64,                           // flash总大小（KB）
  "flash_remain": 28,                             // flash剩余（KB）
}



* api/devices/cables(获取有线设备列表)

  get
  
  return
  <pre>
  {
    "code": 0,
    "devices": [
        {
            "platform": "phone",                    // 设备类型(phone, pad, unknow)
            "total_speed": 200,                     // 总速率
            "down_speed": 100,                      // 下载速度
            "up_speed": 100,                        // 上传速度
            "host_name": "android-a078b707872bc9a", // 主机名
            "connectType": "WIFI5G",                // 连接类型c
            "up": "10MB",                           // 总上传
            "down": "10GB",                         // 总下载
            "type": 1,                         // 设备连接方式(1->cable, 2->wifi2.4g,3->wifi5g)
            "ip": "192.168.1.11",                   // ip地址
            "mac": "97:32:21:44:55:11:42",          // mac地址
            "leftTime": 31223,                      // 租约剩余时间单位(s)
            "time": 31223,                          // 连接时间单位(s)
            "tag": "white"                          // 白名单，黑名单，或没有(white, black, "")
            "up_limit": 400,                        // 上传限速
            "down_limit": 600                       // 下载限速
	“local”,0 or 1 // web使用,是否是当前主机
        },...  
    ]
}
  </pre>


* api/devices/wifis(获取无线设备列表)

  get
  
  return
  <pre>
  {
    "code": 0,
    "devices": [
        {
            "platform": "phone",                    // 设备类型(phone, pad, unknow)
            "total_speed": 200,                     // 总速率
            "down_speed": 100,                      // 下载速度
            "up_speed": 100,                        // 上传速度
            "host_name": "android-a078b707872bc9a", // 主机名
            "connectType": "WIFI5G",                // 连接类型
            "up": "10MB",                           // 总上传
            "down": "10GB",                         // 总下载
            "type": 1,                         // 设备连接方式(1->cable, 2->wifi2.4g,3->wifi5g)
            "ip": "192.168.1.11",                   // ip地址
            "mac": "97:32:21:44:55:11:42",          // mac地址
            "leftTime": 31223,                      // 租约剩余时间单位(s)
            "time": 31223,                          // 连接时间单位(s)
            "tag": "white"                          // 白名单，黑名单，或没有(white, black, "")
            "up_limit": 400,                        // 上传限速
            "down_limit": 600                       // 下载限速
	“single”： 100                            // 信号强度
“local”,0 or 1 // web使用,是否是当前主机
“rssi0”,-30 //信号强度
“rssi1”,0 //信号强度

“rssi2”,-50//信号强度


        },...
    ]
}
  </pre>


* api/devices/edit_hostname (修改设备主机名)

  post
  <pre>
  {
    "mac": "97:32:21:44:55:11:42",
    "host_name": "android-a078b"
  }
  </pre>
  
*/api/devices/blacklist (黑名单列表)
{"devices":[{
            "platform": "phone",                    // 设备类型(phone, pad, unknow)
            "host_name": "android-a078b707872bc9a", // 主机名
            "type": 1,                         // 设备连接方式(1->cable, 2->wifi2.4g,3->wifi5g)
            "mac": "97:32:21:44:55:11:42",          // mac地址

},...],"code":0}

*/api/devices/whitelist (白名单列表)
{"devices":[{
            "platform": "phone",                    // 设备类型(phone, pad, unknow)
            "host_name": "android-a078b707872bc9a", // 主机名
            "type": 1,                         // 设备连接方式(1->cable, 2->wifi2.4g,3->wifi5g)
            "mac": "97:32:21:44:55:11:42",          // mac地址

},...],"code":0}
*/api/devices/graylist (灰名单，请求列表)
{"devices":[{
            "mac”: "AA:BB:CC:DD:EE:FF",                    // 请求设备的MAC地址
            "ip": "192.168.10.100",                                  // IP地址
            "username": "张三丰",                             // 用户名

},...],"code":0}

* api/devices/blacklist_add (加到黑名单)

  post
  <pre>
  {
    "mac": "32:21:44:55:11:42"
  }
  </pre>
  
* api/devices/blacklist_remove (从黑名单移除)

  post
  <pre>
  {
    "mac": "32:21:44:55:11:42"
  }
  </pre>
  
  
* api/devices/whitelist_add (加到白名单)

  post
  <pre>
  {
    "mac": "32:21:44:55:11:42"
  }
  </pre>
  
* api/devices/whitelist_remove (从白名单移除)

  post
  <pre>
  {
    "mac": "32:21:44:55:11:42"
  }
  </pre>

* api/devices/apply_rule (应用防火墙规则)

  post
  <pre>
  {
    "mac": "32:21:44:55:11:42"
  }
  </pre>


## 极客模式相关接口

/api/system/get_expertMode
需要验证 ：yes
方法        ：get
return 
<pre>
  {
    "enabled": true/false
  }
  </pre>

/api/system/set_expertMode
需要验证 ：yes
方法        ：post
post data：
<pre>
  {
    "enabled": true/false
  }
  </pre>

return data：
<pre>
  {
    “code”: 0,
“msg”: “error message when failed” (optional, when error happens)
“password”: “XXXXXXX”    (optional, when expert mode is enabled)
  }
  </pre>

