---
layout: post
title:  websocket+node.js实现一对多双向实时通信
date:   2016-11-26 14:08:00 +0800
categories: 代码
tag: javascript
---

* content
{:toc}

基于socket.io框架的写法
====================================
需求是用设备实时控制公众号页面，同时公众号也可以回传信息给设备，一个设备可以绑定多个公众号

一、安装node.js
------------------------------------
这个就不多说了，官网安装node.js，现在也不需要配置环境变量之类的，我使用的版本是v6.9.1

{% highlight bash %}

C:\Users\zz>node -v
v6.9.1

{% endhighlight %}

二、安装express框架
------------------------------------
Express 是一个基于 Node.js 平台的极简、灵活的 web 应用开发框架，它提供一系列强大的特性，帮助你创建各种 Web 和移动设备应用。（这句是从官网摘抄的）

{% highlight bash %}

$ npm install express --save

{% endhighlight %}

当然，你也可以不使用express框架，或者用其他框架构建node应用，具体看个人喜好

三、web端的代码的编写
------------------------------------
* web端的代码相对简单，但是需要引入一个socket.io.js文件，这个文件要在node_modules里面搜索，我的具体位置是`node_modules/.1.6.0@socket.io-client/socket.io.min.js`，找到它以后，把他引入到web端的HTML页面中。

* body中的内容:

{% highlight html linenos %}
<textarea id="content"></textarea>
<input type="button" onclick="CHAT.submit()" value="提交" />
{% endhighlight %}

body中的内容只是一个简单的提交，用于测试发送信息给服务端

* 定义一个对象CHAT，内部提供websocket初始化方法
{% highlight html linenos %}
w.CHAT = {
        socket:null,
        submit:function(){

        },
        updateMsg:function(){

        },
        init:function(){

        }
    };
{% endhighlight %}
其中socket为实例化的socket.io对象，submit为发送信息方法，updateMsg为收到信息的展示方法，init为初始化socket方法。

* submit:
{% highlight html linenos %}
function(){
    var content = d.getElementById("content").value;
    if(content != ''){
        var obj = {
            type: 'wechat',
            ID: 'wechattestID',
            message: content
        };
        this.socket.emit('message', obj);
        d.getElementById("content").value = '';
    }
    return false;
}
{% endhighlight %}
emit是socket的发送信息方法，第一个参数为约定对接字段，第二个参数为发送的数据，可以是json对象。

* updateMsg:
{% highlight html linenos %}
function(o, message){
    if(message == 'message'){
        console.log('收到推送内容',o);
    }
}
{% endhighlight %}
updateMsg是自定义的方法，第一个参数是收到服务端的数据，第二个是自定义的检测字段。

* init：
{% highlight html linenos %}
function(openID){
    //连接websocket后端服务器
    this.socket = io.connect('ws://192.168.1.106:3000');
    
    //告诉服务器端有用户登录
    var obj = {
        type:'wechat',
        ID: openID
    };
    this.socket.emit('hasConnect', obj);
    
    //监听消息发送
    this.socket.on('message', function(obj){
        CHAT.updateMsg(obj, 'message');
    });
}
{% endhighlight %}
init为初始化方法，用于建立socket连接，建立连接后自动发送一段hasConnect字段给服务器让服务器知道自己的身份和ID。

* 最后是初始化socket，执行：`CHAT.init('wechattestID');`这样web端的代码就编写完了。

四、安卓端代码的编写
------------------------------------
介于我不会写安卓端的代码，安卓端的socket.io也未找到，故暂未实现安卓端socket.io的写法，我只能用web端的写法来模拟设备（和web端代码基本无异），具体见代码：

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta name="format-detection" content="telephone=no"/>
        <meta name="format-detection" content="email=no"/>
<meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0" name="viewport">
        <title>模拟设备</title>
        <!--[if lt IE 8]><script src="./json3.min.js"></script><![endif]-->
        <script src="./vue/websocket/nodeServer/node_modules/.1.6.0@socket.io-client/socket.io.min.js"></script>
    </head>
    <body>
    <textarea id="content"></textarea>
        <input type="button" onclick="CHAT.submit()" value="提交" />
        <script type="text/javascript">
      
    var d = document,
    w = window,
    
    w.CHAT = {
        socket:null,
        //提交消息内容
        submit:function(){
            var content = d.getElementById("content").value;
            if(content != ''){
                var obj = {
                    type: 'device',
                    ID: 'testDeviceID',
                    message: content
                };
                this.socket.emit('message', obj);
                d.getElementById("content").value = '';
            }
            return false;
        },
        updateMsg:function(o, message){
            if(message == 'message'){
                console.log('收到推送内容',o);
            }
        },
        init:function(deviceID){
            this.deviceID = deviceID;
            
            //连接websocket后端服务器
            this.socket = io.connect('ws://192.168.1.106:3000');
            
            //告诉服务器端有用户登录
            var obj = {
                type:'device',
                ID: deviceID
            };
            this.socket.emit('hasConnect', obj);
            
            //监听消息发送
            this.socket.on('message', function(obj){
                CHAT.updateMsg(obj, 'message');
            });

        }
    };

        CHAT.init('testDeviceID');
        </script>
    </body>
