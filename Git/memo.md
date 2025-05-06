# Gitについて覚書

## Gitの構成
リモートリポジトリ　<=|=> (リモートリポジトリのコピー) ローカルリポジトリ <=> インデックス(ステージング) <=> ワークツリー

## ローカルでGitを使えるようにする
`git init` リポジトリを新規作成するコマンド

## ファイルの変更（追加/変更/削除）を確認する
`git status` ワークツリーとインデックスの現在の状況を確認する

## 変更履歴を残す準備をする
`git add ファイル名` ワークツリーからインデックスに移す。

`git add -A または git add .`で全てのファイルを一括でインデックスに追加

## 変更履歴を保存する
`git commit -m "コミットメッセージ"` インデックスに登録された状態を、ローカルリポジトリにコミット

## 差分を確認する
`git diff` 前後の差分を確認する。差分がない場合はメッセージはない。
```
diff --git a/変更ファイル b/変更ファイル
index e69de29..5c04dac 100644
--- a/変更ファイル
+++ b/変更ファイル
@@ -0,0 +1 @@
+変更内容
```

## 変更履歴を確認する
`git log` 「いつ」「誰が」「なぜ」変更したかを、履歴として確認できる。
```
commit コミットの識別番号 (HEAD -&gt; master)
Author: ユーザ名 <メールアドレス>
Date:   git commitを実行した日時

    コミットメッセージ2

commit コミットの識別番号
Author: ユーザ名 <メールアドレス>
Date:   git commitを実行した日時

    コミットメッセージ1

```

## 履歴を基に、前の状態に戻す
`git reset コミットの識別番号または位置`

コミットの識別番号の例: a70a1f5667fc78a5feedc62c131a800089f53bd1<br>
位置の例: HEAD^ (1つ前のコミット)

## ローカルリポジトリとリモートリポジトリを紐付ける
二つの方法
1. リモートリポジトリを複製（clone）して、ローカルリポジトリを作成する。
2. ローカルリポジトリを作成してから、リモートリポジトリにpushする。

### 2の方法
`git remote add origin リポジトリURL`<br>
できたか確認は`git remote -v`
```
origin  https://github.com/kkraoi/git-practice.git (fetch)
origin  https://github.com/kkraoi/git-practice.git (push)
```
が表示されればOK。

ちなみに、紐付け時にスペルミス等でリポジトリのURLを誤って入力してしまった場合、ローカルリポジトリとリモートリポジトリの紐付けを変更することで修正。<br>
`git remote set-url origin リポジトリURL`

下記「[ブランチ名初期化](#ブランチ名初期化)」にてブランチ名初期設定。

`git push origin main`で初回プッシュ。
```
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (6/6), 442 bytes | 442.00 KiB/s, done.
Total 6 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/kkraoi/git-practice.git
 * [new branch]      main -> main
```
のようなメッセージが出ればOK

## ブランチ名初期化
反映させる前に、Gitのブランチ名を揃える必要がある。<br>
VSCodeデフォルトローカルリポの`master`からGitHubでのデフォルト`main`へ。<br>
つまり、VSCodeとGitHubのデフォルトブランチ名を統一する。
`git branch -M main`

## Github と　EC2の連携 in VSCode

### EC2インスタンスに接続する in VSCode
1. 左端ツールバーのリモートエクスプローラー（拡張機能Remote-SSH）をクリック
2. 所定のディレクトリを開く 「→」押下

### VSCode と Github を 公開鍵暗号方式で接続する
EC2インスタンスの`ssh-keygen`で公開鍵と秘密鍵を作成。オプションは全部enterでOK

`ls ~/.ssh`で秘密鍵(id_rsa)、公開鍵(id_rsa.pub)ができているか確認

`cat ~/.ssh/id_rsa.pub`で中身を表示、全部コピーしておく

https://github.com/settings/keys
の「New SSH key」をクリック、titleに適当な名前をつけて、id_rsa.pubの中身をコピーし登録

ターミナルにて`ssh -T git@github.com`でGithubと接続。yesで良い
```
Hi *****! You've successfully authenticated, but GitHub does not provide shell access.
    Connection to github.com closed.
```
と表示されればOK

「公開鍵認証は、ユーザー(Cloud9)が秘密鍵、サーバー(GitHub)が公開鍵を持っている」

## TIPS

## コマンド
`ls -a` -aは.から始まる隠しファイル・ディレクトリを表示する

## SSH接続でのgit
1. リモートリポジトリを作成
1. `cd ディレクトリ`
2. `git branch -M main`
3. `git remote add origin リモートリポジトリURL`

## コミットコメント
- 開始 … `[Start]`
- 終了 … `[Finish]`
- 追加 … `[Add]`
- 更新 … `[Update]`
- 削除 … `[Remove]`
- 不具合修正 … `[Fix]`

## Git の "user.name" と "user.email" を構成していることを確認してください。
```
git config --global user.name "名前"
git config --global user.email "メールアドレス"
```
コマンドを実行して、ユーザー名とメールアドレスを登録する必要がある。

```
git config --list
```
で登録されたか確認。

## 作業ブランチを作成する
```
git switch -c <新しいブランチ名>
```
- バージョン2.23.0以降で使える。
- `switch`はブランチを切り替える。
- `-c`で新しいブランチを作り、切り替える。