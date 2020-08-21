# FlatList使用过程中的关注点

FlatList现在是RN推荐使用的Scrollable的组件了, 使用过程中有一些点需要重点关注.

下面是一个完整的例子, 包括目前使用过的需求和实现说明:

```typescript
<FlatList
    style={{marginTop: 5, marginBottom: 60}}
    contentContainerStyle={{flexGrow: 1}}
    ref={(r) => (this.flatList = r!)}
    initialNumToRender={8}
    data={this.state.data}
    renderItem={({item}) => this.renderItem(item)}
    keyExtractor={(item, index) => {
      return index.toString();
    }}
    
    refreshing={this.state.refreshing}
    onRefresh={() => this.onSearch(this.state.owner)}
    
    onViewableItemsChanged={this._onViewableItemsChanged}
    viewabilityConfig={this._viewabilityConfig}
/>
```



## 1. data为空的时候, 让FlatList显示占据全屏

如果不做设置, FlatList在data为空的时候, 是不会render内容的, 这样一来, container设置的backgroundColor不显示出来, 界面非常不美观, 所以如果要在`data=[]`的时候FlatList也全屏, 可以设置:

`contentContainerStyle={{flexGrow: 1}}`, 同时, <u>FlatList的父容器的flex要设置为1.</u>



## 2. 设置下拉时刷新

需要同时设置下面两个属性, 然后在拉到头的时候, 再往下接就会触发`onRefresh`.

```
refreshing={this.state.refreshing}
onRefresh={() => this.onSearch(this.state.owner)}
```



## 3. 想知道当前显示的是哪一个item

FlatList只会render当前需要显示的item, 那要怎样才能知道当前有些item正在被显示呢?  可以通过以下两个属性探知:

```text
onViewableItemsChanged={this._onViewableItemsChanged}
viewabilityConfig={this._viewabilityConfig}
```

其中*viewabilityConfig*必须要以单独object的方式提供, <u>不能在JSX中嵌套</u>, 如下面这样(表示一个item只要有50的height被显示了, 就认为这个item已经显示):

```text
_viewabilityConfig = {
     itemVisiblePercentThreshold: 50,
};
```

*onViewableItemsChanged*方法在每次滚动时, 都会回调, 告诉我们现在有哪些items已经visible了以及其它我们想要知道的所有信息.

```text
_onViewableItemsChanged = (info: {
    viewableItems: Array<ViewToken>;
    changed: Array<ViewToken>;
  }) => {
    // console.log('viewableItems: ', info);
    this.flatListPosition = info?.viewableItems?.[0].index! ?? 0;
  };
```



## 4. scroll到某个特定的item

FlatList提供了一个方法`scrollToIndex({index: 1})`, 来帮助我们直接滚动到某个特定的item.