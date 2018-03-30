---
layout:     post
title:      "Sublime Text 3 备忘"
subtitle:   "神级文本和代码编辑器"
date:       2018-03-29
author:     "leehp258"
header-img: "img/sublime_theme.jpg"
tags:
    - sublime
    - editor
    - tools
---

# 先安装Package Control

1. 按 Ctrl+` 调出console (也可以通过菜单View/Show Console调出)
2. 复制以下代码并回车：

```python
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```
> 重启Sublime后，在`Perferences->Package Settings`中看到`Package Control`项，说明安装成功。

**通过Package Control安装插件的方法：**
`Ctrl+Shift+P`（MAC下是`Command+Shift+P`）调出命令窗口，输入`Install Package`并回车，等待`Loading repositories`然后在列表中搜索要安装的插件

# 安装SolarizedDark主题和自定义常用配置
1. 安装插件：
- Soda SolarizedDark Theme
- A-File-Icon: 文件图标
2. 然后修改自定义配置：

```shell
{
	"auto_indent": true,
	"auto_match_enabled": true,
	"bold_folder_labels": true,
	"caret_style": "smooth",
	"color_scheme": "Packages/Color Scheme - Default/Solarized (Dark).tmTheme",
	"default_encoding": "UTF-8",
	"detect_indentation": true,
	"dictionary": "Packages/Language – English/en_US.dic",
	"dpi_scale": 1.0,
	"fade_fold_buttons": true,
	"fold_buttons": true,
	"font_options": "subpixel_antialias",
	"font_size": 12.0,
	"gutter": true,
	"highlight_line": true,
	"highlight_modified_tabs": true,
	"hot_exit": true,
	"ignored_packages":
	[
		"Vintage"
	],
	"file_exclude_patterns":
	[
		"*.zip", "*.rar", "*.pyc", "*.pyo", "*.exe", "*.dll", "*.obj","*.o", "*.a", "*.lib", "*.so", "*.dylib", "*.ncb", "*.sdf", "*.suo", "*.pdb", "*.idb", ".DS_Store", "*.class", "*.psd", "*.db", "*.sublime-workspace","*.d","*.tags*","*.cmd","*.mod.*","*.symvers","*.order"
	],
	"line_numbers": true,
	"line_padding_bottom": 2,
	"line_padding_top": 3,
	"margin": 4,
	"match_brackets": true,
	"match_brackets_angle": true,
	"match_brackets_braces": true,
	"match_brackets_content": true,
	"match_brackets_square": true,
	"match_selection": true,
	"match_tags": true,
	"rulers":
	[
		100
	],
	"save_on_focus_lost": true,
	"show_tab_close_buttons": true,
	"tab_completion": true,
	"tab_size": 4,
	"theme": "Soda SolarizedDark.sublime-theme",
	"translate_tabs_to_spaces": true,
	"tree_animation_enabled": true,
	"trim_automatic_white_space": true,
	"trim_trailing_white_space_on_save": true,
	"use_tab_stops": true,
	"word_wrap": true,
	"wrap_width": 100
}
```


# Python支持
> `ctrl + b` 可以执行python文件，见菜单项`Tools->Build`

通过SublimeREPL运行Python
先安装，然后`Tools->SublimeREPL->Python`就可以看到效果了
设置快捷键: `Preferences->Key Bindings-User`，就可以用快捷键调出`python shell`或者执行当前python文件了
```shell
[
    {"keys":["ctrl+shift+c"],
        "caption": "SublimeREPL: Python",
        "command": "run_existing_window_command", "args":
        {"id": "repl_python",
        "file": "config/Python/Main.sublime-menu"}},
    {"keys":["ctrl+shift+r"],
        "caption": "SublimeREPL: Python - RUN current file",
        "command": "run_existing_window_command", "args":
        {"id": "repl_python_run",
        "file": "config/Python/Main.sublime-menu"}},
]

```
修改默认的python解释器：
菜单项`Preferences->Browse Packages`，在打开的目录中找到SublimeREPL/config/Python/Main.sublime-menu文件，
修改其中的cmd值为相应的解释器路径

Jedi - Python autocompletion 智能补全插件
```shell
# 快捷键
"goto": "ctrl+shift+g"
"find_usages": "alt+shift+f"
"docstring": "alt+ctrl+d"
```

SublimeTmpl
SublimeTmpl 提供了常用文件模板，新建文件时很有用。

GitGutter
只是喜欢这个插件能够根据 git 的版本来提示你修改了哪些行，能够比较容易定位。

AutoFileName
在字符串中智能补全路径。

Anaconda
代码分析平台，用于代码规范检查。不过我把里面的 pep8 检查排除了 E501，因为我屏幕分辨率高，不想被80个字符束缚。
使用这个插件之后，建议对每个项目保存为 sublime 项目，然后在项目配置文件中指明使用的 virtualenv 解释器：
```json
"settings":
{
    "python_interpreter": "/path/to/my/python"
}
```

CTags
Python 的智能补全插件其实都不能算太好用，在很多时候还不如 Ctags 来得简单粗暴。
OS X 的 ctags 命令不支持 -R 参数，因此需要自己用 brew 或其他方式安装一个，并在设置中指定。

AutoPep8
自动将 Python 代码按 PEP8 规范格式化，安装完添加如下配置可自动在保存文件的时候格式化。
```json
{
	"format_on_save": true,
}
```

Alignment 等号对齐
按Ctrl+Alt+A，可以是凌乱的代码以等号为准左右对其，适合有代码洁癖的朋友。

> 参考：https://segmentfault.com/a/1190000009318179