</html>
```

五、服务端代码的编写
------------------------------------

* 引入需要的模块：
```
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res){
	res.send('<h1>Welcome Realtime Server</h1>');
});
```

* 建立一个用于储存在线用户信息的列表
`var onlineList = {};`

* 建立socket.io的连接监听
```
io.on('connection', function(socket){
	
})
```

* 连接监听**hasConnect**字段，并对发来的信息进行存入在线列表并分组处理
{% highlight html linenos %}
socket.on('hasConnect', function(obj){
	if(obj.type == 'wechat'){
		//var deviceID = getDeviceIDByOpenID(obj.ID);//真正实现的时候需要编写一个通过openid获取deviceID的方法，这里用测试ID代替
		var deviceID = 'testDeviceID';
		if(onlineList[deviceID]){
			onlineList[deviceID][obj.ID] = socket.id;
		}else{
			onlineList[deviceID] = {}
			onlineList[deviceID][obj.ID] = socket.id;
		}
	}else{
		if(onlineList[obj.ID]){
			onlineList[obj.ID][obj.ID] = socket.id;
		}else{
			onlineList[obj.ID] = {}
			onlineList[obj.ID][obj.ID] = socket.id;
		}
	}
});
{% endhighlight %}
具体的思路是微信端和设备端连接的时候都会发送一个hasConnect字段，然后字段中含有设备/微信的信息和ID，利用这些信息对它们进行分组处理，存入在线列表中。

* 监听**message**字段，根据信息内的内容，寻找其所在分组，利用connected[id]方法转发到指定的微信/设备上去
{% highlight html linenos %}
socket.on('message', function(obj){
	if(obj.type == 'wechat'){
		if(obj.ID){
			//var deviceID = getDeviceIDByOpenID(obj.ID);
			var deviceID = 'testDeviceID';
			var socketID = onlineList[deviceID][deviceID];
			if(io.sockets.connected[socketID]){
				io.sockets.connected[socketID].emit('message', obj.message);
			}
		}
	}else{
		if(obj.ID){
			//var openIDList = getOpenIDListByDeviceID(obj.ID);
			for(var e in onlineList[obj.ID]){
				var socketID = onlineList[obj.ID][e];
				if((io.sockets.connected[socketID])&&(socketID!=socket.id)){
					io.sockets.connected[socketID].emit('message', obj.message);
				}
			}
		}
	}
});
{% endhighlight %}

* 最后监听端口:
{% highlight html linenos %}
http.listen(3000, function(){
	console.log('listening on *:3000');
});
{% endhighlight %}

* 至此，socket.io方法的代码基本写完了，一些对于断开连接要去分组里面清除成员的方法，我还未写上，请需要的朋友自行思考如何写（循环判断就可以了）。

* 最后是全部代码（现在还没上传，后期会附在github上），特别感谢这个博客：[使用Node.js+Socket.IO搭建WebSocket实时应用](http://www.open-open.com/lib/view/open1402479198587.html)。

不使用框架的写法
====================================
前面的东西就不交代了，基本同上

一、web端代码的编写
------------------------------------
html端和上面一样，只是不需要导入socket.io.js这个文件了，这里就只交代js的写法了

* 实例化websocket：
```
var host = 'ws://192.168.1.106:3000';
var socket = new WebSocket(host);
```

* 建立连接：
{% highlight html linenos %}
// 打开Socket
socket.onopen = function (event) {
    alert('您已经成功连接');
    var message = {
                    type: 'wechat',
                    ID: 'testID',
                    message: {信息:'发给设备'}
                };
    message = JSON.stringify(message);
    socket.send(message);
};
{% endhighlight %}
用onopen方法建立和服务端的连接，利用send方法发送信息给服务端。

* 监听服务端发来的消息：
{% highlight html linenos %}
socket.onmessage = function (event) {
    alert('message received');
    var res = event.data;

    var batteryLevel = document.getElementById("battery_level");
    batteryLevel.innerHTML = res;
};
{% endhighlight %}
用onmessage方法监听服务端的消息，event.data为服务端发送来的数据，后面可以自己写逻辑处理这些数据，我只是做了简单的展示。

* 监听Socket的关闭：
{% highlight html linenos %}
socket.onclose = function (event) {
        socket = null;
        alert('您已经断开连接');
    };
{% endhighlight %}
用onclose方法监听socket的连接断开时间，可以在回调函数中自己写业务逻辑。

二、设备端代码的编写
------------------------------------
设备端的代码是基于安卓的，我对安卓的代码不太了解，直接附上代码：
{% highlight html linenos %}
/**
 * Created by rick on 16-11-22.
 */

public class WebSocketUtil {
    private Context mContext = null;
    private WebSocketConnection mConnect = null;
    private String mWsUrl = "ws://192.168.1.106:3000";
    private String TAG = "Rick_WebSocket";
    private Handler mHandler = new Handler();

    public WebSocketUtil(Context context){
        mContext = context;
        mConnect = new WebSocketConnection();
    }

    public void sendLumpState(final String state, final String deviceId){
        WebSocketHandler wsHandler = new WebSocketHandler(){
            @Override
            public void onOpen() {
                Log.d(TAG, " ==== ws callback on open");
                JSONObject tem = new JSONObject();
                JSONObject object = new JSONObject();
                try {
                    tem.put("action", ACTION_LUMP_INFO);
                    tem.put("state", state);
                    object.put("type", "device");
                    object.put("ID", deviceId);
                    object.put("message", tem.toString());
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                mConnect.sendTextMessage(object.toString());
            }

            @Override
            public void onClose(int code, String reason) {
                Log.d(TAG, " ==== ws callback on close");
            }

            @Override
            public void onTextMessage(String payload) {
                Log.d(TAG, " ==== ws callback on receive message");
                mConnect.disconnect();
            }
        };
        try {
            mConnect.connect(mWsUrl, wsHandler);
        } catch (WebSocketException e) {
            e.printStackTrace();
        }
    }

    public void sendBatteryInfo(final int percent, final String deviceId){
        WebSocketHandler wsHandler = new WebSocketHandler(){
            @Override
            public void onOpen() {
                Log.d(TAG, " ==== ws callback on open");
                JSONObject tem = new JSONObject();
                JSONObject object = new JSONObject();
                try {
                    tem.put("action", ACTION_BATTERY_INFO);
                    tem.put("percent", percent);
                    object.put("type", "device");
                    object.put("ID", deviceId);
                    object.put("message", tem.toString());
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                mConnect.sendTextMessage(object.toString());
            }

            @Override
            public void onClose(int code, String reason) {
                Log.d(TAG, " ==== ws callback on close");
            }

            @Override
            public void onTextMessage(String payload) {
                Log.d(TAG, " ==== ws callback on receive message");
                mConnect.disconnect();
            }
        };
        try {
            mConnect.connect(mWsUrl, wsHandler);
        } catch (WebSocketException e) {
            e.printStackTrace();
        }
    }

    public void sendPlayStateInfo(final String playName, final String playState, final String deviceId){
        WebSocketHandler wsHandler = new WebSocketHandler(){
            @Override
            public void onOpen() {
                Log.d(TAG, " ==== ws callback on open");
                JSONObject tem = new JSONObject();
                JSONObject object = new JSONObject();
                try {
                    tem.put("action", ACTION_PLAY_INFO);
                    tem.put("name", playName);
                    tem.put("state", playState);
                    object.put("type", "device");
                    object.put("ID", deviceId);
                    object.put("message", tem.toString());
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                mConnect.sendTextMessage(object.toString());
            }

            @Override
            public void onClose(int code, String reason) {
                Log.d(TAG, " ==== ws callback on close");
            }

            @Override
            public void onTextMessage(String payload) {
                Log.d(TAG, " ==== ws callback on receive message");
                mConnect.disconnect();
            }
        };
        try {
            mConnect.connect(mWsUrl, wsHandler);
        } catch (WebSocketException e) {
            e.printStackTrace();
        }
    }

    public void sendLocalChange(final String deviceID){
        WebSocketHandler wsHandler = new WebSocketHandler(){
            @Override
            public void onOpen() {
                Log.d(TAG, " ==== ws callback on open");
                JSONObject tem = new JSONObject();
                JSONObject object = new JSONObject();
                try {
                    tem.put("action", ACTION_LOCAL_CHANGE);
                    object.put("type", "device");
                    object.put("ID", deviceID);
                    object.put("message", tem.toString());
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                mConnect.sendTextMessage(object.toString());
            }

            @Override
            public void onClose(int code, String reason) {
                Log.d(TAG, " ==== ws callback on close");
            }

            @Override
            public void onTextMessage(String payload) {
                Log.d(TAG, " ==== ws callback on receive message");
                mConnect.disconnect();
            }
        };
        try {
            mConnect.connect(mWsUrl, wsHandler);
        } catch (WebSocketException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

三、服务端代码的编写
------------------------------------
* 导入需要的模块
{% highlight html linenos %}
var app = require('express')();
var http = require('http').Server(app);
var ws = require('ws');
var WebSocketServer = ws.Server;
var getDeviceID = require('getDeviceID');
var useDeviceID = getDeviceID.getActivedDeviceByOpenID;
{% endhighlight %}
其中getDeviceID是自己写的一个获取设备ID的模块，不需要的可以无视

* 实例化websocket：
{% highlight html linenos %}
var io = new WebSocketServer({
    port: 3000,//监听的端口
});
app.get('/', function(req, res){
	res.send('<h1>Welcome Realtime Server</h1>');
});
{% endhighlight %}

* 在线列表（同上）：
`var onlineList = {};`

* 监听连接：
`io.on('connection', function(socket){});`

* 监听Socket的关闭：
{% highlight html linenos %}
socket.on('close', function(obj){
	outerloop:
	for(var e in onlineList){
		for(var m in onlineList[e]){
			if(onlineList[e][m] == this){
				delete onlineList[e][m];
				break outerloop;
			}
		}
	}
})
{% endhighlight %}
在socket断开时，会从在线列表中删除在线的用户。

* 将微信端的信息转发给设备
{% highlight html linenos %}
function forwardMessageToDevice(deviceID,stringObj){
	var socketID = onlineList[deviceID][deviceID];
	if(socketID){
		socketID.send(stringObj);
		console.log("转发给设备成攻");
	}
}
{% endhighlight %}
如果设备在线则转发，不在线不作处理。

* 将设备端的信息转发给微信
{% highlight html linenos %}
function forwardMessageToWechat(obj,stringObj){
	for(var e in onlineList[obj.ID]){
		var socketID = onlineList[obj.ID][e];
		if(socketID && socketID != this){
			socketID.send(stringObj);
			console.log("转发给微信成攻");
		}
	}
}
{% endhighlight %}
如果微信在线则转发，不在线不作处理。

* 监听收来的信息，搜索在线列表，转发给指定对象
{% highlight html linenos %}
socket.onmessage = function(obj){
		var stringMessage;
		try{
			obj = JSON.parse(obj.data);
			try{
				stringMessage = JSON.stringify(obj.message);
			}catch(e){
				console.log(obj.ID,'收到数据中message不是JSON格式！');
			}
		}catch(e){
			obj = obj.data;
			console.log('收到非JSON格式数据！');
		}
		console.log(stringMessage);
		if(obj.type == 'wechat'){
			//微信发来消息的转发给设备
			if(obj.ID){
				//var deviceID = 'testDeviceID';
				try{
					useDeviceID(obj.ID,function(deviceID){
						//查到设备在线就发送数据给设备
						if(onlineList[deviceID]){
							if(onlineList[deviceID][deviceID]){
								onlineList[deviceID][obj.ID] = socket;
								try{
									forwardMessageToDevice(deviceID,stringMessage);
								}catch (e) {
									console.log(e);
								};
							}
						}else{
						//没有人在线就新建一个hash表，用来保存在线用户
							onlineList[deviceID] = {};
							onlineList[deviceID][obj.ID] = socket;
						}
					});
				}catch(e){
					console.log(e);
				}
			}
		}else{
			//设备发来信息转发给微信
			if(obj.ID){
				//查到公众号有人在线就发送数据给公众号
				if(onlineList[obj.ID]){
					onlineList[obj.ID][obj.ID] = socket;
					try{
						forwardMessageToWechat.call(socket,obj,stringMessage);
					}catch (e) {
						console.log(e);
					};
				}else{
				//没有人在线就新建一个hash表，用来保存在线用户
					onlineList[obj.ID] = {};
					onlineList[obj.ID][obj.ID] = socket;
				}
			}
		}
	};
})
{% endhighlight %}
这里有个坑，要注意的就是不使用框架的话，双方通信只能使用字符串或者二进制，如果是json格式的数据需要在双方先编码再发送，另一方要先解码再处理！！！
>node因为是单线程的原因，所以对于不能把握的方法需要都用try catch处理，如果对错误不处理，服务器会直接挂掉！

结束语
====================================
至此，所以编码工作都完成了，其中有些内部的逻辑需要读者自行处理。因为初学node，代码也写得不够完美，node的一些坑也是得自己去踩了才知道，不过node.js擅长处理高并发任务的特点也决定了他未来的地位。
最后，希望大家都能成为大牛。