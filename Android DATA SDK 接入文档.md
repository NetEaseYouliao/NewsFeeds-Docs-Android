# Android SDK 接入文档
## SDK 概述

网易有料NewsFeedsSDK为移动应用提供内容智能分发功能，对外提供较为简洁的API接口，方便第三方应用快速的集成并实现内容分发功能。SDK兼容Android 14+，Demo兼容Android 14+。

NewsFeedsSDK提供的功能如下：

- 获取频道列表
- 获取新闻列表
- 获取新闻详情
- 新闻正文WebView
- 用户行为采集上传

## SDK类说明

网易有料NNewsFeedsSDK主要提供了以下类：

- NNewsFeedsSDK：整个SDK的主入口，单例，主要提供初始化，配置用户信息，加载频道、新闻列表的功能
- NNFTracker：用户行为追踪的单例类
- NNFNewsDetailsWebView：WebView派生类，提供新闻正文展示功能
- NNFChannels：频道列表的model类
- NNFNews：新闻列表的model类

## 开发准备

### 1. Gradle集成

我们提供了两种方式进行Gradle集成，可以根据需要任选其一。

- jcenter远程依赖

第一步，在project module下的build.gradle配置repositories。创建项目时，会自动包含jcenter。若无，请添加。

```java
allprojects {
    repositories {
        jcenter()
    }
}
```

第二步，在app module下的build.gradle中引入我们sdk的依赖，请自行将x.x替换为版本号，目前最新版为1.2


```java
compile 'com.netease.youliao:newsfeeds-data:x.x'
```

为了自动升级到最新的sdk，建议写成下面的形式：

```java
compile 'com.netease.youliao:newsfeeds-data:1.2+'
```


- aar本地依赖

第一步，导入jar包。在工程app结构下新建libs目录，同时将我们提供的`newsfeeds-data-x.x.aar`文件复制到当前libs目录下

第二步，在project module下的build.gradle配置repositories

```java
allprojects {
    repositories {
        jcenter()
        
        // 添加aar文件所在目录
        flatDir {
            dirs 'libs'
        }
    }
}
```

第三步，在app module下的build.gradle中引入我们sdk的aar依赖，请自行将x.x替换为版本号，目前最新版为1.2

我们的data-sdk内部依赖了一些第三方库：

```java
compile 'com.alibaba:fastjson:1.2.8'
compile 'com.getui:sdk:2.11.1.0'
```

因此，采用aar本地引入的方式，需要同时引入data-sdk依赖库 & data-sdk。

```java
dependencies {
    // 新建工程时自动生成
    compile fileTree(include: ['*.jar'], dir: 'libs')
    ...
    // data-sdk依赖库
    compile 'com.alibaba:fastjson:1.2.8'
    compile 'com.getui:sdk:2.11.1.0'
    // 添加data-sdk依赖
    compile 'com.netease.youliao:newsfeeds-data:x.x@aar'
}
```

### 2. 初始化

在自定义Application的OnCreate中添加以下代码，初始化我们的SDK

```
new NNewsFeedsSDK.Builder()
    .setAppKey("4c92fbfc2e6e7046d6e3cafced******")
    .setAppSecret("b430f8362f9f65bc09a639f62b******")
    .setContext(getApplicationContext())
    .setCacheEnabled(true)
    .setMaxCacheTime(1 * 24 * 60 * 60 * 1000)
    .setLogLevel(NFLogUtil.LOG_DEBUG)
    .build();
```

上述初始化代码生成一个NNewsFeedsSDK实例，后续可通过NNewsFeedsSDK.getInstance()拿到该实例进行接口调用。

初始化接口及参数说明

接口 | 参数 | 类型 | 描述
---|---|---|---
setAppKey | appKey | String | 分配给应用的唯一标识，用户在CMS后台新建应用时生成
setAppSecret | appSecret | String |  分配给应用的唯一秘钥，用户在CMS后台新建应用时生成
setContext | context | Context | 传入app的Context，建议传入ApplicationContext
setCacheEnabled | cacheEnabled | boolean | 是否开启新闻正文文本和图片缓存，默认开启
setMaxCacheTime | maxCacheTime | long | 配置新闻正文文本和图片最大缓存时长, 单位毫秒，默认7天
setLogLevel | logLevel | int | Android Studio等开发工具的 控制台Log等级，指定哪些日志需要输出

### 3. 权限（v1.2新增）

从v1.2开始，我们的SDK内部提供广告功能，为了成功拉取到广告，需要在AndroidManifest.xml中添加权限声明：

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />    <!-- 如果需要精确定位的话请加上此权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

以及添加如下广告组件：

