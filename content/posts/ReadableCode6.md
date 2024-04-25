---
title: "リーダブルコード6章まとめ"
date: 2022-07-28
categories: ["本まとめ"]
tags: ["リーダブルコード"]
---

## 6章: コメントは正確で簡潔に

**コメントは領域に対する情報の比率が高くなければいけない**

「それ」や「これ」を使わずに、「それ」や「これ」が指している名詞そのものに書き換えたが良い。

関数の動作を正確に記述した方がいい。  
「行数をカウント」ではなく、改行記号を数えているなら「改行記号をカウント」の方が関数の動作に正確である。

慎重に選んだ入出力の実例をコメントに書いておくのも良い。  
どういう処理なのか質問されることを考えて、全てに答えられる実例を選ぶと良い。

コードの動作だけでなく、コードの意図を書く。  
「listを逆順にイテレートする」ではなく、「値段の高い順に表示する」のように、どうしてその動作が必要なのかわかる様に書いた方が良い。

「キャッシュ」「正規化する」のような処理の名前としてよく使われる情報密度の高い言葉を使う。

### 6章の感想
コメントの文章であっても言葉ではあるので、わかりやすく人に伝えられるテクニックが必要であるということだと考えている。このテクニックを使えばコードのコメントでなくても、日常生活の会話の中でわかりやすく人に伝えることができる。例えば、こそあど言葉をなるべく使わないことで混乱を防いだり、うまい例を使うことで説明よりも何倍も早く伝えることができる。
逆に、誰しも日常の中で人にうまく伝えるノウハウは自分の中に溜まっているはずなので、それをコードのコメントにも活かせばいいのだと思っている。