+++
date = '2026-02-07T04:54:23+09:00'
draft = false
title = '3gx・3DS開発の環境構築｜devkitProのインストール手順'
description = '3DS・3gx開発に必要なdevkitProのインストール方法を解説。日本語の解説が少ないため、手順を備忘録としてまとめました。'
tags = ["3DS", "3gx", "devkitPro"]
keywords = ["3gx", "devkitPro", "3DS", "インストール", "環境構築", "開発環境"]
categories = ["開発"]
showToc = true
+++

去年の記事でこんなことを書きました。

> ある程度学習をした今の私が改めてあの頃のコードを見ても、「よくこんなのやってたな」ってぐらい難しいことやってたんですよね。
>
> — [HugoとGitHub Pagesでブログを立ち上げた手順・感想]({{< relref "posts/2025/12/31/myself.md" >}})

この記事を書くきっかけにもなったのですが、高校生の頃に趣味でやっていたことの1つに「3DSのプラグイン（チートプラグイン）を作る」というものがあります。  
簡単に言うと、ゲーム内のメモリを見て値を変更したりしていました（営利目的じゃないし配布もしてないからギリセーフ）。

最近GitHubのアカウントを変えたので、これを機にあの頃に書いたコードをリファクタリングしようと思いました。  
環境構築をしようとしたら、日本語でまとまった記事があんまり見当たらなくて、ブログの記事も増やしたいなってことで、手順を備忘録として残しておくことにしました。

## プラグインとは

めっちゃ簡単に言うと、例えばプレイヤーがゲーム内のお金（以下、ベル）を2000持ってたとしますよね。  
この2000ベルって値を、貧乏だから1億ベルにしちゃおう！ってなるわけです。ゲームのメモリには関数や値がたくさん入ってるので、プレイヤーが所持してるベル数が入ってるアドレスを見つけて、そこのメモリを書き換えちゃえばいい。  
そうすると持ってたベルが1億になって超大金持ち、ってわけです。~~現実でもできたらいいのにな〜~~

