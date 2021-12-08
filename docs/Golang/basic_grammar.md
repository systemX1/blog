verbs https://pkg.go.dev/fmt

函数可以安全地返回局部变量

```go
var p = f()
func f() *int {
	v := 100
	return &v
}
```

