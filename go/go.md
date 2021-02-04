## 闭包

```go
package main
import "fmt"

func test01() func() int {
    // 没有初始化  值为0
    var x int
    
    // 匿名函数
    return func() int {
        x++
        return x*x
    }
}

func main() {
    /**
     * 闭包的特点：
     * 1. 函数嵌套，内部为匿名函数
     * 2. 外层函数的变量值不会被回收
     */
    fmt.Println(test01()) // 1
    fmt.Println(test01()) // 4
    fmt.Println(test01()) // 9
    fmt.Println(test01()) //16
}
```

