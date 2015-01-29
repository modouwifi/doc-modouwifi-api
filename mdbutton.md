## 魔豆外围设备接入和管理API

### 硬件组成

魔豆路由器
魔豆棒（为例）
魔豆按键（为例）

### 一般使用模式

    魔豆路由器作为载体，魔豆棒作为外围设备的一个接入口，魔豆按键作为一个外围设备的例子。
    使用魔豆按键通过魔豆棒传递特定数据或指令到魔豆路由器，路由器根据设定的规则来展示这
    些数据或者执行这些指令。魔豆棒由魔豆路由器系统自带程序完成初始化，类似魔豆按键的外
    围设备，需要在路由器上执行匹配动作，完成设备和路由器的匹配和注册工作。完成匹配与注
    册之后，魔豆路由器便可处理来自外围设备的消息了。

### 提供的APIs

    匹配api在调用“开始匹配”后将会被加锁，直到调用“结束匹配”后锁被释放；
    修改api在调用“开始修改”后将会被加锁，直到调用“结束修改”后锁被释放；
    注册api在调用“开始注册”后将会被加锁，直到调用“结束注册”后锁被释放；
    在加锁情况下调用“开始＊＊”api，将返回busy。
    获取列表api可以获取当前的设备列表与已注册的列表。
    获取可接入到魔豆棒应用（与按键联动）的脚本、可执行程序信息。

### 魔豆棒探活

	GET /api/mdbang/ping
		return
	   ｛code: -1, msg:”请等待”｝
		or
		return
	   ｛code: 1, msg:”设备不在线“｝
		or
		return
	   ｛code: 0, msg:”设备在线“｝

### 匹配

* 开始匹配：

		GET /api/mdbang/matchStart
			return
			{ code: -1, msg:"busy" }
			or
			return
			{ code: 0,  msg:"ok" }

* 尝试匹配：

		GET /api/mdbang/matchTry
			return
			{ code: -1, msg:"no match" }
			or
			return
			{
			code: 0,
			devid:"the id",
			msg:"ok"
			}

* 结束匹配：

		GET /api/mdbang/matchEnd
			return
			{ code: 0, msg: "ok" }

### 解匹配
* 开始解匹配：

		POST /api/mdbang/disMatchStart
		post data:{
				"devid" : "00"~"99"
		}
			return
			{ code: -1, msg:"busy" }
			or
			return
			{ code: 0,  msg:"ok" }

* 尝试解匹配：

		POST /api/mdbang/disMatchTry
		post data:{
		 		"devid" : "00"~"99"
		}
			return
			{ code: -1, msg:"no dis match" }
			or
			return
			{
			 	code: 0,
			 	devid:"the id"
			 }

* 结束解匹配：

		POST /api/mdbang/disMatchEnd
		post data:{
		 		"devid" : "00"~"99"
		 }
			 return
			 { code: 0, msg: "ok" }

### 列表

* 开始修改列表：

		GET /api/mdbang/listModifyStart
			 return
			 { code: -1, msg:"busy"}
			 or
			 return
			 { code: 0,  msg:"ok" }

* 添加匹配设备到列表：

		POST /api/mdbang/listAddDevice
		post data:{
				 "devid" : "00"~"99"
				 "alias" : "xxxx"
		}

			 return { code: -1, msg:"exist"}
			 or
			 return { code: 0,  msg:"ok"  }

* 从列表删除匹配设备：

		POST /api/mdbang/listRmDevice

			 post data:{
				 "devid" : "00"~"99"
			 }

			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 修改匹配设备备注到列表：

		 POST /api/mdbang/listRepAlias

			 post data:{
				 "devid" : "00"~"99"
				 "alias" : "xxxx"
			 }

			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 结束修改列表：

		 GET /api/mdbang/listModifyEnd
			 return { code: 0, msg: "ok" }

### 注册

* 开始注册：

		 GET /api/mdbang/registerStart
			 return { code: -1, msg:"busy"}
			 or
			 return { code: 0,  msg:"ok"  }

* 注册设备：

		 POST /api/mdbang/registerAddDevice

			 post data:{
				 "devid" : "00"~"99"
			 }
			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 注册设备的操作方法：

		 POST /api/mdbang/registerAddOps

			 post data:{
				 "devid"   : "00"~"99"
				 "cmd" : "0"~"255"
				 "op"     : "xxxx(reboot)"
				 "alias" : "xxxx"
			 }
			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 删除已注册设备：

		 POST /api/mdbang/registerRmDevice

			 post data:{
				 "devid" : "00"~"99"
			 }

			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 删除已注册设备的操作方法：

		 POST /api/mdbang/registerRmOps

		    post data:{
				 "devid"   : "00"~"99"
				 "cmd" : "0"~"255"
			 }
			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 修改已注册设备的操作方法：

		 POST /api/mdbang/registerRepOps

			 post data:{
				 "devid"   : "00"~"99"
				 "cmd" : "0"~"255"
				 "op"     : "xxxx(reboot)"
				 "alias" : "xxxx"
			 }
			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 生效注册：

		 GET /api/mdbang/registerCommit
			 return { code: -1, msg:"error"}
			 or
			 return { code: 0,  msg:"ok"  }

* 结束注册：

		 GET /api/mdbang/registerEnd
			 return { code: 0, msg: "ok" }

### 读取状态

* 获取列表状态（已匹配并为被注册的设备）：

		 GET /api/mdbang/listGetFullList
		 return
		 {
			 "netid":	"10",
			 "devices":	[{
				 "devid":	"1",
				 "alias":	“key 1”
				 }, {
				 "devid":	"2",
				 "alias":	“key 2”
				 }]
		 }

* 获取注册状态

		 GET /api/mdbang/registerGetFullList
		 return
		 {
			 "netid":	"10",
			 "devices":	[{
				 "devid":	"20",
				 "alias":	“key 1”,
				 "ops":	[{
					 "cmd":	"7",
					 "alias":	"时钟",
					 "op":	"clock &"
				 }]
			 }, {
				 "devid":	"15",
				 "alias":	“key 2”,
				 "ops":	[{
						 "cmd":	"7",
						 "alias":	"时钟",
						 "op":	"clock &"
				 }]
			 }]
		 }

### 读取可调用程序列表

	    GET /api/mdbang/getActions

			 return {
			     "13504053": {
			         "package_id": "com.modouwifi.vpnss",
			         "name": "app-ss-vpn",
			         "actions": {
			             "start_shadow_socks": {
			                 "id": "start_shadow_socks",
			 			    "name": "打开SS-VPN",
			                 "relative": true,
			                 "cmd": "/data/apps/14156393/./init start",
			                 "is_sync": false
			             },
			             "stop_shadow_socks": {
			                 "id": "stop_shadow_socks",
			 			    "name": "关闭SS-VPN",
			                 "relative": true,
			                 "cmd": "/data/apps/14156393/./init stop",
			                 "is_sync": false
			             }
			         },
			         "id": "13504053"
			     }
			 }


## 两份文件

* /data/conf/mdbang.list

	该文件维护着已匹配的设备id以及他们的备注信息

* /data/conf/mdbang.conf

	该文件维护着已注册的设备信息和对应的操作方法，系统会根据这份文件的信息，来自动调度和处理已注册的操作方法。
