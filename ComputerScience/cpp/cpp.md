%d       int  32bit  
%ld      long  32bit  
%lld     long long 64bit  

### 字符串  
#### 字符串格式化  
```
string timeFormat(long timeMillis) {
    long totalSecond = timeMillis / 1000;
    int seconds = (int) (totalSecond % 60);
    int minutes = (int) ((totalSecond / 60) % 60);
    int hours = (int) (totalSecond / 3600);
    std::ostringstream buffer;
    buffer << hours << ":" << minutes << ":" << seconds;
    return buffer.str();
}
```
#### 字符串追加  
```
char *appText(char *left, char *right) {
    char *text = (char *) malloc(strlen(left) + strlen(right));
    strcpy(text, left);
    strcat(text, right);
    return text;
}
```