```java
<!-- 声明SDK所需要的组件 -->
<service
    android:name="com.qq.e.comm.DownloadService"
    android:exported="false"/>
<!-- 请开发者注意字母的大小写，ADActivity，而不是AdActivity -->
<activity
    android:name="com.qq.e.ads.ADActivity"
    android:configChanges="keyboard|keyboardHidden|orientation|screenSize"/>
```


如果您打包App时的targetSdkVersion >= 23：请先获取到SDK要求的所有权限，然后调用SDK获取到的新闻列表和新闻详情才会包含广告数据。否则SDK返回的新闻列表和新闻详情将不包含广告数据。我们建议您在App启动时就去获取SDK需要的权限。您可以参考如下权限处理示例代码，权限示例代码写在Activity中：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_splash);
    ...

    // 如果targetSDKVersion >= 23，就要申请权限
    if (Build.VERSION.SDK_INT >= 23) {
        checkAndRequestPermission();
    }
}

/**
 * ----------非常重要----------
 * <p>
 * Android6.0以上的权限适配简单示例：
 * <p>
 * 如果targetSDKVersion >= 23，那么必须要申请到所需要的权限，再调用SDK，否则SDK返回的
 * 数据不含广告。
 * <p>
 * 这里的代码里是一个基本的权限申请示例，请开发者根据自己的场景合理地编写这部分代码来实现权限申请。
 * 注意：下面的 checkSelfPermission 和 requestPermissions 方法都是在Android6.0的SDK中增加的API，如果您的App还没有适配到Android6.0以上，则不需要调用这些方法。
 */
@TargetApi(Build.VERSION_CODES.M)
private void checkAndRequestPermission() {
    List<String> lackedPermission = new ArrayList<String>();
    if (!(checkSelfPermission(Manifest.permission.READ_PHONE_STATE) == PackageManager.PERMISSION_GRANTED)) {
        lackedPermission.add(Manifest.permission.READ_PHONE_STATE);
    }

    if (!(checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED)) {
        lackedPermission.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
    }

    if (!(checkSelfPermission(Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED)) {
        lackedPermission.add(Manifest.permission.ACCESS_FINE_LOCATION);
    }

    if (lackedPermission.size() > 0) {
        // 请求所缺少的权限，在onRequestPermissionsResult中再看是否获得权限
        String[] requestPermissions = new String[lackedPermission.size()];
        lackedPermission.toArray(requestPermissions);
        requestPermissions(requestPermissions, 1024);
    }
}

private boolean hasAllPermissionsGranted(int[] grantResults) {
    for (int grantResult : grantResults) {
        if (grantResult == PackageManager.PERMISSION_DENIED) {
            return false;
        }
    }
    return true;
}

@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    if (!(requestCode == 1024 && hasAllPermissionsGranted(grantResults))) {
        // 如果用户没有授权，那么应该说明意图，引导用户去设置里面授权。
        Toast.makeText(this, "应用缺少必要的权限！请点击\"权限\"，打开所需要的权限。", Toast.LENGTH_LONG).show();
        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
        intent.setData(Uri.parse("package:" + getPackageName()));
        startActivity(intent);
        finish();
    }
}

```

### 4. 文件访问兼容性（v1.2新增）

由于SDK返回的广告可能是App下载类广告，因此，当targetSDKVersion >= 24时，需要进行文件访问兼容处理。如果您打包时的targetSDKVersion >= 24，为了让SDK能够正常下载、安装App类广告，必须按照下面的三个步骤做兼容性处理。注意：如果您的targetSDKVersion < 24，不需要做这个兼容处理。

（1）在 AndroidManifest.xml 中的 Application 标签中添加 provider 标签，接入代码如下所示：

```java
<application
    android:allowBackup="true"
    android:icon="@drawable/gdticon"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">

    <!-- targetSDKVersion >= 24时才需要添加这个provider。provider的authorities属性的值为${applicationId}.fileprovider，请开发者根据自己的${applicationId}来设置这个值 -->
    <provider
        android:name="android.support.v4.content.FileProvider"
        android:authorities="${applicationId}.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/gdt_file_path" />
    </provider>

    <!-- 声明SDK所需要的组件 -->
    <service
        android:name="com.qq.e.comm.DownloadService"
        android:exported="false"/>
    <!-- 请开发者注意字母的大小写，ADActivity，而不是AdActivity -->
    <activity
        android:name="com.qq.e.ads.ADActivity"
        android:configChanges="keyboard|keyboardHidden|orientation|screenSize"/>

    ... ...
</application>
```

需要注意的是provider的authorities值为 `${applicationId}.fileprovider`，对于每一个开发者而言，这个值都是不同的，`${applicationId}` 在代码中和Context.getPackageName()值相等，是应用的唯一id，例如 "com.netease.youliao.demo"。

（2）在项目结构下的res目录下添加一个xml文件夹，再新建一个`gdt_file_path.xml`的文件，文件内容如下：

```java
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 这个下载路径不可以修改，必须是GDTDOWNLOAD -->
    <external-path name="gdt_sdk_download_path" path="GDTDOWNLOAD" />
