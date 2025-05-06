# ImageMagickライブラリについて
画像サイズの変更するLinux由来のライブラリ。

## インストール手順
アプリの階層にて、
1. `sudo yum update -y`
2. `sudo amazon-linux-extras install epel -y`
3. `sudo yum install -y ImageMagick`
でインストール。（各行の解説は下に記載）

`convert --version`で正しくインストールされているか確認。
```
username:~/environment/meshiterro (main) $ convert --version
Version: ImageMagick 6.9.10-97 Q16 x86_64 2024-05-13 https://imagemagick.org
Copyright: © 1999-2020 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC Modules OpenMP(4.5)
Delegates (built-in): bzlib cairo fftw fontconfig freetype gslib gvc jbig jng jp2 jpeg lcms ltdl lzma openexr pangocairo png ps raw rsvg tiff webp wmf x xml zlib
username:~/environment/meshiterro (main) $
```
のようになったらOK。

```
ActiveStrage由来の画像オブジェクト.variant(resize_to_limit: [width, height]).processed
```
という感じで、リサイズする。

## インストール手順のコマンド解説。
### パッケージの情報を最新にする
```
sudo yum update -y
```
- yum: Amazon Linux（RedHat系）のパッケージ管理コマンド。
- update 「インストール済み or 利用可能なパッケージ情報を更新する」操作。
- -y: すべての確認を 'yes' に自動応答する」オプション

### EPEL（Extra Packages for Enterprise Linux）という追加パッケージ集を有効にする
```
sudo amazon-linux-extras install epel -y
```
- amazon-linux-extras: Amazon Linux 特有のパッケージ管理拡張コマンド
- epel: RedHat系にない追加の便利パッケージ群（Extra Packages for Enterprise Linux）。
- install epel: EPEL を使えるようにする

### ImageMagick を実際にインストール
```
sudo yum install -y ImageMagick
```
- yum install: パッケージをインストール。

```
convert --version
```
- convert: ImageMagick由来のコマンド

## ImageMagick由来のコマンド
- convert： 画像変換・編集
- mogrify： 画像を上書き編集
- identify： 画像のメタ情報表示
- montage： 画像を並べて1枚に
- composite： 画像を合成

