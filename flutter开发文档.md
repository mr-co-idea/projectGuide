#### 1、flutter应用

##### 1.1 路由管理：

1）Navigator.push();

```
onPressed: () {
                  Navigator.of(context)
                      .pushNamed('params_page', arguments: '我是被传递的参数');
                }
```

传参：详见/route/TipRouter.dart

2）通过命名路由进行跳转；

```
routes: {
        'new_page': (context) => MainRoute(title: '路由测试页'),
        'params_page': (context) => ParamsRouter(),
        'tips': (context) {
          return TipRoute(text: ModalRoute.of(context).settings.arguments);
        },
        'Widget': (context) => WidgetRoote(),
        'roote': (context) => RouteManage()
        // '/': (context) => MyHomePage(title: 'Flutter Demo Home Page')
      }
```

传参：详见/route/routerArg.dart

##### 1.2 包管理

1)引入在线包（https://pub.dev/）

```
dependencies:
  flutter:
    sdk: flutter


  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.3
  english_words: ^3.1.5
```

2）本地包或git（https://www.dartlang.org/tools/pub/dependencies）

![image-20200825171655445](C:\note\flutter\firstFlutterApp\image\image-20200825171655445.png)

##### 1.3 资源管理（待理解）

##### 1.4 调试app （待理解）

##### 1.5dart线程和捕获异常 （待理解）

#### 2、基础组件

##### 2.1 Widget

1)  Widget是Element的配置数据：Widget树是UI控件树或UI渲染树

2）一个Widget可以对应（创建）多个Element对象。

##### 2.2 StatelessWidget

用于不需要维护状态的场景

##### 2.3 Context

表示当前widget在widget树中的上下文

##### 2.4 StatefulWIdget

和`StatelessWidget`一样，`StatefulWidget`也是继承自`Widget`类，并重写了`createElement()`方法，不同的是返回的`Element` 对象并不相同；另外`StatefulWidget`类中添加了一个新的接口`createState()`

##### 2.5 State(*)

理解State的生命周期对flutter开发非常重要

- initState：当Widget第一次插入到Widget树时会被调用，对于每一个State对象

- `didChangeDependencies()`：当State对象的依赖发生变化时会被调用

- `build()`：主要是用于构建Widget子树的

- `reassemble()`：此回调是专门为了开发调试而提供的

- `didUpdateWidget()`：在widget重新构建时

- `deactivate()`：当State对象从树中被移除时，会调用此回调

- `dispose()`：当State对象从树中被永久移除时调用

![image-20200827085655653](C:\note\flutter\firstFlutterApp\image\image-20200827085655653.png)

##### 2.6为什么要将build方法放在State中，而不是放在StatefulWidget中？

1) 状态访问不方便

2）继承StatefulWIdget不方便

##### 2.7在Widget树中获取State对象

1）State希望暴露：findAncestorStateOfType

2)  State不希望暴露：GlobalKey

##### 2.8Flutter SDK内置组件库

- 基础组件、Material组件、Cupertino组件



#### 3、状态管理

**常见方法：**

- Widget管理自己的状态。
- Widget管理子Widget状态。
- 混合管理（父Widget和子Widget都管理状态）
- 全局注册

**官方原则：**

- 如果状态是用户数据，如复选框的选中状态、滑块的位置，则该状态最好由父Widget管理。
- 如果状态是有关界面外观效果的，例如颜色、动画，那么状态最好由Widget本身来管理。
- 如果某一个状态是不同Widget共享的则最好由它们共同的父Widget管理。

#### 4、组件

##### 1.文本、字体组件

- Text->TextStyle

- TextSpan

- DefualtTextStyle

字体：要使用Package中定义的字体，**必须提供`package`参数**，应用程序本地定义了字体，在创建TextStyle时可以不指定`package`参数

##### 2. 按钮

- RaisedButton
- FlatButton
- OutlineButton
- IconButton
- 自定义按钮

#### 5、flutter与js通讯

##### 1、flutter调用js方法

dart:

```
_webViewController
                ?.evaluateJavascript('callJS("我是flutter传输过来的")')
                ?.then((result) {
              print(result);
            });
```