</paths>
```

（3）混淆配置文件中加上：`-keep class android.support.v4.**{ *;}`，避免support-V4包中的FileProvider代码被混淆。

### 5. 混淆

若您的App开启了混淆，请为我们的SDK添加下述混淆规则


```java
# 网易有料 - 信息流SDK
-keep class com.netease.youliao.newsfeeds.**{*;}
```

从v1.2开始，由于我们的SDK内部依赖了腾讯广点通和个推，因此，混淆时也需要添加如下混淆规则：

```java
# 腾讯广点通
-keep class com.qq.e.** {
    public protected *;
}
-keep class android.support.v4.app.NotificationCompat**{
    public *;
}
# targetSDKVersion >= 24添加该语句，避免support-V4包中的FileProvider代码被混淆
-keep class android.support.v4.**{ *;}


# 个推
-dontwarn com.igexin.**
-keep class com.igexin.** { *; }
-keep class org.json.** { *; }
-keep public class * extends com.igexin.sdk.GTIntentService

-keep class android.support.v4.app.NotificationCompat { *; }
-keep class android.support.v4.app.NotificationCompat$Builder { *; }
```


## 日志管理

- 定义

```java
public Builder setLogLevel(@NNFLogLevel int logLevel);
```

- 调用

在初始化SDK的时候调用

```java
// 初始化
new NNewsFeedsSDK.Builder()
        ...
        .setLogLevel(NNFLogUtil.LOG_INFO)
        .build();
```

- Log等级

Log等级由@NFLogLevel约束，只能取@NFLogLevel指定的整数值

```
@IntDef({NNFLogUtil.LOG_NONE,
        NNFLogUtil.LOG_ERROR,
        NNFLogUtil.LOG_WARN,
        NNFLogUtil.LOG_INFO,
        NNFLogUtil.LOG_DEBUG,
        NNFLogUtil.LOG_VERBOSE})
@Retention(RetentionPolicy.SOURCE)
public @interface NNFLogLevel {
}
```

Log等级 | 说明 
---|---
NFLogUtil.LOG_NONE| 不打印日志
NFLogUtil.LOG_ERROR | 打印 ERROR
NFLogUtil.LOG_WARN | 打印 ERROR、WARN
NFLogUtil.LOG_INFO | 打印 ERROR、WARN、INFO
NFLogUtil.LOG_DEBUG | 打印 ERROR、WARN、INFO、DEBUG
NFLogUtil.LOG_VERBOSE | 打印 ERROR、WARN、INFO、DEBUG、VERBOSE

## 数据接入接口说明

### 配置用户标识

为了获取更加准确的跨平台的个性化推荐内容，鼓励用户配置应用的唯一userId，调用：

```java
NNewsFeedsSDK.getInstance().loginUser("我是userId");
```
若不调用该接口，系统将默认采用匿名的设备Id进行相关用户行为分析。

鼓励用户在登录或者注册时候调用，SDK会记住该字段，原则上只调用一次即可，多次调用以最后一次调用为主，如果用户想清除以前配置的userId，可以将该字段设为空即可。也可以调用登出接口：

```java
NNewsFeedsSDK.getInstance().logoutUser();
```

### 加载频道列表

- 定义

```java
/**
 * 获取所有新闻频道列表
 *
 * @param listener 网络请求回调
 */
public void loadChannelList(NNFHttpRequestListener<NNFChannels> listener);
```

- 调用


```java
NNewsFeedsSDK.getInstance().loadChannelList(new NNFHttpRequestListener<NNFChannels>() {
    @Override
    public void onHttpSuccessResponse(NNFChannels result) {

    }

    @Override
    public void onHttpErrorResponse(int code, String errorMsg) {

    }
});
```

- 网络请求结果回调接口说明

NNFHttpRequestListener 接口类为 网络请求结果回调。包含以下回调函数：

a. 网络请求成功

```java
/**
 * 网络请求成功
 *
 * @param result 网络返回数据
 */
void onHttpSuccessResponse(T result);
```

网络请求成功时触发该回调。其中，result 为网络返回数据，该数据类型为泛型，与用户调用网络请求接口时传入的类型保持一致。例如，获取所有新闻频道列表时，传入的类型为NNFChannels，则回调中result的类型也为NNFChannels。


b. 网络请求失败

```java
/**
 * 网络请求失败
 *
 * @param code     错误码
 * @param errorMsg 错误原因
 */
