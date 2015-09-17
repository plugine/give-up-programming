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

#### V11
V11适用于大中型项目，基本安卓项目就通杀，因为用法太广泛，所以篇幅略长，还是老样子，代码过几天会标准化放入这个git仓库的代码库，可以拿来就用，但是在文章里出现的代码不保证正确性，请勿直接在这里面commandC.

为了节省篇幅，顺便安利一下JDK8在安卓中的应用，一下代码使用符合java8的标准写成

V11包含三个部分，网络请求，缓存系统和基础工具库，接下来自底向上讲解。

首先是架构图，还是以用户登录注册作为示例

![image](https://raw.githubusercontent.com/plugine/give-up-programming/master/images/v11-structure.png)

##### 基础工具库
基础工具库包括json解析，数据传递interface

首先是数据传递的interface，这是一个范型interface，方便传递所有类型的数据

```
public interface NetDataListener<T> {
	void onData(T data);
}
```
这样就定义了传递数据的统一接口

再观察V10的代码，是不是觉察到有很多的重复代码，比如AsyncHttpResponseHandler，仅仅是返回数据不同，因为网络返回code的规范化，我们可以用模版来处理不同数据的问题，下列代码来源于ServerManager

```
public static AsyncHttpResponseHandler getHandler(final NetDataListener listener, final Class<? extends BeanBase> t)){
	return new AsyncHttpResponseHandler(){
		@Override
		public void onSucceess(int statusCode, Header[] headers, byte[] responseBody) {
			String json = new String(responseBody);
			try{
				listener.onData(getMapper().readValue(json, t))
			}catch(IOException e){
				e.printStackTrace();
				//生成一条本地包装的数据，code为500，代表服务器错误，因为它传回的json没法解析
				listener.onData(BeanBase.generate500(t));
			}
		}
	}
}
```

然后是通用的解析json的代码

```
public static <T extends BeanBase> T parseJson(String json, Class<T> type){
	if(json != null){
		try{
			return getMapper().readValue(json, type);
		}catch(IOException e){
			e.pritnStackTrace();
		}
	}
	return null;
}
```

两段代码里都出现了getMapper()方法，这个方法非常简单，只是把全局的ObjectMapper返回而已，是一个Getter，ObjectMapper是jackson json解析库解析json的核心类，如果你喜欢，也可以用fastjson或者json替代。fastjson是alibaba出品的良心开源库，解析和序列化的速度都是上述三者中最快的，唯一的缺点是因为要极速的性能而忽略了详细的调试信息，如果json有错误或者Bean有错误，没有标明错在哪里，俗话说“不求通过，但求报错”，所以我一般用jackson，速度仅次于fastjson且带有完整的错误诊断信息。

### 前台架构

## 与API后台工程师交流的技巧

## 资源
这里应该写github awesome和gitbook的一些好的推荐的东西
