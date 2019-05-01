开启调试, 日志输出  
```
logTag = "IJKMEDIA";  

adb logcat -s "LogTrack" "IJKMEDIA"

```
自动播放#设置选项  
https://github.com/bilibili/ijkplayer/issues/4200  

### setOption  
```
```
### 音量的焦点  
```
private static final class InternalOnAudioFocusChangeListener implements AudioManager.OnAudioFocusChangeListener {

    @Override
    public void onAudioFocusChange(int focusChange) {
        switch (focusChange) {
            //  当标记获得焦点-
            //  onResume
            case AudioManager.AUDIOFOCUS_GAIN:
                GsPlayerUtil.d("onResume");
                break;
            case AudioManager.AUDIOFOCUS_LOSS:
                //  标记失去焦点, 在长时间失去焦点后, 其他软件获得焦点, 会回调这个标记;
                GsPlayerUtil.d("onStop-onDestroy");
                break;
            case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
                //  标记暂时失去焦点, 其他软件获得焦点, 会回调次标记;
            case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
                //  临时失去焦点, 允许继续以低音量播放;
                GsPlayerUtil.d("onPause");
                break;
            default:
                GsPlayerUtil.d("focusChange=" + focusChange);
        }

    }
}
```

### 耳机插拔  


### bug  
1.. 播放失败  
2.. buffering 只有 start, 收不到 end;  
3.. 经常受到 complete;  
4.. ijkPlayer 有时候,收不到 onPrepared 信号;  
5.. 还有一个 case, 就是假死, 可以理解为所有的原生回调, 忽然都静止了;  
      既收不到 onPrepared 也收不到 onComplete, 也收不到 onBuffered;  
      根据 ijkPlayer后台日志, 最后收到的反馈是 ffp_toggle_buffering: eof        
解决办法:  
1.. 暂时没有到;  
2.. 启动定时器, 轮训;  
3.. 在受到 onComplete 的时候 reopen;  
4.. 和 5.. 可以用轮训的办法解决, 实在是没有更好的办法了, 因为我发现, 假死之后, 手动点击 reopen, 视频又活过来了;  
```
V/LogTrack( 7631): [ (IjkPlayer.java:548) #onBufferingUpdate] progress = 0
...
I/IJKMEDIA( 7631): Hit DNS cache hostname = ivi.bupt.edu.cn
I/IJKMEDIA( 7631): Opening 'http://ivi.bupt.edu.cn/hls/cctv1hd-1556101842000.ts' for reading
I/IJKMEDIA( 7631): will delete cache entry, hostname = ivi.bupt.edu.cn
I/IJKMEDIA( 7631): Add dns cache hostname = ivi.bupt.edu.cn, ip = 58.200.131.5
V/LogTrack( 7631): [ (IjkPlayer.java:548) #onBufferingUpdate] progress = 0
...
I/IJKMEDIA( 7631): Hit DNS cache hostname = ivi.bupt.edu.cn
E/IJKMEDIA( 7631): Hit dns cache but connect fail hostname = ivi.bupt.edu.cn, ip =
W/IJKMEDIA( 7631): Failed to reload playlist 0
V/LogTrack( 7631): [ (IjkPlayer.java:548) #onBufferingUpdate] progress = 0
...
I/IJKMEDIA( 7631): ffp_toggle_buffering: completed: OK
D/IJKMEDIA( 7631): ijkmp_get_msg: FFP_MSG_COMPLETED
D/IJKMEDIA( 7631): FFP_MSG_COMPLETED:
I/IJKMEDIA( 7631): ffp_toggle_buffering: eof

关于 ffp_toggle_buffering: eof
参考, 并不是解决方案, 而是确实别人也遇到了;   
 
https://github.com/Bilibili/ijkplayer/issues/903  
https://github.com/Bilibili/ijkplayer/issues/2153   
https://github.com/Bilibili/ijkplayer/issues/1570  
```
### 参考  

ijkPlayer 系列  
https://github.com/bilibili/ijkplayer  
https://github.com/CarGuo/GSYVideoPlayer/wiki  
https://juejin.im/post/582ed1c0a0bb9f0067b296a4  
https://juejin.im/post/5a1d8f10f265da431523a849  
https://github.com/lipangit/JiaoZiVideoPlayer  
https://github.com/Rukey7/IjkPlayerView  
https://github.com/jiajunhui/PlayerBase  
https://github.com/dueeeke/dkplayer  
https://github.com/thiagooo0/lmnplayer  
https://github.com/lingcimi/jjdxm_ijkplayer  
https://blog.csdn.net/word_code/article/details/78292535  
https://blog.csdn.net/huaxun66/article/details/53401231  
https://www.jianshu.com/c/92a1d0b32c55  
https://blog.csdn.net/ch97ckd/article/details/83155283  
https://www.jianshu.com/p/220b00d00deb  
https://blog.csdn.net/shareus/article/details/78585260  
ff_ffplay_options  
https://github.com/Bilibili/ijkplayer/blob/master/ijkmedia/ijkplayer/ff_ffplay_options.h  
https://github.com/Bilibili/ijkplayer/issues/210  
https://www.jianshu.com/p/7b6a2eee3f3a  
https://blog.csdn.net/u013241923/article/details/83544458  

优化  
https://www.jianshu.com/p/80c56f47a870  
https://www.jianshu.com/p/843c86a9e9ad  
https://blog.csdn.net/ch97ckd/article/details/83155283  
https://github.com/Bilibili/ijkplayer/issues/445  
https://www.jianshu.com/p/d6a5d8756eec  
http://blog.csdn.net/hejjunlin/article/details/72860470
https://cloud.tencent.com/developer/article/1357997  
https://blog.csdn.net/shareus/article/details/78585260  

https://blog.csdn.net/yyhjifeng/article/details/71191950  
https://www.jianshu.com/p/d6a5d8756eec  
https://blog.csdn.net/qq_31810357/article/details/83340151  
https://www.jianshu.com/p/512241bd655a  

倍速播放  
https://www.cnblogs.com/renhui/p/6510872.html  
http://www.cnblogs.com/wangguchangqing/p/6003087.html  

深入理解  
https://cloud.tencent.com/developer/article/1032547  
https://blog.csdn.net/liuxiaoheng1992/article/details/80601145  

音量焦点  
https://blog.csdn.net/wusuobupo/article/details/53034506  
