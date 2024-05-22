# RegCPython: A Register-based Python Interpreter for Better Performance

## Credit

Qiang Zhang, Lei Xu, and Baowen Xu. 2022. RegCPython: A Register-based Python Interpreter for Better Performance. ACM Trans. Archit. Code Optim. 20, 1, Article 14 (March 2023), 25 pages. https://doi.org/10.1145/3568973

Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not made or distributed for profit or commercial advantage and that copies bear this notice and the full citation on the first page. Copyrights for components of this work owned by others than the author(s) must be honored. Abstracting with credit is permitted. To copy otherwise, or republish, to post on servers or to redistribute to lists, requires prior specific permission and/or a fee. Request permissions from permissions@acm.org.


## Summary

研究で行われたこと：

1. CPythonをregister-basedに換装したRegCPythonを開発
2. RegCPythonとCPythonやPyPyとを比較

RegCPythonの売り：

- CPythonと完全に互換性がある
- ASTから直接IRにする
- register-based

CPythonとの比較結果：

- P/Vが大きい
- 最大1.287倍、最小0.977倍、平均1.06倍(x86)あるいは1.068倍(ARM)速い
- メモリ効率がほんの少し悪い
- コンパイル時間がほんの少し速い
- コンパイル結果の容量が少し大きい
- 実装がそれほど難しくない


## Note

### 実装方法

RegCPythonはグローバル変数、一時変数、ローカル変数をすべて同じ配列(レジスタ配列)に割り当てるため、ぱっと見ただけではインデックスが決まらない。そのため、DFSでレジスタ割当てを行う。TVC(AST上のスコープに由ってインクリメント・デクリメントされるカウンタ)によって一時変数のインデックスを決めることでメモリを再利用する。

1命令は、命令の種類(1byte)、引数(1byte)が3個で4bytes固定。可変長にするとパースにコストがかかるため不採用にしたらしい。

一般的にはSSA形式を用いると良いらしいが、Pythonの仕様上やレジスタ配列の実装の都合上使えなかったらしい。

### テストケース

テストケースにpyperformanceというものを使っている。純粋にPythonだけのコードがあれば、Cライブラリを呼び出すためだけにPythonが書かれたコードもある。つまり、P/Vを見ると前者は小さく後者は大きい。P/Vが小さいほどregister-basedに換装した恩恵を得ている(そりゃそう)。

ただし、P/VとはP(Physical instructions)(ネイティブな命令語数)とV(Virtual instructions)(VMに対する命令語数)の比のこと。

### なぜ速い

stack-basedなIRに含まれるLOAD_CONST、LOAD_FAST、STORE_FAST、POP_TOP、ROT_TWO等の命令をregister-basedは減らせる、ひいては機械語命令数を減らせるため速い。

JVMの先行研究曰くRegister-basedはキャッシュミスが多いらしいが、CPythonとRegCPythonでは大した差がない。

RegCPythonは分岐予測ミスが多いが、大した影響はない。

### JITコンパイラで良くね?

JITコンパイラは起動時やcold code実行時にコストがかかる。実際、CPython(stack-based)に比べてPyPy(stack-based & JIT compile)は最大40.372倍速いが、最悪0.102倍、平均1.938倍(x86)1.916倍(ARM)となり、必ずしも良い選択であるとは言えない(リスクが大き過ぎる)。

### 他

PGO(Profile-Guided Optimization)をかけるとCPythonでもRegCPythonでも同一コードのビルド結果が変わる。x86ではCPythonが、ARMではRegCPythonが優勢になる。

JVMの先行研究曰く、register-basedは平均26.5倍速くなる。本研究者はJVMは低レイヤ、RegCPythonは高レイヤだから仕方ないとしている。


## Impression

なんでもかんでもJITコンパイルにすればいいというわけではないことを示していて勉強になった。実装が案外簡単そうで意外だった。

Pythonの言語仕様(リフレクション)のせいで妥協した点があるように思えた。Register-based特化な言語仕様を考えるのも面白いかもしれない。
