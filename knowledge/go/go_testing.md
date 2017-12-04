go测试
====
来自 http://lanlingzi.cn/post/technical/2016/0824_go_testing/

Go语言内置了测试框架，编写单元测试非常方便。

### 1 命名约定

测试代码位于以_test.go结尾的源文件中，一般与源码在同一个package中。

位于同一个package中的主要原因是：测试可以访问package中不可导出的变量，方法等元素。

测试源码可以修改package名，带上_test结尾

修改的package名称，不需要再单独新建立目录，也与源码在一个目录下。参考标准库的bytes中的测试代码，方便使用被测试的元素，可以采用.来import测试的package：

```go
package bytes_test

import (
    . "bytes"
    "io"
    ...
)
```
以Test开头的功能测试函数

以Benchmark开头的性能测试函数

以Example开头的样例代码

```go
func ExampleHello() {
    hl := hello()
    fmt.Println(hl)
    // Output: hello.
}
```
示例函数无需接收参数，但需要使用注释的Output: 标记说明示例函数的输出值，未指定Output:标记或输出值为空的示例函数不会被执行。go doc工具会解析示例函数的函数体作为对应包/函数/类型/类型方法的用法。

示例函数需要归属于某个包/函数/类型/类型的方法，具体命名规则如下：

```
func Example() { ... }      // 包的示例函数
func ExampleF() { ... }     // 函数F的示例函数
func ExampleT() { ... }     // 类型T的示例函数
func ExampleT_M() { ... }   // 类型T的M方法的示例函数

// 多示例函数 需要跟下划线加小写字母开头的后缀
func Example_suffix() { ... }
func ExampleF_suffix() { ... }
func ExampleT_suffix() { ... }
func ExampleT_M_suffix() { ... }
```

## 2 测试类型

### 2.1 功能测试

功能测试函数以*testing.T类型为单一参数t，testing.T类型用来管理测试状态和格式化测试日志。测试日志在测试执行过程中积累，完成后输出到标准错误输出。

常用方法:

测试预期不符，使用t.Error()或t.Errorf()记录日志并标记测试失败

```go
func TestToTitle(t *testing.T) {
    for _, tt := range ToTitleTests {
        if s := string(ToTitle([]byte(tt.in))); s != tt.out {
            t.Errorf("ToTitle(%q) = %q, want %q", tt.in, s, tt.out)
        }
    }
}
```

测试预期不符，使用t.Fatal()和t.Fatalf()跳出该测试函数

```go
func TestUnreadByte(t *testing.T) {
    b := new(Buffer)
    b.WriteString("abcdefghijklmnopqrstuvwxyz")

    _, err := b.ReadBytes('m')
    if err != nil {
        t.Fatalf("ReadBytes: %v", err)
    }
    ...
}
```
记录日志， 使用t.Log()和t.Logf()
```go
func TestFowler(t *testing.T) {
    files, err := filepath.Glob("testdata/*.dat")
    if err != nil {
        t.Fatal(err)
    }
    for _, file := range files {
        t.Log(file)
        testFowler(t, file)
    }
}
```
跳过某条测试用例，使用t.Skip()和t.Skipf()
```go
func TestZip64(t *testing.T) {
    if testing.Short() {
        t.Skip("slow test; skipping")
    }
    const size = 1 << 32 // before the "END\n" part
    buf := testZip64(t, size)
    testZip64DirectoryRecordLength(buf, t)
}
```
并发执行测试用例，使用t.Parallel()标记
```go
func TestStackGrowth(t *testing.T) {
    t.Parallel()
    var wg sync.WaitGroup

    // in a normal goroutine
    wg.Add(1)
    go func() {
        defer wg.Done()
        growStack()
    }()
    wg.Wait()
    ...
}
```
### 2.2 性能测试

性能测试函数以接收*testing.B类型为单一参数b，性能测试函数中需要循环b.N次调用被测函数。testing.B类型用来管理测试时间和迭代运行次数，也支持和testing.T相同的方式管理测试状态和格式化测试日志，不一样的是testing.B的日志总是会输出。

