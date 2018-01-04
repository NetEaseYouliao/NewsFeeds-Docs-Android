# 一、UI SDK更新记录

## v1.4.5

- 增加正文WebView
- 去除EventBus依赖
- 广告支持大图和三图模式
- 新增内容负反馈
- UI 样式自定义

## v1.3.6

- 移除微信分享依赖
- fix: view.getContext()在4.x上的兼容性问题

## v1.3.5

- 新增分享回调
- 视频提供相关推荐
- 定位到某一个频道

## v1.3.1 & v1.3.2

- 文章详情页细化资源配置

	* 状态栏颜色
	* 导航栏颜色
	* 导航栏标题颜色
	* 导航栏返回图片资源

## v1.3
- 频道增加自营类型
- 图集提供相关推荐


# 二、DATA SDK更新记录

## v1.4.5

- 移除正文WebView
- 整合列表&正文缓存
- 信息流&正文广告支持大图和三图模式
- 支持内容负反馈
	* NNFNewsInfo新增feedbacks字段
	* 新增NNFeedbackInfo
	* 新增负反馈行为上报接口
- 针对非Activity的Context调用startActivity时，添加FLAG_ACTIVITY_NEW_TASK标记
- loadDetails无需传入infoType和签名，废弃原有接口

## v1.3.5

- 正文WebView支持分享回调
- 为支持视频分享回流，NNFNewsDetails添加producer、summary、videos字段
- NNFNewsDetails添加updateTime字段，NNFNewsInfo添加publishTime字段
- 添加分享行为上报接口
- fix: v1.3新增 DELIVER_ID 导致数据库升级bug

## v1.3
- 频道增加自营类型
	* 数据库新增 DELIVER_ID 字段
	* 数据库版本号升级到4
	* NNFChannelInfo增加channelType字段
	* loadNewsList接收NNFChannelInfo参数，废弃原有接口
- 新增获取相关推荐接口
- 接入易推广
- 集成个推推送（改成provided形式依赖）

## v1.2.4

- 新闻正文举报功能
- inmobi 广告接入
- 缓存新闻列表

## v1.1

- 新闻正文相关推荐
- 集成个推推送（以compile形式依赖）
- 集成广点通

## v1.0

- 新增头图和置顶
