# pt3 Linux driver
- 変更点 ビルド時のワーニング除去（4箇所）

## ビルド
$ cd pt3  
$ make

## カーネルへの組み込み
[コマンド操作で組み込む]  
セキュアブート環境の場合、MOKで署名する  
（署名に失敗またはカーネルへの組み込みに失敗する時はlinux(ubuntu)セキュアブートを参照  
$ sudo kmodsign sha512 /var/lib/shim-signed/mok/MOK.priv /var/lib/shim-signed/mok/MOK.der pt3_drv.ko  
$ sudo make install  
$ sudo modprobe pt3_drv  

[DKMSを使用してカーネルに組み込む]  
$ sudo ${SHELL} ./dkms.install  
※dkmsを実行後modprobeを行わなくてもリブートすることでカーネルに組み込まれる

組み込みに成功すると/dev/pt3videoXが作成される  

[uninstall]  
$ sudo ${SHELL} ./dkms.uninstall  
$ sudo rmmod pt3_drv  
$ sudo make uninstall  
$ make clean  

## オプション指定
オプション指定する時は  
/etc/modprobe.d/options-pt3.confを作成し（ファイル名は任意）  
options pt3_drv lnb=X lnb_force=X debug=X  
と記述する  
lnb   LNBのデフォルト値を指定  
      0: OFF(default)  
      1: 11V  
      2: 15V  
lnb_force 常時lnbで設定した値を利用  
      0: 無効(default)  
      1: 有効  
debug メッセージの表示を制御  
      0: 出来るだけ表示しない (default)  
      1: pt1と同じ程度表示  
      7: デバッグ用メッセージも表示  

## video groupの登録
userをvideo groupに登録する
$ sudo gpasswd -a user video

# linux(ubuntu)セキュアブート

## MOKとは
Machine Owner Key の略  
セキュアブート環境では  
linux(ubuntu)shimバイナリー(UEFIローダー) はMicrosoftが秘密キーで署名している  
UEFIはMicrosoft公開キーを使用してlinux(ubuntu)shimバイナリーの電子署名を検証し起動  
linux(ubuntu)shimバイナリーには予めCanonical公開キーが組み込まれている  
shimバイナリーがCanonical公開キーを使用してGrubまたはMOKManagerの電子署名を検証し起動する  
(Grub、MOKManagerはCanonical秘密キーにて署名されている）  
そこから起動されるlinuxカーネル、標準的に組み込まれるカーネルモジュール（ドライバー）はCanonical秘密キーにて署名されている  
署名されていないものは起動できない仕組み  
pt1_drv や pt3_drvのような自分でビルドしたカーネルオブジェクトは署名してカーネルに組み込まないとデバイスを使用することができない  
Canonicalの秘密キーを使用して署名することは不可能なので自分で独自の秘密キー・公開キーを作成しカーネルオブジェクトに署名する  
この仕組みがMOKである  
MOK managerを使用し、linux(ubuntu)shimバイナリーに自分で作成した独自の公開キーを登録することでカーネルオブジェクトのロードに成功する  

※Microsoftが署名したshimバイナリーを用意しているlinuxディストリビューションのみがセキュアブートでインストールできる  
※UEFIはMicrosoftの公開キーを所持しており、セキュアブート時それを使用してshimの署名を検証する  
※主にubuntu(Canonical)を例にしたが他ディストリビューションも同様の仕組みでセキュアブートを実現している  

## セキュアブート環境でのLinux(ubuntu)OSインストール
インストールの準備画面でサードパーティー製のソフトウェアをインストールするオプションを選択しパスワードを設定する  
このオプションを選択することでMOK用の公開キー・秘密キーが作成される  
/var/lib/shim-signed/mok/MOK.priv (秘密鍵)  
/var/lib/shim-signed/mok/MOK.der (証明書、公開鍵)  
このパスワードはインストール後リブートで表示されるMOK manager(青い画面) でlinux(ubuntu)shimバイナリーにMOK用公開キーを登録する為のもの  
MOK managerでEnroll MOKを選択し同じパスワードを入力することで登録完了となる  
Continueを選ぶと登録せずに終わってしまうので注意  

### セキュアブート確認
$ mokutil --sb-state  
または  
sudo dmesg | grep secureboot  

### MOKのインストールに失敗している時(MOK manager(青い画面)でcontinueを選択してしまった等)]
既存のキーをshimに登録する  
$sudo update-secureboot-policy --enroll-key  
    既に登録済みの時  
    Nothing to do.  
    登録されていない時  
    再起動後に公開キーを登録するためのパスワードの入力を求められる  
### インストール後にセキュアブートに変更した場合
新しくMOKを生成  
$ sudo update-secureboot-policy --new-key  
出力先は  
/var/lib/shim-signed/mok/MOK.priv (秘密キー)  
/var/lib/shim-signed/mok/MOK.der (証明書、公開キー)  

公開キーをshimに登録する  
$ sudo mokutil --import /var/lib/shim-signed/mok/MOK.der  
    input password: xxxxx  
    input password again: xxxxx  
リブートするとMOK manager(青い画面)が起動するので  
[Enroll MOK]  
を選択し、先程設定したパスワードxxxxxを使用し公開キーを登録し再起動する  

### カーネルオブジェクトに署名
セキュアブート環境では署名していないカーネルオブジェクトはinsmod(modprobe)で失敗するので事前にMOKで署名する  
$ sudo kmodsign sha512 /var/lib/shim-signed/mok/MOK.priv /var/lib/shim-signed/mok/MOK.der pt3_drv.ko  
$ sudo insmod /lib/modules/`uname -r`/kernel/drivers/video/pt3_drv.ko  
または  
$ sudo modprobe pt3_drv  
※modprobeは依存関係を調べて全て組み込む
※modprobe前にdepmodを実行する    

### DKMS(Dynamic Kernel Module Support)
DKMSはカーネルアップデート時にカーネルオブジェクトをリビルドし組み込むアプリケーション  
またセキュアブート環境では自動的にMOKで電子署名をするので自分で署名する必要がない  

### shimバイナリー
UEFIから起動するローダープログラム  
セキュアブートの場合、UEFIに組み込まれたMicrosoft公開キーを使用してshimバイナリーのMicrosoft署名が検証される  

### カーネルオブジェクトの署名確認
modinfo オブジェクト名  例 modinfo pt3_drv  
