# pt3 driver
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
