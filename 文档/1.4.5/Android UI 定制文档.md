### 简介

为了信息流的界面风格能与集成有料UI SDK 的 APP 能够整体统一，有料SDK 提供了简洁的 UI 和功能自定义配置选项。对于配置分为全局配置和局部配置，局部配置的优先级高于全局配置，对于配置针对每个页面做了功能配置和UI配置的区分。对于样式的配置，通过`Map<String, Object>`的形式传入，`String`为属性名称，`Object`为相应的属性值。在全局配置中，每一个页面的配置项对应相应的Key。例如：


- 新闻详情页

功能配置参数在`NNFArticleFuncOption`中，自定义时可通过 `NNFArticleFuncOptionKey`作为键值传入
。UI样式配置参数在`NNFArticleUIOption`中，自定义时可通过 `NNFArticleUIOptionKey`作为键值传入。

- 信息流首页

功能配置参数在`NNFFeedsFuncOption`中，自定义时可通过 `NNFFeedsFuncOptionKey`作为键值传入。UI样式配置参数在`NNFFeedsUIOption`中，自定义时可通过 `NNFFeedsUIOptionKey`作为键值传入。

- 信息流入口

其中针对不同类型的入口具备不同的配置项。功能配置参数在`NNFChannelFuncOption`中，自定义时可通过 `NNFChannelFuncOptionKey`作为键值传入。UI样式配置参数在`NNFChannelUIOption`中，自定义时可通过 `NNFChannelUIOptionKey`作为键值传入。

开发者可在初始化 SDK 时做配置或者在运行时任意时候修改配置，当需要与 SDK 提供的默认界面不一样表现的地方，就修改对应的项，否则不赋值即可，界面会保留默认表现。修改各设置项后，都需要等到下次进入信息流界面才会看到相应的更改。


### 定制界面

#### 新闻详情页

- **功能自定义（NNFArticleFuncOptionKey）**

参数名 | 类型 | 参数说明 | 取值说明
---|--- |--- |--- |
 showRelated |boolean  |文章详情内是否显示相关推荐 | 默认为true，显示相关推荐
 platforms   |int      |支持的社交平台常量值      | 默认为朋友圈，微信好友都显示

 1. 对于分享平台`platforms`配置，开发者需要创建一个int数组，数组内添加要显示的分享平台类型。当前只支持朋友圈和微信好友。默认提供两个常量值NFSharePlatformWXSession(微信好友),NFSharePlatformWXTimeline（朋友圈）。根据int数组的传入顺序决定显示顺序。

- **UI自定义（NNFArticleUIOptionKey）**

参数名 | 类型 | 参数说明 | 取值说明
---|--- |--- |--- |
navigationBackgroundColor |String |一键集成文章详情导航栏背景色   |颜色值 
navType                   |int    | 导航栏标题显示类型常量值     |默认为文章来源
navigationTitle           |String |导航栏自定义标题内容          |
navBottomLineColor        |String |导航栏底部分割线颜色          |颜色值
navBackImage              |int    |导航栏返回按钮图标           |Drawable ResId
navShareImage             |int    |导航栏分享按钮图标           | Drawable ResId
navTintColor              |String |导航栏字体颜色              |颜色值
backgroundColor           |String |文章详情背景色              |颜色值
titleColor                |String |文章标题颜色                |颜色值
sourceTextColor           |String |文章来源颜色                |颜色值
timeTextColor             |String |时间颜色                   |颜色值
seperatorColor            |String |文章标题内容分割线颜色        |颜色值
articleTextColor          |String |文章内容颜色值               |颜色值
articleTextSize           |int    |文章内容字体大小             |单位为px
paragraphIndent           |int    |段首空格                   |默认为0
paragraphSpace            |float    |段间距                     |默认为2
lineSpace                 |float    |行间距                     |默认为1.5f
imagePlaceholderColor     |String |图片占位图颜色               |颜色值
feedsUIOption             |NNFFeedsUIOption|文章详情相关推荐样式控制，同首页信息流样式设置|自定义NNFFeedsUIOption实例


