# 曝光事件

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

注意： 同一篇新闻曝光可以多次上传，后台会过滤，建议为了节省流量，app开发者应尽量避免同一篇新闻的曝光事件多次上传

trackNewsExposure接口会收集待曝光的新闻，并每次间隔15s上报一次，用户只需合理调用该接口即可

### 普通新闻列表的曝光

新闻列表暴露在当前屏幕0.5s以上调用，若是曝光时间太短，影响推荐数据

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

### banner轮播图曝光说明

用户一般不会快速地滑动轮播图，因此，我们无需考虑轮播图曝光时间的问题，当切换到轮播图的某一页时，进行曝光打点。已曝光的页无需重复曝光。

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

### 新闻正文相关推荐曝光

滑动到正文底部时，如有相关推荐，则曝光相关推荐。调用下面的接口进行曝光：

```java
NNFTracker.getInstance().trackNewsExposure(newsInfo);
```

### 图集相关推荐曝光

浏览图集中的图片，滑动到最后一页时，展示图集相关推荐，一次性曝光页面上展示的多个相关图集。调用下面的接口进行曝光：

```java
NNFTracker.getInstance().trackNewsExposure(newsInfo);
```

### 视频相关推荐曝光
若用户通过列表的形式展示视频及相关视频，可参考下面的曝光方案：

视频及相关视频列表的展示与普通新闻列表类似，因此曝光逻辑也类似。

##### 1. 在视频单元格添加到列表的时候记录曝光开始时间

示例代码参考普通新闻曝光

##### 2. 在视频单元格从列表移除的时候曝光打点

示例代码参考普通新闻曝光

##### 3. 列表未滚动时，视频列表当前屏的曝光统计
由于上述曝光事件只有在视频单元格从新闻列表移除时才会触发，也就是只有视频列表滚动的时候才会触发。当页面未滚动时，停留在当前页面的视频单元格按照上述方法无法上传曝光事件，因此需要额外时机上传当前页面的视频单元格的曝光事件。

一般情况下，视频页面是独立的Activity页面，因此只需在视频页面退出时，统计视频列表当前屏的曝光。

若视频页面是Fragment页面，则建议App开发人员在以下两种情形下，统计视频列表当前屏的曝光：

```
1. 视频Fragment由可见变成不可见时，对视频列表ListItem进行曝光统计；
2. 视频Fragment依附的Activity退出时，若视频Fragment可见，则对视频ListItem进行曝光统计。
```

示例代码参考普通新闻曝光。




