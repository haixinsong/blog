# vscode 配置笔记

[^_^]: # (url:a-note-about-vscode-config)
[^_^]: # (tag:note,config,vscode,#tech)
[^_^]: # (excerpt:这是笔者配置 vscode 开发环境的笔记)
> 此篇为个人vscode的配置笔记，仅供参考。有任何疑问或建议，欢迎留言探讨。

## Settings

### 通用配置

- 禁用自动更新和后台更新

  ```json
  "update.mode": "manual",
  "update.enableWindowsBackgroundUpdates": false,
  ```

- 关闭反馈

  ```json
  "telemetry.enableCrashReporter": false,
  "telemetry.enableTelemetry": false
  ```

- 自动换行

  ```json
  "editor.wordWrap": "on"
  ```

- 鼠标滚轮滑动缩放

  ```json
  "editor.mouseWheelZoom": true
  ```

- 标题栏显示文件完整路径

  ```json
  "window.title": "${activeEditorLong}"
  ```

- 取消针对不同文件应用不同设置(可解决 windows 与 ubuntu tab 键缩进空格数不一致的问题)

  ```json
  "editor.detectIndentation": false
  ```

- 删除文件不需要二次确认

  ```json
  "explorer.confirmDelete": false
  ```

- 输入时自动格式化代码

  ```json
  "editor.formatOnType": true
  ```

- 保存时自动格式化代码

  ```json
  "editor.formatOnSave": true
  ```

### 扩展配置

- Material Theme

  ```json
  "workbench.iconTheme": "eq-material-theme-icons",
  "workbench.colorTheme": "Material Theme"
  ```

### 全部配置

- all settings

  ```json
  "update.mode": "manual",
  "update.enableWindowsBackgroundUpdates": false,
  "telemetry.enableCrashReporter": false,
  "telemetry.enableTelemetry": false,
  "editor.wordWrap": "on",
  "editor.mouseWheelZoom": true,
  "window.title": "${activeEditorLong}",
  "editor.detectIndentation": false,
  "explorer.confirmDelete": false,
  "editor.formatOnType": true,
  "editor.formatOnSave": true,
  "workbench.iconTheme": "eq-material-theme-icons",
  "workbench.colorTheme": "Material Theme",
  ```

## Extensions

### 通用扩展

- 主题(包括图标)  
  [Material Theme](https://marketplace.visualstudio.com/items?itemName=Equinusocio.vsc-material-theme#overview)  
  `ext install vsc-material-theme`
- 彩虹括号  
  [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)  
  `ext install vsc-bracket-pair-colorizer`
- 代码运行  
  [Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner)  
  `ext install formulahendry.code-runner`
- 代码截图
  [Polacode](https://marketplace.visualstudio.com/items?itemName=pnp.polacode)  
  `ext install pnp.polacode`
- 代码特效  
  [Power Mode](https://marketplace.visualstudio.com/items?itemName=hoovercj.vscode-power-mode)  
  `ext install hoovercj.vscode-power-mode`
- 效率分析  
  [WakaTime](https://marketplace.visualstudio.com/items?itemName=WakaTime.vscode-wakatime)  
  `ext install WakaTime.vscode-wakatime`
- TODO 高亮  
  [TODO Highlight](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)  
  `ext install wayou.vscode-todo-highlight`
- 配置同步  
  [Settings Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync)  
  `ext install Shan.code-settings-sync`
- 书签
  [Bookmarks](https://marketplace.visualstudio.com/items?itemName=alefragnani.Bookmarks)
  `ext install alefragnani.Bookmarks`
- 项目管理
  [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager)
  `ext install alefragnani.project-manager`
- 代码格式化  
  [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)  
  `ext install esbenp.prettier-vscode`
- 在浏览器打开  
  [open in browser](https://marketplace.visualstudio.com/items?itemName=techer.open-in-browser)  
  `ext install techer.open-in-browser`
- Live Server  
  [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)  
  `ext install ritwickdey.LiveServer`
- VS Live Share Extension Pack  
  [VS Live Share Extension Pack](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare-pack)  
  `ext install MS-vsliveshare.vsliveshare-pack`
- Git Extension Pack  
  [Git Extension Pack](https://marketplace.visualstudio.com/items?itemName=donjayamanne.git-extension-pack#overview)  
  `ext install donjayamanne.git-extension-pack`

### 针对扩展

- markdown

  - [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)  
    `ext install DavidAnson.vscode-markdownlint`

- c/c++

  - [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)  
    `ext install ms-vscode.cpptools`

- java

  - [java extension pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)  
    `ext install vscjava.vscode-java-pack`

- docker

  - [Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
    `ext install PeterJausovec.vscode-docker`

- go

  - [Go](https://marketplace.visualstudio.com/items?itemName=ms-vscode.Go)
    `ext install ms-vscode.Go`

- python
  - [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)  
     `ext install ms-python.python`
  - pylint