js:

```
 function callJS(params){
            document.getElementById('X').innerHTML = params;
            return '我是执行结果'
        }
```

##### 2、js调用flutter方法

- **javascriptChannels**

  js:

  ```
  Toast.postMessage('我是传输结果');
  ```

  dart:

  ```
    javascriptChannels:
              <JavascriptChannel>[_alertJavascriptChannel(context)].toSet()
              
    JavascriptChannel _alertJavascriptChannel(BuildContext context) {
      return JavascriptChannel(
          name: 'Toast',
          onMessageReceived: (JavascriptMessage message) {
            showToast(message.message, context);
          });
    }
  ```

- **navigationDelegate**

  js:

  ```
  document.location = "https://www.baidu.com?a=haha"
  ```

  dart:

  ```
  navigationDelegate: (NavigationRequest request) {
            if (request.url.startsWith('https://www.baidu.com')) {
              showToast('我是劫持结果：$request', context);
              return NavigationDecision.prevent;
            }
            return NavigationDecision.navigate;
          }
  ```

##### 3、android默认不支持http请求(webview直接使用报错)

需要配置：

```
 <application
        android:usesCleartextTraffic="true">
 </application>
```



#### 6、全局变量和状态共享

##### 1、全局变量

- 包shared_preferences：对变量持久化处理

- 示例：

  ```
  const _themes = <MaterialColor>[
    Colors.blue,
    Colors.cyan,
    Colors.teal,
    Colors.green,
    Colors.red
  ]; //app颜色主题列表
  
  class Global {
    static SharedPreferences _prefs; //持久化存储
    static Profile profile = Profile(); //app描述
  
    //基础存储
    static NetCache netCache = NetCache(); //网络缓存
    static List<MaterialColor> get themes => _themes; //可选主题列表
    static bool get isRelease =>
        bool.fromEnvironment("dart.vm.product"); //是否为发布版本
  
    //初始化全局信息，在APP启动时执行
    static Future init() async {
      _prefs = await SharedPreferences.getInstance();
      var _profile = _prefs.getString('profile');
      if (_profile != null) {
        try {
          profile = Profile.fromJson(jsonDecode(_profile));
        } catch (e) {
          print(e);
        }
      }
  
      //如果没有缓存策略，设置默认缓存策略
      profile.cache = profile.cache ?? CacheConfig()
        ..enable = true
        ..maxAge = 3600
        ..maxCount = 100;
    }
  
    //持久化Profile信息
    static saveProfile() =>
        _prefs.setString("profile", jsonEncode(profile.toJson()));
  }
  ```

  

##### 2、全局状态共享

- 使用插件provider：通过context来控制状态共享

- 示例：

  创建

  ```
  //共享状态
  class ProfileChangeNotifier extends ChangeNotifier {
    Profile get _profile => Global.profile;
  
    @override
    void notifyListeners() {
      Global.saveProfile(); //保存Profile变更
  
      super.notifyListeners(); //通知涉及的Wiget更新
    }
  }
  
  //用户状态
  class UserModel extends ProfileChangeNotifier {
    User get user => _profile.user;
  
    bool get isLogin => user != null; //判断App是否为空
  
    set user(User user) {
      if (user?.identity != _profile.user?.identity) {
        _profile.lastLogin = _profile.user?.identity;
        _profile.user = user;
        notifyListeners();
      }
    } //用户状态发生改变，依赖的Widget更新
  
  } //用户状态在登录状态发生变化是更新，通知其依赖项
  ```

  使用

  ```
  MultiProvider(
        providers: [
          ChangeNotifierProvider.value(value: UserModel()),
        ],
        child: Consumer<ThemeModel>(
            builder: (BuildContext context, themeModel, Widget child) {
          return MaterialApp(
            theme: null,
            title: '',
            home: HomeRoute(),
            routes: <String, WidgetBuilder>{
            },
          );
        }),
      );
  ```

  调用

  ```
  UserModel user = Provider.of<UserModel>(context);
  ```

#### 7、数据处理和网络请求

##### 1、数据处理

- json通过json.decode(json)转化为map -> model -> class

