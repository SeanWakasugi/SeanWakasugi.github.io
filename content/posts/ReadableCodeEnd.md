---
title: "リーダブルコード読了感想"
date: 2022-10-28
categories: ["本まとめ"]
tags: ["リーダブルコード", "swift"]
---

読みやすいコードを書くための本として一番おすすめされている本なので購入しました。

私は複数人でコードを共有するような開発の経験が浅く、コメントの書き方とか命名とかに悩んでいたのでそれを解決したいと考えていました。

読み終わってみてわかったことは、読みやすいコードとはコメントや命名だけでなく、コードの構造も含まれるし、実装のロジックも含まれるということがわかりました。


この本では、私が最初に改善していたいと思っていた「表面上の改善」から始まって、「ループとロジックの単純化」でコードの中身やロジックを修正し、「コードの再構成」でそもそも何をコードに書くのか、どこに書くのかという根本まで迫っていく3部構成になっています。

それぞれの章でとても具体的な改善例と改善するためにどう考えればいいのかという思考プロセスが示されており、すぐに実行に移したいと思います。


また、本の最後で解説として（本の作者と別の方が）

読みやすいコードを書くということはたまに思い出してやることではない。習慣にして常に実践することなんだ。 

と述べられています。

これを胸に、リーダブルコードをたまに見返すのではなく、何かコードを書くたびにこれは読みやすいコードだろうか？　と自問自答するようにします。