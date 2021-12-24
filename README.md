# Arch + i3wm download ＆ set

![](https://i.imgur.com/z36LIpO.jpg)

## download ISO

[NCTU CSCC Mirror site](http://linux.cs.nctu.edu.tw/archlinux/iso/latest/)


## write on USB

Ubuntu 會自動幫忙寫入超讚
Windows 可能就要用 [Rufus](https://rufus.ie/en/)
![](https://i.imgur.com/ZxbaCQj.png)

> 一開始用時 USB 爆掉讀不到
> 結果換一個就好了


## mount USB


1. 查看磁碟分配
```
fdisk -l
```

2. 更新時間
```
timedatectl set-ntp true
```
> 無線網路有點麻煩
> 所以我先連用有線的

將你裝 arch 的 USB mount 上去

3. 切割和格式化磁碟
```
#格式 
mkfs.ext4 /dev/sdx
```

4. 掛載根目錄 "/" 
```
mount /dev/[要放根目錄 "/" 的磁區] /mnt
```

5. 掛載 boot
```
mkdir /mnt/boot
mount /dev/[要放開機 boot 的磁區] /mnt/boot
```

6. 更改鏡像檔優先權
```
vim /etc/pacman.d/mirrorlist
```

7. 下載必要 pkg
```
pacstrap /mnt base base-devel linux linux-firmware dhcpcd
```

8. 開機自動掛載
```
genfstab -U /mnt >> /mnt/etc/fstab

# 再次確認
cat /mnt/etc/fstab
```

9. 進入新系統
```
arch-chroot /mnt
```


## 設定 Arch
1. 時區設定(Taipei)
```
ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
timedatectl set-local-rtc 1 --adjust-system-clock
hwclock --systohc
```

2. 安裝基本工具
```
pacman -S

# 文字編輯器
vim 
tmux

# 系統監視管理
htop

# 網路管理
dialog wpa_supplicant
networkmanager 
netctl

# intel 自動更新
intel-ucode

# 協助 gub 掃 OS
os-prober ntfs-3g
```

3. 開啟網路服務
```
systemctl enable NetworkManager
```

4. local 調成 Taiwan
```
vim /etc/locale.gen

把 zh_TW.UTF-8 和 en_US.UTF-8 
取消註解

> lzh_TW UTF-8 ?
```

5. 設定 hostname
```
vim /etc/hostname
```

6. 設定 localhost
```
vim /etc/hosts

127.0.0.1	localhost
::1		localhost
127.0.1.1	[hostname].localdomain	[hostname]
```

7. 設定 user
```
# 建立使用者與家目錄
useradd -m Username

# 設定使用者密碼
passwd [username]

# 使用者加上 sudo 權限
vim /etc/sudoers
加入這行
[username] ALL=(ALL) ALL
```

8. 安裝 bootloader
```
# 安裝 grub2
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub

# 產生 config
grub-mkconfig -o /boot/grub/grub.cfg
```

9. 取消掛載
```
exit
umount /mnt/boot
umount /mnt
```

9. 重啟
```
reboot
```

10. 建立 swap
> 一開始忘記建立 swap
> 結果開機時跑出錯誤碼
> ```
> Failed to activate swap Swap Partition
> ```
> 這是因為我的有兩個系統卻只有一個 swap
> 所以要先裝 swap

```
# 查看內存佔用情況
free -h

# 新增 6G 的 swap
dd if=/dev/[要寫入的磁區] of=/var/swap bs=1024 count=6144000

# 創建 swap 文件
mkswap -f /var/swap

# 加載文件
swapon /var/swap

# 查看是否生效
free -h
> Swap: 6.0G 0B 6.0G

# 添加至 /etc/fstab
vim /etc/fstab
最後面加上 /var/swap swap swap defaults 0 0
```

## Arch 配置
1. 啟用 Wi-fi 服務
> 好吧也不能永遠都用有線
> 來設置一下 Wi-fi 好了

```
wifi-menu
```

> 沒出意外的話又要出意外了Qq
> Job for [network.service] failed because the control process exited with error code.
> 於是用另一個開啟

```
# 開啟服務
sudo system start wpa_supplicant.service
sudo systemctl start wpa_supplicant.service

# 列出無線網路
nmcli dev wifi list

# 連接
nmcli device wifi connect [wifi-name] password "[wifi-password]"
> 別人用 history 就看得到你的 wifi 密碼了
> 所以不建議用
> 下面那個超讚

# 另一個網路連線
nmtui
```

2. 安裝
```
pacman -S

# 圖形驅動程序
xf86-video-intel nvidia nvidia-utils nvidia-settings lib32-nvidia-utils

# 顯示管理器,窗口管理器以及 Google CJK 字體
sudo pacman -S --noconfirm xorg i3-gaps i3lock i3status lightdm lightdm-gtk-greeter noto-fonts-cjk

# 應用城程式啟動器
ranger dmenu rofi

# 終端機圖片顯示
w3m imlib2

# 聲音驅動
alsa-utils pulseaudio pulseaudio-alsa pulseaudio-bluetooth pavucontrol mpd

# RDP
sudo pacman -S --noconfirm freerdp remmina

# 桌布
feh

# 螢幕亮度
light

# Enable the display manager
sudo systemctl enable lightdm
sudo systemctl start lightdm
```


> 補充t常用 i3 快捷建
```
$mod + Enter 開啟 terminal
$mod + w 分頁模式
$mod + e 平铺模式
$mod + f 進入/退出全屏
$mod + Shift + Space 切换窗口浮動/平铺
$mod + Shift + q 關閉窗口
$mod + 1-9 切換到桌面 1-9
$mod + Shift + [1-9] 将当前窗口移至桌面 [1-9]
$mod +[hjkl] 切换 focus 窗口
$mod + Shift + [hjkl] 當前窗口移動至桌面的左/下/上/右
按住 $mod 和鼠標右鍵並拖動：調整浮動的窗口大小
```

3. terminal 美化

```
! ****************
! urxvt config
! ****************

!font
URxvt.font: xft:monaco:size=25

URxvt.inputMethod: fcitx
URxvt.inheritPixmap: True
URxvt.tintColor: #30FFFF
Xft.dpi: 96
Xft.hinting: true
Xft.hintstyle: hintslight
Xft.antialias: true
Xft.rgba: rgb
Xft.autohint: false
Xft.lcdfilter: lcddefault

!color
!URxvt.background: black
URxvt.foreground: #00FFFF
URxvt.colorBD: white
URxvt.colorUL: #FF1212
URxvt.color1: #FF3030
!hard red
URxvt.color8: #FFFF3B
!hard yellow
URxvt.color2: #BFFFFF
!blue
URxvt.color9: #FFDEA1
!orange
URxvt.color3: #FFD4D4
!pink
URxvt.color4: #EDD4FF
!purple
URxvt.color11: #00FFFF
!main blue
URxvt.color10: #FFFF82 
!yellow
URxvt.color12: #FFFFFF 
!white
URxvt.color13: #1212FF
!hard blue
URxvt.color14: #7FFFD4
!hard green
URxvt.color15: #4545FF
!blue

!ctrl+shift+c/v
URxvt.iso14755: false
URxvt.iso14755_52: false

URxvt.keysym.Shift-Control-V: eval:paste_clipboard
URxvt.keysym.Shift-Control-C: eval:selection_to_clipboard
URxvt.keysym.Control-Meta-c: builtin-string:
URxvt.keysym.Control-Meta-v: builtin-string:

!back
URxvt.depth: 64
URxvt.background:rgba:0000/0000/0000/dddd
URxvt.transparent: true
URxvt.shading: 25
```

4. 桌面美化

Polybar theme
```
#安裝 git
pacamn -S git

# clone git
git clone --depth=1 https://github.com/adi1090x/polybar-themes.git

# run
cd polybar-themes
chmod +x setup.sh
./setup.sh

# 選擇主題
bash ~/.config/polybar/launch.sh --[主題]

# 寫入 config
vim ~/.config/i3/config
末端寫入
    exec bash ~/.config/polybar/launch.sh --[主題]
```
> error: module/mpd: Connection refused

```
# 開啟 mpd
systemctl start mpd
systemctl enable mpd
```

5. 設定桌布
```
# 設定桌布
feh --bg-scale [桌面路徑]

# 放入 config
vim ~/.config/i3/config
exec feh --bg-scale [桌面路徑]
```

6. AUR - yay
```
# 安裝 yay
yaourt -S yay
sudo pacman -S git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

# AUR 中安裝軟件包或應用程序
yay -S <package-name>

# 在官方存儲庫和 AUR 中搜索應用程序，添加標誌“ s”
yay -Ss <package-name>

# 安裝本地軟件包
yay -U ruta-del-paquete
```

7. 輸入法安裝
```
# ibus + 新酷音
yay -S ibus ibus-chewing

vim /etc/environment

GTK_IM_MODULE=ibus
QT_IM_MODULE=ibus
XMODIFIERS=@im=ibus
```
> 之後發現 ibus 有一些問題
> 像是 telegram 會不能輸中文字
> 所以改用 fcitx 問題就解決了

```
yay -S fcitx-im fcitx-chewing
```

切換語言設置：
![](https://i.imgur.com/rA2ewED.png)



## 其他
1. flameshot
```
# 安裝
yay -S flameshot

# 設置
flameshot config
```

2. telegram

```
# 使用 ibus 要手動加環境變數
env QT_IM_MODULE=ibus telegram-desktop
```

3. Vivaldi
> 超級超級讚的瀏覽器

```
# 安裝
sudo pacman -S vivaldi vivaldi-ffmpeg-code

# 自訂 keyboard
Notes     Alt+N
Tab Bar   Alt+B
```

4. neofetch
```
#安裝
sudo pacman -S neofetch

# 配置
vim ~/.config/neofetch/config.conf 

# 右邊標章的顏色
ascii_colors([你想要的顏色])

# 隱藏右邊的色塊
color_blocks="off"

# 設定右邊文字顏色
colors=(12 12 12 11 12 12 12)
```

最後成效：
![](https://i.imgur.com/6JhvQqM.png)

# flameshot
## 參考資料

[小知筆記](https://github.com/YukinaMochizuki/arch-linux-on-7559)
[Failed to activate swap Swap Partition](https://bbs.archlinux.org/viewtopic.php?id=215890&fbclid=IwAR3fb-kXyNu2ezL2ruANp97aUuzLDmRi7dE4KokQfk_wInTiJfaWhb7Dhtg)
[swap 配置](https://cloud.tencent.com/developer/article/1525090?fbclid=IwAR0W52OrJkCnynMXS1FZd4cl_TepPz0Q46tUis5vyfasKflQ3p4moLphH9E)
[i3 set & download](https://blog.swwind.me/post/archlinux-setup)
[polybar-theme](https://github.com/adi1090x/polybar-themes)
[archlinux](https://archlinux.org)
