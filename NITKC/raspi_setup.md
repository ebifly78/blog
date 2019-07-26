# RaspberryPi セットアップ方法

created on 2019-07-26

- [microSDカードの準備](#microSDカードの準備)
- [SSH接続と初期設定](#SSH接続と初期設定)

## microSDカードの準備

```
~$ df -ah
Filesystem   Size Used Avail Capacity iused ifree %iused Mounted on
...
/dev/disk2s1 43Mi 22Mi 21Mi  51%      0     0     100%   /Volumes/boot
```

上の出力の "disk2s1" に注意する．この "2" の部分は環境によって変わる．たとえば，"disk3s1" であった場合，以下の操作に含まれる "disk2" の部分をすべて "disk3" に書き換える．

```
~$ diskutil eraseDisk FAT32 RPI /dev/disk2
...
Finished erase on disk2
~$ diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
```

ここで，Etcherを使ってSDカードにOSイメージを書き込む．
Etcherをインストールしていない場合は[こちら](https://www.balena.io/etcher/)から．
Etcherによる書き込みが終わったら，ターミナルから再度操作する．

```
~$ nano /Volumes/boot/config.txt
```

最後の行に，`dtoverlay=dwc2`を追加する．
次に，cmdline.txt を編集する．

```
~$ nano /Volumes/boot/cmdline.txt
```

rootwait と quit の間に，

```
modules-load=dwc2,g_ether g_ether.host_addr=02:22:82:ff:ff:01 g_ether.dev_addr=02:22:82:ff:ff:11
```

を追記する．なお，この "addr=" 以降に書かれたアドレスを書き換えることで，同一のネットワークと認識させるか，異なるネットワークと認識させるか選択できる．

Macのインターネット共有を使用するとき，ここのアドレスが同じSDカードは，すべて同一の共有設定が適用される．また，"ネットワーク環境設定" で，同一のサービス名が適用される．

次に，"ssh" という名前の空ファイルを作成する．その後，SDカードを取り出す．

```
~$ touch /Volumes/boot/ssh
~$ sudo diskutil eject /dev/disk2
```

## SSH接続と初期設定

USBケーブルを用いてMacとラズパイを接続する．
ターミナルから，以下のコマンドでSSH接続し，一度シャットダウンする．

```
~$ ssh pi@raspberrypi.local
password: raspberry
~$ sudo poweroff
```

Macの「システム環境設定 > 共有 > インターネット共有」から，"RNDIS/Ethernet Gadget" にチェックを入れる．
「インターネット共有」にもチェックを入れると，確認ウインドウが出るので，「開始」をクリックする．
USBケーブルを挿しなおし，ラズパイを起動する．

`~$ ping 8.8.8.8`で出力が帰ってくれば，インターネットに接続できています．
ここで，ラズパイの言語とタイムゾーンを日本に切り替えます．

```
~$ sudo raspi-config
```

"4 Localisation Options" から，
"I1 Change Locale" "ja_jp.UTF-8 UTF-8（スペースキーで選択）" "ja_JP.UTF-8" の順に選択．
"I2 Change Timezone" "Asia" "Tokyo" の順に選択．
再起動後から，日本語設定になります．
ターミナルから，

```
~$ sudo apt-get update
~$ sudo apt -y upgrade
```

を実行し，アップデートして初期設定は終了です．

```
再起動
~$ sudo reboot
シャットダウン
~$ sudo poweroff
```