void onHttpErrorResponse(int code, String errorMsg);
```

网络请求失败时触发该回调。其中，code 为错误码，errorMsg 为错误原因。具体错误码和错误原因说明请参考网易有料Api Server文档。

### 加载新闻列表

SDK提供多个加载新闻列表的接口，用户可根据需要传入不同的参数组合。

- 定义

```java
/**
 * @param channelid 频道ID
 * @param listener  网络请求回调
 */
public void loadNewsList(String channelid, NNFHttpRequestListener<NNFNews> listener);
```

```java
/**
 * @param channelid 频道ID
 * @param num       新闻列表长度，默认10条
 * @param listener  网络请求回调
 */
public void loadNewsList(String channelid, int num, NNFHttpRequestListener<NNFNews> listener);
```


```java
/**
 * @param channelid 频道ID
 * @param num       新闻列表长度，默认10条
 * @param loadType  新闻列表内容组合，默认为0
 * @param listener  网络请求回调
 */
public void loadNewsList(String channelid, int num, int loadType, NNFHttpRequestListener<NNFNews> listener)
```

- 调用

```java
NNewsFeedsSDK.getInstance().loadNewsList(channelid, new NNFHttpRequestListener<NNFNews>() {
    @Override
    public void onHttpSuccessResponse(NNFNews result) {

    }

    @Override
    public void onHttpErrorResponse(int code, String errorMsg) {

    }
});
```

- 请求参数 loadType 

loadType为新闻列表内容组合类型，取值0或1，默认取0。取值说明如下： 

0：接口返回推荐系统或用户编辑的普通新闻。

1：接口返回推荐系统或用户编辑的普通新闻、接口返回用户编辑的头图新闻、用户编辑的置顶新闻

一个典型的Feed流，上拉加载更多时，传入0；下拉刷新时，传入1。

- 请求参数 num

num为期望返回的新闻列表长度，默认10条。num仅为普通新闻的条数，头图或置顶新闻不计算在内。

- 接口返回 NNFNews

新闻列表数据模型（NNFNews）各字段说明如下：

字段 | 类型 | 描述
---|---|---
infos| NNFNewsInfo[]| 普通新闻
banners | NNFNewsInfo[] |轮播图或头图新闻
tops | NNFNewsInfo[] |置顶新闻

其中，单个新闻（NNFNewsInfo）按类型（infoType）可分为：文章(article)、图集(picset)、视频(video)。数据模型具体字段说明，请参考附录。

从v1.2开始，我们的SDK提供广告功能，如果拉取广告的必要权限均被授权，会在轮播图（banners），普通新闻（infos）出现广告。广告具体说明，请参考广告模块。

(1) 轮播图

如果拉取广告的必要权限均被授权，以loadType 为 1 请求新闻列表时，若返回的 NNFNews.banners 有数据，即表示有轮播图，那么会在轮播图中随机插入一条广告，且广告不是在第一个位置。

(2) 普通新闻列表

如果拉取广告的必要权限均被授权，当请求的新闻条数小于10条时，不会出现广告；当请求的新闻条数大于等于10，小于15时，在第4条位置出现广告；当请求条数大于等于15条时，第4、10条位置将会出现广告。

- 错误码说明

error的特殊处理：并不是所有error都是网络请求失败，其中

```
errorCode == 4000    表示后台管理系统删除了某个频道，App在未重新拉取更新频道列表的情况下，获取该频道下的新闻列表会返回code为4000的错误

errorCode == 4004    表示该频道下拉取不到任何的新闻列表信息，暂时没有可推荐的新闻，会返回code为4004的错误
```

## 加载新闻详情

- 定义

```java
/**
 * @param infoType 新闻类型，文章/图集/视频
 * @param infoId 新闻ID
 * @param producer 新闻提供者，user表示用户自编辑新闻，recommendation表示来自个性化推荐系统
 * @param listener 网络请求回调
 */
public void loadNewsDetails(String infoType, String infoId, String producer, NNFHttpRequestListener<NNFNewsDetails> listener);
```
- 调用

```java
NNewsFeedsSDK.getInstance().loadNewsDetails(newsInfo.infoType, newsInfo.infoId, newsInfo.producer, new NNFHttpRequestListener<NNFNewsDetails>() {
    @Override
    public void onHttpSuccessResponse(NNFNewsDetails result) {
        
    }
    
    @Override
    public void onHttpErrorResponse(int code, String errorMsg) {
    
    }
});
```
- 接口返回 NNFNewsDetails

从v1.2开始，我们的SDK提供广告功能，如果拉取广告的必要权限均被授权，用户请求新闻详情成功后， 返回的 NNFNewsDetails 中可能包含广告。广告具体说明，请参考广告模块。


## 文章类新闻正文WebView

SDK将infoType为文章（article）的新闻正文的展示封装成NNFNewsDetailsWebView。

第一步，在布局文件中添加

```java
<com.netease.youliao.newsfeeds.webview.NNFNewsDetailsWebView
    android:id="@+id/webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

