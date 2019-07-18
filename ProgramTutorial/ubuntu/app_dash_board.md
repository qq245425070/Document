dash_board_fix.sh  
./ dash_board_fix.sh  
```
gsettings set org.gnome.desktop.app-folders folder-children "['accessories', 'chrome-apps', 'games', 'office',  'science', 'sound---video',  'universal-access', 'wine']"


gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/accessories/ name "A附件"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/accessories/ categories "['Utility', 'System', 'Settings']"

gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/sound---video/ name "A媒体"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/sound---video/ categories "['Graphics', 'AudioVideo', 'Audio', 'Video', 'Network', 'WebBrowser', 'Email']"


gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/office/ name "A办公"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/office/ categories "['Office']"

gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/chrome-apps/ name "Chrome Apps"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/chrome-apps/ categories "['chrome-apps']"


gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/games/ name "Games"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/games/ categories "['Game']"



gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/science/ name "Science"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/science/ categories "['Science']"



gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/universal-access/ name "Universal Access"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/universal-access/ categories "['Accessibility']"


gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/wine/ name "Wine"
gsettings set org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/wine/ categories "['Wine', 'X-Wine', 'Wine-Programs-Accessories']"
```

### 参考   
https://github.com/BenJetson/gnome-dash-fix  
https://www.fossmint.com/gnome-dash-fix-automatically-organize-apps-by-categories-in-gnome/  