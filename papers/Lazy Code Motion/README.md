# Lazy Code Motion

## Credit

Jens Knoop, Oliver Rüthing, and Bernhard Steffen. 1992. Lazy code motion. In Proceedings of the ACM SIGPLAN 1992 conference on Programming language design and implementation (PLDI '92). Association for Computing Machinery, New York, NY, USA, 224–234. https://doi.org/10.1145/143095.143136

Permission to copy without fee all or part of this material is granted provided that the copias are not made or distributed for direct commercial advantaga, the ACM copyright notica and the title of the publication and its date appear, and notice is given that copying is by permission of the Association for Computing Machinery. To copy otherwise, or to republish, raquires a fee and/or specific permission.



## Summary

Flow graphのcode motionにおいて、必要であるものに限り・なるべく直前にコードを移動するアルゴリズムを提唱する。

アルゴリズム：

- 直前ではないあるいは孤立していないノードの計算を、
- 直前かつ孤立していないノードのエントリに移動する。

このようにすることで、register pressureの少ないcode motionを、しかも（当時の手法に比べて）完全に・計算量を抑えながら実現できる。

## Note

### Code Motion

Code motionとは、再計算を防ぐためのflow graph上の最適化。
ソースコードで例えるならば、次のコード：

```c
for (int i = 0; i < 4; ++i) {
    x = a + b;
    y = x + i;
}
```

を次のように移動する：

```c
x = a + b;
for (int i = 0; i < 4; ++i) {
    y = x + i;
}
```

### The Safe-Earliest Transformation

ノード $n$ について、backward analysisによって得られる次の $\text{D-SAFE(n)}$ のgreatest solutionをD-Safeと言う：

$$ \text{D-SAFE}(n) = n が最後のノードでない \land ( n に再計算を防ぎたい計算が含まれる \lor 値が変わっていない \land \prod_{m \in succ(n)} \text{D-SAFE}(m) ) $$

ノード $n$ について、forward analysisによって得られる次の $\text{EARLIEST(n)}$ のleast solutionをEarliestと言う：

$$ \text{EARLIEST}(n) = n が最初のノードである \lor \sum_{m \in pred(n)} ( 値が変わっている \lor \lnot \text{D-SAFE}(m) \land \text{EARLIEST}(m) ) $$

次の手順でcode motionするアルゴリズムをSafe-Earliest Transformationと定義する：

- 再計算を防ぎたい計算を、
- D-SafeおよびEarliestを満たすノードのエントリに移動する。

しかし、このアルゴリズムでは、不必要に遠くへ移動してしまう。
もっと直前に移動したい。

### The Latest Transformation

ノード $n$ について、forward analysisによって得られる次の $\text{DELAY(n)}$ のgreatest solutionをLatestと言う：

$$ \text{DELAY(n)} = \text{D-SAFE}(n) \land \text{EARLIEST}(n) \lor n が最初のノードでない \land \prod_{m \in pred(n)} \lnot Used(m) \land \text{DELAY}(m) $$

ノード $n$ について、次の述語 $Latest(n)$ を定義する：

$$ \text{Latest(n)} = \text{DELAY}(n) \land ( n に再計算を防ぎたい計算が含まれる \lor \lnot \prod_{m \in succ(n)} \text{DELAY}(m) ) $$

次の手順でcode motionするアルゴリズムをLatest Transformationと定義する：

- 再計算を防ぎたい計算を、
- DelayおよびLatestを満たすノードのエントリに移動する。

しかし、このアルゴリズムでは、一度しか計算されない計算も移動してしまう。
一度しか計算されない計算は移動したくない。

### The Lazy Code Motion Transformation

ノード $n$ について、backward analysisによって得られる次の $\text{ISOLATED(n)}$ のgreatest solutionをIsolatedと言う：

$$ \text{ISOLATED(n)} = \prod_{m \in succ(n)} ( \text{Latest}(m) \lor m に再計算を防ぎたい計算が含まれない \land \text{ISOLATED}(m) ) $$

移動先ノードの集合 $\text{OCP}$ を次のように定義する：

$$ \text{OCP} = \lbrace n | \text{Latest}(n) \land \lnot \text{ISOLATED}(n) \rbrace $$

移動元ノードの集合 $\text{RO}$ を次のように定義する：

$$ \text{RO} = \lbrace n | n に再計算を防ぎたい計算が含まれる \land \lnot ( \text{Latest}(n) \land \text{ISOLATED}(n) ) \rbrace $$ 

次の手順でcode motionするアルゴリズムをLazy Code Motion Transformationと定義する：

- $\text{RO}$ の計算を、
- $\text{OCP}$ の対応するノードのエントリに移動する。



## Impression

私の所属する研究室出の博士の博論を理解するために必要だと言われて読んだ。
理論系の論文を初めて読んだ。
大量の証明が羅列されていて驚いた。
そうしなければならないのは、考えてみれば当然だ。

この論文では新しい定義をするたびに補題を提示し、最終的にLazy Code Motion Transformationが計算的・ライフタイム的に最適であることの証明をしている。
今回は手法を理解すればいいと割り切って、また省略されている証明も多からずあり、証明を深く追わなかった。

原文を読んでも、ChatGPTで翻訳しても、全く理解できない文章が多々あった。
自分の英語力の無さに悲しくなった。
……論文の内容に関係ないな。
