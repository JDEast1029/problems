# IOS端微信 js-sdk 录音问题
如果使用html5的pushState或者replaceState方法进入录音调用的页面(SPA)，ios端会导致录音的startRecord和stopRecord调用失败，报the permission value is offline verifying。建议：
```javascript
window.loaction.replace('http://youaddress');
或
//通过一个全局变量来记录进入页面是的URL
if (_global.landingPage !== `${location.origin}${location.pathname}${location.search}`) {
    _global.landingPage = `${location.origin}${location.pathname}${location.search}`;
    location.reload();
}
```
在录音过程中，如果用户刷新了页面，Android 和IOS 就会进入假录音状态，导致无法调用startRecord，解决如下：
```javascript
//在录音的操作界面显示的时候调用一次stopRecord(),这时不会返回任何内容，
//猜测只是将内部的状态从 ‘正在录音’ 改为 ‘录音已停止’
componentWillMount() {
    wx.stopRecord();
}
```