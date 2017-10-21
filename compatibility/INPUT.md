# IOS 键盘遮挡输入框
如果input 固定在页面底部，IOS在呼出键盘时会将输入框遮挡，解决：
```javascript
//通过将body.scrollTop设置为一个值，将 body滚动到最底部
//在输入框获取焦点时，执行多次的scrollTop,因为在键盘彻底呼出之前设置scrollTop不会有效果
handleTextClick = async (e) => {
    if(this.timer) return;
    this.timer = 1;
    try{
        e.preventDefault();
        for(let i = 0; i <= 12; i++) {
            await this.promiseDelay();
            document.body.scrollTop = 10000;
        }
        this.timer = 0;
    }catch(e){
        this.timer = 0;
        console.log(e);
    }
};

promiseDelay() {
    return new Promise((resolve) => {
        setTimeout(()=>{
            resolve();
        }, 50)
    })
}
```
以上办法IOS端或出现光标快速闪烁的问题（RNMMP）