- json

  ```
  {
      "example":"",
  }
  ```

- class

  ```
  class Example {
    final String example;
    
    Example(this.example);
  
    Example.fromJson(Map<String, dynamic> json)
        : example = json['example'];
  
    Map<String, dynamic> toJson() =>
      <String, dynamic>{
        'example': example
      };
  }
  ```

- model

  ```
  Map exampleMap = json.decode(json);
  var example = new Example.fromJson(exampleMap);
  String json = json.encode(example);
  ```

- 自动生成json model

  1. dependencies：

     json_annotation

      json_model

  2. dev-dependencies

      build_runner

      json_serializable

##### 2、网络请求

- 示例

  ```
  class NetWork {
    //在网络请求过程中可能需要使用当前context，例如请求失败时
    //打开一个新路由需要context
    NetWork([this.context]) {
      _options = Options(extra: {"context": context});
    }
  
    BuildContext context;
    Options _options;
    static Dio dio = new Dio(
        BaseOptions(baseUrl: '', headers: {HttpHeaders.acceptHeader: ""}));
  
    static void init() {
      dio.interceptors.add(Global.netCache); //添加缓存插件
      dio.options.headers[HttpHeaders.authorizationHeader] =
          Global.profile.token; //设置用户taken（可能为空）
  
      if (!Global.isRelease) {
        (dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate =
            (client) {
          client.findProxy = (uri) {
            return "PROXY 10.1.10.255:8088";
          };
          client.badCertificateCallback =
              (X509Certificate cert, String host, int port) =>
                  true; //代理工具会提供一个抓包的自签证书，需要禁用证书验证
        };
      } //在调试模式下抓包调试，所以使用代理，并禁用HTTPS证书校验
    }
  
    //登录接口，返回登录成功后的用户信息
    Future<User> login(String login, String pwd) async {
      String basic = 'Basic' + base64.encode(utf8.encode('$login:$pwd'));
      var result = await dio.get('',
          options: _options.merge(
              headers: {HttpHeaders.authorizationHeader: basic},
              extra: {"noCache": true}));
      //登录成功后更新公共头（authorization），此后的所有请求都会带上用户身份信息
      dio.options.headers[HttpHeaders.authorizationHeader] = basic;
      Global.netCache.cache.clear(); //清空所有缓存
      Global.profile.token = basic; //更新profile中的token信息
      return User.fromJson(result.data);
    }
  }
  
  ```

- 缓存

  示例

  ```
  //缓存机制:通过拦截器实现缓存
  
  class CacheObject {
    CacheObject(this.response)
        : timeStamp = DateTime.now().millisecondsSinceEpoch;
    Response response;
    int timeStamp; //缓存创建的时间
  
    @override
    bool operator ==(other) {
      return response.hashCode == other.hashCode;
    }
  
    @override
    int get hashCode => response.realUri.hashCode;
  } //保存缓存信息
  
  //网络缓存
  class NetCache extends Interceptor {
    //保持迭代器和插入时间的顺序一致性，采用LinkedHashMap
    var cache = LinkedHashMap<String, CacheObject>();
    @override
    Future onRequest(RequestOptions options) async {
      if (!Global.profile.cache.enable) return options;
      //refresh标记是否是“下拉”刷新
      bool refresh = options.extra["refresh"] == true;
      //如果是下拉刷新，则清除请求
      if (refresh) {
        if (options.extra["list"]) {
          //如果是列表，则只需要url中包含当前path的缓存全部删掉
          cache.removeWhere((key, value) => key.contains(options.path));
        } else {
          //如果不是列表，只删除url相同的缓存
          delete(options.uri.toString());
        }
      }
      if (options.extra["noCache"] != true &&
          options.method.toLowerCase() == 'get') {
        String key = options.extra["cacheKey"] ?? options.uri.toString();
        var ob = cache[key];
        if (ob != null) {
          //缓存未过期，则返回缓存信息
          if ((DateTime.now().millisecondsSinceEpoch - ob.timeStamp) / 1000 <
              Global.profile.cache.maxAge) {
            return cache[key].response;
          } else {
            //缓存过期,继续向服务器请求
            delete(key);
          }
        }
      }
    } //对缓存方法进行封装
  
    @override
    onError(DioError err) async {} //错误状态不缓存
  
    @override
    onResponse(Response response) async {
      if (Global.profile.cache.enable) {
        _saveCache(response);
      }
    } //如果启用缓存，则返回结果保存到缓存
  
    _saveCache(Response object) {
      RequestOptions options = object.request;
      if (options.extra["noCache"] != true &&
          options.method.toLowerCase() == "get") {
        if (cache.length == Global.profile.cache.maxCount) {
          cache.remove(cache[cache.keys.first]);
        } //如果缓存数量超过最大的数量限制，则先移除最早的一条记录
        String key = options.extra['cacheKey'] ?? options.uri.toString();
        cache[key] = CacheObject(object);
      }
    }//保存
  
    void delete(String key) {
      cache.remove(key);
    } //删除缓存
  }
  
  ```