1. 对于其中的feedsUIOption，用户自定义的时候，传入值为`Map<String, Object>`。

#### 信息流首页

- **功能自定义（NNFChannelFuncOptionKey）**

| 参数名          | 类型                   | 参数说明                 | 取值说明                                     |
| :-----------     | :------------------- | :------------------- | :------------------------------- |
| thumbMode        | int      | 当前信息列表页每个cell的图片显示模式 | 默认为正常显示         |
| refreshCount     | int      | 每次下拉刷新请求的条数             | 范围5-20， 默认值10    |            
| defaultChannelID | String   | 一键集成默认进入的频道             | 默认进入切换为第一个    |
| slidable         | boolean  | 频道是否可以滑动切换               | 默认为频道可滑动切换    |


1 . thumbMode，默认提供以下几种常量值供配置使用。NFFeedsOptionThumModNone(无图)、 NFFeedsOptionThumModSingle(单图)、 NFFeedsOptionThumModNormal(按照新闻摘要的图片数显示, 默认值)

- **UI自定义（NNFFeedsUIOptionKey）**

| 参数名                           | 类型        | 参数说明           |           取值说明            |
| :---------------------------- | :-------- | :------------- | :-----------------------: |
| backgroundColor               | String  | 信息流背景色             |  颜色值           |
| cellBackgroundColor           | String  | 单条新闻cell的背景色     |  颜色值            |
| cellEnableBackgroundColor     | String  | 点击新闻cell的高亮背景色  |  颜色值            |
| cellTitleColor                | String  | 新闻cell的标题字体颜色    |  颜色值            |
| cellTitleSize                 | int     | 新闻cell的标题字号大小    |  可取值14，16，18   |
| sourceTextColor               | String  | 新闻cell的来源字体颜色    |  颜色值            |
| showSeparator                 | boolean | 是否显示信息流列表分割线   |  默认显示          |
| separatorColor                | String  | 信息流列表分割线颜色       |  颜色值           |
| lastReadBackgroundDrawable    | int     | 上次已读信息提示背景图     |  Drawable ResId  |
| lastReadTextColor             | String  | 上次已读信息提示字体颜色   |   颜色值           |
| pullingText                   | String  | 下拉刷新控件一般状态文字   |   默认值"下拉刷新"   |
| releaseToRefreshText          | String  | 下拉刷新控件松开状态文字   |   默认值"松开刷新"   |
| loadingText                   | String  | 下拉刷新控件加载中状态文字 |   默认值"加载中"     |
| refreshTextColor              | String  | 下拉刷新控件的字体颜色     |  颜色值            |
| refreshSuccessBackgroundColor | String  | 刷新成功后提示控件背景色   |  颜色值             |
| refreshSuccessTextColor       | String  | 刷新成功后提示控件字体颜色  | 颜色值              |
| loadMoreText                  | String  | 加载更多控件正在加载文字   |  默认值"正在载入更多"  |
| loadMoreTextColor             | String  | 加载更多控件字体颜色      |  颜色值              |
| backgroundColor               | String  | 频道条目背景色           |  颜色值              |
| normalTitleColor              | String  | 未选中状态下标题颜色      |  颜色值              |
| selectedTitleColor            | String  | 选中状态下标题颜色        |  颜色值              |
| selectedIndicatorColor        | String  | 选中状态下指示器颜色      |  颜色值              |
| showBottomLine                | boolean | 是否展示底部分割线        |  默认为显示          |
| bottomLineColor               | String  | 底部分割线颜色           |  颜色值              |

##### 信息流小入口

 - **功能自定义（SmallEntranceFuncOptionKey）**
 
| 参数名	          | 类型	     | 参数说明               | 取值说明
| ------------------| -------- | --------------------- | ----------------- |
| countInPage	         | int	|  入口每页滚动显示的新闻数  | 取值1，2，默认值
| refreshCount        | int	|  每次刷新获取的新闻数     | 如下注释1
| autoScrollInterval  | long  |	自动翻页的时间间隔      | 默认为3000，单位毫秒

