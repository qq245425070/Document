❀ ijkplayer_jni.c  
```
static void
IjkMediaPlayer_seekTo(JNIEnv *env, jobject thiz, jlong msec) {
    ijkmp_seek_to(mp, msec);
}
```
❀ ijkplayer.c  
```
int ijkmp_seek_to(IjkMediaPlayer *mp, long msec) {
    int retval = ijkmp_seek_to_l(mp, msec);
}

//  发送消息  FFP_REQ_SEEK  
int ijkmp_seek_to_l(IjkMediaPlayer *mp, long msec) {
    ffp_remove_msg(mp->ffplayer, FFP_REQ_SEEK);
    ffp_notify_msg2(mp->ffplayer, FFP_REQ_SEEK, (int) msec);
}

//  处理消息  
int ijkmp_get_msg(IjkMediaPlayer *mp, AVMessage *msg, int block) {
                case FFP_REQ_SEEK:
                            MPTRACE("ijkmp_get_msg: FFP_REQ_SEEK\n");
                    continue_wait_next_msg = 1;
    
                    pthread_mutex_lock(&mp->mutex);
                    if (0 == ikjmp_chkst_seek_l(mp->mp_state)) {
                        mp->restart_from_beginning = 0;
                        if (0 == ffp_seek_to_l(mp->ffplayer, msg->arg1)) {
                            av_log(mp->ffplayer, AV_LOG_DEBUG, "ijkmp_get_msg: FFP_REQ_SEEK: seek to %d\n",
                                   (int) msg->arg1);
                        }
                    }
                    pthread_mutex_unlock(&mp->mutex);
                    break;
}
```



