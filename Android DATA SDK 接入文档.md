# Android SDK 接入文档

# SDK 概述

网易有料NewsFeedsSDK为移动应用提供内容智能分发功能，对外提供较为简洁的API接口，方便第三方应用快速的集成并实现内容分发功能。SDK兼容Android 14+，Demo兼容Android 14+。

NewsFeedsSDK提供的功能如下：

- 获取频道列表
- 获取新闻列表
- 获取新闻详情
- 获取（文章、图集、视频）相关推荐
- 用户行为采集上传


# SDK类说明

网易有料NNewsFeedsSDK主要提供了以下类：

- NNewsFeedsSDK：整个SDK的主入口，单例，主要提供初始化，配置用户信息，加载频道、新闻列表、新闻详情的功能
- NNFTracker：用户行为追踪的单例类
- NNFChannels：频道列表的model类
- NNFChannelInfo：单个频道的model类
- NNFNews：新闻列表的model类
- NNFNewsInfo：单个新闻的model类
- NNFNewsDetails：新闻详情的model类
- NNFAdInfo：单个广告的model类

# 开发准备

### 1. 外部依赖说明

- Json解析

我们的SDK远程依赖了fastjson，fastjson版本号为1.2.8，若您的App也依赖了fastjson，请确保您依赖的fastjson版本与1.2.8兼容。

```java
compile 'com.alibaba:fastjson:1.2.8'
```

- 广告（v1.2新增）

从v1.2开始，我们的SDK内嵌了腾讯广点通广告SDK，用户在使用本SDK的时候，确保工程中未使用广点通的SDK。

- 推送（v1.2新增）

我们的SDK依赖了推送SDK，使用的是个推的第三方推送SDK，相关使用参考个推官网开发使用文档：http://docs.getui.com/mobile/android/androidstudio_maven/

从v1.3开始，用户可以自主选择是否集成个推SDK。若不需要推送服务，请直接进入下一步。