第二步，在Activity或Fragment中实例化NNFNewsDetailsWebView


```java
private NNFNewsDetailsWebView mNewsDetailsWebView;

mNewsDetailsWebView = (NNFNewsDetailsWebView) this.findViewById(R.id.webview);
```

第三步，加载新闻正文

- 接口定义

```java
/**
 * @param newsInfo        列表页可展现的新闻信息
 * @param webViewListener
 */
public void loadNewsDetails(NNFNewsInfo newsInfo, NNFWebViewListener webViewListener)
```


```java
/**
 * @param newsInfo        列表页可展现的新闻信息
 * @param reportAction    当WebView销毁时，是否上报新闻正文的浏览时长和浏览进度。默认为true。
 * @param webViewListener
 */
public void loadNewsDetails(NNFNewsInfo newsInfo, boolean reportAction, NNFWebViewListener webViewListener);
```

- 参数 reportAction

注意，reportAction 为 boolean类型，表示当WebView销毁时，是否上报新闻正文的浏览时长和浏览进度。默认为true。详细说明请参考用户行为统计。

- 接口调用

```java
mNewsDetailsWebView.loadNewsDetails(newsInfo, new NNFWebViewListener() {
    @Override
    public void onWebViewPageFinished() {
        Log.d(TAG, "新闻详情请求成功");
    }

    @Override
    public void onWebViewPageFailure(int errCode, String errorMsg) {
        Log.e(TAG, "新闻详情请求失败->@code = " + errCode + ", @errorMsg = " + errorMsg);
    }

    @Override
    public void onWebImageClick(int clickedIndex, String src, NNFImageInfo[] imageInfos) {

    }
    
    @Override
    public void onRelatedNewsClick(NNFNewsInfo newsInfo) {
        
    }

    @Override
    public void onIssueReporting(String issueDescription) {
        
    }

    @Override
    public void onIssueReportFinished() {
        
    }
});
```

若用户不希望NNFNewsDetailsWebView自动上报新闻正文的浏览时长和浏览进度，可按照如下示例进行调用：

```java
mNewsDetailsWebView.loadNewsDetails(newsInfo, false, webViewListener);
```

- NNFNewsDetailsWebView回调接口说明

NNFWebViewListener为回调抽象类，提供文章类新闻正文NNFNewsDetailsWebView交互事件回调，目前支持的交互事件回调有：网页主框架加载完毕、网页主框架加载失败、正文中的图片被点击、文章详情页面跳转到报错页面的回调、报错完成返回到文章详情页面的回调。

a. 网页主框架加载完毕

```java
public abstract void onWebViewPageFinished();
```

SDK内部为WebView设置WebViewClient，在WebViewClient的onPageFinished中调用onWebViewPageFinished。表示网页主框架加载完毕。

b. 网页主框架加载失败

```java
/**
 *
 * @param errCode 错误码
 * @param errorMsg 错误原因
 */
public abstract void onWebViewPageFailure(int errCode, String errorMsg);
```

网络请求失败时调用。其中errCode为错误码，errorMsg为错误原因。

c. 正文中的图片被点击

```java
/**
 * @param clickedIndex 被点击图片在图片数组中的索引
 * @param source       图片原始链接
 * @param imageInfos   图片数组
 */
public abstract void onWebImageClick(int clickedIndex, String source, NNFImageInfo[] imageInfos);
```

当正文中的图片被点击时，触发该回调。其中，clickedIndex为被点击图片在图片数组中的索引，source为图片原始链接；imageInfos为文章中的图片数组。

d. 点击相关推荐（v1.2新增）

```java
/**
 * 文章类新闻详情页面相关推荐新闻点击时的回调
 *
 * @param newsInfo
 */
public void onRelatedNewsClick(NNFNewsInfo newsInfo)
```
阅读完文章正文后，在文章正文底部，可能会有该篇文章的相关推荐，点击相关推荐时触发该回调。

e. 文章详情页面跳转到报错页面的回调（v1.2新增）

```java
/**
 * 举报中
 *
 * @param issueDescription 举报描述
 */
public void onIssueReporting(String issueDescription)
```
点击文章底部的举报按钮时，弹出举报选择框，触发该回调。其中issueDescription为报错页面的相关描述，方便更改页面title之类的参数。

f. 报错完成返回到文章详情页面的回调（v1.2新增）

```java
/**
 * 举报完成
 */
public void onIssueReportFinished()
```

报错操作结束后触发该回调。

## 文章类新闻正文缓存

SDK支持将infoType为文章（article）的新闻正文的文本和图片进行缓存。

