## 基本情報

| パッケージ名 | ファイルパス | 閲覧時点の最新コミット |
| :---: | :---: | :--- |
| io | src/io/io.go | [7781fed24ea79d819bc0ecfdafe8c24151a83c31](https://github.com/golang/go/commit/7781fed24ea79d819bc0ecfdafe8c24151a83c31) |

## ファイル説明

ioパッケージは、基本的な I/Oプリミティブを提供します。
特に、io.go で行っているのは、各I/Oプリミティブのインターフェース定義が主です。
単純に `Read`関数 や `Write`関数を持つものを `Reader` や `Writer` と定義することで、
様々な入出力を抽象的かつ統合的に扱うことができるようになります。

### エラー定義

ioパッケージで使用する各種エラーを定義しています。

```go
// ErrShortWrite means that a write accepted fewer bytes than requested
// but failed to return an explicit error.
var ErrShortWrite = errors.New("short write")

// ErrShortBuffer means that a read required a longer buffer than was provided.
var ErrShortBuffer = errors.New("short buffer")

// EOF is the error returned by Read when no more input is available.
// Functions should return EOF only to signal a graceful end of input.
// If the EOF occurs unexpectedly in a structured data stream,
// the appropriate error is either ErrUnexpectedEOF or some other error
// giving more detail.
var EOF = errors.New("EOF")

// ErrUnexpectedEOF means that EOF was encountered in the
// middle of reading a fixed-size block or data structure.
var ErrUnexpectedEOF = errors.New("unexpected EOF")

// ErrNoProgress is returned by some clients of an io.Reader when
// many calls to Read have failed to return any data or error,
// usually the sign of a broken io.Reader implementation.
var ErrNoProgress = errors.New("multiple Read calls return no data or error")
```

#### Point of View

`errors.New`関数で新しいエラーを定義できます。`error`型の実態は `Error() string` を持つ interface なので、より高度なエラーを定義したいときには次のように書けます（[原文:Effective Go](https://golang.org/doc/effective_go.html#errors)）。

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

