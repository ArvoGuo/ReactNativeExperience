关于RN扫一扫功能的实现
=========

上下文：

```json
  "react": "16.0.0-alpha.12",
  "react-native": "0.48.3",
  "react-native-camera": "^0.10.0",
```

需求：扫一扫，页面打开像微信扫一扫类似，中间正方形透明，旁边背景黑色半透明。

问题：

- 1.扫一扫的 `扫` 这个功能怎么做，有没有现成的库
- 2.在明白第一个问题后，继而会思考怎么把界面画成和正常的扫一扫样子一样

分析：在寻找方案的过程中有看到很多库都实现了扫一扫功能，并且有各自的自定义界面，发现所有的库都是基于`react-native-camera@^0.10.0`实现的，无非都是添加了自定义界面，转接了下参数。

解决：

- 问题1答案：`扫`这个功能`react-native-camera@^0.10.0`这个库已经实现了，其实在相机打开运行的时候就已经在扫了，只是界面上和拍照模式长的一样，需要自己去画。需要做的就是接入`react-native-camera@^0.10.0`这个库后给`onBarCodeRead`这个参数传个回调即可，完整代码在文末。

- 问题2答案：画的难点在于如何画出遮罩效果且中间正方形镂空，由于RN采用的是弹性盒模型，并且不像css有伪元素之类的，大多css能想到的方法都尝试了都不行。主流的一个方式就是平铺的方式，大概是5个View，上下左右各一个View背景遮罩，中间一个View背景透明，就感觉有点蠢有点麻烦。`我的解决方式是：利用borderWidth作为遮罩`，代码如下。

```js
import React from 'react';
import {
  Dimensions,
  StyleSheet,
  Text,
  Linking,
  View
} from 'react-native';
import Camera from 'react-native-camera';
export default class Scanning extends React.Component {
  static navigationOptions = ({navigation}) => ({
    title: '扫描'
  })
  _onBarCodeRead = (e) => {
    Linking.openURL(e.data)
      .then(_ => {
        this.props.navigation.goBack();
      })
      .catch( e => {
        console.log(e)
      });
  }
  render() {
    return (
      <View style={styles.container}>
        <Camera
          ref={(cam) => {
            this.camera = cam;
          }}
          style={styles.preview}
          onBarCodeRead={this._onBarCodeRead}
          >
          <View style={styles.scanContainer}>
            <View style={styles.scan}>
              <View style={[styles.sanCorner, styles.scanCornerLeftTop]}></View>
              <View style={[styles.sanCorner, styles.scanCornerRightTop]}></View>
              <View style={[styles.sanCorner, styles.scanCornerRightBottom]}></View>
              <View style={[styles.sanCorner, styles.scanCornerLeftBottom]}></View>
            </View>
          </View>
        </Camera>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
  },
  preview: {
    flex: 1
  },
  scanContainer: {
    borderColor: 'rgba(0, 0, 0, 0.8)',
    borderTopWidth: 100,
    borderBottomWidth: (Dimensions.get('window').height),
    borderLeftWidth: 50,
    borderRightWidth: 50
  },
  scan: {
    width: Dimensions.get('window').width - 100,
    height: Dimensions.get('window').width - 100
  },
  sanCorner: {
    borderColor: 'rgb(0, 166, 255)',
    height: 20,
    width: 20
  },
  scanCornerLeftBottom: {
    borderLeftWidth: 2,
    borderBottomWidth: 2,
    position: 'absolute',
    left: 0,
    bottom: 0,
  },
  scanCornerLeftTop: {
    borderLeftWidth: 2,
    borderTopWidth: 2,
    position: 'absolute',
    left: 0,
    top: 0,
  },
  scanCornerRightBottom: {
    borderRightWidth: 2,
    borderBottomWidth: 2,
    position: 'absolute',
    right: 0,
    bottom: 0,
  },
  scanCornerRightTop: {
    borderRightWidth: 2,
    borderTopWidth: 2,
    position: 'absolute',
    top: 0,
    right: 0,
  }
});
```