普通ならデバッガーとかのツールが必要なんですけど、[CTRPluginFramework](https://gitlab.com/thepixellizeross/ctrpluginframework)（通称: CTRPF）を使うと、こういうメモリの閲覧・変更をプラグイン側からわりと簡単に実装できます。  
で、そのフレームワーク（CTRPF）で書いたコードをビルドするには、devkitProの環境構築が必要ってわけです。

**CTRPF（CTRPluginFramework）の環境構築のやり方**をまとめていきます。OS は Windows 想定です。

---

## 必要なもの

何を用意すればいいか、軽く表にしておきます。

| 項目 | 説明 |
|------|------|
| **devkitPro** | 3DS用のクロスコンパイラ環境（devkitARM が入ってるやつ） |
| **libctrpf** | CTRPluginFramework のライブラリ |
| **3gxtool** | .elf を .3gx に変換するツール |
| **make** | ビルドする用（MSYS2 とかで入れる） |

---

## 手順1: devkitPro を入れる（Windows）

まず devkitPro を入れます。

1. **インストーラを取ってくる**
   - [devkitPro - Getting Started](https://devkitpro.org/wiki/Getting_Started) から **devkitProUpdater** を落とす  
   - または [SourceForge - devkitPro](https://sourceforge.net/projects/devkitpro/) から最新の `devkitProUpdater-*.exe` を取る

2. **インストール**
   - インストーラを実行して、**「Download and Install」** を選ぶ
   - コンポーネントは **devkitARM** を選んでおけばOK（3DS用。devkitPPC とかは不要なら外していい）
   - インストール先は **`C:\devkitPro`** にしとくのを推奨

3. **環境変数**
   - インストーラがだいたい自動で設定してくれる想定
   - 手動で確認するなら:
     - `DEVKITPRO` = `C:\devkitPro`
     - `DEVKITARM` = `C:\devkitPro\devkitARM`
     - `PATH` に `C:\devkitPro\devkitARM\bin` と `C:\devkitPro\tools\bin` が入ってるか確認

4. **make について**
   - Windows には標準で `make` がないので、**MSYS2** を入れて `pacman -S make` で make を入れたシェルでビルドするか、devkitPro インストール時に MSYS2 が入ってれば、その MSYS2 のシェルで `make` を叩く形になります。

---

## 手順2: libctrpf を入れる

Makefile では `CTRPFLIB ?= $(DEVKITPRO)/libctrpf` を参照してるので、**`C:\devkitPro\libctrpf`** にヘッダとライブラリを置く必要があります。

### 方法A: プリビルドを使う場合

CTRPluginFramework の **BlankTemplate** とか **リリース** に同梱されてる **libctrpf** を使うやり方が一般的です。

- 入手先の例:
  - [Nanquitas/CTRPluginFramework-BlankTemplate](https://github.com/Nanquitas/CTRPluginFramework-BlankTemplate) の README や Release
  - 日本語の解説だと「libctrpf と libctru を MediaFire とかから落とす」って書いてあることもあります
- 配置の仕方:
  - 解凍した中に `include` と `lib` があるやつなら、それらを **`C:\devkitPro\libctrpf`** の直下に置いて、
  - 結果として `C:\devkitPro\libctrpf\include` と `C:\devkitPro\libctrpf\lib` になってればOKです。

### 方法B: プロジェクトの asset/install.sh を使う場合（WSL や Git Bash など）

`asset/install.sh` はこんな感じで使う想定です:

```bash
# $1 に libctrpf の bzip2 アーカイブのパスを指定
./asset/install.sh /path/to/libctrpf-xxx.tar.bz2
```

このスクリプトは **`/opt/devkitpro`** を前提にしてるので、**Windows でやる場合は WSL や Git Bash 上で DEVKITPRO を設定する**か、スクリプト内の `DEVKITPRO` を `C:/devkitPro` に合わせて書き換える必要があります。  
実行したあと、Windows から見て `C:\devkitPro\libctrpf`（または WSL なら `/opt/devkitpro/libctrpf`）に `include` と `lib` が展開されてるか確認してください。

---

## 手順3: 3gxtool を入れる

Makefile だとビルドの最後に **3gxtool** で .elf から .3gx を生成するようになってます。

- **置き場所**: `C:\devkitPro\tools\bin\3gxtool.exe` に置いとくと、devkitPro の PATH でそのまま使えることが多いです。
- **入手**:  
  - Luma3DS とか 3gx 関連のリポジトリに同梱されてることがあります。  
  - 検索するなら `3gxtool.exe` とか `3gxtool` + `Luma3DS` / `CTRPluginFramework` とかで探すとよいです。

`3gxtool` がないと、リンクまでは成功しても「.3gx の生成」でこけます。

---

## 手順4: ビルドする

1. **環境の確認**
   - `DEVKITPRO` と `DEVKITARM` が設定されてるシェル（例: MSYS2 の MinGW シェル）を開く。

2. **プロジェクトでビルド**
   ```bash
   cd C:\Users\onetw\Documents\github-projects\vc.3gx
   make
   ```
   クリーンビルドしたい場合:
   ```bash
   make re
   ```

3. **成果物**
   - 成功すると、プロジェクト直下に **`.3gx`** ファイル（Makefile の `TARGET` に由来する名前）ができます。

---

## トラブルシューティング

こけたときに見るといいとこを表にしておきます。

| 症状 | 確認すること |
|------|------------------|
| `DEVKITARM not set` | 環境変数 `DEVKITPRO` / `DEVKITARM` が設定されてるか。使ってるシェルで `echo $DEVKITARM`（または `%DEVKITARM%`）で確認。 |
| `libctrpf` のヘッダやライブラリが見つからない | `C:\devkitPro\libctrpf\include` と `C:\devkitPro\libctrpf\lib` が存在するか。Makefile の `LIBDIRS` で `$(CTRPFLIB)` を参照してる。 |
| `make: command not found` | MSYS2 など make が入ってる環境で実行してるか。`pacman -S make` で入れる。 |
| .3gx ができない / 3gxtool でエラー | `C:\devkitPro\tools\bin\3gxtool.exe` があるか、PATH に `devkitPro\tools\bin` が含まれてるか。 |

---

## まとめ（やること一覧）

1. **devkitPro** を `C:\devkitPro` にインストール（devkitARM を選択）。
2. **libctrpf** の `include` / `lib` を `C:\devkitPro\libctrpf` に配置。
3. **3gxtool** を `C:\devkitPro\tools\bin\3gxtool.exe` とか PATH の通る場所に配置。
4. **make** が使えるシェル（MSYS2 推奨）で `make` または `make re` を実行。