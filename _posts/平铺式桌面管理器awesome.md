---
title: "平铺式桌面管理器AwesomeWM"
date: 2017-08-08 0800
cover: "2.jpg"
category: "Linux"
tags:
    - Linux
    - AwesomeWM
---

# 平铺式桌面管理器awesomewm
第一篇就来写一写我很早就想记录下来的一些东西，一个目前我很喜欢的window manager：awesome。

## 目的：“纯粹”
把博客从Hexo迁移回github的一个原因就是“纯粹”，换句话说就是简单易用，满足所有需求的最轻量选择。

我从刚一开始Ubuntu的Unity折腾到Gnome，又从Gnome折腾到awesomewm，倒是没有使用过几个不同的桌面环境，但是到了awesome这里我在“酷炫桌面”这个主题上的折腾之心就收敛了，因为到了这里我对“好用又好看的桌面”的要求被很好地满足了。

## 特性：“高效”
起初是看到隔壁的前辈在用这个wm，毕竟人家一手HHKB，总要物尽其用：所有的桌面切换操作都使用键盘来完成——我自己就是一个键盘党，恨不得抛弃鼠标，包括在浏览网页时（这个的确可以实现，最后会提及）。

Awesomewm能够将所有切换桌面、控制窗口（client）的操作映射为指定的快捷键，并且具有高度的可配置性。

我个人的需求非常简单：
1. 同屏可显示的信息尽可能多，屏幕空间应尽可能显示信息而非无用内容
2. 单一屏幕空间足够大，多窗口不至于过于凌乱
3. 可以方便地控制显示位置、工作区和屏幕

其中需求1和2实际上是存在冲突的，显示的信息越多，意味着窗口或者tab就越多，相应地就存在越大可能使整个屏幕看起来非常混乱，窗口到处都是又彼此重叠，切换还要记忆顺序按好几次快捷键（如果有快捷键的话，没错说的就是Windows)，这样极大地影响了用户体验。

Awesomewm能够在尽量满足1\2的同时，也提供了众多快捷键来增强用户对桌面显示内容的控制，这一点简直不能更好。

