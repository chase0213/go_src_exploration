# golang/goソースコード探索紀

この記事は、golang/go のソースコードを斜め読みすることで、golang の設計理念や、"良い"実装、go言語の振る舞いなどを理解することを目的としています。解釈の違いや明らかな間違いなどの指摘は issue や pull-request をベースにしていただけると幸いです。

## 眺望

golang/go のルート直下には、いくつか重要なディレクトリがあります。

- src
- test
- api
- doc

本記事では主として src ディレクトリ以下を見ていきます。