### 1. 正文缓存开启

通过该接口配置是否开启正文缓存，默认情况为true。

- 定义

```java
/**
 * 是否开启WebView缓存，默认开启
 *
 * @param cacheEnabled
 * @return
 */
public Builder setCacheEnabled(boolean cacheEnabled)
```

- 调用

在初始化SDK的时候调用

```java
// 初始化
new NNewsFeedsSDK.Builder()
        ...
        .setCacheEnabled(false)
        .build();
```

### 2. 配置最大缓存时长

在开启了正文缓存的前提下，可配置正文的文本和图片缓存的最大时长，单位毫秒，默认7天。

- 定义

```java
/**
 * 配置新闻详情文本和图片最大缓存时长
 *
 * @param maxCacheTime 最大缓存时长，单位毫秒，默认7*24*60*60*1000
 */
public Builder setMaxCacheTime(long maxCacheTime);
```

- 调用

在初始化SDK的时候调用

```java
// 初始化
new NNewsFeedsSDK.Builder()
        ...
        .setMaxCacheTime(3 * 24 * 60 * 60 * 1000)
        .build();
```

### 3. 获取新闻正文图片缓存

在开启了正文缓存的前提下，SDK会对infoType为文章（article）的新闻正文的文本和图片进行缓存。

若用户有对正文展示的图片进行操作（例如查看大图）的需求，调用该接口可以获取已经缓存的图片，若正文图片未缓存，SDK会下载该正文图片并缓存该图片到本地，可以避免同一张图片加载两次而占用过多的存储。

- 定义

```java
/**
 * 从缓存中加载新闻图片，如果本地没有，则从网络下载，并缓存本地
 *
 * @param infoId   新闻ID
 * @param srcUrl   图片原地址
 * @param listener 图片下载监听器
 * @param isMain   true: 回调必须在主线程，默认为true; false: 回调可在非主线程
 */
public void loadImgFromCache(String infoId, String srcUrl, final NNFDownloadRequestListener listener, boolean isMain)

/**
 * 从缓存中加载新闻图片，如果本地没有，则从网络下载，并缓存本地
 *
 * @param infoId   新闻ID
 * @param srcUrl   图片原地址
 * @param listener 图片下载监听器
 */
public void loadImgFromCache(String infoId, String srcUrl, final NNFDownloadRequestListener listener);
```

- 调用

```java
// 调用SDK接口获取图片
NNewsFeedsSDK.getInstance().loadImgFromCache(mInfoId, url, new NNFDownloadRequestListener() {
    @Override
    public void onDownloadProgressUpdate(long contentLength, long bytesRead) {

    }

    @Override
    public void onDownloadSuccess(File file) {

    }

    @Override
    public void onDownloadFailure(Exception e) {

    }
});
```

- 图片获取回调接口说明

NNFDownloadRequestListener接口类，为调用SDK接口获取图片时的图片下载监听器。该监听器包含以下回调函数：

a. 图片文件下载进度

```java
/**
 * 图片文件下载进度
 *
 * @param contentLength 图片文件长度
 * @param bytesRead     已下载长度
 */
void onDownloadProgressUpdate(long contentLength, long bytesRead);
```

该回调函数用于监听图片文件下载进度。其中，contentLength为图片文件长度；bytesRead为文件已下载长度。


b. 图片下载成功

```java
/**
 * 图片下载成功
 *
 * @param file 已缓存或已下载的文件
 */
void onDownloadSuccess(File file);
```

当图片下载成功时触发该回调。其中，file为已下载的文件。

c. 图片下载异常信息

```java
/**
 * 图片下载异常信息
 *
 * @param e 异常信息
 */
void onDownloadFailure(Exception e);
```

### 4. 清除某一条新闻的正文缓存

用户可根据一定的缓存清除策略删除某一新闻的正文及图片缓存。

- 定义

```java
/**
* 根据新闻ID删除新闻详情缓存
*
* @param infoId 新闻ID
*/
public void removeNewsDetails(String infoId);
```

- 调用

```java
NNewsFeedsSDK.getInstance().removeNewsDetails(infoId);
```

### 5. 清除所有的正文缓存

用户可手动清除所有的新闻正文及图片缓存。

- 定义

```java
/**
 * 清空缓存的新闻文本和图片
 *
 * @param listener 执行结果回调
 */
public void clearDetailsCache(NNFClearDetailsCacheListener listener);
```

- 调用

SDK将清除缓存的操作放在线程中执行，待执行完毕后触发回调`onClearedFinish`。

```java
NNewsFeedsSDK.getInstance().clearDetailsCache(new NNFClearDetailsCacheListener() {
    @Override
    public void onClearedFinish() {
        
    }
});
```

## 用户行为统计

