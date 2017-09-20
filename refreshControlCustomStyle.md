关于RefreshControl的自定义样式
============================

上下文：

```json
{
  "react": "16.0.0-alpha.12",
  "react-native": "0.48.3"
}
```

需求：自定义`RefreshControl`样式

问题：官方仅提供部分属性修改，无法全部替换

分析：查看了源码

```js
// ScrollView.js
...
if (refreshControl) {
  if (Platform.OS === 'ios') {
    // On iOS the RefreshControl is a child of the ScrollView.
    // tvOS lacks native support for RefreshControl, so don't include it in that case
    return (
      <ScrollViewClass {...props} ref={this._setScrollViewRef}>
        {Platform.isTVOS ? null : refreshControl}
        {contentContainer}
      </ScrollViewClass>
    );
  } else if (Platform.OS === 'android') {
    // On Android wrap the ScrollView with a AndroidSwipeRefreshLayout.
    // Since the ScrollView is wrapped add the style props to the
    // AndroidSwipeRefreshLayout and use flex: 1 for the ScrollView.
    // Note: we should only apply props.style on the wrapper
    // however, the ScrollView still needs the baseStyle to be scrollable

    return React.cloneElement(
      refreshControl,
      {style: props.style},
      <ScrollViewClass {...props} style={baseStyle} ref={this._setScrollViewRef}>
        {contentContainer}
      </ScrollViewClass>
    );
  }
  ...
```

```js
// RefreshControl.js
...
render() {
  return (
    <NativeRefreshControl
      {...this.props}
      ref={ref => {this._nativeRef = ref;}}
      onRefresh={this._onRefresh}
    />
  );
},
...
if (Platform.OS === 'ios') {
  var NativeRefreshControl = requireNativeComponent(
    'RCTRefreshControl',
    RefreshControl
  );
} else if (Platform.OS === 'android') {
  var NativeRefreshControl = requireNativeComponent(
    'AndroidSwipeRefreshLayout',
    RefreshControl
  );
}
...
```

```objc
// RCTScrollView.m
- (void)insertReactSubview:(UIView *)view atIndex:(NSInteger)atIndex
{
  [super insertReactSubview:view atIndex:atIndex];
#if !TARGET_OS_TV
  if ([view isKindOfClass:[RCTRefreshControl class]]) {
    [_scrollView setRctRefreshControl:(RCTRefreshControl *)view];
  } else
#endif
  {
    RCTAssert(_contentView == nil, @"RCTScrollView may only contain a single subview");
    _contentView = view;
    RCTApplyTranformationAccordingLayoutDirection(_contentView, self.reactLayoutDirection);
    [_scrollView addSubview:view];
  }
}

```

综上代码，
- 1.必须使用`RCTRefreshControl`的继承类(`因为：[view isKindOfClass:[RCTRefreshControl class]]`)
- 2.发现`<NativeRefreshControl {...this.props}`, 该写法是能够直接嵌入子元素的


解决：

方案1:

```js
<FlatList
  ...
  refreshControl={
    <RefreshControl
      refreshing={this.state.refreshing}
      onRefresh={this._refresh}>
      <View>Your component</View>
    </RefreshControl>
  }
/>
```

根据`RefreshControl`提供的属性设置将原有的转菊花设置为透明等，嵌入一个自定义的子元素。

方案2：

写native module。。。