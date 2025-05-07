# pt3 Linux driver
- 変更点 ビルド時のワーニング除去（2箇所）

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

# linux(ubuntu)セキュアブート

MOKとは  
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

※Microsoftが署名したshimバイナリーを用意しているlinuxディストリビュータのみがセキュアブートでインストールできる  
※UEFIはMicrosoftの公開キーを所持しており、セキュアブート時それを使用してshimの署名を検証する  
※主にubuntu(Canonical)を例にしたが他ディストリビュータも同様の仕組みでセキュアブートを実現している  