我们鼓励合作方接入用户行为统计接口，通过采集用户行为，结合网易有料专业数据挖掘能力和智能推荐算法得到的用户画像将会更精准。

值得注意的是，用户行为统计接口均封装在 NNFTracker 中。请确保 SDK 初始化后再调用 NNFTracker 中的接口，否则会抛 NullPointerException。

### 点击事件

当文章或图集被点击，视频开始播放时，建议调用该接口进行点击行为上报。

- 定义

```java
/**
 * 新闻点击的事件收集
 *
 * @param newsInfo
 */
public void trackNewsClick(NNFNewsInfo newsInfo);
```

- 调用


```java
NNFTracker.getInstance().trackNewsClick(newsInfo);
```


### 浏览事件

当文章或图集浏览结束，视频播放暂停或结束时，建议调用该接口进行浏览行为上报。用户可以自主统计浏览时长和浏览进度一并上报。

- 定义

```
/**
 * 文章或图集浏览结束、视频播放结束的事件收集
 *
 * @param newsInfo
 * @param cost     浏览时长 - 单位毫秒（点击进去，到返回的时间）
 * @param progress 浏览进度
 */
public void trackNewsBrowse(NNFNewsInfo newsInfo, long cost, double progress);
```

- 调用

```java
NNFTracker.getInstance().trackNewsBrowse(newsInfo, cost, progress);
```

### 新闻正文浏览事件

SDK将infoType为文章（article）的新闻正文的展示封装成NNFNewsDetailsWebView。同时，为便于新闻正文浏览事件的统计，当WebView销毁时，我们默认上报新闻正文的浏览时长和浏览进度。

用户若不希望上报浏览事件或想自主上报，则可在 NNFNewsDetailsWebView 加载数据时，将 reportAction 置为false即可。 

- 数据加载接口定义

```java
/**
 * @param newsInfo        列表页可展现的新闻信息
 * @param reportAction    当WebView销毁时，是否上报新闻正文的浏览时长和浏览进度。默认为true。
 * @param webViewListener
 */
public void loadNewsDetails(NNFNewsInfo newsInfo, boolean reportAction, NNFWebViewListener webViewListener);
```

- 禁止上报浏览事件

```java
mNewsDetailsWebView.loadNewsDetails(newsInfo, false, webViewListener);
```



## 广告（v1.2新增）

从v1.2开始，我们的SDK内嵌了腾讯广点通广告SDK，用户在使用本SDK的时候，确保工程中未使用广点通的SDK。

如果拉取广告的必要权限均被授权，会在轮播图，普通新闻列表和新闻详情出现广告。

(1) 轮播图

如果拉取广告的必要权限均被授权，以loadType 为 1 请求新闻列表时，若返回的 NNFNews.banners 有数据，即表示有轮播图，那么会在轮播图中随机插入一条广告，且广告不是在第一个位置。

(2) 普通新闻

如果拉取广告的必要权限均被授权，当请求的新闻条数小于10条时，不会出现广告；当请求的新闻条数大于等于10，小于15时，在第4条位置出现广告；当请求条数大于等于15条时，第4、10条位置将会出现广告。

(3) 新闻详情

如果拉取广告的必要权限均被授权，拉取新闻详情时，若成功拉取到广告，则返回的NNFNewsDetails中包含广告信息。

### 广告渲染

SDK返回给用户的广告数据模型为：

- 单条广告数据模型：NNFAdInfo

名称 | 类型 | 描述
---|---|---|---
title | String | 标题，短文字,14字以内
desc | String | 描述，长文字,30字以内
iconUrl | String | 获取Icon图片地址
imgUrl | String | 获取大图地址
isApp | boolean | 返回是否为APP广告
appStatus | int | 获取应用状态，0：未开始下载；1：已安装；2：需要更新; 4:下载中; 8:下载完成; 16:下载失败
progress | int | 获取APP类广告下载中的下载进度
downloadCount | long | 获取APP类广告的下载数
appScore | int | 获取应用评级
appPrice | Double | 获取APP类应用价格

返回的广告数据模型NNFAdInfo包含展示一条广告所需的标题、描述、图片地址等等。广告视图的渲染由app开发人员完成。

### 广告点击事件

- 定义

```java
/**
 * 广告点击时调用
 *
 * @param adInfo 广告数据
 * @param view   被点击的view组件
 */
public void onAdClicked(NNFAdInfo adInfo, View view)
```

- 调用示例

```java
NNFAdCell adParams = mNewsDetails.ad;
if (null != adParams) {
    NNewsFeedsSDK.getInstance().onAdClicked(adParams.adInfo, NNFNewsDetailsWebView.this);
}
```

广告SDK内部封装了广告点击后的页面跳转逻辑，用户只需调用 onAdClicked 传入广告数据实例 adInfo 及 被点击的view 实例。SDK内部会进行广告曝光和广告计费。

