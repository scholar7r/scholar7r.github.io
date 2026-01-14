+++
title = '表达式预编译'
date = 2025-12-01T11:06:10+08:00
draft = false
author = "scholar7r"
authorTwitter = "scholar7r"
+++
最近学习到 [expr](https://github.com/expr-lang/expr) 这个库，饶有兴致，正好有一个需求是取出数据库中的表达式并预编译运行，遂记录一下。

<!--more-->

```shell
go get github.com/expr-lang/expr
```

我现在正在开发 DNS 相关的一个功能，要求从数据库中获取表达式字符串，然后预编译运行，下面是一个简单的实现。

```go
type record struct {
    Name      string
    Type      uint16
    Value     string
    Condition string
}

type zone struct {
    Records []record
}
```

这是一个最简化的记录，其中的 Condition 字段保存了一段条件表达式，用来筛选记录。解析记录的类型使用 `uint16` 表示，如果换成枚举之类的实现需要**将常量枚举等数据使用 `expr.Env` 通过 `Compile` 导入到表达式环境中**。

每个记录都可以编写表达式，那就说明表达式肯定是包含重复的内容，因此需要筛选掉重复的进行过滤。我们在运行时的环境都是一样的，因此不需要考虑编译时传递环境的问题。当然，这个用例里面也没有用到这种方法，不需要考虑这个问题。

```go
type conditionCompiler struct {
    store map[string]*vm.Program
}
```

这里我们用了一个 `map` 来保存编译的结果，下次匹配到相同的表达式就会直接从 `map` 中取出这个值，从而实现了预编译，当引入了新的条件表达式之后，需要实现追加编译。

```go
func (cc *conditionCompiler) Precompile() error {
    for k := range cc.store {
        prog, err := expr.Compile(k)
        if err != nil {
            return err
        }

        cc.store[k] = prog
    }

    return nil
}
```

在进行预编译之前，需要将 `Condition` 不为空的记录的 `Condition` 值写入到 `map` 的 `key` 中，用于后续编译。从上面的 `Precompile` 方法可以看出，`expr.Compile(k)` 中的 `k` 直接使用的 `map` 的 `key`，这样设计后续更加容易匹配表达式和检查是否需要追加编译。

```go
func main() {
    zones := map[string]*zone{
    "scholar7r.cn.": {
        Records: []record{
            { // should be true
                Name:      "api.scholar7r.cn.",
                Type:      0, // A
                Value:     "127.0.0.1",
                Condition: "Value == '127.0.0.1' && Type == 0",
            },
            { // should be false
                Name:      "api-t.scholar7r.cn.",
                Type:      0, // A
                Value:     "192.168.0.1",
                Condition: "Value == '127.0.0.1'",
            },
            { // should be true
                Name:      "status.scholar7r.cn.",
                Type:      1, // AAAA
                Value:     "::1",
                Condition: "Type == 1",
            },
            { // should be false
                Name:      "gitea.scholar7r.cn.",
                Type:      2, // CNAME
                Value:     "gt.scholar7r.cn.",
                Condition: "Type == 1",
            },
        },
    },
 }

    zone := zones["scholar7r.cn."]

    cc := &conditionCompiler{
        store: map[string]*vm.Program{},
    }

    for _, r := range zone.Records {
        if r.Condition != "" {
            cc.store[r.Condition] = nil
        }
    }

    if err := cc.Precompile(); err != nil {
        log.Panicf("precompile: %v", err)
    }

    log.Println("=== Evaluating Records ===")

    for _, rec := range zone.Records {
        env := map[string]any{
            "Name":  rec.Name,
            "Type":  rec.Type,
            "Value": rec.Value,
        }

        if rec.Condition == "" {
            log.Printf("[ALLOW] %-25s (no condition)\n", rec.Name)
            continue
        }

        prog := cc.store[rec.Condition]

        out, err := expr.Run(prog, env)
        if err != nil {
            log.Printf("[ERROR] %-25s condition=%q error=%v\n",
                rec.Name, rec.Condition, err)
            continue
        }

        match := out.(bool)

        if match {
            log.Printf("[ALLOW] %-25s condition=%q => true\n", rec.Name, rec.Condition)
        } else {
            log.Printf("[DENY] %-25s condition=%q => false\n", rec.Name, rec.Condition)
        }
    }

    log.Print("done")
}
```

追加编译的话，每次调用表达式的时候都是需要在 `map` 通过 `key` 查找 `prog` 的，如果不存在，则编译追加到 `map` 中。

```goat
key                               value
+-----------+                     +---------+
| Condition | -- expr compile --> | Program |
+-----+-----+                     +---------+
      |                                ^
      +----------- exists -------------+
      |                                |
      +---- not exists -- compile -----+
```

再写一个不带预编译的版本，跑一个 Benchmark 看看提升如何。

```go
package main

import (
    "log"

    "github.com/expr-lang/expr"
    "github.com/expr-lang/expr/vm"
)

type record struct {
    Name      string
    Type      uint16
    Value     string
    Condition string
}

type zone struct {
    Records []record
}

type conditionCompiler struct {
    store map[string]*vm.Program
}

func (cc *conditionCompiler) Precompile() error {
    for k := range cc.store {
        prog, err := expr.Compile(k)
        if err != nil {
            return err
        }

        cc.store[k] = prog
    }

    return nil
}

func Condition(zones map[string]*zone) {
    zone := zones["scholar7r.cn."]

    log.Println("=== Evaluating Records ===")

    for _, rec := range zone.Records {
        env := map[string]any{
            "Name":  rec.Name,
            "Type":  rec.Type,
            "Value": rec.Value,
        }

        if rec.Condition == "" {
            log.Printf("[ALLOW] %-25s (no condition)\n", rec.Name)
            continue
        }

        prog, err := expr.Compile(rec.Condition)
        if err != nil {
            log.Printf("[ERROR] %-25s condition=%q error=%v\n", rec.Name, rec.Condition, err)
        }

        out, err := expr.Run(prog, env)
        if err != nil {
            log.Printf("[ERROR] %-25s condition=%q error=%v\n",
                rec.Name, rec.Condition, err)
            continue
        }

        match := out.(bool)

        if match {
            log.Printf("[ALLOW] %-25s condition=%q => true\n", rec.Name, rec.Condition)
        } else {
            log.Printf("[DENY] %-25s condition=%q => false\n", rec.Name, rec.Condition)
        }
    }
}

func ConditionWithPrecompile(zones map[string]*zone) {
    zone := zones["scholar7r.cn."]

    cc := &conditionCompiler{
        store: map[string]*vm.Program{},
    }

    for _, r := range zone.Records {
        if r.Condition != "" {
            cc.store[r.Condition] = nil
        }
    }

    if err := cc.Precompile(); err != nil {
        log.Panicf("precompile: %v", err)
    }

    log.Println("=== Evaluating Records ===")

    for _, rec := range zone.Records {
        env := map[string]any{
            "Name":  rec.Name,
            "Type":  rec.Type,
            "Value": rec.Value,
        }

        if rec.Condition == "" {
            log.Printf("[ALLOW] %-25s (no condition)\n", rec.Name)
            continue
        }

        prog := cc.store[rec.Condition]

        out, err := expr.Run(prog, env)
        if err != nil {
            log.Printf("[ERROR] %-25s condition=%q error=%v\n",
                rec.Name, rec.Condition, err)
            continue
        }

        match := out.(bool)

        if match {
            log.Printf("[ALLOW] %-25s condition=%q => true\n", rec.Name, rec.Condition)
        } else {
            log.Printf("[DENY] %-25s condition=%q => false\n", rec.Name, rec.Condition)
        }
    }
}
```

调用逻辑整理到 Benchmark 测试中了，所以删除了 `main` 函数。再看看 Benchmark 测试的内容。

```go
package main

import (
    "fmt"
    "testing"
)

var zones map[string]*zone

func init() {
    zones = make(map[string]*zone, 0)
    zones["scholar7r.cn."] = generateZone(1000)
}

func generateZone(n int) *zone {
    records := make([]record, n)
    for i := range n {
        records[i] = record{
            Name:      fmt.Sprintf("host-%d.scholar7r.cn.", i),
            Type:      uint16(i % 3), // 0:A, 1:AAAA, 2:CNAME
            Value:     fmt.Sprintf("192.168.0.%d", i%255),
            Condition: "",
        }

        if i%2 == 0 {
            t := i % 3
            records[i].Condition = fmt.Sprintf("Type == %d && Value == '192.168.0.%d'", t, i%255)
        }
    }
    return &zone{Records: records}
}

func BenchmarkCondition(b *testing.B) {
    b.ReportAllocs()
    for b.Loop() {
        Condition(zones)
    }
}

func BenchmarkConditionWithPrecompile(b *testing.B) {
    b.ReportAllocs()
    for b.Loop() {
        ConditionWithPrecompile(zones)
    }
}
```

运行 Benchmark 测试。我没有在具体实现中删除 `log` 相关代码，因此会占用相当一部分的 I/O 资源，对实际数据会产生一点影响。

```shell
go test -bench=. -benchmem # | grep op

# BenchmarkCondition-16               225 5525636 ns/op 5765045 B/op 40018 allocs/op
# BenchmarkConditionWithPrecompile-16 368 3233540 ns/op 3175136 B/op 23850 allocs/op
```

| 指标            | Condition | ConditionWithPrecompile | 提升幅度   |
| ------------- | --------- | ----------------------- | ------ |
| **耗时 ns/op**  | 5,525,636 | 3,233,540               | ≈41% 快 |
| **内存 B/op**   | 5,765,045 | 3,175,136               | ≈45% 少 |
| **分配 allocs/op** | 40,018    | 23,850                  | ≈40% 少 |

当然还有更快的处理方式，本例中使用的 `map` 本身是一个动态数据解构，会占用内存和分配对象，大量的 `map` 写入和增长都会导致产生多次分配。可以考虑尝试使用 `slice` 等来实现，降低内存占用。
