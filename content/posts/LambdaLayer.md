---
title: "AWS Lambda Layerでつまづいたところメモ"
date: 2024-08-09
categories: ["AWS"]
tags: ["AWS", "Lambda", "python"]
---

AWS Lambdaを使用してPythonスクリプトを実行する際、必要なPythonライブラリをインストールするのに `pip` などのライブラリ管理は利用できません。
ライブラリを自分でビルドし、Lambda Layerに追加することが一般的です。

Lambda Layerに追加するのにいくつかつまづいたところがあるのでメモとして残しておきます。

包括的な手順については、公式のガイドを参照してください。
[Python Lambda 関数にレイヤーを使用する - AWS Lambda](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/python-layers.html)

## Lambda LayerにアップロードするZIPの構造が誤っている

Lambda LayerにアップロードするZIPファイルは以下のディレクトリ構造を守らなければなりません。

`python/lib/python3.x/site-packages/`

この構造に従わない場合、Lambdaがライブラリを認識できず、エラーが発生します。ファイルをZIP化した後にリネームするのは問題ないです。

### 参考スクリプト

[AWS公式のスクリプト](https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/layer-python/layer)

特に特殊なことはやっておらず、中身は以下です。

```bash
python3.12 -m venv create_layer
source create_layer/bin/activate
pip install -r requirements.txt
mkdir python
cp -r create_layer/lib python/
zip -r layer_content.zip python
```

このスクリプトは、仮想環境を作成し、必要なライブラリをインストールした後、適切なディレクトリ構造を持つZIPファイルを生成します。

## 環境の違いによりPythonライブラリが実行できない

上記のLambda Layerを利用したLambda関数を実行する際に特定ライブラリだけ `Module not found` エラーなどが出る場合は、AWS Lambda環境と特定ライブラリの想定環境が異なっている可能性があります。
ライブラリによっては実行OSやCPUアーキテクチャで異なるバイナリが生成されることがあるからです。

AWS LambdaはAWS Linux環境で実行されるため、ライブラリのビルドも同じ環境で行うことが推奨されます。AWS Linux環境をシミュレートするには、AWSが用意したDockerコンテナを使用するのが便利です。また、Lambdaの実行環境がarm64かx86_64かに注意し、ビルド環境を合わせることが重要です。

```bash
docker pull amazonlinux:2023
docker run -it amazonlinux:2023 /bin/bash
```

このコンテナ内で使用予定のバージョンのPythonとPythonライブラリをインストールし、Lambda Layer用のZIPファイルを作成するとエラーが解決します。

## ファイルサイズが上限を超えている

Lambda Layerに登録できるZIPファイルのサイズには制限があります。具体的には、圧縮前のファイルサイズが250MBを超えないように注意する必要があります。これを超えると、Layer登録時にエラーが発生します。

### ライブラリを分割する

大きなライブラリは複数のLayerに分けて登録することで解決します。

登録したいzipファイルの `python/lib/python3.x/site-packages/` 下のフォルダーを適宜分けて、複数のzipファイルとして登録すればOKです。

無論、使用しないライブラリを削除するのも有効です。

なお、1つのLambda関数に登録できるLambda Layerは最大5個までですので、Layerの分割しすぎも注意が必要です。