## 推送（v1.2新增）

我们的SDK依赖了推送SDK，使用的是个推的第三方推送SDK，相关使用参考个推官网开发使用文档：http://docs.getui.com/mobile/android/androidstudio_maven/

## 附录

### 数据模型

- 频道列表：NNFChannels

名称 | 类型 | 描述
---|---|---
channels| NNFChannelInfo[]| 频道列表


- 单个频道：NNFChannelInfo

名称 | 类型 | 示例 | 描述
---|---|---|---
channelId| String| | 频道ID
channelName | String ||频道名称
order | int | 1 | 频道显示的顺序

- 新闻列表：NNFNews

名称 | 类型 | 描述
---|---|---
infos| NNFNewsInfo[]| 普通新闻
banners | NNFNewsInfo[] |轮播图
tops | NNFNewsInfo[] |置顶新闻

- 单个新闻：NNFNewsInfo

名称 | 类型 | 描述
---|---|---|---
producer | String | 新闻提供者，user表示用户自编辑新闻，recommendation表示来自个性化推荐系统
recId | String | 单次推荐唯一标示
algInfo| String| 推荐策略及权重信息
channelId | String | 频道ID
infoId| String| 新闻ID
infoType| String| 新闻类型：文章(article)、图集(picset)、视频(video)
title | String | 新闻标题
source | String | 新闻来源
updateTime | String | 新闻更新时间
summary | String | 新闻简介
thumbnails | NNFThumbnailInfo[] | 新闻缩略图
imgType | int | 缩略图类型，3：三图模式， 2：大图模式，1：缩略图模式，0：无图
hasVideo | boolean | 文章是否包含视频，true：有，false：没有
num | int | 图集中图片数量，仅当infoType为图集picset时才有值
videos | NNFVideoCell[] | 视频源信息，仅当infoType为视频video时才有值 
ad | NNFAdCell | 广告信息，当且仅当infoType为ad时infoType
readStatus | int | 0：未读 ； 1： 已读

- 视频模型：NNFVideoCell

名称 | 类型 | 描述
---|---|---
cover | String | 视频封面图
largeCover | String | 视频封面大图
playsize | int | 播放类型，0：表示4：3宽高比，1：表示16：9宽高比
duration |  int | 视频时长，单位为秒，仅当infoType为视频时才有值
mp4SdUrl | String | mp4格式视频链接，标清
mp4HdUrl | String | mp4格式视频链接，高清
m3u8SdUrl | String | m3u8格式视频链接，标清
m3u8HdUrl | String | m3u8格式视频链接，高清
sdUrl | String | flv格式视频链接，标清
hdUrl | String | flv格式视频链接，高清
shdUrl | String | flv格式视频链接，shd

- 缩略图模型：NNFThumbnailInfo

名称 | 类型 | 描述
---|---|---
url | String | 缩略图url
height | int | 缩略图高度
width | int | 缩略图宽度

- 新闻详情：NNFNewsDetails

名称 | 类型 | 示例 | 描述
---|---|---|---
infoId| String| | 新闻ID
infoType| String| article/picset/video | 新闻类型，文章/图集/视频
category | String || 新闻类目
title | String || 新闻标题
publishTime | String || 发布时间
source | String || 新闻来源
sourceLink | String || 原文地址
content | String |  | 新闻正文
imgs | NNFImageInfo[] |  | 图片列表
tag | String |  | 标签
ad | NNFAdCell | | 广告

- 新闻图片模型：NNFImageInfo

名称 | 类型 | 描述
---|---|---|---
url | String | 新闻图片url
height | int | 新闻图片高度
width | int | 新闻图片宽度
picType | int | 新闻图片类型
note | String | 图片描述，infoType为picset该字段才有值

- 单条广告参数模型：NNFAdCell

名称 | 类型 | 描述
---|---|---|---
adPlacementId | String | 广告位
mediumId | String | 广告表中的媒体id
producer | String | 广告生产者
adInfo | NNFAdInfo | 广告数据

- 单条广告数据模型：NNFAdInfo

名称 | 类型 | 描述
---|---|---|---
title | String | 标题，短文字,14字以内
desc | String | 描述，长文字,30字以内
iconUrl | String | 获取Icon图片地址
imgUrl | String | 获取大图地址
isApp | boolean | 返回是否为APP广告
appStatus | int | 获取应用状态，0：未开始下载；1：已安装；2：需要更新; 4:下载中; 8:下载完成; 16:下载失败
progress | int | 获取APP类广告下载中的下载进度
downloadCount | long | 获取APP类广告的下载数
appScore | int | 获取应用评级
appPrice | Double | 获取APP类应用价格
