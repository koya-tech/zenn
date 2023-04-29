---
title: "【イラストで分かる】windowsで動くDockerの仕組み"
emoji: "🐋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [docker]
published: false
---
# はじめに 
こんにちは。ソフトウェアエンジニアをしています、Koyaです。

筆者がDockerについて調べていると、こんな文言が飛び込んできました。
**「DockerはLinux上でしか動かない」**

私はココで混乱しましたね。

**いやいや、PCのOSはwindowsですが動いてますよ？**
**公式が間違ってるんか？他の記事やサイトが間違ってんのか？**

本気でそう思って、しばらくDockerがよくわからない状態が続きました（笑）

今回はこの問題を解決するために調べた内容をまとめました。
したがって、本記事では特にwindows上で動かす際の仕組みについて説明します。

また筆者は視覚優位な特徴(*1)を持ちます。
同じ視覚優位な特徴を持つ人たちに向けて、わかりやすいように可能な限りイラスト・図を使って説明したいと思います。

よろしくお願いします！

*1 : [物事の理解の仕方は人によって異なると言われています](https://www.edu-c.pref.miyagi.jp/midori/tokushi/tomomanabi/tokuseirikai/tokusei_all.pdf)


# 本題

上述したように、この文言が私を混乱させました😅

**「DockerはLinux上でしか動かない」**

私以外にもこのように感じた人がいたみたいです。安心しました。
https://qiita.com/AA_RU/items/7cce697eea822471840f

では、これを解決するために具体的にDockerの仕組みを深堀していこうと思います。

## DockerはLinux上でしか動かない、とは？
見出しにあるように、大前提としてDockerは**Linux**上でしか動きません！
公式ドキュメントでは、以下のように説明されています。
「Linuxカーネル活用」＝「Linux上でしか動かない」に近い意味と思っています。
>Docker は Go プログラミング言語 で書かれており、Linux カーネルの機能をうまく活用して、さまざまな機能性を実現しています。　[Docker-docs-jaより](https://docs.docker.jp/get-started/overview.html#:~:text=%E5%89%8A%E9%99%A4%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%99%E3%80%82-,%E5%9F%BA%E7%A4%8E%E6%8A%80%E8%A1%93,-Docker%20%E3%81%AF%20Go)


## ハイパーバイザー, WSLで解決
まず大前提の「DockerはLinux上でしか動かない」、ココは変わりません。
では、なぜWindows上でDockerを動作させることができるのでしょうか？

これを解決するには２つの技術が絡んできます。
1. ハイパーバイザー
1. wsl

ひとつずつ説明します。
#### ハイパーバイザー
>ハイパーバイザーとは、仮想マシン（VM）の作成と実行を担うプロセスです。（中略）2つのタイプがあります。
>1つはホストハードウェア上で直接動作する「ベアメタル」と呼ばれるハイパーバイザーです。
>もう1つは「ホスト型」と呼ばれ、通常のコンピュータープログラムのように、オペレーティングシステム上でソフトウェアレイヤーとして動作します。   [vmware公式](https://www.vmware.com/jp/topics/glossary/content/hypervisor.html)

イメージはこんな感じです。
![](/images/9ba0df8ca3e4d0/hyper-type.png =500x)

違いはハイパーバイザーが入る位置です。
windows上でDockerを構築する際は「ベアメタル型」が使用されています。
※ホスト型でもできると思いますが、一般的にベアメタル型が多いと思います。

ここで重要なのはこの文言「**ホストのハードウェア上で直接動作する**」です。
普段のPCでは、ハード→カーネル→OS→APPですが、ハードの上にハイパーバイザーが入ることで、複数のOSを構築できるようになります。
これにより、私たちのwindowsPCで、windowsとLinuxといった他のOSを動かせられるようになるわけです。

ただし、これではまだLinuxは動かせません。Linuxのカーネルが無いからです。
ここでWSLの出番です。

![](/images/9ba0df8ca3e4d0/hyper-loc.png =500x)

:::message
ハイパーバイザー＝Hyper-vではありません。
ハイパーバイザー"を可能にするMicrosoftの製品名が"Hyper-v"です。
他にも、VMware社の製品"vSphere"、Linuxで使われるKVM（Kernel-based Virtual Machine）などがあります。
:::

#### WSL
>Linux 用 Windows サブシステムを使用すると、開発者は、従来の仮想マシンまたはデュアルブート セットアップのオーバーヘッドなしで、ほとんどのコマンド ライン ツール、ユーティリティ、アプリケーションを含む GNU/Linux 環境を変更せずそのまま Windows 上で直接実行できます。  [Microsoft公式](https://learn.microsoft.com/ja-jp/windows/wsl/about)

簡単に要約すると、「開発者向けに作られたwindows上でもLinux環境を準備できるツール」です。

WSL1とWSL2があります。詳細は省きますが、簡単に言うと
・WSL1 → Linuxカーネルに似た動きをするもの
・WSL2 → Linuxカーネルとほぼ同じ

現在ではWSL2が主流だと思いますので、WSL=WSL2という前提で話を進めます。

WSL2をインストールすることで、Linux用のカーネルを装備でき、ハイパーバイザー上で動くOSの一つをLinuxにすることができます。
これでwindowsPCでも、前提である「DockerはLinux上で動く」ための条件を満たしましたね。
:::message
WSL2では、デフォルトでubuntuもインストールされます
:::

![](/images/9ba0df8ca3e4d0/hyper-container.png =500x)


ここでこんな疑問を持った人がいるかもしれません。
・**そもそもHyper-vで複数OSを構築できるんでしょ？WSLっている？**

こちらの答えは、以下のサイトがわかりやすくまとめていたので参考にしてみてください。
簡単に言うと、
1. WSLはWindowsとLinux環境との相互アクセスが可能。（Hyper-vはできない）
1. WSLは構築が用意。（Hyper-vは1から実施。WSLより手間）

などがあります。

開発者として、何かしらサクッとLinux環境を準備したいときに便利です。

https://tooljp.com/windows/chigai/html/WSL/WSL-Linux-on-HypverV-chigai.html



# まとめ
最終的には以下のイラストのようになります。
ハイパーバイザーを使用することで、異なるOSを起動できる土台ができ、WSLをインストールすることでLinuxが動かせるようになるわけです。
![](/images/9ba0df8ca3e4d0/final-model.png =500x)

---

# 参考
・https://www.edu-c.pref.miyagi.jp/midori/tokushi/tomomanabi/tokuseirikai/tokusei_all.pdf
・https://docs.docker.jp/get-started/overview.html#isolated
・https://udemy.benesse.co.jp/development/system/docker.html
・https://zenn.dev/longbridge/articles/d9f544f5b4cb82
・https://docs.docker.com/engine/install/
・https://www.vmware.com/jp/topics/glossary/content/hypervisor.html
・https://learn.microsoft.com/ja-jp/windows/wsl/about