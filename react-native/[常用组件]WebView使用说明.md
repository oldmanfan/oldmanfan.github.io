# WebView使用说明

WebView指的是`react-native-webview`中的最新的实现版本, 相当多的细节需要阅读[WebView的官方文档](https://github.com/react-native-community/react-native-webview/blob/master/docs/Guide.md), 这里只罗列一些需要重点关注的地方.

```js
<WebView
            ref={(ref) => {
              this.webview = ref;
            }}
            mixedContentMode={'always'}
            bounces={false}
            automaticallyAdjustContentInsets={false}
            injectedJavaScript={jsInjectedToWebView()}
            style={styles.webView}
            source={{uri: linkUrl}}
            decelerationRate="normal"
            renderLoading={this.renderLoading}
            onLoadEnd={() => {
              setTimeout(() => {
                this.setState({loading: false});
              }, 1500);
            }}
            onMessage={this.handleMessage}
            onLoad={() => this.sendMessage('isApp')}
            onNavigationStateChange={(e) => {
              this.setState({
                title: e.title || e.url,
              });
            }}
  />
```



## 1. load的页面, 使用了混合http和https的资源

android和ios最新的版本, 都已经默认不再支持http了, 如果确实要在加载的资源中混合使用http和https的资源, 需要明确设置`mixedContentMode={'always’}`这个属性为`always`或`compatability`, 这个值默认为`none`, 从而http的资源不会下载.



## 2. Web和Native交互

参考文档中的说明.