## 使用：“无他，唯手熟尔”
### 安装
参考：[AwesomeWM 4.1 install on Ubuntu 17.04](https://www.reddit.com/r/awesomewm/comments/66qiue/awesomewm_41_install_on_ubuntu_1704/)
```shell
# Get awesome
wget https://github.com/awesomeWM/awesome-releases/raw/master/awesome-4.1.tar.bz2

# Extract Awesome
tar xvjf awesome-4.1.tar.bz2
cd awesome-4.1

# Install Dependencies
sudo apt install cmake libgdk-pixbuf2.0-dev libcairo2-dev libxcb-cursor-dev libxcb-randr0-dev libxcb-xtest0-dev libxcb-xinerama0-dev libxcb-shape0-dev libxcb-util-dev libxcb-keysyms1-dev libxcb-icccm4-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev libstartup-notification0-dev libxdg-basedir-dev libxcb-xrm-dev lua5.2 liblua5.2-dev lua-lgi-dev

# Make awesome
make

# Install awesome
sudo make install
```
在安装依赖的过程中，可能会遇到一个包`libxcb-xrm-dev`找不到的情况，导致安装的Awesome不可用。

这里使用[No package 'xcb-xrm' found](https://unix.stackexchange.com/questions/338628/no-package-xcb-xrm-found)的解决方法:
```shell
sudo add-apt-repository ppa:aguignard/ppa
sudo apt-get update
sudo apt-get install xcb-util-xrm
```
就可以正确安装依赖。

### 配置
AwesomeWM的配置文件位于`~/.config/awesome/rc.lua`，这个文件可以在`/etc/xdg/awesome/`下找到，如果没有，那么可以自己新建一个，按照官方文档进行配置。

但是从头开始配置一个完整的`rc.lua`实在太耗时，本着快速上手越快越好的原则，我使用了[awesome-copycats](https://github.com/copycat-killer/awesome-copycats)提供的配置文件和主题。

下面是一些我根据自己的情况修改的配置项，有些通用的选项还是比较有必要的：
以主题`multicolor`为例：

#### 设置常用项：
```lua
local editor       = "vim"
local browser      = "google-chrome"
local terminal     = "gnome-terminal"
```
由于我的系统原来使用的是Gnome，所以比较习惯原来的terminal，当然你也可以换成其他的xterm之类的。

#### 布局(layouts):
这里建议只保留`tile`类别下的各个布局。毕竟我们安装这个东西就是为了平铺布局。  
我的设置是：
```lua
awful.layout.layouts = {
    awful.layout.suit.tile,
    awful.layout.suit.tile,
    awful.layout.suit.tile.left,
    awful.layout.suit.tile.bottom,
    awful.layout.suit.tile.top,
    awful.layout.suit.corner.nw,
    awful.layout.suit.corner.ne,
    awful.layout.suit.corner.sw,
    awful.layout.suit.corner.se,
    lain.layout.cascade,
    lain.layout.cascade.tile,
    lain.layout.centerwork,
    lain.layout.centerwork.horizontal,
    lain.layout.termfair,
    lain.layout.termfair.center,
}
```

#### 图标大小
以我的屏幕分辨率(2560x1440)感觉，12还算可以。可以在文件中搜索数字，来选择替换。

#### 菜单
```lua
local myawesomemenu = {
    { ">>___Reboot___<<", "reboot"},
    { ">>__ Shutdown__<<", "shutdown -h now"},
    { "Hotkeys", function() return false, hotkeys_popup.show_help end },
    { "Manual", terminal .. " -e man awesome" },
    { "Config", string.format("%s -e %s %s", terminal, editor, awesome.conffile) },
    { "Restart", awesome.restart },
    { "Quit", function() awesome.quit() end }
}

local appmenu = freedesktop.menu.build({
    icon_size = beautiful.menu_height or 12,
})

awful.util.mymainmenu = freedesktop.menu.build({
    icon_size = beautiful.menu_height or 12,
    before = {
        { "VSCode", "code" },
        { "Awesome", myawesomemenu, beautiful.awesome_icon },
    },
    after = {
        { "Lock", "gnome-screensaver-command -l" },
    }
})
```
这里我在一级菜单(`mymainmenu`)下添加了编辑器、awesome菜单和锁屏选项；在awesome这个一级菜单项下添加了重启和关机这两个二级菜单选项。

#### 规则(Rules)
这里主要是控制窗口行为的一些规则。
```lua
awful.rules.rules = {
    -- All clients will match this rule.
    { rule = { },
      properties = { border_width = beautiful.border_width,
                     border_color = beautiful.border_normal,
                     focus = awful.client.focus.filter,
                     raise = true,
                     keys = clientkeys,
                     buttons = clientbuttons,
                     screen = awful.screen.preferred,
                     placement = awful.placement.no_overlap+awful.placement.no_offscreen,
                     size_hints_honor = false
     }
    },

    -- Titlebars
    { rule_any = { type = { "dialog", "normal" } },
      properties = { titlebars_enabled = true } },
    { rule = { class = "shadowsocks-qt5" },
      properties = { screen = 1, tag = screen[1].tags[5] } },
    { rule = { class = "Gimp", role = "gimp-image-window" },
      properties = { maximized = true } },
    { rule = { class = "Google-chrome" },
      properties = { floating = false,
                     maximized = false,
                     maximized_vertical = false,
                     maximized_horizontal = false, } },
}
```
这些选项的主要目的是：
    1. 新建窗口时，新窗口的默认样式和行为
    2. 按照程序(窗口类别)设置新窗口样式和行为

比如这里我设置ss-qt5客户端自启动后显示在屏幕1的标签5中。
另一个必要的设置是：
> 让Chrome新窗口默认不处于最大化状态，禁用浮动。这个设置可以在Chrome窗口存在的情况下，和其他窗口平铺。否则新建一个窗口，如终端窗口，会盖在Chrome上方，Chrome会自动被设置为浮动窗口，而不能正常平铺。

#### 声音调整
使用下列设置可以将`alt + ↑`、`alt + ↓`和`alt + m`映射成音量加、音量减和切换静音。
注意：这些设置需要在`Key bindings`下修改，可以搜索"ALSA volume control"来修改原有的声音设置。
```lua
awful.key({ altkey }, "Up",
    function ()
        os.execute("amixer -D pulse sset Master 5%+")
        beautiful.volume.update()
    end),
awful.key({ altkey }, "Down",
    function ()
        os.execute("amixer -D pulse sset Master 5%-")
        beautiful.volume.update()
    end),
awful.key({ altkey }, "m",
    function ()
        os.execute("amixer -D pulse sset Master 1+ toggle")
        beautiful.volume.update()
    end),
```

### 启动
使用AwesomeWM需要在`.xprofile`中进行相关设置。
```shell
#!/usr/bin/sh
xrandr --output HDMI-2 --above eDP-1
#fcitx need
export XMODIFIERS=@im=fcitx
export QT_IM_MODULE=fcitx
export GTK_IM_MODULE=fcitx
fcitx-autostart

gnome-settings-daemon &
gnome-power-manager &
gnome-screensaver &
xcompmgr -c &
nm-applet --sm-disable &
update-notifier &
sh /home/mingholy/Downloads/start.sh &
ss-qt5 &

exec awesome
```
所有的准备工作和自启动项都要放在`exec awesome`之前。
其中第一行
```
xrandr --output HDMI-2 --above eDP-1
```
是上下双屏排布的命令，如果是左右排布的，可以`--left-of`或`--right-of`，外加
```
xrandr --output eDP-1 --pos [笔记本屏幕左上角的x轴坐标]x[y轴坐标]
```
来调整两个屏幕的相对位置。

## 感受
通过`Mod4`键组合得到的各种切换窗口、切屏功能完全满足了我在浏览器、vim、vscode和终端之间切换的需求，切换窗口、工作上下文的成本得到了极大的降低，能够很好地提高工作效率。配合Chrome的插件`vimium`基本可以做到不用鼠标。
> `vimium`可以将页面上的所有链接转化为快捷键，在页面上点击f键，可以看到每个可点击链接的快捷键，接下来输入快捷键会自动聚焦点击该链接或区域。
