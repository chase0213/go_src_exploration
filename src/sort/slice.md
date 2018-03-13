## 基本情報

| パッケージ名 | ファイルパス | 閲覧時点の最新コミット |
| :---: | :---: | :--- |
| sort | src/sort/slice.go | [3caa02f603fcb895763f2f5c3f737ef69fa9cf0a](https://github.com/golang/go/commit/3caa02f603fcb895763f2f5c3f737ef69fa9cf0a) |

## ファイル説明

slice.go では、任意のスライスに対して与えられた `less`関数による評価を基にソートした結果を返す `Slice`関数を提供します（[ソースコード](https://github.com/golang/go/blob/master/src/sort/slice.go)）。
`Slice`関数は安定なソートではなく、安定なソートを提供する関数として `SliceStatble`関数が提供されています。また、スライスがソート済みか否かを返す `SliceIsSorted`関数も提供されています。

## ソースコード・リーディング

### Slice関数

`Slice(slice interface{}, less func(i, j int) bool)` は、任意のスライスを対象として、与えられた `less`関数に従ってソートする関数です（[ソースコード](https://github.com/golang/go/blob/master/src/sort/slice.go#L17-L22)）。

```go
func Slice(slice interface{}, less func(i, j int) bool) {
	rv := reflect.ValueOf(slice)
	swap := reflect.Swapper(slice)
	length := rv.Len()
	quickSort_func(lessSwap{less, swap}, 0, length, maxDepth(length))
}
```

#### Point of View

[`reflect`パッケージ](https://golang.org/pkg/reflect/)を使用することで、任意の型のスライスについて同様のインターフェースを提供しています。

`Slice`関数は最終的に `quickSort_func`関数でスライスをクイックソートした結果を返すため、クイックソートに必要な `less`関数と `swap`関数、スライスの長さをそれぞれ用意しています。ソート対象のスライス自体は渡していませんが、`swap`関数は対象のスライスへのポインタを使用してそのスライス自体を変更するため、問題ありません（[Swapperの実装](https://golang.org/src/reflect/swapper.go?s=337:383#L3)）。また、`Swapper`関数は、引数がスライスでない場合は panic を起こします。従って、この `Slice`関数も同様に引数がスライスでなければ panic を発生します。

### SliceStable関数

`SliceStable(slice interface{}, less func(i, j int) bool)`は、`Slice`関数と同様に任意のスライスに対してソートを行いますが、`Slice`関数との違いは、その名の通り安定的なソートを提供することです。

```go
func SliceStable(slice interface{}, less func(i, j int) bool) {
	rv := reflect.ValueOf(slice)
	swap := reflect.Swapper(slice)
	stable_func(lessSwap{less, swap}, rv.Len())
}
```

### SliceIsSorted関数

`SliceIsSorted(slice interface{}, less func(i, j int) bool) bool`は、引数として与えたスライスがソート済みかどうかを判定して、真偽値（true/false）を返します。

```go
func SliceIsSorted(slice interface{}, less func(i, j int) bool) bool {
	rv := reflect.ValueOf(slice)
	n := rv.Len()
	for i := n - 1; i > 0; i-- {
		if less(i, i-1) {
			return false
		}
	}
	return true
}
```