启用内存使用分析，使用t.ReportAllocs()
```go
func BenchmarkWriterFlush(b *testing.B) {
    b.ReportAllocs()
    bw := NewWriter(ioutil.Discard)
    str := strings.Repeat("x", 50)
    for i := 0; i < b.N; i++ {
        bw.WriteString(str)
        bw.Flush()
    }
}
```
停止/重置/启动时间计值，使用b.StopTimer()、b.ResetTimer()、b.StartTimer()
```go
func BenchmarkScanInts(b *testing.B) {
    b.ResetTimer()
    ints := makeInts(intCount)
    var r RecursiveInt
    for i := b.N - 1; i >= 0; i-- {
        buf := bytes.NewBuffer(ints)
        b.StartTimer()
        scanInts(&r, buf)
        b.StopTimer()
    }
}
```
记录在一个操作中处理的字节数，使用b.SetBytes()
```go
func BenchmarkFields(b *testing.B) {
    b.SetBytes(int64(len(fieldsInput)))
    for i := 0; i < b.N; i++ {
        Fields(fieldsInput)
    }
}
```
并发执行被测对象，使用b.RunParallel()和*testing.PB类型的Next()
```go
func BenchmarkValueRead(b *testing.B) {
    var v Value
    v.Store(new(int))
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            x := v.Load().(*int)
            if *x != 0 {
                b.Fatalf("wrong value: got %v, want 0", *x)
            }
        }
    })
}
```
### 3 测试执行

在某一包下执行测试: go test
执行指定的包测试: go test $pkg_in_gopath
执行某一目录下以及子目录下所有测试: go test $pkg_in_gopath/...
执行包下某一些用例: go test -run=xxx，-run参数支持使用正则表达式来匹配要执行的功能测试函数名
执行包下性能测试: go test -bench=.
查看性能测试时的内存情况: go test -bench=. -benchmem
查看每个函数的执行结果: go test -v
查看覆盖率: go test -cover
输出覆盖率到文件: 增加参数-coverprofile，并使用go tool cover来查看，用法请参考go tool cover -help

### 4 测试工具

#### 4.1 IO测试

testing/iotest包中实现了常用的出错的Reader和Writer:

触发数据错误dataErrReader，通过DataErrReader()函数创建
读取一半内容的halfReader，通过HalfReader()函数创建
读取一个byte的oneByteReader，通过OneByteReader()函数创建
触发超时错误的timeoutReader，通过TimeoutReader()函数创建
写入指定位数内容后停止的truncateWriter，通过TruncateWriter()函数创建
读取时记录日志的readLogger，通过NewReadLogger()函数创建
写入时记录日志的writeLogger，通过NewWriteLogger()函数创建

#### 4.2 HTTP测试

net/http/httptest包提供了HTTP相关代码的测试工具

httptest.Server用来构建临时的Server，测试发送与接收HTTP请求
httptest.ResponseRecorder用来记录应答

#### 4.3 黑盒测试

testing/quick包实现了帮助黑盒测试

Check函数，测试的只返回bool值的黑盒函数f，Check会为f的每个参数设置任意值并多次调用

```go
func TestOddMultipleOfThree(t *testing.T) {
    f := func(x int) bool {
        y := OddMultipleOfThree(x)
        return y%2 == 1 && y%3 == 0
    }
    if err := quick.Check(f, nil); err != nil {
        t.Error(err)
    }
}
```
CheckEqual函数，比较给定的两个黑盒函数是否相等

func CheckEqual(f, g interface{}, config *Config) (err error)
### 5 测试框架

stretchr/testify是个人觉得目前最好的测试框架，相比标准库中testing包支持如下特性：

- [Easy assertions](https://github.com/stretchr/testify/blob/master/README.md#assert-package)
- [Mocking](https://github.com/stretchr/testify/blob/master/README.md#mock-package)
- [HTTP response trapping](https://github.com/stretchr/testify/blob/master/README.md#http-package)
- [Testing suite interfaces and functions](https://github.com/stretchr/testify/blob/master/README.md#suite-package)

参考：

[1] https://golang.org/pkg/testing
[2] https://golang.org/pkg/testing/iotest
[3] https://golang.org/pkg/testing/quick
[4] https://golang.org/pkg/net/http/httptest
[5] https://github.com/stretchr/testify
