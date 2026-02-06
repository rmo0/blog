+++
date = '2026-02-07T04:54:23+09:00'
draft = true
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

環境
---

## 必要なもの

| 項目 | 説明 |
|------|------|
| **devkitPro** | 3DS用のクロスコンパイラ環境（devkitARM が入ってるやつ） |
| **libctrpf** | CTRPluginFramework のライブラリ |
| **3gxtool** | .elf を .3gx に変換するツール |
| **make** | ビルドする用（MSYS2 とかで入れる） |

---

## 手順1: devkitPro を入れる（Windows）

- [devkitPro - Getting Started](https://devkitpro.org/wiki/Getting_Started) か [SourceForge - devkitPro](https://sourceforge.net/projects/devkitpro/) から **devkitProUpdater**（`devkitProUpdater-*.exe`）を落とす
- インストーラを実行して **「Download and Install」** を選ぶ。コンポーネントは **devkitARM** でOK（3DS用。devkitPPC は不要なら外していい）。インストール先は **`C:\devkitPro`** にしとく
- 環境変数はインストーラが自動で入れてくれる。自分で確認するなら `DEVKITPRO` = `C:\devkitPro`、`DEVKITARM` = `C:\devkitPro\devkitARM`、あと `PATH` に `C:\devkitPro\devkitARM\bin` と `C:\devkitPro\tools\bin` が入ってるか
- `make` は Windows にないので、**MSYS2** を入れて `pacman -S make` したシェルでビルドする。devkitPro インストールで MSYS2 が入ってれば、そのシェルで `make` 叩けばいい

---

## 手順2: libctrpf を入れる

Makefile が `CTRPFLIB ?= $(DEVKITPRO)/libctrpf` を参照してるので、**`C:\devkitPro\libctrpf`** にヘッダとライブラリを置く。

[CTRPluginFramework-BlankTemplate](https://github.com/Nanquitas/CTRPluginFramework-BlankTemplate) の Release とか、日本語の解説で「libctrpf を MediaFire から落とす」ってあるやつを取ってくる。解凍して `include` と `lib` があるやつを **`C:\devkitPro\libctrpf`** の直下に置いて、`C:\devkitPro\libctrpf\include` と `C:\devkitPro\libctrpf\lib` になってればOK。

---

## 手順3: 3gxtool を入れる

Makefile がビルドの最後に **3gxtool** で .elf から .3gx を吐くようになってる。`C:\devkitPro\tools\bin\3gxtool.exe` に置いとけば devkitPro の PATH でそのまま使える。Luma3DS とか 3gx 関連のリポジトリに同梱されてることがあるので、`3gxtool.exe` で探す。

ないとリンクまでは通っても .3gx の生成でこける。

---

## 手順4: ビルドする

`DEVKITPRO` と `DEVKITARM` が通ってるシェル（MSYS2 の MinGW シェルとか）で、プロジェクトのディレクトリに移動して `make`。クリーンからやりたければ `make re`。

```bash
cd （プロジェクトのパス）
make
```

成功するとプロジェクト直下に **`.3gx`** ができる（Makefile の `TARGET` の名前）。

---

## トラブルシューティング

| 症状 | 確認すること |
|------|------------------|
| `DEVKITARM not set` | 環境変数 `DEVKITPRO` / `DEVKITARM` が設定されてるか。使ってるシェルで `echo $DEVKITARM`（または `%DEVKITARM%`）で確認。 |
| `libctrpf` のヘッダやライブラリが見つからない | `C:\devkitPro\libctrpf\include` と `C:\devkitPro\libctrpf\lib` が存在するか。Makefile の `LIBDIRS` で `$(CTRPFLIB)` を参照してる。 |
| `make: command not found` | MSYS2 など make が入ってる環境で実行してるか。`pacman -S make` で入れる。 |
| .3gx ができない / 3gxtool でエラー | `C:\devkitPro\tools\bin\3gxtool.exe` があるか、PATH に `devkitPro\tools\bin` が含まれてるか。 |

---

## まとめ

1. devkitPro を `C:\devkitPro` にインストール（devkitARM）
2. libctrpf の `include` / `lib` を `C:\devkitPro\libctrpf` に置く
3. 3gxtool を `C:\devkitPro\tools\bin\3gxtool.exe` に置く
4. MSYS2 とかで `make` か `make re`