#### 8、原生api



#### 9、打包

##### 1、添加图标

##### 2、创建密钥库

```
keytool -genkey -v -keystore c:\Users\USER_NAME\key.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias key
```

##### 3、在app中引入密钥库

创建一个名为 `<app dir>/android/key.properties` 的文件，它包含了密钥库位置的定义：

```
storePassword=<上一步骤中的密码>
keyPassword=<上一步骤中的密码>
keyAlias=key
storeFile=<密钥库的位置，e.g. /Users/<用户名>/key.jks>
```

##### 4、在gradle中配置签名

通过编辑 `<app dir>/android/app/build.gradle` 文件来为我们的 app 配置签名：

- Add code before `android` block:

- 在 `android` 代码块之前添加：

```
   def keystoreProperties = new Properties()
   def keystorePropertiesFile = rootProject.file('key.properties')
   if (keystorePropertiesFile.exists()) {
       keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
   }

   android {
         ...
   }
```

- Add code before `buildTypes` block:
- 在 `buildTypes` 代码块之前添加：

```
   buildTypes {
       release {
           // TODO: Add your own signing config for the release build.
           // Signing with the debug keys for now,
           // so `flutter run --release` works.
           signingConfig signingConfigs.debug
       }
   }
```

替换为我们的配置内容：

```
   signingConfigs {
       release {
           keyAlias keystoreProperties['keyAlias']
           keyPassword keystoreProperties['keyPassword']
           storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
           storePassword keystoreProperties['storePassword']
       }
   }
   buildTypes {
       release {
           signingConfig signingConfigs.release
       }
   }
```

5、启用混淆器

- 创建 `/android/app/proguard-rules.pro` 文件并添加下面的规则：

```
## Flutter wrapper
-keep class io.flutter.app.** { *; }
-keep class io.flutter.plugin.**  { *; }
-keep class io.flutter.util.**  { *; }
-keep class io.flutter.view.**  { *; }
-keep class io.flutter.**  { *; }
-keep class io.flutter.plugins.**  { *; }
-dontwarn io.flutter.embedding.**
```

-  启用混淆以及/或压缩

  在 `/android/app/build.gradle` 文件找到 `buildTypes` 的定义。在 `release` 配置中设置 `minifiyEnabled` 和 `useProguard` 为 true。

  ```
  android {
  
      ...
  
      buildTypes {
  
          release {
  
              signingConfig signingConfigs.release
  
              minifyEnabled true
              useProguard true
  
              proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
  
          }
      }
  }
  ```

  [R8](https://developer.android.google.cn/studio/build/shrink-code) 是谷歌推出的最新代码压缩器，当你打包 release 版本的 APK 或者 AAB 时会默认开启。要关闭 R8，请向 `flutter build apk` 或 `flutter build appbundle` 传 `--no-shrink` 标志。

- 

##### 5、检查 app manifest 文件

编辑 [`application`](https://developer.android.google.cn/guide/topics/manifest/application-element) 标签中的 `android:label` 来设置 app 的最终名字

##### 6、构建一个 APK

- 运行 `flutter build apk` （`flutter build` 默认带有 `--release` 参数）

