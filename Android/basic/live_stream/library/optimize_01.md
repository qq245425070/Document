问题描述, 在播放录播文件时, 录播文件是m3u8文件, ijk 认为他是直播;  
在网络异常, ffplayer 会在 read_thread 里面, 界定为直播结束, 并回调 onComplete;  
解决办法是, 手动传标记位, 认为是录播, 让其走 error 回调;  
❀ IjkMediaPlayer  
```
tv.danmaku.ijk.media.player.IjkMediaPlayer#setStreamEnum  

@Override
public native void setStreamEnum(@StreamEnum int streamEnum);

```
❀ ijkplayer_jni.c  
IjkMediaPlayer_setStreamEnum  
```
static void
IjkMediaPlayer_setStreamEnum(JNIEnv *env, jobject thiz, jint stream_enum) {
            MPTRACE("%s\n", __func__);
    ALOGI("LogTrack.Jni.日志: stream_enum=%d", stream_enum);
    IjkMediaPlayer *mp = jni_get_media_player(env, thiz);
    mp->ffplayer->stream_enum = stream_enum;
    LABEL_RETURN:
    ijkmp_dec_ref_p(&mp);
}
```
g_methods  
```
{       "seekTo",                "(J)V",                                                   (void *) IjkMediaPlayer_seekTo},
{       "setStreamEnum",         "(I)V",                                                   (void *) IjkMediaPlayer_setStreamEnum},
{       "_pause",                "()V",                                                    (void *) IjkMediaPlayer_pause},
```
❀ ff_ffplay_def.h  
```
typedef struct FFPlayer {
    int render_wait_start;
    int stream_enum;
} FFPlayer;
```
❀ ff_ffplay.c  
```
static int read_thread(void *arg) {
    if ((!is->paused || completed) &&(!is->audio_st || (is->auddec.finished == is->audioq.serial && frame_queue_nb_remaining(&is->sampq) == 0)) &&(!is->video_st ||(is->viddec.finished == is->videoq.serial && frame_queue_nb_remaining(&is->pictq) == 0))) {
        if (ffp->loop != 1 && (!ffp->loop || --ffp->loop)) {
        } else if (ffp->autoexit) {
        } else {
            if (completed) {
            } else {
                completed = 1;
                ffp->auto_resume = 0;
                ffp_toggle_buffering(ffp, 0);
                toggle_pause(ffp, 1);
                //  fix  只需要改这里  
                av_log(ffp, AV_LOG_INFO, "LogTrack.Jni.日志: ffp=%d\n", ffp->stream_enum);
                if (ffp->error || ffp->stream_enum==FFP_STREAM_ENUM_VOD) {
                    av_log(ffp, AV_LOG_INFO, "ffp_toggle_buffering: error: %d\n", ffp->error);
                    ffp_notify_msg1(ffp, FFP_MSG_ERROR);
                } else {
                    av_log(ffp, AV_LOG_INFO, "ffp_toggle_buffering: completed: OK\n");
                    ffp_notify_msg1(ffp, FFP_MSG_COMPLETED);
                }
            }
        }
    }
}
```
