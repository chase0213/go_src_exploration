## 基本情報

| パッケージ名 | ファイルパス | 閲覧時点の最新コミット |
| sort | src/sort/search.go | [af2ac479fc9e4833357a3281a67cd517d2da07ad](https://github.com/golang/go/commit/af2ac479fc9e4833357a3281a67cd517d2da07ad) |

## ファイル説明

search.go はバイナリーサーチを実装します（[原文](https://github.com/golang/go/blob/master/src/sort/search.go#L5)）。
`Search` は区間[0, n) において `f(i)` が真（true）となる最小のインデックス値 `i` を返します。ただし、`f(i+1)` が真（true）という仮定をおきます。

## ソースコード・リーディング

### Search関数

search.go の主幹となる関数は、`Search(n int, f func(int) bool) int` です（[ソースコード](https://github.com/golang/go/blob/master/src/sort/search.go#L59-L74)）。

```go
func Search(n int, f func(int) bool) int {
	// Define f(-1) == false and f(n) == true.
	// Invariant: f(i-1) == false, f(j) == true.
	i, j := 0, n
	for i < j {
		h := int(uint(i+j) >> 1) // avoid overflow when computing h
		// i ≤ h < j
		if !f(h) {
			i = h + 1 // preserves f(i-1) == false
		} else {
			j = h // preserves f(j) == true
		}
	}
	// i == j, f(i-1) == false, and f(j) (= f(i)) == true  =>  answer is i.
	return i
}
```

#### Point of View

`Search`関数に `f func(int) bool` という関数を引数にとる（依存性注入: DI）ことで、int型を引数にとる任意の bool関数を与えて探索ができるように実装されています。

```go
func Search(n int, f func(int) bool) int {
```

これにより、後述する関数が非常に簡潔に書けるようになり、可読性の向上や、ソースコードのサイズ節約の効果が見込まれます。
この引数 `f` は、二分探索を行う際の評価関数として使われており、`f` の引数である int型の値は、スライスのインデックス値です。
この関数 `f` だけだとどう使うか想像しにくいですが、例えば次の `SearchInts`関数などの実装を見ると、この実装が非常に優れていることがわかります。

具体的に `Search`関数で行われる処理について見ていきます。
取り急ぎここでは天下り的に、`f` は探索中の値が、現在見ている箇所よりも左にあれば `false`、右にあれば `true` を返す関数と考えてください。

まず、`h` の定義式（更新式）についてです。

```go
h := int(uint(i+j) >> 1) // avoid overflow when computing h
```

`i` と `j` は探索中の左右のポインタです。
二分探索のアルゴリズムに則って、`h` を `i` と `j` の間の中央値として定義したいのですが、

```go
h := (i+j) / 2
```

としてしまうと、2つの問題が生じます。
1つは、`i`、`j` が共に int型の最大値の場合、`i+j` がオーバーフローするという点です。
これを解決するために、符号なし整数型である uint にキャストして、1桁余分に確保しています。
ここで、最終的な結果が uint になってしまうとインデックスの型が異なってしまうので、
最後に int型に再びキャストし直しています。

```go
h := int(uint(i+j) / 2)
```

2つ目は、`/ 2` という演算結果が混乱を招きやすいという点です。
go言語（ver. 1.9）においては、整数型を整数型で除した場合、結果も整数型となり、小数点以下は切り捨てとなります。
しかし、この動作はプログラミング言語やバージョンによって異なる場合があり、バグの温床になります。
ここで、1ビットの右ビットシフト `>> 1` を行うことで、実際には `/ 2` と同じ（ような）演算を施しながら、小数点以下が切り捨てであり、型が変わらないということを明示することができます。

```go
h := int(uint(i+j) >> 1)
```

このインデックスの中央値 `h` と判定関数 `f` を用いて、`f(h)` の真偽を見ることで、二分探索を行っています。


### SearchInts関数

`SearchInts(a []int, x int) int` は、int型のスライスから値を二分探索するための関数です（[ソースコード](https://github.com/golang/go/blob/master/src/sort/search.go#L83-L85)）。

```go
func SearchInts(a []int, x int) int {
	return Search(len(a), func(i int) bool { return a[i] >= x })
}
```

#### Point of View

前述の `Search`関数を利用することで、非常に簡潔に実装されています。
`SearchInts` が内部的に呼んでいる `Search`関数の第二引数をピックアップしてみます。

```go
func (i int) bool {
  return a[i] >= x
}
```

`a` は `SearchInts`関数の引数で、int型のスライスです。従ってこの無名関数は、`SearchInts`関数に与えられたスライスの `i`番目の要素 `a[i]` と、探索中の数値 `x` とを比較して、`a[i]` の方が大きいか、または等しければ真（`true`）、そうでなければ偽（`false`）を返します。
これを `Search`関数の判定関数として注入することで、int型のスライスに対して二分探索を実装します。

### SearchFloat64s関数

実装の方針は前述の `SearchInts`関数と同じため、詳細は割愛しますが、引数として与えるスライスと探索値がそれぞれ `[]float64`、`float64` に変更されています。これにより、float64型の `Search`関数を実装しています。

```go
func SearchFloat64s(a []float64, x float64) int {
	return Search(len(a), func(i int) bool { return a[i] >= x })
}
```

### SearchStrings関数

こちらも実装の方針は前述の `SearchInts`関数と同じため、詳細は割愛しますが、引数として与えるスライスと探索値がそれぞれ `[]string`、`string` に変更されています。これにより、string型の `Search`関数を実装しています。

```go
func SearchStrings(a []string, x string) int {
	return Search(len(a), func(i int) bool { return a[i] >= x })
}
```

### 各スライス型への組み込み

それぞれの型に対する `Search`関数は前述の通り `Search*` として実装されています。
しかし、このままだとユーザーは常にスライスがどの型なのか意識して `Search*`関数を呼び出さなければならず、あまり利便性が高くありません。
これに対するアプローチとして、go言語では各スライス構造体へ関数を追加する術を提供しています。

```go
// Search returns the result of applying SearchInts to the receiver and x.
func (p IntSlice) Search(x int) int { return SearchInts(p, x) }

// Search returns the result of applying SearchFloat64s to the receiver and x.
func (p Float64Slice) Search(x float64) int { return SearchFloat64s(p, x) }

// Search returns the result of applying SearchStrings to the receiver and x.
func (p StringSlice) Search(x string) int { return SearchStrings(p, x) }
```

#### Point of View

実際に各関数を見ると分かる通り、関数名の前に `(p IntSlice)` などの宣言が含まれています。
これは、`p` の関数（メソッド）として `Search` を追加するという文で、`p` が intのスライスや float64のスライス、stringのスライスであっても共通して、

```go
p.Search(x)
```

というように呼び出せることを意味しています。
これでユーザーは、`p` がどの型のスライスなのか意識しなくても、二分探索を使うことができます（実際には探索値`x` と `p` のスライス型が一致する必要があるため、全く意識しなくて良いわけではないですが、コードを読む際には `Search` が二分探索であるということだけを知っていれば良いので、理解にかかるコストが小さくできます）。
