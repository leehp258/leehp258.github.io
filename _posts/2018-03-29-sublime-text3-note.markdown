---
layout:     post
title:      "Sublime Text 3 备忘"
subtitle:   "神级文本和代码编辑器"
date:       2018-03-29
author:     "leehp258"
header-img: "img/home-bg.jpg"
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
`Ctrl+Shift+P`调出命令窗口，输入`Install Package`并回车，等待`Loading repositories`然后在列表中搜索要安装的插件

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
		80
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
	"word_wrap": true
}

```

> 参考：https://segmentfault.com/a/1190000009318179
