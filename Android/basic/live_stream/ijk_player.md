### setOption  
```
private void tryFastOpen3() {
    // 跳过循环滤波
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_CODEC, "skip_loop_filter", 48);
    // 设置最长分析时长
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_FORMAT, "analyzemaxduration", 100L);
    // 通过立即清理数据包来减少等待时长
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_FORMAT, "flush_packets", 1L);
    // 暂停输出直到停止后读取足够的数据包
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "packet-buffering", 0L);
    // 网络不好的情况下进行丢包
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "framedrop", 1L);
    // 去掉音频, 只保留视频;  
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "an", 1);
    //  去掉视频, 只保留音频;  
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "vn", 1);
    // 不查询stream_info，直接使用
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "find_stream_info", 0);
    // 等待开始之后才绘制
    mediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "render-wait-start", 1);
}
```


### 参考  

ijkPlayer 系列  
https://github.com/bilibili/ijkplayer  
https://github.com/CarGuo/GSYVideoPlayer/wiki  
https://juejin.im/post/582ed1c0a0bb9f0067b296a4  
https://juejin.im/post/5a1d8f10f265da431523a849  
https://github.com/lipangit/JiaoZiVideoPlayer  
https://github.com/Rukey7/IjkPlayerView  
https://github.com/dueeeke/dkplayer  
https://github.com/thiagooo0/lmnplayer  
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

优化  
https://www.jianshu.com/p/843c86a9e9ad  
https://blog.csdn.net/ch97ckd/article/details/83155283  

录播优化  http://blog.csdn.net/hejjunlin/article/details/72860470
https://cloud.tencent.com/developer/article/1357997  

直播优化  
https://blog.csdn.net/shareus/article/details/78585260  

https://blog.csdn.net/yyhjifeng/article/details/71191950  
https://www.jianshu.com/p/d6a5d8756eec  
https://blog.csdn.net/qq_31810357/article/details/83340151  
倍速播放  
https://www.cnblogs.com/renhui/p/6510872.html  
http://www.cnblogs.com/wangguchangqing/p/6003087.html  

深入理解  
https://cloud.tencent.com/developer/article/1032547  
https://blog.csdn.net/liuxiaoheng1992/article/details/80601145  