1. refreshCount：当countInPage=1时，范围1-5，默认值3。当countInPage=2时，范围2-10，默认值6.

	
- **UI自定义（SmallEntranceUIOptionKey）**
 
| 参数名                 |	类型     |  参数说明                     | 取值说明
| -------               | -------- | -----------                  | ----------------- |
| iconImage             | int      |	 navigationbar返回按钮图片     | Drawable ResId
|backgroundColor	       | String	  | 入口背景颜色                  | 颜色值
|selectBackgroundColor  | String   | 入口选中背景颜色               | 颜色值
|borderColor	           | String   | 上下边界颜色                  | 颜色值
|titleColor	           | String   | 标题颜色                     | 颜色值
|titleFontSize	        | int      | 标题文字大小                 | 默认值为17sp
|lineSpace	           | int       | 两行标题之间的间隔            | 只有当countInPage=2时生效



### 配置代码实例

对于自定义样式的配置，开发接口需要用户通过Map的方式导入，用户可以手动构建Map，也可以通过Json文件，生成相应的Map，来用于自定义样式。如果通过Json文件配置则不支持对于Drawable的设置，Drawable必须通过手动创建Map的方式来进行修改相应的值。

#### 全局配置

对于全局统一的样式，建议采用全局配置的方式，全局配置的代码要在使用SDK创建相应的页面之前进行配置

- 手动创建Map配置

将需要修改的属性，在传入的Map中作修改，如果没有值的传入，使用默认值。

```
public void initGlobalConfigure() {
    //信息流列表UI样式配置，属性和对应的值通过Key，Value形式传入
    Map<String, Object> feedsUIOptionMap = new HashMap<>();
    feedsUIOptionMap.put("cellBackgroundColor",  "#FFFFFF");
    feedsUIOptionMap.put("channelBackgroundColor",  "#FFFFFF");
    feedsUIOptionMap.put("backgroundColor", "#00BFFF");

    //信息流列表功能配置，属性和对应的值通过Key，Value形式传入
    Map<String, Object> feedsFuncMap = new HashMap<>();
    feedsFuncMap.put("slidable", false);
    feedsFuncMap.put("thumbMode", NFFeedsOptionThumModNone);
    
    //文章详情列表配置，属性和对应的值通过Key，Value形式传入
    Map<String, Object> articleUIMap = new HashMap<>();
    articleUIMap.put("backgroundColor", "#00BFFF");
    articleUIMap.put("titleColor", "#ab2b2b");
    articleUIMap.put("articleTextColor", "#0099FB");
    
    //文章详情功能配置，属性和对应的值通过Key，Value形式传入
    Map<String, Object> articleFuncMap = new HashMap<>();
    articleFuncMap.put("showRelated", true);
    articleFuncMap.put("platforms", new int[]{NFSharePlatformWXTimeline});

    //传入各个独立的配置项
    Map<String, Object> customConfigureMap = new HashMap<>();
    customConfigureMap.put(NNFCustomConfigure.NNFFeedsUIOptionKey, feedsUIOptionMap);
    customConfigureMap.put(NNFCustomConfigure.NNFFeedsFuncOptionKey, feedsFuncMap);
    customConfigureMap.put(NNFCustomConfigure.NNFArticleUIOptionKey, articleUIMap);
    customConfigureMap.put(NNFCustomConfigure.NNFArticleFuncOptionKey, articleFuncMap);

    //调用setCustomConfigure，传入全局的配置
    NNewsFeedsUI.setCustomConfigure(customConfigureMap);
}
```


- 通过Json模版文件配置

相比于通过手动Map配置，通过提供的Json模版文件来进行创建，代码更简洁，只需要修改掉需要自定义的值，其它保持默认值即可。

```
public void initGlobalConfigureByJson() {
    String json = null;
    try {
        InputStream inputStream = getAssets().open("json/customconfigure.json");
        json = IOUtil.readInputStream(inputStream);
    } catch (IOException e) {

    }
    Map<String, Object> customConfigureMap = JSON.parseObject(json, Map.class);
    NNewsFeedsUI.setCustomConfigure(customConfigureMap);
}

```

#### 局部配置

