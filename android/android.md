# 安卓开发整理
配环境这种小事情大家百度都有一大堆，就不再赘述了，我们今天主要考虑架构和界面的问题

不过，会翻墙还是很必要的，很多莫名其妙的错误百度不出来，只有google这时候还有点作用，虽然俺们英语也好不到哪里去～

## 架构
架构分为前台架构和后台架构
### 后台架构
**后台的范围** 网络访问，缓存机制，数据库，对象持续化
**后台架构的必要性** 

我整合过V10、V11和V20三种架构模式，这种土的掉渣的命名方式也只有我能想得出来
#### v10
v10架构适用于少量网络请求的小型应用程序，仅包含网络访问模块和简单的Cache

架构图如下

![image](https://raw.githubusercontent.com/plugine/give-up-programming/master/images/v10-structure.png)

如图，Logic.java即是应用的核心，一个示例如下(使用AsyncHttpClient库做网络请求)

```java
class Logic {
	private static String LOGIN_URL= ServerManager.SERVER_API_ROOT + "/auth/login";
	
	public static void Login(String userName, String password, OnDataListener listener){
		AsyncHttpClient client = ServerManager.GetClient();
		RequestParams params = new RequestParams();
		params.put("user_name", userName);
		params.put("password", password);
		client.Post(LOGIN_URL, params, new AsyncHttpRequestHandler(){
			@Override
			public void onSuccess(int statusCode, []byte responseBody){
				String json = new String(responseBody);
				jsonObject = Json.parseObject(json);
				if(jsonObject.containKey("code"))){
					int code = jsonObject.getString("code");
					switch(code){
						case 200:
							listener.onData(jsonObject);
							break;
						case 404:
							listener.onData(jsonObject);
							Log.d("Logic.Login", "用户不存在");
							break;
						case 422:
							listener.onData(null);
							Log.d("Logic.Login", "密码错误");
							break;
						default:
							listener.onData(null);
							Log.d("Logic.Login", "未知错误");
					}
				}else{
					listener.onData(null);
				}
			}
			
			@Override
			public void onFailure(int statusCode, []byte responseBody){
				String response = new String(responseBody);
				if(response == null){
					Log.d("Logic.Login", "本地网络未连接");
					return;
				}
				Log.d("Logic.Login", "发生错误，服务器返回状态码:"+statusCode);
				listener.onData(null);
			}
		});
	}
}
```


### 前台架构

## 与API后台工程师交流的技巧

## 资源
这里应该写github awesome和gitbook的一些好的推荐的东西