若要接入推送服务，请联系网易有料CMS后台获取相应的推送`APP_ID`、`APP_KEY`、`APP_SECRET`的值。推送完整文档请参考：[个推Android接入文档](http://docs.getui.com/getui/mobile/android/androidstudio_maven/)

---


### 2. Gradle集成网易有料

- jcenter远程依赖

我们的sdk已同步到Jcenter仓库，开发人员只需在app module下的build.gradle中引入我们sdk的依赖，请自行将x.x替换为版本号，目前最新版为1.4.5

```java
compile 'com.netease.youliao:newsfeeds-data:x.x'
```

---

### 3. 权限（v1.2新增）

从v1.2开始，我们的SDK内部提供广告功能，为了成功拉取到广告，需要在CMS上开启广告业务开关，并在AndroidManifest.xml中添加权限声明：

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

如果您打包App时的targetSdkVersion >= 23：请先获取到SDK要求的所有权限，然后调用SDK获取到的新闻列表和新闻详情才会包含广告数据。否则SDK返回的新闻列表和新闻详情将不包含广告数据。我们建议您在App启动时就去获取SDK需要的权限。您可以参考如下权限处理示例代码，权限示例代码建议写在启动页SplashActivity中：

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

---

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

---

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

# fastjson
-keep class javax.ws.rs.** { *; }
-dontwarn com.alibaba.fastjson.**
-keep class com.alibaba.fastjson.** { *; }
```

# 初始化

在自定义Application的OnCreate中添加以下代码，初始化我们的SDK。由于您的应用可能不止一个进程，建议只在主进程下初始化我们的SDK。示例代码如下：

```
public class YLApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        String processName = getProcessName();
        // 判断进程名，保证只有主进程才初始化网易有料DATA SDK
        if (!TextUtils.isEmpty(processName) && processName.equals(this.getPackageName())) {
            /**
             * 初始化SDK：在自定义Application中初始化网易有料DATA SDK
             */
            new NNewsFeedsSDK.Builder()
                .setAppKey("3c11d60d903e49d5a47ad2a58bb0db97")
                .setAppSecret("ca5137e40b874abd893e762f1d53d839")
                .setContext(getApplicationContext())
                .setCacheEnabled(true)
                .setMaxCacheNum(60)
                .setMaxCacheTime(1 * 24 * 60 * 60 * 1000)
                .setLogLevel(NNFLogUtil.LOG_VERBOSE)
                .build();
        }
    }

    public static String getProcessName() {
        try {
            File file = new File("/proc/" + android.os.Process.myPid() + "/" + "cmdline");
            BufferedReader mBufferedReader = new BufferedReader(new FileReader(file));
            String processName = mBufferedReader.readLine().trim();
            mBufferedReader.close();
            return processName;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

初始化接口及参数说明

接口 | 参数 | 类型 | 描述
---|---|---|---
setAppKey | appKey | String | 分配给应用的唯一标识，用户在CMS后台新建应用时生成
setAppSecret | appSecret | String |  分配给应用的唯一秘钥，用户在CMS后台新建应用时生成
setContext | context | Context | 传入app的Context，建议传入ApplicationContext
setCacheEnabled | cacheEnabled | boolean | 设置频道列表、新闻列表、新闻详情缓存的开启状态，默认true
setMaxCacheNum | maxCacheNum | int | 配置每个频道最大缓存新闻数量，默认60条
setMaxCacheTime | maxCacheTime | long | 配置新闻列表及新闻详情文本最大缓存时长，单位毫秒，默认1天（1 * 24 * 60 * 60 * 1000）
setLogLevel | logLevel | int | Android Studio等开发工具的 控制台Log等级，指定哪些日志需要输出


# 日志管理

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

Log等级指的是Android Studio等开发工具的控制台Log等级，指定哪些日志需要输出。Log等级只能取指定的整数值：

Log等级 | 说明 
---|---
NNFLogUtil.LOG_NONE| 不打印日志
NNFLogUtil.LOG_ERROR | 打印 ERROR
NNFLogUtil.LOG_WARN | 打印 ERROR、WARN
NNFLogUtil.LOG_INFO | 打印 ERROR、WARN、INFO
NNFLogUtil.LOG_DEBUG | 打印 ERROR、WARN、INFO、DEBUG
NNFLogUtil.LOG_VERBOSE | 打印 ERROR、WARN、INFO、DEBUG、VERBOSE

# 数据接入接口说明

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

---

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

- 示例


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

- 接口返回 NNFChannels

名称 | 类型 | 描述
---|---|---
channels| NNFChannelInfo[]| 频道列表


其中，单个频道的数据模型NNFChannelInfo各字段含义如下：

名称 | 类型 | 示例 | 描述
---|---|---|---
channelId| String| | 频道ID
channelName | String ||频道名称
channelOrder | int | 1 | 频道显示的顺序
channelType | int | 1 | （v1.3新增）频道类型，其中4表示自营频道

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

---

### 加载新闻列表

SDK提供多个加载新闻列表的接口，用户可根据需要传入不同的参数组合。v1.3之前的版本只需传入channelId即可获取新闻列表；v1.3版本以后，由于NNFChannelInfo新增了channelType字段用于区分自营频道与非自营频道，所以，强烈建议开发者传入NNFChannelInfo类型的实例作为列表请求参数。自营频道新闻列表数据返回策略与非自营频道有所不同，具体策略请参考网易有料Api Server文档。

- （v1.3新版）定义

```java
/**
 * @param channelInfo 频道实例
 * @param listener    网络请求回调
 */
public void loadNewsList(NNFChannelInfo channelInfo, NNFHttpRequestListener<NNFNews> listener)
```

```java
/**
 * @param channelInfo 频道实例
 * @param num         新闻列表长度，默认10条
 * @param listener    网络请求回调
 */
public void loadNewsList(NNFChannelInfo channelInfo, int num, NNFHttpRequestListener<NNFNews> listener)
```

```java
/**
 * @param channelInfo 频道信息
 * @param num         新闻列表长度，默认10条
 * @param loadType    新闻列表内容组合 0：接口返回推荐系统或用户编辑的普通新闻 1：接口返回用户编辑的头图新闻、用户编辑的置顶新闻、推荐系统或用户编辑的普通新闻
 * @param listener    网络请求回调
 */
public void loadNewsList(NNFChannelInfo channelInfo, int num, int loadType, NNFHttpRequestListener<NNFNews> listener)
```

接收经纬度

```java
/**
 * @param channelInfo 频道信息
 * @param num         新闻列表长度，默认10条
 * @param loadType    新闻列表内容组合 0：接口返回推荐系统或用户编辑的普通新闻 1：接口返回用户编辑的头图新闻、用户编辑的置顶新闻、推荐系统或用户编辑的普通新闻
 * @param longitude   经度，地球坐标格式，经纬度要么都填要么都不填
 * @param latitude    纬度，地球坐标格式，经纬度要么都填要么都不填
 * @param listener    网络请求回调
 */
public void loadNewsList(NNFChannelInfo channelInfo, int num, int loadType, double longitude, double latitude, NNFHttpRequestListener<NNFNews> listener)
```

- （旧版）定义

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

- 示例

```java
NNewsFeedsSDK.getInstance().loadNewsList(channelId, 10, isRefresh ? 1 : 0, GeoInfo.getLongitude(), GeoInfo.getLatitude(), new NNFHttpRequestListener<NNFNews>() {
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

1：下拉刷新时，传入1，返回普通新闻、头图新闻、置顶新闻。

0：上拉加载更多时，传入0，只返回普通新闻。

- 请求参数 num

num为期望返回的新闻列表长度，默认10条。num仅为普通新闻的条数，头图或置顶新闻不计算在内。

- 接口返回 NNFNews

新闻列表数据模型（NNFNews）各字段说明如下：

字段 | 类型 | 描述
---|---|---
banners | NNFNewsInfo[] | 轮播图或头图新闻
tops | NNFNewsInfo[] | 置顶新闻
infos| NNFNewsInfo[]| 普通新闻

其中，单个新闻（NNFNewsInfo）按类型（infoType）可分为：文章(article)、图集(picset)、视频(video)、广告(ad)。

(1) 轮播图

轮播图包含的新闻类型有：文章(article)、图集(picset)、广告(ad)。

这里要注意的是，只有在CMS上开启广告业务开关且拉取广告的必要权限均被授权的条件下，广告才能拉取成功，返回的轮播图中才可能出现广告。

(2) 置顶新闻

置顶新闻包含的新闻类型有：文章(article)、图集(picset)、视频(video)。

这里要注意的是，置顶新闻不含广告(ad)。

(3) 普通新闻列表

普通新闻包含的新闻类型有：文章(article)、图集(picset)、视频(video)、广告(ad)。

这里要注意的是，只有在CMS上开启广告业务开关且拉取广告的必要权限均被授权的条件下，广告才能拉取成功，返回的普通新闻列表中才可能出现广告。

下拉刷新时，接口返回头图新闻、置顶新闻、普通新闻，典型的Feed流展示逻辑是按照从上到下为头图新闻、置顶新闻、普通新闻的顺序，将下拉刷新的数据插入到当前Feed流头部；上拉加载更多时，接口只返回普通新闻，典型的Feed流展示逻辑是将上拉加载更多的数据追加到当前Feed流尾部。App开发人员可参考如下示例代码进行数据展示：

```java
private void bindData(NNFNews news) {
    ...
    // 新增的普通新闻的条数
    int delta = 0;
    // 移除旧的头图和置顶
    if (isRefresh) {
        // 移除原有头图
        removeBanner();
        // 删除原有置顶
        removeTops();
    }
    
    if (null != news.infos) {
        // 解析普通新闻
        NNFNewsInfo[] newsInfos = news.infos;
        if (null != newsInfos && newsInfos.length > 0) {
            int length = newsInfos.length;
            delta = length;
            if(isRefresh) { // 下拉刷新
	            for (int i = 0; i < length; i++) {
	                NNFNewsInfo newsInfo = newsInfos[i];
	                // 在头部插入
	                addTopImp(newsInfo);
	            }
            } else { // 上拉加载更多
	            for (int i = 0; i < length; i++) {
	                NNFNewsInfo newsInfo = newsInfos[i];
	                // 添加到尾部
	                addBottomImp(newsInfo);
		         }
            }
        }
    }
    
    // 展示新的头图和置顶
    if (isRefresh) {
        // 解析置顶
        NNFNewsInfo[] tops = news.tops;
        // 在头部插入
        bindTops(tops);

        // 解析头图
        NNFNewsInfo[] banners = news.banners;
        // 在头部插入
        bindBanner(banners);
    }

    // 刷新视图
    mContactView.UpdateNewsListView(delta, isRefresh);
}
```

单个新闻摘要NNFNewsInfo部分字段说明如下，完整字段请参考附录：

名称 | 类型 | 描述
---|---|---|---
infoId| String| 新闻ID
infoType| String| 新闻类型：文章(article)、图集(picset)、视频(video)
title | String | 新闻标题
source | String | 新闻来源
updateTime | String | 新闻更新时间
summary | String | 新闻简介
thumbnails | NNFThumbnailInfo[] | 新闻缩略图
imgType | int | 缩略图类型，3：三图模式， 2：大图模式，1：单图模式，0：无图
num | int | 图集中图片数量，仅当infoType为图集picset时才有值
videos | NNFVideoCell[] | 视频源信息，仅当infoType为视频video时才有值 
ad | NNFAdCell | 广告信息，当且仅当infoType为ad时infoType
readStatus | int | 0：未读 ； 1： 已读

这里要注意的是，单个新闻摘要按照 imgType 可分为三图模式、大图模式、单图模式、无图模式。不管是哪一种imgType，具体的缩略图均存于thumbnails字段。

- 错误码说明

error的特殊处理：并不是所有error都是网络请求失败，其中

```
errorCode == 4000    表示后台管理系统删除了某个频道，App在未重新拉取更新频道列表的情况下，获取该频道下的新闻列表会返回code为4000的错误

errorCode == 4004    表示该频道下拉取不到任何的新闻列表信息，暂时没有可推荐的新闻，会返回code为4004的错误
```

---

### 加载相关推荐列表

- 定义

```java
/**
 * @param infoId   新闻ID
 * @param infoType 新闻类型，文章/图集/视频
 * @param listener 网络请求回调
 */
public void loadRelatedNews(String infoId, String infoType, NNFHttpRequestListener<NNFNews> listener)
```

- 示例 

```java
NNewsFeedsSDK.getInstance().loadRelatedNews(mNewsInfo.infoId, mNewsInfo.infoType, new NNFHttpRequestListener<NNFNews>() {
    @Override
    public void onHttpSuccessResponse(NNFNews result) {
        mContactView.hideProgressDialog();
        mRelatedNNFNewsInfos = result.infos;
        mContactView.parseDetails(mNNFNewsDetails, mRelatedNNFNewsInfos);
    }

    @Override
    public void onHttpErrorResponse(int code, String errorMsg) {
        mContactView.hideProgressDialog();
        mContactView.parseDetails(mNNFNewsDetails, mRelatedNNFNewsInfos);
    }
});

```

----

### 各种类型的新闻展现

当新闻列表中的单个新闻（NNFNewsInfo）被点击时，App开发人员需要根据新闻类型（infoType）实现对应的展现页面。例如，当infoType为video时，需要开始播放视频或者跳转到视频播放页面；当infoType为ad时，需要跳转到广告落地页面；当infoType为article时，需要跳转到文章详情页面；当infoType为picset时，需要跳转到图集展示页面。可参考如下代码片段：

```java
String infoType = newsInfo.infoType;
switch (infoType){
	case "video":
	    // 跳转到视频类新闻展示页
	    MoreVideoActivity.start(context, newsInfo);
	    ...
	    break;
	case "ad":
	    // 跳转到广告落地页
	    NNFAdCell ad = newsInfo.ad;
	    if (null != ad) {
	        NNewsFeedsSDK.getInstance().onAdClicked(ad.adInfo, view);
	    }
	    break;
	case "article":
	    // 跳转到文章类新闻展示页
	    NewsDetailsWebActivity.start(context, newsInfo);
	    break;
	case "picset":
	    // 跳转到图集类新闻展示页
	    GalleryActivity.start(context, newsInfo);
	    break;
}
```

单个新闻数据模型NNFNewsInfo部分字段说明如下：

名称 | 类型 | 描述
---|---|---|---
infoId| String| 新闻ID
infoType| String| 新闻类型：文章(article)、图集(picset)、视频(video)
thumbnails | NNFThumbnailInfo[] | 新闻缩略图
imgType | int | 缩略图类型，3：三图模式， 2：大图模式，1：缩略图模式，0：无图
num | int | 图集中图片数量，仅当infoType为图集picset时才有值
videos | NNFVideoCell[] | 视频源信息，仅当infoType为视频video时才有值 
ad | NNFAdCell | 广告信息，当且仅当infoType为ad时才有值 

##### 1. 视频类新闻展现

由NNFNewsInfo数据模型可知，当 infoType 为 video 时，NNFNewsInfo中的videos字段会给出视频源信息，App开发人员可根据该视频源信息完成视频的下载、播放等交互逻辑。

视频源数据模型NNFVideoCell字段说明如下：

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

App开发人员可以根据视频封面和视频宽高比渲染视频视图，并根据视频链接拉取视频流实现播放。

---

##### 2. 广告类新闻展现

当 infoType 为 ad 时，NNFNewsInfo中的ad字段会给出广告信息，广告信息包含展示一条广告所需的标题、描述、图片地址等等。

- 单条广告数据模型NNFAdInfo主要字段

名称 | 类型 | 描述
---|---|---|---
title | String | 标题，短文字,14字以内
desc | String | 描述，长文字,30字以内
iconUrl | String | 获取Icon图片地址
imgUrl | String | 获取大图地址
isApp | boolean | 返回是否为APP广告
producer | String | 广告生产者（gdt/inmobi）
landingURL | String | 落地页（仅producer为inmobi或ytg时有值）
trackingMap | Map | 点击和曝光行为上报相关信息（仅producer为inmobi有值）
thumbnailInfos | NNFThumbnailInfo[] | 缩略图数组
imageType | int | 缩略图类型，3：三图模式， 2：大图模式，1：单图模式，0：无图

广告视图的渲染由App开发人员完成。我们的SDK封装了广告点击后的落地页，为了确保广告准确计费，需要App开发人员自行调用广告曝光和广告点击接口，**在调用广告点击接口之前，一定要先调用广告曝光接口**。

- 广告曝光接口定义

```java
public void reportExposure(View view)
```

其中，view为被点击的广告视图实例。

- 广告曝光接口调用示例

```java
adInfo.reportExposure(view);
```

其中，adInfo 为广告数据模型，view为被点击的广告视图实例。

- 广告点击接口定义

```java
public void reportAdClickAndOpenLandingPage(View view)
```

其中，view为被点击的广告视图实例。

- 广告点击接口调用示例

```java
adInfo.reportAdClickAndOpenLandingPage(view);
```

其中，adInfo 为广告数据模型，view为被点击的广告视图实例。

---

##### 3. 文章类新闻展现

NNFNewsInfo数据模型只给出了文章类新闻的摘要信息，若要获取文章正文，需要调用`loadNewsDetails`接口拉取文章正文信息。

- 定义

```java
/**
 * v1.4.5版开始，无需传入infoType
 *
 * @param infoId   新闻ID
 * @param producer 新闻提供者，user表示用户自编辑新闻，recommendation表示来自个性化推荐系统
 * @param listener 网络请求回调
 */
public void loadNewsDetails(String infoId, String producer, NNFHttpRequestListener<NNFNewsDetails> listener)
```
- 示例

```java
NNewsFeedsSDK.getInstance().loadNewsDetails(newsInfo.infoId, newsInfo.producer, new NNFHttpRequestListener<NNFNewsDetails>() {
    @Override
    public void onHttpSuccessResponse(NNFNewsDetails result) {
        
    }
    
    @Override
    public void onHttpErrorResponse(int code, String errorMsg) {
    
    }
});
```

- 接口返回 NNFNewsDetails

新闻详情NNFNewsDetails字段说明：

名称 | 类型 | 示例 | 描述
---|---|---|---
infoId| String| | 新闻ID
infoType| String| article/picset | 新闻类型，文章/图集
category | String || 新闻类目
title | String || 新闻标题
publishTime | String || 发布时间
source | String || 新闻来源
sourceLink | String || 原文地址
content | String |  | 新闻正文
imgs | NNFImageInfo[] |  | 图片列表
tag | String |  | 标签
ad | NNFAdCell | | 广告

从模型字段描述可知，当NNFNewsDetails.infoType为article时，文章类新闻正文由content字段给出，为支持图片异步加载，content中img标签被替换成了`${{index}}$`，index从0开始。正文中图片链接存于imgs字段。具体说明请参考网易有料Api Server文档。

从v1.2开始，如果CMS上开启了广告业务开关且拉取广告的必要权限均被授权，用户请求新闻详情成功后，返回的 NNFNewsDetails 中可能包含广告，广告信息存于ad字段。广告的渲染与点击操作与新闻列表中的广告类似。

---

##### 4. 图集类新闻展现

NNFNewsInfo数据模型只给出了图集类新闻的缩略图信息，若要获取图集的图片列表，需要调用`loadNewsDetails`接口拉取图集详情信息。`loadNewsDetails`接口定义及示例请参考文章类新闻展现部分。

当NNFNewsDetails.infoType为picset时，imgs字段表示图集中图片列表。

新闻详情NNFNewsDetails部分字段说明：

名称 | 类型 | 示例 | 描述
---|---|---|---
infoId| String| | 新闻ID
infoType| String| article/picset | 新闻类型，文章/图集
imgs | NNFImageInfo[] |  | 图片列表

单个图片数据模型NNFImageInfo字段说明：

名称 | 类型 | 描述
---|---|---|---
url | String | 新闻图片url
height | int | 新闻图片高度
width | int | 新闻图片宽度
picType | int | 新闻图片类型
note | String | 图片描述，infoType为picset该字段才有值

App开发人员需要根据图片列表自行实现图集展示页。

### 缓存相关

#### 缓存开关设置

- 定义

```java
/**
 * 设置频道列表、新闻列表、新闻详情缓存的开启状态
 * 注意：若用户同时使用UI SDK，强烈不建议用户关闭缓存
 *
 * @param cacheEnabled 缓存开启状态，默认true
 * @return
 */
public Builder setCacheEnabled(boolean cacheEnabled)
```

- 示例

```java
new NNewsFeedsSDK.Builder()
    .setCacheEnabled(true)
    .build();
```
用户只使用数据SDK可以关闭缓存，由用户自定义缓存的实现；若用户使用了UI SDK，不建议关闭缓存，会引起部分数据的显示等异常。

---

#### 最大缓存时长设置

- 定义

```java
/**
 * 配置新闻列表及新闻详情文本最大缓存时长
 *
 * @param maxCacheTime 最大缓存时长，单位毫秒，默认1*24*60*60*1000，即一天
 */
public Builder setMaxCacheTime(long maxCacheTime)
```

- 示例

```java
new NNewsFeedsSDK.Builder()
    .setMaxCacheTime(1 * 24 * 60 * 60 * 1000)
    .build();
```

---


#### 频道最大缓存数量设置

- 定义

```java
/**
 * 配置每个频道最大缓存新闻数量
 *
 * @param maxCacheNum 最大缓存新闻数量，默认60条
 */
public Builder setMaxCacheNum(int maxCacheNum)
```

- 示例

```java
new NNewsFeedsSDK.Builder()
    .setMaxCacheNum(60)
    .build();
```

---

#### 删除过期的新闻

- 定义

```java
/**
 * 删除过期新闻及其详情
 *
 * @param listener 删除进度回调
 */
public void deleteExpireCachedNews(NNFClearCacheListener listener)
```

- 示例

```java
NNewsFeedsSDK.getInstance().deleteExpireCachedNews(new NNFClearCacheListener() {
    @Override
    public void onClearFinish() {
        
    }
});
```

---

#### 删除所有缓存的新闻

- 定义

```java
/**
 * 删除缓存的新闻及其详情
 *
 * @param listener 删除进度回调
 */
public void deleteAllCachedNews(NNFClearCacheListener listener)
```

- 示例

```java
NNewsFeedsSDK.getInstance().deleteAllCachedNews(new NNFClearCacheListener() {
    @Override
    public void onClearFinish() {
        
    }
});
```

---

#### 计算当前的缓存大小

- 定义

```java
/**
 * 计算缓存的大小
 * 缓存的大小，单位 bytes
 * 缓存较大的时候时候计算时间较长，请合理使用
 *
 * @return The length, in bytes
 */
public long getCacheSize()
```

- 示例

```java
long cacheSize = NNewsFeedsSDK.getInstance().getCacheSize();
```

---

# 用户行为统计

我们鼓励合作方接入用户行为统计接口，通过采集用户行为，结合网易有料专业数据挖掘能力和智能推荐算法得到的用户画像将会更精准。

值得注意的是，用户行为统计接口均封装在 NNFTracker 中。**请确保 SDK 初始化后再调用 NNFTracker 中的接口，否则会抛 NullPointerException。**

### 曝光事件（v1.2新增）

为了增强推荐效果，当新闻曝光时，建议调用曝光接口进行曝光的行为事件统计。

- 定义

```java
/**
 * 新闻曝光事件搜集
 *
 * @param newsInfo
 */
public void trackNewsExposure(NNFNewsInfo newsInfo)
```

- 示例

```java
NNFTracker.getInstance().trackNewsExposure(newsInfo);
```

==注意：== 同一篇新闻曝光可以多次上传，后台会过滤，建议为了节省流量，app开发者应尽量避免同一篇新闻的曝光事件多次上传

==使用说明：==

快速滑动新闻列表时，滑过屏幕的新闻来不及阅读，并不能算作新闻曝光，因此，我们建议单个新闻单元格暴露在当前屏幕0.5s以上时进行曝光打点。

- 新闻列表曝光说明

下面以RecyclerView为例，给出一种曝光统计方案：

##### 1. 在新闻单元格添加到列表的时候记录曝光开始时间

```java
@Override
public void onChildViewAttachedToWindow(View view) {
    // 曝光统计开始
    RecyclerView.ViewHolder viewHolder = mRecyclerView.getChildViewHolder(view);
    if (null != viewHolder) {
        int pos = viewHolder.getAdapterPosition();
        if (pos < 0 || pos >= mPresenter.getTAdapterItems().size()) {
            return;
        }
        TAdapterItem adapterItem = mPresenter.getTAdapterItems().get(pos);
        if (null != adapterItem) {
            Object model = adapterItem.getDataModel();
            if (model instanceof WrapNewsInfo) {
                WrapNewsInfo wrapNewsInfo = (WrapNewsInfo) model;
                wrapNewsInfo.explosureTime = System.currentTimeMillis();
                YLLog.v(TAG, "曝光统计开始->@channelPos = " + mChannelPos + ", @newsTitle = " + wrapNewsInfo.newsInfo.title);
            }
        }

    }
}
```

---

##### 2. 在新闻单元格从列表移除的时候曝光打点

```java
@Override
public void onChildViewDetachedFromWindow(View view) {

    // 曝光统计结束
    RecyclerView.ViewHolder viewHolder = mRecyclerView.getChildViewHolder(view);
    if (null != viewHolder) {
        int pos = viewHolder.getAdapterPosition();
        explosureImp(pos);
    }
}

public void explosureImp(int pos) {
    int size = mPresenter.getTAdapterItems().size();
    if (pos < 0 || pos >= size) {
        return;
    }
    TAdapterItem adapterItem = mPresenter.getTAdapterItems().get(pos);
    if (null != adapterItem) {
        Object model = adapterItem.getDataModel();
        if (model instanceof WrapNewsInfo) {
            WrapNewsInfo wrapNewsInfo = (WrapNewsInfo) model;
            long duration = System.currentTimeMillis() - wrapNewsInfo.explosureTime;
            if (!wrapNewsInfo.isExplosured && duration >= 500) {
                NNFTracker.getInstance().trackNewsExposure(wrapNewsInfo.newsInfo);
                wrapNewsInfo.isExplosured = true;
                YLLog.v(TAG, "曝光统计结束->@channelPos = " + mChannelPos + ", @newsTitle = " + wrapNewsInfo.newsInfo.title);
            }
        }
    }
}
```

---

##### 3. 列表未滚动时，新闻列表当前屏的曝光统计

由于上述曝光事件只有在新闻单元格从新闻列表移除时才会触发，也就是只有新闻列表滚动的时候才会触发。当页面未滚动时，停留在当前页面的新闻单元格按照上述方法无法上传曝光事件，因此需要额外时机上传当前页面的新闻单元格的曝光事件。建议App开发人员在以下三种情形下，统计新闻列表当前屏的曝光：

```
1. 频道切换时，对上一频道的可见ListItem进行曝光统计；
2. 底部Tab若由信息流Tab切换到其他Tab时，对信息流Tab当前频道的可见ListItem进行曝光统计；
3. 应用退出时，若当前Tab为信息流Tab，对信息流Tab当前频道的可见ListItem进行曝光统计。
```

新闻列表当前屏的曝光统计示例代码如下：

```java
/**
 * 统计新闻列表当前屏的曝光
 */
public void calExplosure() {
    if (null == mRecyclerView) return;
    RecyclerView.LayoutManager layoutManager = mRecyclerView.getLayoutManager();
    //判断是当前layoutManager是否为LinearLayoutManager
    // 只有LinearLayoutManager才有查找第一个和最后一个可见view位置的方法
    if (layoutManager instanceof LinearLayoutManager) {
        LinearLayoutManager linearManager = (LinearLayoutManager) layoutManager;
        //获取最后一个可见view的位置
        int lastItemPosition = linearManager.findLastVisibleItemPosition();
        //获取第一个可见view的位置
        int firstItemPosition = linearManager.findFirstVisibleItemPosition();

        HTLog.v("统计新闻列表当前屏的曝光->@channelPos = " + mChannelPos + ", @firstItemPosition = " + firstItemPosition + ", @lastItemPosition = " + lastItemPosition);
        for (int i = firstItemPosition; i <= lastItemPosition; i++) {
            explosureImp(i);
        }
    }
}
```

总的来说，新闻列表的曝光统计策略如下：

```
1. ListItem添加到List时，记录开始时间；
2. ListItem从List移除时，计算曝光时长，达到500ms打点曝光事件（已打点的ListItem不重复打点）。
3. 频道切换时，对上一频道的可见ListItem进行曝光统计；
4. 底部Tab若由信息流Tab切换到其他Tab时，对信息流Tab当前频道的可见ListItem进行曝光统计；
5. 应用退出时，若当前Tab为信息流Tab，对信息流Tab当前频道的可见ListItem进行曝光统计。
```

以上只是以RecyclerView为例说明新闻列表的一种曝光统计方案，App开发人员可以根据实际情况设计自己的曝光统计方案。

- banner轮播图曝光说明

用户一般不会快速地滑动轮播图，因此，我们无需考虑轮播图曝光时间的问题，当切换到轮播图的某一页时，进行曝光打点。

```java
@Override
public void onPageSelected(int position) {
    // 下标从0开始
    int index = mBanner.toRealPosition(position);

    // 头图曝光采集
    if (index < 0 || index >= mWrapNewsInfos.length) {
        HTLog.e("OnBannerPageSelected 数组越界");
        return;
    }
    WrapNewsInfo wrapNewsInfo = mWrapNewsInfos[index];
    if (!wrapNewsInfo.isExplosured) {
        NNFTracker.getInstance().trackNewsExposure(wrapNewsInfo.newsInfo);
        wrapNewsInfo.isExplosured = true;
    }
}
```

---

### 点击事件

* 点击行为场景如下：

	* 列表中的新闻被点击
	* 相关文章被点击
	* 相关图集被点击

发生以上行为时，建议调用该接口进行点击行为上报。

- 定义

```java
/**
 * 新闻点击的事件收集
 *
 * @param newsInfo
 */
public void trackNewsClick(NNFNewsInfo newsInfo);
```

- 示例


```java
NNFTracker.getInstance().trackNewsClick(newsInfo);
```

---

### 浏览事件

当文章或图集浏览结束，视频播放暂停或结束时，建议调用该接口进行浏览行为上报。用户可以自主统计浏览时长和浏览进度一并上报。

- 定义

```
/**
 * 文章或图集浏览结束、视频播放结束的事件收集
 *
 * @param newsInfo
 * @param cost     浏览时长 - 单位毫秒（点击进去，到返回的时间）
 * @param progress 浏览进度 - 例如视频的播放进度，图集已浏览的图片比率等等
 */
public void trackNewsBrowse(NNFNewsInfo newsInfo, long cost, double progress);
```

- 示例

```java
NNFTracker.getInstance().trackNewsBrowse(newsInfo, cost, progress);
```

- 图集浏览行为统计

通过 loadNewsDetails 接口请求图集详情数据，请求成功时，可以认为图集浏览开始，记录开始时间。

```java
NNewsFeedsSDK.getInstance().loadNewsDetails(newsInfo.infoType, newsInfo.infoId, newsInfo.producer, new NNFHttpRequestListener<NNFNewsDetails>() {
    @Override
    public void onHttpSuccessResponse(NNFNewsDetails result) {
    	// 记录用户行为开始时间为网络请求成功后的时间
    	mStartTime = System.currentTimeMillis();
    }
    
    @Override
    public void onHttpErrorResponse(int code, String errorMsg) {
    
    }
});
```

页面退出，例如Activity被销毁时，可以认为图集浏览结束，上报图集浏览结束事件。其中，图集浏览时间 = 当前时间 - 开始时间；浏览进度 =（当前图片序号 + 1）/ 总的图片数。

```java
@Override
public void onDestroy() {
    super.onDestroy();

	// 用户行为上报，表明图集浏览结束
    long cost = System.currentTimeMillis() - mStartTime;
    NNFTracker.getInstance().trackNewsBrowse(newsInfo, cost, (mCurrPos + 1) * 1.0 / mPhotoSets.length);
}
```

- 视频浏览行为统计

App开发人员需要监听视频播放各个过程。开始视频播放时，记录视频开始播放时间。当视频暂停或播放结束时，上报视频浏览事件。其中，播放时长 = 当前时间 - 开始播放时间；播放进度 = 当前播放帧时间 / 视频总时长。示例代码片段如下：

```java
@Override
public void onPlay() {
    mStartPlayTime = System.currentTimeMillis();
    // 上报用户行为：视频开始播放
    NNFTracker.getInstance().trackNewsClick(mNewsInfo);
}

@Override
public void onError() {

}

@Override
public void onAutoComplete() {
    onVideoBrowseEnd();
}

@Override
public void onPause() {
    // 上报用户行为：视频播放结束
    onVideoBrowseEnd();
}

public void onVideoBrowseEnd() {
    // 上报用户行为：视频播放结束
    mProgress = mVideoPlayer.getVideoProgress();
    long cost = System.currentTimeMillis() - mStartPlayTime;
    NNFTracker.getInstance().trackNewsBrowse(mNewsInfo, cost, mProgress);
}
```

获取视频播放进度示例代码如下：

```java
/**
 * 获取视频播放进度，转化为获取播放进度条的进度
 *
 * @return
 */
public double getVideoProgress() {
    int max = bottomProgressBar.getMax();
    return max == 0 ? 0 : bottomProgressBar.getProgress() * 1.0 / max;
}
```

- 文章类新闻浏览事件

文章类新闻一般通过WebView展示，由于在阅读文章期间，WebView可能被遮挡或App退到后台导致WebView不可见，WebView不可见的时间不应该计算到浏览时间内。因此，文章类新闻的浏览时间即转化为WebView可见的累积时间。

当WebView页面加载成功时，可以认为浏览事件开始，记录开始时间，示例代码如下：

```java
/**
 * 浏览时长统计
 */
public void onPageFinished(WebView view, String url) {
    if (null == mEventTimer) {
        mEventTimer = new EventTimer(TimeUnit.MILLISECONDS);
    } else {
        mEventTimer.reset();
    }
}
```

可通过 EventTimer 来保存浏览事件的开始时间、累积时间等：

```java
public static class EventTimer {
    EventTimer(TimeUnit timeUnit) {
        this.occurTime = System.currentTimeMillis();
        this.startTime = System.currentTimeMillis();
        this.timeUnit = timeUnit;
        this.eventAccumulatedDuration = 0;
    }
    ...
    private final TimeUnit timeUnit;
    private long startTime;
    private long eventAccumulatedDuration;
}
```

可以通过重写WebView的onVisibilityChanged方法，监听WebView的可见状态。当WebView可见时，重置开始时间；当WebView不可见时，计算浏览累积时间并保存。累积时间 = 累积时间 +（当前时间 - 开始时间）。示例代码如下：

```java
/**
 * 浏览时长统计
 *
 * @param changedView
 * @param visibility
 */
@Override
protected void onVisibilityChanged(View changedView, int visibility) {
    super.onVisibilityChanged(changedView, visibility);
    if (visibility == View.VISIBLE) { // WebView进入前台
        if (mEventTimer != null) {
            mEventTimer.setStartTime(System.currentTimeMillis());
        }
    } else { // WebView被遮挡或进入后台
        if (mEventTimer != null) {
            long eventAccumulatedDuration = mEventTimer.getEventAccumulatedDuration() + System.currentTimeMillis() - mEventTimer.getStartTime();
            mEventTimer.setEventAccumulatedDuration(eventAccumulatedDuration);
        }
    }
}
```

可通过重写WebView的onScrollChanged方法实时监测滚动条，用于统计文章新闻的浏览进度。由于用户可能看完一篇文章后，来回上下滑动滚动条重看已看过的文章内容，因此，用户退出文章详情页面时，WebView滚动条的位置并不能准确记录用户的浏览进度，我们推荐App开发者记录WebView可见期间WebView滚动条的最大偏移量作为计算用户浏览进度的依据。示例代码如下：

```java
/**
 * 实时监测滚动条
 *
 * @param l
 * @param t
 * @param oldl
 * @param oldt
 */
@Override
protected void onScrollChanged(int l, int t, int oldl, int oldt) {
    super.onScrollChanged(l, t, oldl, oldt);
    mMaxScrollOffset = Math.max(mMaxScrollOffset, t);
}
```

当 WebView 从父控件移除时，可以认为浏览事件结束，上报浏览结束事件。重写WebView的onDetachedFromWindow方法，可以监听WebView移除事件。上报的浏览时长cost为WebView可见的累积时间，浏览进度progress可取值为（垂直滚动条最大偏移量 + WebView高度）/ 垂直滚动条滚动范围。示例代码如下：

```java
/**
 * 浏览时长统计
 */
@Override
protected void onDetachedFromWindow() {
    super.onDetachedFromWindow();
    NNFLogUtil.v(TAG, "onDetachedFromWindow");
    // 发送统计时长和浏览进度等信息
    long cost = null == mEventTimer ? 0 : mEventTimer.duration();
    double progress = 0;
    int vScrollOffset = computeVerticalScrollOffset();
    vScrollOffset = Math.max(mMaxScrollOffset, vScrollOffset);
    int vScrollRange = computeVerticalScrollRange();
    int contentHeight = this.getContentHeight();
    int measuredHeight = this.getMeasuredHeight();
    if (vScrollRange > 0 && contentHeight > 0) {
        progress = (vScrollOffset + measuredHeight) * 1.0 / vScrollRange;
    }

    if (null != mNewsInfo) {
        NNFTracker.getInstance().trackNewsBrowse(mNewsInfo, cost, progress);
    }
    mEventTimer = null;
}
```

### 负反馈事件

从`v1.4.5`开始，SDK返回的`NNFNewsInfo`数据模型新增`feedbacks`字段，用于支持用户负反馈功能，对不感兴趣或推荐不当的新闻进行反馈，从而修正用户模型，提高推荐准确性。

- feedbacks 字段示例

```java
"feedbacks": [
  {
    "name": "不感兴趣",
    "value": "D_0"
  },
  {
    "name": "内容质量差",
    "value": "D_1"
  },
  {
    "name": "看过了",
    "value": "D_2"
  },
  {
    "name": "不想看:逸动城市网",
    "value": "S_逸动城市网"
  }
]
```

根据`feedbacks字段`给出的反馈项，开发人员可参考如下UI示例进行负反馈搜集，并调用`trackNewsFeedback`进行行为上报。

 ![内容反馈](http://nos.netease.com/knowledge/f7be0108-4554-4256-88ce-562070794ca6) 

- 定义

```java
/**
 * 负反馈事件搜集
 */
public void trackNewsFeedback(NNFNewsInfo newsInfo, String[] reasons)
```

其中`reasons`为反馈的原因，对应`feedbacks`字段中的反馈项value组合，例如用户选择了 "不感兴趣"、"内容质量差" 两项原因，则`reasons`取值应为`["D_0", "D_1"]`。

- 示例

```java
// 上报负反馈数据
HashSet<NNFeedbackInfo> selectedBeans = feedbackAdapter.getSelectedBeans();
String[] reasons = new String[selectedBeans.size() + 1];
// 默认传递第0项
reasons[0] = feedbacks[0].value;
Iterator<NNFeedbackInfo> iterator = selectedBeans.iterator();
int i = 1;
while (iterator.hasNext()) {
    if (i < reasons.length)
        reasons[i] = iterator.next().value;
    i++;
}
NNFTracker.getInstance().trackNewsFeedback(mNewsInfo, reasons);
```

# 附录

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
channelOrder | int | 1 | 频道显示的顺序
channelType | int | 1 | （v1.3新增）频道类型，其中4表示自营频道

- 新闻列表：NNFNews

名称 | 类型 | 描述
---|---|---
infos| NNFNewsInfo[]| 普通新闻
banners | NNFNewsInfo[] |轮播图
tops | NNFNewsInfo[] |置顶新闻

- 单个新闻摘要：NNFNewsInfo

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
publishTime | String | 新闻创建时间
summary | String | 新闻简介
thumbnails | NNFThumbnailInfo[] | 新闻缩略图
tripleImgs | NNFThumbnailInfo[] | 兼容安徽头条
imgType | int | 缩略图类型，3：三图模式， 2：大图模式，1：单图模式，0：无图
hasVideo | boolean | 文章是否包含视频，true：有，false：没有
num | int | 图集中图片数量，仅当infoType为图集picset时才有值
videos | NNFVideoCell[] | 视频源信息，仅当infoType为视频video时才有值 
ad | NNFAdCell | 广告信息，当且仅当infoType为ad时infoType
readStatus | int | 0：未读 ； 1： 已读
deliverId | long | 兼容自营频道
feedbacks | NNFeedbackInfo[] | 用户可选择的负反馈内容，最多7项，最少3项，广告没有负反馈，后台不会下发内容

- 单个负反馈项：NNFeedbackInfo

名称 | 类型 | 描述
---|---|---|---
name | String | 客户端展示内容
value | String | 客户端上报内容

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
infoType| String| article/picset | 新闻类型，文章/图集
category | String || 新闻类目
title | String || 新闻标题
publishTime | String || 发布时间
source | String || 新闻来源
sourceLink | String || 原文地址
content | String |  | 新闻正文
imgs | NNFImageInfo[] |  | 图片列表
tag | String |  | 标签
ad | NNFAdCell | | 广告
videos | NNFVideoCell[] | 视频源信息，仅当infoType为视频video时才有值 

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
ip | String | 移动端公网ip
adInfo | NNFAdInfo | 广告数据

- 单条广告数据模型：NNFAdInfo

名称 | 类型 | 描述
---|---|---|---
title | String | 标题，短文字,14字以内
desc | String | 描述，长文字,30字以内
iconUrl | String | 获取Icon图片地址
imgUrl | String | 获取大图地址
isApp | boolean | 返回是否为APP广告
producer | String | 广告生产者（gdt/inmobi）
appStatus | int | 获取应用状态，0：未开始下载；1：已安装；2：需要更新; 4:下载中; 8:下载完成; 16:下载失败
progress | int | 获取APP类广告下载中的下载进度
downloadCount | long | 获取APP类广告的下载数
appScore | int | 获取应用评级
appPrice | Double | 获取APP类应用价格
landingURL | String | 落地页（仅producer为inmobi有值）
trackingMap | Map | 点击和曝光行为上报相关信息（仅producer为inmobi有值）
thumbnailInfos | NNFThumbnailInfo[] | 缩略图数组
imageType | int | 缩略图类型，3：三图模式， 2：大图模式，1：单图模式，0：无图