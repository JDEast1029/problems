# IOS端 audio标签不能自动播放
在开发聊天室时， 发现ios端audio不能连播，这是由于ios的安全机制不允许audio和video自动播放，所以想要获取页面上的audio标签然后给src值，然后使用自动播放那是实现不了的，即使给play()也是播放过不了；但可以通过手动创建audio对象实例来解决这个问题。（React）
+ document注册‘touchstart’监听，实例化一个audio对象
```javascript
//碰到页面就创建一个audio对象，之后就可以通过添加src播放了
 componentDidMount() {
        document.addEventListener('touchstart', this.handleAudioPlay, false);
 }

 handleAudioPlay() {
     if (!window.PxyAudio.isAudio) {
         window.PxyAudio.isAudio = true;
         window.PxyAudio.audio = new Audio();
         document.removeEventListener('touchstart', this.handleAudioPlay, false);
     }
 }
```
+ 获取到当前列表中所有的录音Item的Node，方便更新Item页面状态
```javascript
componentDidMount() {
    if (this.props.chat.content_type === '1') {
        window.audioNodes = {
            ...window.audioNodes,
            [this.props.id]: this.refs.audio
        };
    }
}
componentDidUpdate() {
    if (this.props.chat.content_type === '1') {
        //hack
        //发送消息或者撤回等导致Item重绘的操作，回使refs变为空Object
        //所以重新给audioNodes赋值
        window.audioNodes = {
            ...window.audioNodes,
            [this.props.id]: this.refs.audio
        };
    }
}
```
+ 创建控制音频播放的文件AudioPlayer.js
```javascript
//audioList为音频Item的数据

window.PxyAudio = {
    audio: {},
    isAudio: false,
    audioOnce: null,
    chatId: ''
};

/**
 * 录音播放控制中心
 * @param audioList  音频列表
 * @param index      当前要播放的音频的位置
 * @param playId     要播放的ID
 */
const playAudio = (audioList, index, playId) => {
    if (index > audioList.length - 1) return;
    const audioNodes = window.audioNodes;
    let audio = window.PxyAudio.audio;
    let lastChatId = window.PxyAudio.chatId;

    let playItem = audioNodes[playId];
    let url = audioNodes[playId].props.src;
    if (playId === lastChatId) {
        if (audio.paused) { //暂停状态
            audio.autoplay = true;
            audio.play();
            playItem.handleStart();
        } else {
            audio.pause();
            playItem.handlePause();
        }
    } else {//点击的不是同一个
        //其他暂停
        toPause();

        window.PxyAudio.chatId = playId;

        if (window.PxyAudio.audio) {
            audio = window.PxyAudio.audio;
            audio.src = url;
        } else {
            window.PxyAudio.audio = new Audio();
            audio = window.PxyAudio.audio;
            audio.src = url;
        }
        audio.load();
        audio.onloadstart = function() {
            // playItem.setAttribute('class', 'voice-icon  playloading');
        };
        audio.oncanplaythrough = function() {
            playItem.handleStart();
            audio.autoplay = true;
            audio.play();
        };
        audio.ontimeupdate = function () {
            playItem.handlePlaying();
        };
        audio.onended = function() {
            playItem.handlePause();
            // playAudio(audioList, index + 1, audioList[index + 1].content_id);
            playItem.handleEnd();
        };
        audio.onerror = function () {
            playItem.handlePause();
        }
    }

};

const toPause = () => {
    const audioNodes = window.audioNodes;

    for (let key in audioNodes) {
        audioNodes[key].handlePause()
    }
};
export default playAudio;
```
