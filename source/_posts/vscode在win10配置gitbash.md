---
title: vscode在win10配置git bash
date: 2022-11-11 14:10:42
tags:
 - vscode
 - win10
 - git
categories:
 - vscode
 - git
---



打开设置搜索`Terminal Integrated window`

找到`settings.json`文件打开

![image-20221111143208620](assets/vscode在win10配置gitbash/image-20221111143208620.png)



找到`terminal.integrated.profiles.windows`

![image-20221111143549740](assets/vscode在win10配置gitbash/image-20221111143549740.png)

```json
"terminal.integrated.profiles.windows": {
    "PowerShell": {
        "source": "PowerShell",
        "icon": "terminal-powershell"
    },
    "Command Prompt": {
        "path": [
            "${env:windir}\\Sysnative\\cmd.exe",
            "${env:windir}\\System32\\cmd.exe"
        ],
        "args": [],
        "icon": "terminal-cmd"
    },
    "GitBash": {
        //"source": "Git Bash",
        "path": ["D:\\Program Files\\Git\\bin\\bash.exe"],
        "icon": "terminal-bash"
    }
},
"terminal.integrated.defaultProfile.windows": "GitBash",
```

添加Git Bash配置

```json
"GitBash": {
   //"source": "Git Bash",
   "path": ["D:\\Program Files\\Git\\bin\\bash.exe"], 
   "icon": "terminal-bash"
}
```

`path` 的参数是git的安装路径下的`/bin/bash.exe`

注意`GitBash`可以是`Git-Bash`但不可以是`Git Bash` 配置文件中避免出现空格



这段配置将`Git Bash`设置为默认终端

`"terminal.integrated.defaultProfile.windows": "GitBash",` 




