# Renamingless Capture-Avoiding Substitution for Definitional Interpreters

## Credit

Casper Bach Poulsen. Renamingless Capture-Avoiding Substitution for Definitional Interpreters. In Eelco Visser Commemorative Symposium (EVCS 2023). Open Access Series in Informatics (OASIcs), Volume 109, pp. 2:1-2:10, Schloss Dagstuhl – Leibniz-Zentrum für Informatik (2023)
https://doi.org/10.4230/OASIcs.EVCS.2023.2

© Casper Bach Poulsen; licensed under Creative Commons License CC-BY 4.0


## Summary

アルファ変換を行わないラムダ計算の実装方法2種類を紹介する論文。

- 簡単な区切り文字を用いる方法
- スコープを考慮した接頭辞を用いる方法


## Note

2種類とも新しい手法ではなく、認知度を高めるための論文。

アルファ変換を行わなければ、自由変数が誤って束縛されてしまう。
かといって、アルファ変換には次の問題がある。

- 探索が重い
- 変数名が元から変わるためエラー表示がわかりづらくなる

### 簡単な区切り文字を用いる方法

代入のたびに代入する値を区切る。

区切られた式を評価する際に区切り文字を取り払う。

実行コストが小さい。
一方、区切られた式が最外で評価されることを前提としているため、そうでないラムダ式には矛盾が生じる。

### スコープを考慮した接頭辞を用いる方法

代入のたびに自由変数に接頭辞を付ける。
接頭辞は同名かつスコープが内側であるほど増える。

矛盾が生じない。
一方、実行コストが重い。


## Impression

JavaやPythonやのインタプリタの高速化について知りたかったため、目的と違う分野の論文であった。

しかし、もし関数型言語の最適化について研究することがあれば、役に立つかもしれない。