分步配置需要在创建相应的页面时，通过`HashMap<String, Object>`的结构，将相应的配置信息传入。如全局配置代码示例所示，既可以通过手动创建Map的方式来实现，也可以通过提供的Json模块来做配置。

- 信息流样式配置

```
public void initNNFeedsConfigure() {
    Map<String, Map<String, Object>> map = new HashMap<>();

    //配置信息流页面UI样式
    Map<String, Object> feedsUIMap = new HashMap<>();
    feedsUIMap.put("cellBackgroundColor",  "#ab2b2b");
    feedsUIMap.put("channelBackgroundColor",  "#0099FB");
    feedsUIMap.put("backgroundColor", "#FFFFFF");
    feedsUIMap.put("channelBackgroundColor", "#ab2b2b");
    feedsUIMap.put("pullingText", "这就是下滑");
    feedsUIMap.put("refreshSuccessBackgroundColor", "#FFFFFF");

    //配置信息流页面功能
    Map<String, Object> feedsFuncMap = new HashMap<>();
    feedsFuncMap.put("slidable", false);
    feedsFuncMap.put("thumbMode", NFFeedsOptionThumModNone);

    //添加UI，功能配置项
    map.put(NNFCustomConfigure.NNFFeedsUIOptionKey, feedsUIMap);
    map.put(NNFCustomConfigure.NNFFeedsFuncOptionKey, feedsFuncMap);

    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    
    //集成创建单个信息流页面时传入
    mFeedsFragment = NNewsFeedsUI.createFeedsFragment(null, null, map);
    ft.replace(R.id.fragment_container, mFeedsFragment);
    ft.commitAllowingStateLoss();
}

```

- 信息流入口配置

此处代码演示通过json字符串的方式来配置信息

```
public void initSmallEntrance() {
	//小入口属性值Json
  String jsonStr = "{ \"smallEntranceFuncOption\": {\n" +
                "    \"autoScrollInterval\": 3000,\n" +
                "    \"countInPage\": 2,\n" +
                "    \"refreshCount\": 6\n" +
                "  },\n" +
                "  \"smallEntranceUIOption\": {\n" +
                "    \"backgroundColor\": \"#FFFFFFFF\",\n" +
                "    \"borderColor\": \"#00000000\",\n" +
                "    \"lineSpace\": 20,\n" +
                "    \"selectBackgroundColor\": \"#f1f4f7\",\n" +
                "    \"titleColor\": \"#333333\",\n" +
                "    \"titleFontSize\": 17\n" +
                "  } }";
    Map<String, Map<String,Object>> map = JSON.parseObject(jsonStr, Map.class);

    FragmentManager fm = getSupportFragmentManager();
    FragmentTransaction ft = fm.beginTransaction();
    //创建Map时导入
    NNFChannelContentFragment fragment = NNewsFeedsUI.createChannelContentViewFragment(null, map, NNFEntranceFragmentType.NNFChannelFragmentSmallEntrance, null, "1245");
    ft.replace(R.id.fragment_container, fragment);
    ft.commitAllowingStateLoss();
}

```

- 文章详情配置

通过NNFArticleUIOptionKey，NNFArticleFuncOptionKey传入相应的配置项进行构建Map，或通过Json字符串的方式创建Map传入。


#### Q&A

-  颜色值该如何传入?

颜色值为十六进制颜色码，通过字符串的方式传入。例如纯白则传入字符串`#FFFFFF`，纯黑则传入`#000000`。透明度的添加在原有颜色十六进制颜色码前添加。如70%透明黑色，则传入`#4c000000`。常见透明度值转换。
>
00%=FF（不透明）    5%=F2    10%=E5    15%=D8    20%=CC    25%=BF    30%=B2    35%=A5    40%=99    45%=8c    50%=7F    
55%=72    60%=66    65%=59    70%=4c    75%=3F    80%=33    85%=21    90%=19    95%=0c    100%=00（全透明）

- 通过Json文件如何定义drawable资源？

目前不支持通过Json字符串修改传入drawable ID，如果要对Drawable进行修改，只能通过手动创建Map的方式来修改相应的值。