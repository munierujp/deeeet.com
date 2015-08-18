+++
date = "2015-08-17T23:47:14+09:00"
title = "1コマンドでkubernetesを立ち上げるboot2kubernetesというツールをつくった"
+++

[tcnksm/boot2kubernetes](https://github.com/tcnksm/boot2kubernetes)

[kubernetes 1.0がリリースされた](http://googlecloudplatform.blogspot.jp/2015/07/Kubernetes-V1-Released.html)．これから実際に試す機会も増えそうなのでDockerを使って簡単に（1コマンドで）kubernetesクラスタを立てるコマンドをつくった．

## デモ

以下はOSX上でシングルNodeのkubernetesクラスタを立てて`kubectl`でリクエストを投げるデモ．

<img src="/images/boot2k8s.gif" class="image">

## 使い方

以下のコマンドでクラスタを立ち上げる．

```bash
$ boot2k8s up
```

このコマンドで必要なDockerイメージがPullされコンテナが起動する．boot2docker上でDockerを動かしている場合ローカルから`kubectl`でクラスタにアクセスするにはport forwardが必要になる．その場合`boot2k8s`はport forwardサーバーも同時に起動する．

終了する（コンテナを削除する）には以下を実行する．

```bash
$ boot2k8s destroy
```

`boot2k8s`が立ち上げたkubernetesコンテナとkubernetesによって立ち上げられたpodを削除することができる．


## インストール

OSXの場合はHomebrewでインストールできる．

```bash
$ brew tap tcnksm/boot2k8s
$ brew install boot2k8s
```

他のプラットフォームの場合は[リリースページ](https://github.com/tcnksm/boot2kubernetes/releases)にバイナリがある．

## 内部の実装

Dockerを使ってシングルNodeのKubernetesクラスタをローカル開発環境に立てる方法はkubernetesの公式が提供している（cf. [Running Kubernetes locally via Docker](https://github.com/kubernetes/kubernetes/blob/release-1.0/docs/getting-started-guides/docker.md)）．基本はこのドキュメントとやっていることは同じで，面倒な部分を含めて全てを1つのコマンドにまとめている．

同じように1コマンドでKubernetesクラスタを立ち上げる試みとして["1 command to Kubernetes with Docker compose"](http://sebgoa.blogspot.jp/2015/04/1-command-to-kubernetes-with-docker.html)がある．これは`docker-compose`を使っている．同様に`boot2k8s`は`docker-compose`のGo言語実装である[libcompose](https://github.com/docker/libcompose)をライブラリとして利用して複数コンテナを立ち上げている（`boot2k8s`のcomposeファイルは[k8s.yml](https://github.com/tcnksm/boot2kubernetes/blob/0.1.0/config/k8s.yml)にあるので`docker-compose`のみを使いたい人はこれをそのまま利用できる）．

`boot2k8s`は[go-bindata](https://github.com/jteeuwen/go-bindata)でcomposeファイルをbyteとして埋め込み`libcompose`で起動する．コードで示すと以下のようになる．

```golang
// Read .yml file as bytes by go-bindata
compose, _ := config.Asset("k8s.yml")

// Setup new docker-compose project
context := &docker.Context{
    Context: project.Context{
        ComposeBytes: compose,
        ProjectName:  "boot2k8s",
    },
}

project, _ := docker.NewProject(context)

project.Up()
```

`boot2k8s`の利点はport forwardingやコンテナの削除機能を持っていることが挙げられる．さらに今後はDocker machineと連携して様々な環境でkubernetesを簡単に立ち上げられるようにする予定．

## 最後に

バグや意見はGitHubの[Issue](https://github.com/tcnksm/boot2kubernetes/issues), もしくは[@deeeet](https://twitter.com/deeeet)までお願いします．

[@rrreeeyyy](https://twitter.com/rrreeeyyy)くんに事前に意見をもらいました．ありがとう．

### 参考

- ["1 command to Kubernetes with Docker compose"](http://sebgoa.blogspot.jp/2015/04/1-command-to-kubernetes-with-docker.html)
- ["Libcompose and our journey with Docker Compose"](http://rancher.com/our-journey-with-docker-compose-and-the-introduction-of-libcompose/)
