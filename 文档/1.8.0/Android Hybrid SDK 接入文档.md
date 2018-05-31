### SDK概述

网易有料 NewsFeeds Hybrid SDK 是网易有料出品的一种轻量化解决方案。封装了信息流的核心视图控件，方便第三方应用快速的集成并实现内容分发功能。SDK兼容Android 14+。SDK通过Web方式接入网易有料内容，并使用jsbridge使native能够完成如微信分享等功能。

[![Release](https://img.shields.io/badge/release-v1.2.0-green.svg)](CHANGELOG.md)

### SDK开发准备

#### 1.SDK接入

- jcenter远程依赖

第一步，在project module下的build.gradle配置repositories。创建项目时，会自动包含jcenter。若无，请添加。

```java
allprojects {
    repositories {
        jcenter()
    }
}
```

第二步，在app module下的build.gradle中引入依赖。


```java
compile 'com.netease.youliao:newsfeeds-hybrid:x.x'
```


#### 2. 混淆

若您的App开启了混淆，请为我们的SDK添加下述混淆规则


```java
# 网易有料 - 信息流SDK
-keep class com.netease.newsfeedshybrid.feeds.**{*;}
```

#### 3.SDK初始化使用

- 接入代码

```
NNFHybridSDK.init(new UserCustomCallBack(iwxapi), BuildConfig.appKey, BuildConfig.secretKey, null);
HybridFeedsFragment hybridfeedsFragment = NNFHybridSDK.createHybridFeedsFragment();
getSupportFragmentManager().beginTransaction()
                .add(R.id.content, hybridfeedsFragment, "FeedsFragment")
                .commit();
```

您需要在SDK的初始化接口，继承CustomCallback类，处理相应的web回调。同时需要传入APPKey，SecretKey，UniqueID，APPKey，SecretKey必须传入，UniqueId如果不传入，则会默认生成。通过`createHybridFeedsFragment`接口，可以创建出Hybrid Fragment，然后可以将其嵌入到您的页面之中，用来使用。

#### 4. SDK接口参数说明

继承CustomCallback，完成web和native的通信。当前HybridSDK支持三种事件，分别是init， openLink， share  , nagative_feed_back。执行后，返回true表示消耗掉该事件，返回为false表示不消耗该事件。当有内容需要回调的时候需要手动调用<b>submit(webView,result,INIT)</b>方法。


事件处理演示代码

```
@Override
public boolean action(HybridView webView, String action, String data) {
    return process(webView, action, data);
}

private boolean process(HybridView webView, String action, String data) {
            case INIT:
                return init(webView);
            case SHARE:
                return share(webView.getContext(), data);
            case OPEN_AD:
                return openAD(webView.getContext(), data);
            case OPEN_LINK:
                return openLink(webView.getContext(), data);
            case NEGATIVE_FEED_BACK:
                return negative(webView,data);
            default:
                return false;
}

```


- init

init是web页面加载时发起的事件，需要用户返回appKey， secretKey，supportActions，supportSharePlatforms。

其中supportActions返回当前native支持的事件名，因为web端的更新会超前于native，所以web会隐藏native不支持的action。

supportSharePlatforms返回当前native支持的分享平台，其中0表示微信分享用户，1表示微信朋友圈。native需要自行判断微信的安装情况来返回参数。暂不支持其它平台。

```
private boolean init(HybridView webView) {
    try {
        JSONObject supportData = new JSONObject();
        JSONObject result = new JSONObject();

        JSONArray supportPlatforms = new JSONArray();
        supportPlatforms.put(0);
        supportPlatforms.put(1);
        supportData.put("supportSharePlatforms", supportPlatforms);

        JSONArray supportActions = new JSONArray();
        supportActions.put("share");
        supportData.put("supportActions", supportActions);
        supportData.put("appKey", BuildConfig.appKey);
        supportData.put("secretKey", BuildConfig.secretKey);
        supportData.put("closePullRefresh", true);
        supportData.put("customNegativeFeedBack", true);

        supportData.put("closeShare", true);

        result.put("code", 0);
        result.put("message", "success");
        result.put("data", supportData);
        submit(webView,result,INIT);
        return true;
    } catch (JSONException e) {
        Log.e("eTest", e.getMessage());
        e.printStackTrace();
    }
    return false;
}
```

- openLink

当打开新的页面时，会发起openLink事件，同时携带url参数，该事件不关心返回结果。对于打开新的页面，这里页面分为两种类型，一种是广告，一种是新闻详情，这里在返回的字段中，通过`type`字段进行标示，分别为`ad`和`content`。

```
private boolean openLink(Context context, String data) {
    try {
        JSONObject jsonObject = new JSONObject(data);
        HybridNewsInfo newsInfo = parseJson(context, jsonObject);
        DetailActivity.start(context, newsInfo);
    } catch (JSONException e) {

    }
    return true;
}

private HybridNewsInfo parseJson(Context context, JSONObject data) {
    HybridNewsInfo hybridNewsInfo = new HybridNewsInfo();
    try {
        JSONObject jsonObject = data.getJSONObject("search");
        if (jsonObject.has("url")) {
            hybridNewsInfo.url = jsonObject.getString("url");
        } else {
            hybridNewsInfo.ak = BuildConfig.appKey;
            hybridNewsInfo.dt = jsonObject.getString("dt");
            hybridNewsInfo.id = jsonObject.getString("id");
            hybridNewsInfo.info = jsonObject.getString("info");
            hybridNewsInfo.it = jsonObject.getString("it");
            hybridNewsInfo.p = jsonObject.getString("p");
            hybridNewsInfo.rid = jsonObject.getString("rid");
            hybridNewsInfo.sk = BuildConfig.secretKey;
            hybridNewsInfo.st = jsonObject.getString("st");
            if (NNFHybridSDK.getUnidId() == null) {
                hybridNewsInfo.unid = DeviceInfo.getUniqueID(context);
            } else {
                hybridNewsInfo.unid = NNFHybridSDK.getUnidId();
            }
            if (jsonObject.has("ctag")) {
                hybridNewsInfo.ctag = jsonObject.getString("ctag");
            }
        }

    } catch (JSONException e) {
        e.printStackTrace();
    }
    return hybridNewsInfo;
}
```

- share

当用户点击分享时，会发起该事件。事件参数是platform，link，imgUrl，title，desc。platform和supportSharePlatforms相同。该事件不关心返回结果。


```
private boolean share(Context context, String data) {
    try {
        JSONObject jsonObject = new JSONObject(data);
        String title = jsonObject.getString("title");
        String desc = jsonObject.getString("desc");
        String link = jsonObject.getString("link");
        String imgUrl = jsonObject.getString("imgUrl");
        int platform = jsonObject.getInt("platform");

        Map<String, String> shareInfo = new HashMap<>();
        shareInfo.put(Constants.FIELD_TITLE, title);
        shareInfo.put(Constants.FIELD_SUMMARY, desc);
        shareInfo.put(Constants.FIELD_LINK, link);
        shareInfo.put(Constants.FIELD_ICONURL, imgUrl);
        ShareUtil.shareImp(context, iwxapi, shareInfo, platform);
        return true;
    } catch (JSONException e) {
        return false;
    }
}


```
- nagative_feed_back

当在初始化时候设置了supportData.put("customNegativeFeedBack", true)时，点击feed流中item的关闭按钮会触发该事件，传入的参数为一个关闭原因的列表。其中name字段为具体原因，value字段为原因的编码。当选择了具体原因后需要调用submit(webView,result,NEGATIVE_FEED_BACK)来提交结果。

```
private boolean negative(final HybridView webView, String data) {
    try {
        final FeedbackInfoModel feedbackInfoModel = new FeedbackInfoModel();
        feedbackInfoModel.feedbackInfo = new ArrayList<>();

        JSONObject feedbackJson = new JSONObject(data);
        feedbackInfoModel.feedbackInfoId = feedbackJson.optString("feedbackInfoId");
        JSONArray feedbackInfoJson = feedbackJson.optJSONArray("feedbackInfo");
        for (int i = 0; i < feedbackInfoJson.length(); i++) {
            JSONObject object = feedbackInfoJson.optJSONObject(i);
            FeedbackInfoModel.FeedbackItem item = new FeedbackInfoModel.FeedbackItem();
            item.name = object.optString("name");
            item.value = object.optString("value");
            feedbackInfoModel.feedbackInfo.add(item);
            }

        /**
         * 通过feedbackInfoModel来构建native对话框，主要是操作feedbackInfoModel里面的列表feedbackInfo，该列表中包含model--FeedbackItem，其包含
         * 两个属性name和value，name用来展示给用户，value用来回调。
         */

        new AlertDialog.Builder(webView.getContext())
                .setTitle("标题")
                .setMessage(data)
                .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        try {
                            JSONObject negativeReason = new JSONObject();
                            JSONObject result = new JSONObject();
                            JSONArray dReason = new JSONArray();
                            for (int i = 0;i<2;i++){
                                dReason.put(feedbackInfoModel.feedbackInfo.get(i).value);
                            }
                            negativeReason.put("dreason",dReason);
                            result.put("code", 0);
                            result.put("message", "success");
                            result.put("data", negativeReason);

                            //每次都要调用，即使无返回结果，用来删除其中存放的回调
                            submit(webView,result,NEGATIVE_FEED_BACK);
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }

                    }
                })
                .show();

        return true;
    } catch (JSONException e) {
        Log.e("eTest", e.getMessage());
        e.printStackTrace();
    }
    return false;
}
```

