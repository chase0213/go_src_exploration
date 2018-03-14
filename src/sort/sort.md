## 基本情報

| パッケージ名 | ファイルパス | 閲覧時点の最新コミット |
| :---: | :---: | :--- |
| sort | src/sort/sort.go | [3caa02f603fcb895763f2f5c3f737ef69fa9cf0a](https://github.com/golang/go/commit/3caa02f603fcb895763f2f5c3f737ef69fa9cf0a) |

## ファイル説明

sort.go は、任意の型へソート機能を提供するファイルです（[ソースコード](https://github.com/golang/go/blob/master/src/sort/sort.go)）。
ソーティングアルゴリズムには、安定なソートと不安定なソートとがあり、このファイルではそのどちらも実装されています。
本記事は go のソースコードを読み解くことを目的としており、実装されているアルゴリズムに説明を与えるものではないため、ソーティングアルゴリズムの詳細については割愛します。

## ソースコード・リーディング

### Interface型

sort.go の肝となるのは、ファイル冒頭で定義されている Interface型です。
予約語である interface と紛らわしいですが、こちらはユーザー定義の型で、
`Len`、`Less`、`Swap` の 3つの関数を持つようなインターフェースとして定義されています。

```go
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```

#### Point of View

多くのソーティングアルゴリズムは、ソート対象の配列の長さと、配列中の任意の 2要素の大小関係、それから任意の 2要素の交換処理で記述することができます。従って、それらを満たすような集合であれば、仮に他の演算が定義されないような集合でも、ソートを行うことができます。
例えば通常、文字列の集合では 2つの単語間で積を定義することはしませんが、配列の長さは集合の大きさ（要素数）、大小関係は辞書式順序関係、交換処理は集合中のインデックスの入れ替えで定義できるため、文字列の集合に対するソートを定義できます。
これを計算機に扱える形に抽象化したものが、ここで定義されている Interface型です。

### insertionSort関数

`insertionSort`関数は、挿入ソートを実装します。
特筆すべきところはないですが、第一引数として `data Interface` と受け取っている点に注目です。
先述の通り、Interface型は `Less`、`Swap`関数を定義しています。

```go
func insertionSort(data Interface, a, b int) {
	for i := a + 1; i < b; i++ {
		for j := i; j > a && data.Less(j, j-1); j-- {
			data.Swap(j, j-1)
		}
	}
}
```

### Sort関数

`Sort`関数は、不安定なソートを提供します。

```go
func Sort(data Interface) {
	n := data.Len()
	quickSort(data, 0, n, maxDepth(n))
}
```

#### Point of View

現時点では、最も速く実装の簡単なクイックソートが内部的に使用されています。
この関数の直下でクイックソートを実装してしまうと、もしクイックソートよりも高速なアルゴリズムに変更したいとなった場合、可読性やテストの観点で不都合が生じるため、実際のソーティングアルゴリズムはプライベート関数として外出しにして、それを呼び出すだけという実装になっています。

### Stable関数

`Stable`関数は、安定なソートを提供します。

```go
func Stable(data Interface) {
	stable(data, data.Len())
}

func stable(data Interface, n int) {
	blockSize := 20 // must be > 0
	a, b := 0, blockSize
	for b <= n {
		insertionSort(data, a, b)
		a = b
		b += blockSize
	}
	insertionSort(data, a, n)

	for blockSize < n {
		a, b = 0, 2*blockSize
		for b <= n {
			symMerge(data, a, a+blockSize, b)
			a = b
			b += 2 * blockSize
		}
		if m := a + blockSize; m < n {
			symMerge(data, a, m, n)
		}
		blockSize *= 2
	}
}
```

#### Point of View

現時点では、挿入ソートとマージソートを組み合わせたアルゴリズムが内部的に使用されています。
`symMerge`関数は、ソート済みの部分配列を SymMergeアルゴリズムによって高速にマージする関数です。
Pok-Son Kim と Arne Kutzer によって "Stable Minimum Storage Merging by Symmetric Comparisons" という論文で提案されています。
詳しくは、ソースコードのコメントである下記を参照してください。

```go
// SymMerge merges the two sorted subsequences data[a:m] and data[m:b] using
// the SymMerge algorithm from Pok-Son Kim and Arne Kutzner, "Stable Minimum
// Storage Merging by Symmetric Comparisons", in Susanne Albers and Tomasz
// Radzik, editors, Algorithms - ESA 2004, volume 3221 of Lecture Notes in
// Computer Science, pages 714-723. Springer, 2004.
//
// Let M = m-a and N = b-n. Wolog M < N.
// The recursion depth is bound by ceil(log(N+M)).
// The algorithm needs O(M*log(N/M + 1)) calls to data.Less.
// The algorithm needs O((M+N)*log(M)) calls to data.Swap.
//
// The paper gives O((M+N)*log(M)) as the number of assignments assuming a
// rotation algorithm which uses O(M+N+gcd(M+N)) assignments. The argumentation
// in the paper carries through for Swap operations, especially as the block
// swapping rotate uses only O(M+N) Swaps.
```

### 各ユーティリティ関数

sort.go の中には、`func (p IntSlice) Sort() { Sort(p) }` といったように、いくつかの便利なメソッドが定義されています。
これにより、例えば int型のスライス `a` に対して、`a` の型を基にすること無く、`a.Sort()` という書き方ができるようになります。

### その他

sort.go の内容は、go特有の記述というよりも、論文を基にしたアルゴリズムの実装がメインとなっています。
冒頭に示した通り、本記事はアルゴリズムの説明に重きを置くものではないため、それらの説明は省きますが、
このように論文をエビデンスとして実装された関数やファイルは、issue を書いたり pull-request を送ったりするときに参考になります。
