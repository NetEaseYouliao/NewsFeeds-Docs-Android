# Android UI SDK 接入文档

为便于用户接入我们的 UI SDK，我们提供集成了 UI SDK 的演示Demo，用户可参考Demo源码及文档使用我们的 UI SDK。

[NewsFeeds-Docs-Android](https://github.com/NetEaseYouliao/NewsFeeds-Docs-Android)

## UI SDK 概述

网易有料 NewsFeeds UI SDK 封装了信息流的核心视图控件，方便第三方应用快速的集成并实现内容分发功能。UI SDK兼容Android 14+，UI Demo兼容Android 14+。

NewsFeeds UI SDK提供的功能如下：

- 信息流首页视图
- 文章详情页视图
- 图集浏览页面
- 视频浏览页面
- 文章图片浏览页面
- 负反馈

接入的模式有两种：

- 快速集成信息流

> 信息流的基本功能包含信息流主页、文章类新闻展示页、图集类新闻展示页等，若用户选择快速集成方式，则使用UI SDK提供的所有默认页面。

- 自定义集成信息流

> 用户可以根据UI SDK提供的回调，自定义交互逻辑。例如，点击新闻列表后的目标调转页面、点击相关推荐后的目标调转页面等等。

## SDK类说明

网易有料NNewsFeeds UI SDK主要提供了以下类：

- NNFeedsFragment：信息流主页，提供频道切换和新闻列表的默认实现，采用ViewPager内嵌Fragment完成。

- NNFArticleWebFragment：文章类新闻展示的默认实现，采用Fragment内嵌WebView完成。

- NNFPicSetGalleryFragment：展示图集类新闻

- NNFArticleGalleryFragment：展示文章类新闻正文中的图片集

提供多样化信息流入口：

- NNFSmallEntranceFragment：信息流小入口

![信息流小入口示例](http://nos.netease.com/knowledge/e1ba4abd-b043-4ee9-830b-e8ba05392f2e)

## 开发准备

### 1. Gradle集成

我们推荐通过Gradle集成我们的sdk。由于ui-sdk是在data-sdk的基础上开发的，因此，使用ui-sdk必须同时依赖data-sdk，data-sdk的接入，请参考data-sdk接入文档。

- 第三方依赖说明

我们的data-sdk和ui-sdk内部依赖了一些第三方库，

其中, data-sdk以`provided`的形式依赖了如下第三方库：

```java
provided 'com.alibaba:fastjson:1.2.8'
```

ui-sdk以`provided`的形式依赖了如下第三方库：

```java
// 状态栏
provided "com.readystatesoftware.systembartint:systembartint:1.0.+"
// 图片库
provided 'com.github.bumptech.glide:glide:3.7.0'
// recyclerview列表
provided "com.android.support:recyclerview-v7:25.3.1"
```

若您的App也依赖了这些第三方库，请确保您依赖的第三方库的版本与我们sdk依赖的第三方库版本兼容。

- jcenter远程依赖

第一步，在project module下的build.gradle配置repositories。创建项目时，会自动包含jcenter。若无，请添加。

```java
allprojects {
    repositories {
        jcenter()
    }
}
```

第二步，在app module下的build.gradle中同时引入我们的data-sdk和ui-sdk的依赖，请自行将x.x替换为版本号，目前ui-sdk最新版为1.6.0，data-sdk最新版为1.7.0


```java
// data-sdk依赖库
compile 'com.alibaba:fastjson:1.2.8'
// ui-sdk依赖库
compile "com.readystatesoftware.systembartint:systembartint:1.0.+"
compile 'com.github.bumptech.glide:glide:3.7.0'
compile "com.android.support:recyclerview-v7:25.3.1"

compile 'com.netease.youliao:newsfeeds-data:x.x'
compile 'com.netease.youliao:newsfeeds-ui:x.x'
```

### 2. 初始化

在自定义Application的OnCreate中添加以下代码，初始化我们的ui-sdk。由于您的应用可能不止一个进程，建议只在主进程下初始化我们的ui-sdk。示例代码如下：

```
public class YLApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        Context context = getApplicationContext();
        String processName = getProcessName(context, Process.myPid());
        // 判断进程名，保证只有主进程才初始化网易有料UI SDK
        if (!TextUtils.isEmpty(processName) && processName.equals(this.getPackageName())) {
            /**
             * 初始化SDK：在自定义Application中初始化网易有料UI SDK
             */
            new NNewsFeedsUISDK.Builder()
                    .setAppKey(BuildConfig.APP_KEY)
                    .setAppSecret(BuildConfig.APP_SECRET)
                    .setContext(getApplicationContext())
                    .setMaxCacheNum(60)
                    .setMaxCacheTime(60 * 60 * 1000)
                    .setAutoRefreshInterval(60 * 60 * 1000)
                    .setLogLevel(NNFLogUtil.LOG_VERBOSE)
                    .build();
        }
    }

	/**
	 * 根据进程 ID 获取进程名
	 *
	 * @param pid
	 * @return
	 */
	public static String getProcessName(Context context, int pid) {
	    ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
	    List<ActivityManager.RunningAppProcessInfo> processInfoList = am.getRunningAppProcesses();
	    if (processInfoList == null) {
	        return null;
	    }
	    for (ActivityManager.RunningAppProcessInfo processInfo : processInfoList) {
	        if (processInfo.pid == pid) {
	            return processInfo.processName;
	        }
	    }
	    return null;
	}
}
```

- 初始化接口及参数说明

接口 | 参数 | 类型 | 描述
---|---|---|---
setAppKey | appKey | String | 分配给应用的唯一标识，用户在CMS后台新建应用时生成
setAppSecret | appSecret | String |  分配给应用的唯一秘钥，用户在CMS后台新建应用时生成
setContext | context | Context | 传入app的Context，建议传入ApplicationContext
setMaxCacheNum | maxCacheNum | int | 配置每个频道最大缓存新闻数量，默认60条
setMaxCacheTime | maxCacheTime | long | 配置新闻列表及新闻详情文本最大缓存时长，单位毫秒，默认1天（1 * 24 * 60 * 60 * 1000）
setAutoRefreshInterval | autoRefreshInterval | long | 设置新闻列表自动刷新间隔，单位毫秒，默认1小时（1 * 60 * 60 * 1000）
setLogLevel | logLevel | int | Android Studio等开发工具的 控制台Log等级，指定哪些日志需要输出

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

### 3. 混淆

若您的App开启了混淆，请为我们的SDK添加下述混淆规则


```java
# 网易有料 - 信息流SDK
-keep class com.netease.youliao.newsfeeds.**{*;}
```

由于我们的data-sdk内部依赖了腾讯广点通和个推，因此，混淆时也需要添加如下混淆规则：

```java
# fastjson
-keep class javax.ws.rs.** { *; }
-dontwarn com.alibaba.fastjson.**
-keep class com.alibaba.fastjson.** { *; }
```

## ui-sdk接口说明

### ui-sdk主入口

NNewsFeedsUI 为ui-sdk主入口，提供全局设置及多个Fragment的实例化调用。

#### 设置全局分享点击回调

为实现分享引流拉新，我们的ui-sdk预留了分享按钮，当开发人员设置了分享回调，新闻详情页面、图集详情页面、视频页面都会显示分享按钮，若未设置，则隐藏。

- 定义

```java
public static void setShareCallback(NNFOnShareCallback shareCallback)
```

其中 NNFOnShareCallback 为 interface ，定义如下：

```java
public interface NNFOnShareCallback {
    /**
     * 分享
     *
     * @param shareInfo 分享必要字段集合
     * @param index     0：微信好友 1：朋友圈
     */
    void onWebShareClick(Map<String, String> shareInfo, int index);
}
```

其中，shareInfo以键值对的形式存储了单条新闻的必要描述信息，开发人员可根据这些信息构造分享视图，详细说明如下：

key | value描述
---|---
title | 新闻标题
infoId | 新闻ID
infoType | 新闻类型，article/picset/video/svideo
producer  | 新闻提供者，user表示用户自编辑新闻，recommendation表示来自个性化推荐系统
recId | 单次推荐唯一标识
algInfo | 推荐策略及权重信息
summary | 新闻简介
source | 新闻来源
iconUrl | 新闻内部某张图片

开发人员需要实现 onWebShareClick 方法，给出分享操作的具体实现。

- 示例

```java
// 设置全局分享回调
NNewsFeedsUI.setShareCallback(new NNFOnShareCallback() {
    @Override
    public void onWebShareClick(Map<String, String> shareInfo, int index) {
        ShareUtil.shareImp(getApplicationContext(), api, shareInfo, index);
    }
});
```

微信分享示例：

```java
public class ShareUtil {
    private static final int THUMB_SIZE = 150;

    public static void shareImp(Context context, final IWXAPI api, Map<String, String> shareInfo, final int type) {
        WXWebpageObject webpage = new WXWebpageObject();
        webpage.webpageUrl = buildShareReq(shareInfo);
        Log.d("SHARE", "webpageUrl->" + webpage.webpageUrl);
        final WXMediaMessage msg = new WXMediaMessage(webpage);
        msg.title = shareInfo.get(NNFUIConstants.FIELD_TITLE);
        String summary = shareInfo.get(NNFUIConstants.FIELD_SUMMARY);
        msg.description = TextUtils.isEmpty(summary) ? shareInfo.get(NNFUIConstants.FIELD_SOURCE) : summary;
        String iconUrl = shareInfo.get(NNFUIConstants.FIELD_ICONURL);

        if (!TextUtils.isEmpty(iconUrl)) {
            Glide.with(context).load(iconUrl).asBitmap().into(new SimpleTarget<Bitmap>() {
                @Override
                public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {
                    sendShareReq(api, resource, type, msg);
                }
            });
        } else {
            // 使用默认图标
            Bitmap bmp = BitmapFactory.decodeResource(context.getResources(), R.mipmap.ic_launcher);
            sendShareReq(api, bmp, type, msg);
        }
    }

    private static void sendShareReq(IWXAPI api, Bitmap bmp, int type, WXMediaMessage msg) {
        int targetScene = SendMessageToWX.Req.WXSceneSession;
        switch (type) {
            case 0:
                targetScene = SendMessageToWX.Req.WXSceneSession;
                break;
            case 1:
                targetScene = SendMessageToWX.Req.WXSceneTimeline;
                break;
        }
        Bitmap thumbBmp = Bitmap.createScaledBitmap(bmp, THUMB_SIZE, THUMB_SIZE, true);
        msg.setThumbImage(thumbBmp);

        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.transaction = buildTransaction("webpage");
        req.message = msg;
        req.scene = targetScene;
        api.sendReq(req);
    }

	// h5分享链接构造
    private static String buildShareReq(Map<String, String> shareInfo) {
        String infoId = shareInfo.get(NNFUIConstants.FIELD_INFOID);
        String infoType = shareInfo.get(NNFUIConstants.FIELD_INFOTYPE);
        String producer = shareInfo.get(NNFUIConstants.FIELD_PRODUCER);
        String source = shareInfo.get(NNFUIConstants.FIELD_SOURCE);
        String recId = shareInfo.get(NNFUIConstants.FIELD_RECID);
        String algInfo = shareInfo.get(NNFUIConstants.FIELD_ALGINFO);
        producer = TextUtils.isEmpty(producer) ? "recommendation" : producer;
        String urlFormat = BuildConfig.SHARE_SERVER + "m/#/info?fss=1&ak=%s&sk=%s&id=%s&it=%s&p=%s" +
                "&aou=%s" +
                "&iou=%s" +
                "&st=%s" +
                "&rid=%s" +
                "&info=%s";
        String openUrl = "youliao%3A%2F%2Fyouliao.163yun.com%3FinfoId%3D" + infoId + "%26infoType%3D" + infoType + "%26producer%3D" + producer;
        return String.format(urlFormat,
                BuildConfig.APP_KEY,
                BuildConfig.APP_SECRET,
                infoId,
                infoType,
                producer,
                openUrl,
                openUrl,
                source,
                recId,
                algInfo);
    }

    private static String buildTransaction(final String type) {
        return (type == null) ? String.valueOf(System.currentTimeMillis()) : type + System.currentTimeMillis();
    }
}
```

==注意==：

- 快速集成模式：实现该全局回调，则新闻详情页面、图集详情页面、视频页面都会显示分享按钮；未实现该全局回调，则不会显示分享按钮

- 自定义集成模式：该分享回调和各个页面级分享回调只要实现一个则会显示分享按钮，若两个回调都实现，则点击分享的时候，触发页面级分享回调。

h5分享链接的构造规则请参考网易有料h5文档，也可参考UI SDK演示Demo。此外，我们的ui-sdk仅提供分享的回调，具体的分享行为还需开发人员参考微信官方文档。

---

#### 创建信息流主页NNFeedsFragment实例

```java
/**
 * 信息流主页，提供频道切换和新闻列表的默认实现
 *
 * @param onFeedsCallback 回调
 * @param extraData       用户自定义数据
 * @return
 */
public static NNFeedsFragment createFeedsFragment(OnFeedsCallback onFeedsCallback, Object extraData)
```
其中 extraData 为用户自定义数据，该参数会在onFeedsCallback回调中回传。

==注意==：提供两种集成模式

- 第一种，未自定义回调，则所有页面已经整合到一起，及文章详情页面、图集页面、新闻正文图片浏览页面都已封装

```java
/**
 * 接入信息流UI SDK，快速集成信息流主页 NNFeedsFragment
 */
private void initFeedsByOneStep() {
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    // 回调设置为null，表示使用SDK内部回调
    mFeedsFragment = NNewsFeedsUI.createFeedsFragment(null, null);
    ft.replace(R.id.fragment_container, mFeedsFragment);
    ft.commitAllowingStateLoss();
}
```

- 第二种，实现自定义回调，那么用户在点击新闻列表的时候跳转到用户回调，由用户自定义页面跳转

```java
/**
 * 第一步：接入信息流UI SDK，自定义集成信息流主页 NNFeedsFragment
 */
private void initFeedsStepByStep() {
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    mFeedsFragment = NNewsFeedsUI.createFeedsFragment(new FeedsCallbackSample(), null);
    ft.replace(R.id.fragment_container, mFeedsFragment);
    ft.commitAllowingStateLoss();
}

/**
 * 第二步：为信息流主页 NNFeedsFragment 设置点击事件回调
 */
private class FeedsCallbackSample extends NNFOnFeedsCallback {
    @Override
    public void onNewsClick(Context context, NNFNewsInfo newsInfo, Object extraData) {
        if (null != newsInfo && null != newsInfo.infoType) {
        // 普通新闻被点击
        switch (newsInfo.infoType) {
            case NNFUIConstants.INFO_TYPE_ARTICLE:
                /**
                 * 第三步：自定义文章类新闻展示页面
                 */
                SampleArticleActivity.start(context, newsInfo);
                break;
            case NNFUIConstants.INFO_TYPE_PICSET:
                /**
                 * 第四步：自定义图集类新闻展示页面
                 */
                SamplePicSetGalleryActivity.start(context, newsInfo);
                break;
            case NNFUIConstants.INFO_TYPE_VIDEO:
                /**
                 * 第五步：自定义视频类新闻展示页面
                 */
                DefaultMoreVideosActivity.start(context, newsInfo);
                break;
            case NNFUIConstants.INFO_TYPE_SVIDEO:
                /**
                 * 第六步：自定义短视频新闻展示页面
                 */
                DefaultMoreVideosActivity.start(context, newsInfo);
                break;

			}
    }
}
```
---

#### 信息流主页回调接口说明

NNFOnFeedsCallback为回调抽象类，提供信息流主页交互事件回调，目前支持的交互事件回调有：新闻被点击。

- 新闻被点击

```java
/**
 * 新闻被点击
 *
 * @param context
 * @param newsInfo  当前新闻对应新闻列表中的数据源
 * @param extraData 用户自定义数据
 */
public abstract void onNewsClick(Context context, NNFNewsInfo newsInfo, Object extraData);
```
 
 注意这里的extraData是初始化NNFeedsFragment实例时，传入的用户自定义数据。
 
 若用户未实现该回调，则SDK会根据新闻的infoType自动跳转到对应的默认展示页面。
 
 若用户实现该回调，则跳转到用户回调。
 
---

#### 创建文章类新闻展示页NNFArticleWebFragment实例

```java
/**
 * 文章类新闻展示的默认实现
 *
 * @param newsInfo          当前新闻对应新闻列表中的数据源
 * @param onArticleCallback 回调
 * @param extraData         用户自定义数据
 * @return
 */
public static NNFArticleWebFragment createArticleFragment(NNFNewsInfo newsInfo, NNFOnArticleCallback onArticleCallback, Object extraData) 

/**
 * 文章类新闻展示的默认实现
 *
 * @param newsInfo          当前新闻对应新闻列表中的数据源
 * @param onArticleCallback 回调
 * @param onShareCallback   分享回调
 * @param extraData         自定义参数
 * @return
 */
public static NNFArticleWebFragment createArticleFragment(NNFNewsInfo newsInfo, NNFOnArticleCallback onArticleCallback, NNFOnShareCallback onShareCallback, Object extraData)
```

用户选择自己创建新闻详情页面时，可以通过该接口创建新闻详情视图实例，其中newsInfo为当前新闻对应新闻列表中的数据源，onArticleCallback为自定义回调，extraData 为用户自定义数据，该参数会在onArticleCallback回调中回传。

NNFOnShareCallback为页面级分享回调，该页面级分享回调和全局分享回调只要实现一个则会显示分享按钮，若两个回调都实现，则点击分享的时候，触发页面级分享回调。


==注意==：Activity配置

由于文章类新闻展示页采用内嵌WebView实现，为了使WebView中的编辑框在编辑时能够弹出软键盘输入，请按照如下示例在AndroidManifest.xml中配置NNFArticleWebFragment依附的Activity的windowSoftInputMode属性：

```java
<activity
    android:name=".SampleArticleActivity"
    android:configChanges="orientation|screenSize|keyboardHidden"
    android:screenOrientation="portrait"
    // 设置后，该Activity总是调整屏幕的大小以便留出软键盘的空间
    android:windowSoftInputMode="adjustResize">
</activity>
```

==注意==：提供两种模式

- 快速集成，SDK提供一套界面交互事件的默认实现

```java
/**
 * 第一步：接入信息流UI SDK，快速集成文章类展示页 NNFArticleWebFragment
 */
private void initArticleByOneStep() {
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    NNFArticleWebFragment articleWebFragment = NNewsFeedsUI.createArticleFragment(mNewsInfo, null, null);
    ft.replace(R.id.fragment_container, articleWebFragment);
    ft.commit();
}
```

开发人员在创建`NNFArticleWebFragment`实例时可设置页面级分享回调，示例代码如下：

```java
NNFArticleWebFragment articleWebFragment = NNewsFeedsUI.createArticleFragment(mNewsInfo, null, new NNFOnShareCallback() {
    @Override
    public void onWebShareClick(Map<String, String> shareInfo, int index) {
        ShareUtil.shareImp(SampleArticleActivity.this, api, shareInfo, index);
    }
}, null);
```

- 自定义集成，用户指定界面交互回调逻辑。

```java
/**
 * 第一步：接入信息流UI SDK，自定义集成文章类展示页 NNFArticleWebFragment
 */
private void initArticleStepByStep() {
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    NNFArticleWebFragment articleWebFragment = NNewsFeedsUI.createArticleFragment(mNewsInfo, new NNFOnArticleCallback() {
        /**
         * 第二步：为文章类展示页 NNFArticleWebFragment 设置点击事件回调；
         */

        @Override
        public void onRelatedNewsClick(Context context, NNFNewsInfo newsInfo, Object extraData) {
            /**
             * 第三步：自定义相关推荐新闻展示页面
             */
            SampleArticleActivity.start(context, newsInfo);
        }

        @Override
        public void onWebImageClick(Context context, String infoId, int index, String source, NNFImageInfo[] imageInfos, Object extraData) {
            /**
             * 第四步：点击文章正文内的图片后的响应事件
             */
            SampleArticleGalleryActivity.start(context, infoId, index, imageInfos);
        }

		@Override
		public void onArticleLoaded(NNFNewsDetails details, Object extraData) {
	    	/**
	         * 第五步：显示分享图标等
	         */
	        initSharePopView(details);
	        showShare();
		}

        @Override
        public void onIssueReporting(String issueDescription, Object extraData) {
            /**
             * 第六步：点击文章底部的举报按钮时，弹出举报选择框，此时，WebView标题发生变化，将当前Activity Title展示为WebView标题
             */
            if (null != issueDescription) {
                mTextView.setText(issueDescription);
            }
        }

        @Override
        public void onIssueReportFinished(Object extraData) {
            /**
             * 第七步：举报完成后，展示原来的标题
             */
            if (null != mNewsInfo) {
                mTextView.setText(mNewsInfo.source);
            }
        }
    }, null);

    ft.replace(R.id.fragment_container, articleWebFragment);
    ft.commit();
}
```
---

#### 文章类新闻展示页回调接口说明

NNFOnArticleCallback为回调抽象类，提供文章类新闻展示页交互事件回调，目前支持的交互事件回调有：点击正文中的相关推荐、点击正文中的图片、文章类新闻详情加载成功、文章详情页面跳转到报错页面的回调、报错完成返回到文章详情页面的回调。

- 点击正文中的相关推荐

```java
/**
 * 点击正文中的相关推荐
 *
 * @param context
 * @param newsInfo
 * @param extraData 用户自定义数据
 */
public abstract void onRelatedNewsClick(Context context, NNFNewsInfo newsInfo, Object extraData);
```
文章类新闻详情页面相关推荐新闻点击时的回调

---

- 点击正文中的图片

```java
/**
 * 点击正文中的图片
 *
 * @param context
 * @param infoId     所属新闻ID
 * @param index      当前图片是正文第几张图片
 * @param source     图片网络链接
 * @param imageInfos 正文图片数组
 * @param extraData  用户自定义数据
 */
public abstract void onWebImageClick(Context context, String infoId, int index, String source, NNFImageInfo[] imageInfos, Object extraData);
```
新闻正文中图片点击的回调，其中，imageInfos为图片集合，index为当前点击图片下标

---

- 文章类新闻详情加载成功

```java
/**
 * 文章类新闻加载成功
 *
 * @param details   当前新闻详情数据源
 * @param extraData 自定义参数
 */
public void onArticleLoaded(NNFNewsDetails details, Object extraData)
```
当文章类新闻加载成功时触发

---

- 文章详情页面跳转到报错页面的回调

```java
/**
 * 举报中
 *
 * @param issueDescription 举报描述
 * @param extraData        用户自定义数据
 */
public void onIssueReporting(String issueDescription, Object extraData)
```
点击文章底部的举报按钮时，弹出举报选择框，触发该回调。其中issueDescription为报错页面的相关描述，方便更改页面title之类的参数

---

- 报错完成返回到文章详情页面的回调

```java
/**
 * 举报完成
 *
 * @param extraData 用户自定义数据
 */
public void onIssueReportFinished(Object extraData) 
```

报错操作结束后触发该回调。

---

#### 创建图集类新闻展示页NNFPicSetGalleryFragment实例

```java
/**
 * 展示图集类新闻
 *
 * @param newsInfo                当前新闻对应新闻列表中的数据源
 * @param onPicSetGalleryCallback 回调
 * @param extraData               用户自定义数据
 * @return
 */
public static NNFPicSetGalleryFragment createPicSetGalleryFragment(NNFNewsInfo newsInfo, NNFOnPicSetGalleryCallback onPicSetGalleryCallback, Object extraData)

/**
 * 展示图集类新闻
 *
 * @param newsInfo                当前新闻对应新闻列表中的数据源
 * @param onPicSetGalleryCallback 回调
 * @param onShareCallback         分享回调
 * @param extraData               自定义参数
 * @return
 */
public static NNFPicSetGalleryFragment createPicSetGalleryFragment(NNFNewsInfo newsInfo, NNFOnPicSetGalleryCallback onPicSetGalleryCallback, NNFOnShareCallback onShareCallback, Object extraData)
```
可以通过该接口返回图集页面的实例，其中newsInfo为当前新闻对应新闻列表中的数据源，onPicSetGalleryCallback为自定义回调，extraData 为用户自定义数据，该参数会在onPicSetGalleryCallback回调中回传。

NNFOnShareCallback为页面级分享回调，该页面级分享回调和全局分享回调只要实现一个则会显示分享按钮，若两个回调都实现，则点击分享的时候，触发页面级分享回调。

==注意==：提供两种模式

- 快速集成

```java
/**
 * 第一步：快速集成图集类展示页 NNFPicSetGalleryFragment，展示图集类新闻
 */
public void initGalleryByOneStep(NNFNewsInfo newsInfo) {
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    NNFPicSetGalleryFragment picSetGalleryFragment = NNewsFeedsUI.createPicSetGalleryFragment(newsInfo, null, null);
    ft.replace(R.id.fragment_container, picSetGalleryFragment);
    ft.commit();
}
```

开发人员在创建`NNFPicSetGalleryFragment`实例时可设置页面级分享回调，示例代码如下：

```java
mArticleWebFragment = NNewsFeedsUI.createArticleFragment(mNewsInfo, null, new NNFOnShareCallback() {
    @Override
    public void onWebShareClick(Map<String, String> shareInfo, int index) {
        ShareUtil.shareImp(SampleArticleActivity.this, api, shareInfo, index);
    }
}, null);
```

- 自定义集成

```java
/**
 * 第一步：自定义集成图集类展示页 NNFPicSetGalleryFragment，展示图集类新闻
 */
public void initGalleryStepByStep(NNFNewsInfo newsInfo) {
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();

    /**
     * 第二步：为图集展示页 NNFPicSetGalleryFragment 设置点击事件回调；
     */
    NNFOnPicSetGalleryCallback onPicSetGalleryCallback = new NNFOnPicSetGalleryCallback() {

        @Override
        public void onBackClick(Context context) {
            /**
             * 第三步：设置图集展示页左上角返回按钮点击后的行为
             */
            SamplePicSetGalleryActivity.this.finish();
        }
        
        @Override
        public void onPicSetClick(Context context, NNFNewsInfo newsInfo) {
            super.onPicSetClick(context, newsInfo);
            /**
             * 第四步：设置相关图集被点击后的行为
             */
            SamplePicSetGalleryActivity.start(context, newsInfo);
            // 避免OOM，展示相关图集时，销毁上一图集
            if (context instanceof SamplePicSetGalleryActivity) {
                SamplePicSetGalleryActivity activity = (SamplePicSetGalleryActivity) context;
                activity.finish();
            }
        }
    };

    NNFPicSetGalleryFragment picSetGalleryFragment = NNewsFeedsUI.createPicSetGalleryFragment(newsInfo, onPicSetGalleryCallback, null);

    ft.replace(R.id.fragment_container, picSetGalleryFragment);
    ft.commit();
}
```

---

#### 图集类新闻展示页回调接口说明

NNFOnPicSetGalleryCallback为回调抽象类，提供图集类新闻展示页交互事件回调，目前支持的交互事件回调有：图集加载成功时的回调、左上角返回按钮的点击响应。

- 图集加载成功时的回调

```java
/**
 * 图集类新闻加载成功
 *
 * @param details   当前新闻详情的数据源
 * @param extraData 自定义参数
 */
public void onPicSetLoaded(NNFNewsDetails details, Object extraData)
```

当图集新闻加载成功时触发

---

- 左上角返回按钮的点击响应

```java
/**
 * 左上角返回按钮的点击响应
 *
 * @param context 返回按钮上下文
 */
public abstract void onBackClick(Context context);
```

当图集类新闻展示页的左上角返回按钮被点击时触发

---

- 相关图集被点击

```java
/**
 * 相关图集被点击
 *
 * @param context
 * @param newsInfo 图集数据摘要
 */
public void onPicSetClick(Context context, NNFNewsInfo newsInfo)
```

图集相关推荐在图集新闻页的最后一页，当该页上的相关新闻被点击时触发。

#### 创建文章类新闻正文图片集展示页NNFArticleGalleryFragment实例

```java
/**
 * 展示文章类新闻正文中的图片集
 *
 * @param infoId                   所属新闻ID
 * @param startIndex               当前图片是正文第几张图片
 * @param imageInfos               正文图片数组
 * @param onArticleGalleryCallback 回调
 * @param extraData                用户自定义数据
 * @return
 */
public static NNFArticleGalleryFragment createArticleGalleryFragment(String infoId, int startIndex, NNFImageInfo[] imageInfos, NNFOnArticleGalleryCallback onArticleGalleryCallback, Object extraData)
```
可以通过该接口返回新闻详情图片浏览页面的实例，其中infoId为当前新闻的ID，startIndex为当前的图片的索引，imageInfos为新闻详情的图片数组，onArticleGalleryCallback为自定义回调，extraData 为用户自定义数据，该参数会在onArticleGalleryCallback回调中回传。

---

==注意==：提供两种模式

- 快速集成

```java
private void initGalleryByOneStep(int startIndex, NNFImageInfo[] imageInfos, String infoId) {
    /**
     * 第一步：实例化 NNFArticleGalleryFragment，展示文章类新闻正文中的图片集
     */
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    NNFArticleGalleryFragment articleGalleryFragment = NNewsFeedsUI.createArticleGalleryFragment(infoId, startIndex, imageInfos, null, null);
    ft.replace(R.id.fragment_container, articleGalleryFragment);
    ft.commit();
}
```

- 自定义集成

```java
private void initGalleryStepByStep(int startIndex, NNFImageInfo[] imageInfos, String infoId) {
    /**
     * 第一步：实例化 NNFArticleGalleryFragment，展示文章类新闻正文中的图片集
     */
    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();

    /**
     * 第二步：NNFArticleGalleryFragment 设置点击事件回调；
     */
    NNFOnArticleGalleryCallback onArticleGalleryCallback = new NNFOnArticleGalleryCallback() {
        @Override
        public void onPicClick(Context context, NNFImageInfo imageInfo, Object extraData) {
            /**
             * 第三步：设置图片点击行为
             */
            SampleArticleGalleryActivity.this.finish();
        }
    };
    NNFArticleGalleryFragment articleGalleryFragment = NNewsFeedsUI.createArticleGalleryFragment(infoId, startIndex, imageInfos,
            onArticleGalleryCallback, null);
    ft.replace(R.id.fragment_container, articleGalleryFragment);
    ft.commit();
}
```

#### 文章类新闻正文图片集展示页回调接口说明

NNFOnArticleGalleryCallback为回调抽象类，提供文章类新闻正文图片集展示页交互事件回调，目前支持的交互事件回调有：图片被点击。

- 图片被点击

```java
/**
 * 图片被点击
 *
 * @param context   图片上下文
 * @param imageInfo 图片数据源
 * @param extraData 初始化NNFArticleGalleryFragment时传入的用户自定义数据
 */
public abstract void onPicClick(Context context, NNFImageInfo imageInfo, Object extraData);
```

当图片集中的某张图片被点击时，触发该回调

---

### NNFeedsFragment类接口说明

#### 返回新闻列表顶部

```java
/**
 * 点击导航栏返回到顶部，模仿ios点击状态栏返回到顶部的功能
 */
public void scrollToTop()
```
调用该接口可返回当前列表顶部。可用于模仿ios点击状态栏返回到顶部的功能。

---

#### 强制刷新当前列表

```java
/**
 * 为当前频道执行下拉刷新
 */
public void performRefresh()
```
可调用该接口实现强制下拉刷新当前列表的功能。

---

#### 定位到指定频道

```java
/**
 * 允许用户直接定位到某一个频道，请传入channelTag
 *
 * @param channelTag
 */
public void setSelectedChannel(String channelTag)
```
可调用该接口定位到指定频道。

---

#### 列表自定义数据展现

当发起新闻列表请求时（例如下拉刷新或上拉加载更多），app开发人员可能希望往信息流新闻列表中插入自己的数据，为实现该扩展功能，app开发人员需要为`NNFeedsFragment`实例设置监听回调`NNFOnFeedsExtendCallback`

```java
/**
 * 设置自定义扩展回调
 *
 * @param callback
 */
public void setOnFeedsExtendCallback(NNFOnFeedsExtendCallback callback)
```

其中`NNFOnFeedsExtendCallback`定义如下：

```java
public abstract class NNFOnFeedsExtendCallback {
    public abstract void onStart(NNFChannelInfo channelInfo, int loadType, Object extraData);

    public abstract void onFinish(NNFChannelInfo channelInfo, int loadType, Object extraData);
}
```

当发起新闻列表请求时（例如下拉刷新），`NNFOnFeedsExtendCallback`的`onStart`方法被触发，`channelInfo`为当前发起新闻列表请求的频道；`loadType`为0时，表示下拉刷新，为1时，表示上拉加载更多，`extraData`为用户初始化`NNFeedsFragment`实例时传入的自定义数据。

当网络请求成功时，SDK内部并没有结束此次刷新状态（比如列表头部一直转圈中），而是触发`NNFOnFeedsExtendCallback`的`onFinish`方法并处于等待中，直到app开发人员调用`insertUserData`方法，传入要扩展的的新闻数据才会结束此次刷新操作。

```java
public void insertUserData(NNFChannelInfo channelInfo, int loadType, int[] indexArr, NNFExtendNewsInfo[] infos)
```

`indexArr` 表示待插入的位置列表； `infos` 表示待插入的新闻摘要列表。为区别于SDK内部的新闻摘要`NNFNewsInfo`，约定app开发人员传入的新闻摘要只能是`NNFNewsInfo`的派生类`NNFExtendNewsInfo`。app开发人员只需按照`NNFNewsInfo`的字段格式组织新闻即可。

当用户插入的新闻摘要被点击时，SDK默认的点击回调不会处理，app开发人员需要自行实现点击事件，因此，设置了`NNFOnFeedsExtendCallback`回调，就必须自定义`NNFOnFeedsCallback`。

- 示例代码

```java
private void initFeedsStepByStep() {
    HashMap<String, CustomOption> map = new HashMap<>();

    initCustomStyle(map);

    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    mFeedsFragment = NNewsFeedsUI.createFeedsFragment(new FeedsCallbackSample(), null, null);
    mFeedsFragment.setOnFeedsExtendCallback(new SampleExtendCallback(mFeedsFragment));
    ft.replace(R.id.fragment_container, mFeedsFragment);
    ft.commitAllowingStateLoss();
}

private class SampleExtendCallback extends NNFOnFeedsExtendCallback {

    private WeakReference<NNFeedsFragment> mWeakReference;
    HashMap<Integer, NNFExtendNewsInfo[]> mHashMap;

    public SampleExtendCallback(NNFeedsFragment feedsFragment) {
        mWeakReference = new WeakReference<NNFeedsFragment>(feedsFragment);
        mHashMap = new HashMap<>();
    }

    @Override
    public void onStart(NNFChannelInfo channelInfo, int loadType, Object extraData) {
        Log.d("数据扩展", "onStart@channel = " + channelInfo.channelName + ", @loadType = " + loadType);

        // 读取缓存或发起网络请求获取待插入的新闻数据
        String json = "{\n" +
                "\t\t\t\"producer\": \"recommendation\",\n" +
                "\t\t\t\"recId\": \"736633ceedf146e78a5086acaa821d2b_930340049\",\n" +
                "\t\t\t\"algInfo\": \"|HOT_GBREAK|g_break-36\",\n" +
                "\t\t\t\"infoId\": \"DBNEGGRU00097U7S\",\n" +
                "\t\t\t\"infoType\": \"ad\",\n" +
                "\t\t\t\"title\": \"iOS 11沦陷，美执法部门已能破解几乎所有iPhone\",\n" +
                "\t\t\t\"summary\": \"2月28日消息，据《福布斯》杂志报道，美国的执法部门最近取得了一项破解iPhone手机的重大技术突破，然而对全球众多苹果手机的消费者来说，这或许会对他们造成新的隐私问题。\",\n" +
                "\t\t\t\"source\": \"网易科技报道\",\n" +
                "\t\t\t\"imgType\": 2,\n" +
                "\t\t\t\"thumbnails\": [{\n" +
                "\t\t\t\t\"width\": 300,\n" +
                "\t\t\t\t\"height\": 200,\n" +
                "\t\t\t\t\"url\": \"http://irec.nosdn.127.net/icovr-20180228-2229684e11bf912a0e1aeefed2ad5f4e.jpg\",\n" +
                "\t\t\t\t\"note\": null\n" +
                "\t\t\t}],\n" +
                "\t\t\t\"deliverId\": null,\n" +
                "\t\t\t\"publishTime\": \"2018-02-28 10:42:43\",\n" +
                "\t\t\t\"feedbacks\": [{\n" +
                "\t\t\t\t\"name\": \"不感兴趣\",\n" +
                "\t\t\t\t\"value\": \"D_0\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"内容质量差\",\n" +
                "\t\t\t\t\"value\": \"D_1\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"看过了\",\n" +
                "\t\t\t\t\"value\": \"D_2\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"不想看:网易科技报道\",\n" +
                "\t\t\t\t\"value\": \"S_网易科技报道\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"不想看:手机\",\n" +
                "\t\t\t\t\"value\": \"C_36\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"不想看:执法部门\",\n" +
                "\t\t\t\t\"value\": \"P_C手机_I执法部门\"\n" +
                "\t\t\t}, {\n" +
                "\t\t\t\t\"name\": \"不想看:ios\",\n" +
                "\t\t\t\t\"value\": \"P_C手机_Iios\"\n" +
                "\t\t\t}]\n" +
                "\t\t}";

        NNFExtendNewsInfo newsInfo = NNFJsonUtils.fromJson(json, NNFExtendNewsInfo.class);
        NNFExtendNewsInfo[] infos = new NNFExtendNewsInfo[]{newsInfo};

        mHashMap.put(channelInfo.hashCode(), infos);
    }

    @Override
    public void onFinish(NNFChannelInfo channelInfo, int loadType, Object extraData) {
        Log.d("数据扩展", "onFinish@channel = " + channelInfo.channelName + ", @loadType = " + loadType);
        // 该方法被触发，表示SDK内部完成网络请求；等待数据插入
        int[] indexArr = new int[]{3};
        NNFExtendNewsInfo[] infos = mHashMap.get(channelInfo.hashCode());

        // 通知SDK，插入结束
        mFeedsFragment.insertUserData(channelInfo, loadType, indexArr, infos);
    }
}
```

例如上例中插入了一条infoType为ad的新闻，则app开发人员需要自定义点击回调，示例如下：

```java
private class FeedsCallbackSample extends NNFOnFeedsCallback {
    @Override
    public void onNewsClick(Context context, NNFNewsInfo newsInfo, Object extraData) {
        if (null == newsInfo || null == newsInfo.infoType) return;
        if (newsInfo instanceof NNFExtendNewsInfo) {
            // 用户自定义新闻被点击
            switch (newsInfo.infoType) {
                case NNFUIConstants.INFO_TYPE_AD:
                    Intent intent = new Intent();
                    intent.setAction("android.intent.action.VIEW");
                    Uri content_url = Uri.parse("https://www.baidu.com/");
                    intent.setData(content_url);
                    if (!(context instanceof Activity)) {
                        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                    }
                    context.startActivity(Intent.createChooser(intent, "请选择浏览器"));
                    break;
            }
        } else {
            // 普通新闻被点击
            switch (newsInfo.infoType) {
                case NNFUIConstants.INFO_TYPE_ARTICLE:
                    /**
                     * 第三步：自定义文章类新闻展示页面
                     */
                    SampleArticleActivity.start(context, newsInfo);
                    break;
                case NNFUIConstants.INFO_TYPE_PICSET:
                    /**
                     * 第四步：自定义图集类新闻展示页面
                     */
                    SamplePicSetGalleryActivity.start(context, newsInfo);
                    break;
                case NNFUIConstants.INFO_TYPE_VIDEO:
                    /**
                     * 第五步：自定义视频类新闻展示页面
                     */
                    DefaultMoreVideosActivity.start(context, newsInfo);
                    break;
                case NNFUIConstants.INFO_TYPE_SVIDEO:
                    /**
                     * 第六步：自定义短视频新闻展示页面
                     */
                    DefaultMoreVideosActivity.start(context, newsInfo);
                    break;

            }
        }
    }
}
```

---

### 信息流入口

#### 创建信息流小入口实例

```
FragmentManager fm = getSupportFragmentManager();
FragmentTransaction ft = fm.beginTransaction();
NNFChannelContentFragment fragment = NNewsFeedsUI.createChannelContentViewFragment(null,null, NNFEntranceFragmentType.NNFChannelFragmentSmallEntrance, null, "1170");
ft.replace(R.id.fragment_container, fragment);
ft.commit();
```

#### 信息流入口创建接口说明


```
/**
 * 单个频道信息流入口
 * @param channelContentCallback 信息流内文章点击回调
 * @param map 自定义数据
 * @param type 单个频道信息流入口类型
 * @param extraData 自定义参数
 * @return
 */
 public static NNFChannelContentFragment createChannelContentViewFragment(NNFChannelContentCallback channelContentCallback, Map<String, Map<String, Object >> map, int type, Object extraData, String channelID) {
```