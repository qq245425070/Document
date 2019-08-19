sublime 单行展示:  view -> layout -> single  
主题的位置:  
```
C:\Users\\Administrator\AppData\Roaming\Sublime Text 2\Packages\User\Preferences.sublime  
```
### preference  
```
{
	"auto_match_enabled": false,  //  自动匹配
	"color_scheme": "Packages/User/alston.tmTheme",
	"font_face": "Calibri",  //  字体 
	"font_size": 15,  //  字号   
	"theme": "Adaptive.sublime-theme",
	"margin": 2, //  文本框 编距
	"create_window_at_startup": false, 
	 "open_files_in_new_window": false,
	 "show_encoding": true,  //  展示 编码 信息
	 "translate_tabs_to_spaces": true,  //  tab 转换 成 空格  
	 "show_full_path": true,  //  展示 全路径
	 "highlight_line": false,  //  高亮 当前行  
	 "highlight_modified_tabs": true,  
	 "update_check" : false,  // 自动更新
	 "auto_complete": false,  // 自动完成  
	 "draw_centered"    : false,  // 是否居中显示
	  "extensions":[
	  	"md"
	  ],
	  //  不记录打开的文件
	  "hot_exit": false,
      "remember_open_files": false
}

```
### 快捷键  
super(mac) = ctrl(windows)  
```
[
    { "keys": ["super+d"], "command": "duplicate_line" },  //  command + D  复制 当前行，插入到下一行
    { "keys": ["super+shift+d"], "command": "null" },  //  取消 功能  

    { "keys": ["alt+shift+up"], "command": "swap_line_up" },  // 向上 移动 当前行  
    { "keys": ["alt+shift+down"], "command": "swap_line_down" },  // 向下 移动 当前行 

    { "keys": ["super+shift+u"], "command": "upper_case" },  //  变为 大写
    { "keys": ["super+u"], "command": "lower_case" }, //  变为 小写

]
```
### 参考  
主题  
http://tmtheme-editor.herokuapp.com/#!/editor/theme/Monokai  
https://github.com/dempfi/ayu  