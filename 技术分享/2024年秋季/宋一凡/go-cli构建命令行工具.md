# urfave/cli

## 简介

urfave/cli 是一个声明性的、简单、快速且有趣的包，用于用 Go 构建命令行工具，其特点是：

- 具有别名和前缀匹配支持的命令和子命令(可以定义主命令和子命令，并为它们指定别名。例如，可以定义一个命令 greet，并为其设置别名 g)
- 灵活且宽容的帮助系统（提供了自动生成的帮助信息 `--help`）
- 动态 shell 补全，包括bash、zsh、fish和powershell
- 除了 Go 标准库之外没有其他依赖,这使得它非常轻量级和易于管理
- 简单类型、简单类型片段、时间、持续时间等的输入标志
- 复合短旗支撑（-a -b -c可缩短为-abc）
- 支持通过 urfave/cli-docs 模块生成 man 和 Markdown 格式的文档，方便你为 CLI 工具编写详细的使用说明
- 输入查找自：
  - 环境变量
  - 纯文本文件
  - 结构化文件格式（通过 urfave/cli-altsrc模块支持）

## 安装

```shell
go get -u github.com/urfave/cli/v2@latest
```

## 使用

### 基本使用

使用非常简单，理论上创建一个cli.App结构的对象，然后调用其Run()方法，传入命令行的参数即可

```go
package main

import (
 "fmt"
 "github.com/urfave/cli/v2"
)

func main() {
 app := &cli.App{
  Name:  "hello",
  Usage: "hello world",
  Action: func(c *cli.Context) error { 
        // Action是调用该命令行程序时实际执行的函数，需要的信息可以从参数cli.Context获取
   fmt.Println("Hello world!")
   return nil
  },
 }

 err := app.Run(os.Args)
 if err != nil {
  panic(err)
 }
}
```

### 命令行参数

通过`cli.Context`的相关方法我们可以获取传给命令行的参数信息：

- NArg(): 返回参数个数;  
- Args(): 返回cli.Args对象, 调用其Get(i)获取位置i上的参数。

示例:

```go
func main() {
  app := &cli.App{
    Name:  "arguments",
    Usage: "arguments example",
    Action: func(c *cli.Context) error {
      for i := 0; i < c.NArg(); i++ {
        fmt.Printf("%d: %s\n", i+1, c.Args().Get(i))
      }
      return nil
    },
  }

  err := app.Run(os.Args)
  if err != nil {
    log.Fatal(err)
  }
}
```

当运行这个程序并传递参数时，它会输出每个参数的位置和值。例如：

```shell
$ go run main.go hello world
1: hello
2: world
```

hello 和 world 就是传递给程序的参数。这些参数会被 cli.Context 捕获并存储起来，供我们在程序中使用。

Go标准库里的flag包也可以定义和解析命令行标志(flags)，但是urfave/cli包提供了更强大和灵活的功能，可以更方便地定义和解析命令行参数。

- flag包中的Args()和NArg():
  - Args(): 返回一个字符串切片，包含所有未被解析为标志的非标志参数
  - NArg(): 返回未被解析为标志的非标志参数的数量

- cli包中的Args()和NArg():  
  - Args(): 返回一个cli.Args对象，该对象封装了所有未被解析为标志的非标志参数

  ```go
  // 底层:
  // Args returns the command line arguments associated with the context.
  func (cCtx *Context) Args() Args {
  ret := args(cCtx.flagSet.Args())
  return &ret
  }
  ```

  - NArg(): 返回未被解析为标志的非标志参数的数量



  ```go
  // 底层:
  // NArg returns the number of the command line arguments.
  func (cCtx *Context) NArg() int {
  return cCtx.Args().Len()
  }

  func (cCtx *Context) lookupFlag(name string) Flag {
  for _, c := range cCtx.Lineage() {
  if c.Command == nil {
    continue
  }

  for _, f := range c.Command.Flags {
    for _, n := range f.Names() {
    if n == name {
      return f
    }
    }
  }
  }

  if cCtx.App != nil {
  for _, f := range cCtx.App.Flags {
    for _, n := range f.Names() {
    if n == name {
      return f
    }
    }
  }
  }

  return nil
  }

  ```

### 选项(Flags)

cli设置和获取选项非常简单。在cli.App{}结构初始化时，设置字段Flags即可添加选项。Flags字段是[]cli.Flag类型，cli.Flag实际上是接口类型。cli为常见类型都实现了对应的XxxFlag，如BoolFlag/DurationFlag/StringFlag等。它们有一些共用的字段，Name/Value/Usage（名称/默认值/释义）。看示例：

```go
func main() {
 // 创建一个新的 CLI 应用程序实例
 app := &cli.App{
  // 定义应用程序支持的标志（flags）
  Flags: []cli.Flag{
   &cli.StringFlag{
    Name:  "lang",       // 标志的名称为 "lang"
    Value: "english",    // 默认值为 "english"
    Usage: "language for the greeting", // 标志的描述信息
   },
  },
  // 定义应用程序的主要操作函数
  Action: func(c *cli.Context) error {
   name := "world" // 默认名字为 "world"
   if c.NArg() > 0 { // 如果提供了额外的命令行参数
    name = c.Args().Get(0) // 使用第一个参数作为名字
   }

   // 根据 "lang" 标志的值选择语言并打印问候语
   if c.String("lang") == "english" {
    fmt.Println("hello", name) // 如果是英文，打印 "hello <name>"
   } else {
    fmt.Println("你好", name) // 否则，打印 "你好 <name>"
   }
   return nil // 返回 nil 表示没有错误发生
  },
 }

 // 运行应用程序，传入命令行参数
 err := app.Run(os.Args)
 if err != nil { // 如果运行过程中出现错误
  log.Fatal(err) // 记录错误并退出程序
 }
}
```

注意选项是通过c.Type(name)来获取的，Type为选项类型，name为选项名.
编译,运行:

```shell
$ go build -o flags

# 默认调用
$ ./flags
hello world

# 设置非英语
$ ./flags --lang chinese
你好 world

# 传入参数作为人名
$ ./flags --lang chinese dj
你好 dj
```

golang的标准库flag包也可以定义和解析命令行标志(flags)

```golang
配置flag
func XXX(name string, value xxx, usage string) *xxx
func XXXVar(p *xxx, name string, value xxx, usage string)
```

XXX为各个类型名，有：bool、duration、float64、int64、int、uint、uint64、string等.
name string为flag名—name区分大小写
value xxx为默认值
usage string为帮助信息
p *xxx为Type指针
即如下内容：

```golang
flag.Type(flag名, 默认值, 帮助信息)*Type
flag.TypeVar(Type指针, flag名, 默认值, 帮助信息) 
```

它们的区别是: 前者直接返回指向对应类型指针并分配对应指针指向的对象，可以直接通过解引用来获取转换后的对应值
后者需要指定指针指向的对象.

例如:定义姓名、年龄、婚否三个命令行参数

```golang
name := flag.String("name", "张三", "姓名")
age := flag.Int("age", 18, "年龄")
married := flag.Bool("married", false, "婚否")
delay := flag.Duration("d", 0, "时间间隔")
```

或者

```golang
var name string
var age int
var married bool
var delay time.Duration
flag.StringVar(&name, "name", "张三", "姓名")
flag.IntVar(&age, "age", 18, "年龄")
flag.BoolVar(&married, "married", false, "婚否")
flag.DurationVar(&delay, "d", 0, "时间间隔")
```

### 存入变量(Destination)

除了通过c.Type(name)来获取选项的值，我们还可以将选项存到某个预先定义好的变量中。只需要设置Destination字段为变量的地址即可：

```go
func main() {
  var language string

  app := &cli.App{
    Flags: []cli.Flag{
      &cli.StringFlag{
        Name:        "lang",
        Value:       "english",
        Usage:       "language for the greeting",
        Destination: &language, // 设置这个为变量的地址,这样就可以将选项的值存入这个变量中
      },
    },
    Action: func(c *cli.Context) error {
      name := "world"
      if c.NArg() > 0 {
        name = c.Args().Get(0)
      }

      if language == "english" {
        fmt.Println("hello", name)
      } else {
        fmt.Println("你好", name)
      }
      return nil
    },
  }

  err := app.Run(os.Args)
  if err != nil {
    log.Fatal(err)
  }
}
```

aass.

### 占位值

```go
func main() {
  app := & cli.App{
    Flags : []cli.Flag {
      &cli.StringFlag{
        Name:"config",
        Usage: "Load configuration from `FILE`",
      },
    },
  }

  err := app.Run(os.Args)
  if err != nil {
    log.Fatal(err)
  }
}
```

设置占位值之后，帮助信息中，该占位值会显示在对应的选项后面，对短选项也是有效的：

```shell
$ go build -o placeholder
$ ./placeholder --help
NAME:
   placeholder - A new cli application

USAGE:
   placeholder [global options] command [command options] [arguments...]

COMMANDS:
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config FILE  Load configuration from FILE
   --help, -h     show help (default: false)
```

在这个帮助信息中，--config 选项后面显示了占位值 FILE. 这告诉用户需要提供一个文件名作为参数。例如：

```shell
./placeholder --config config.yaml

```

### 命令和子命令

cli通过设置cli.App的Commands字段添加命令，设置各个命令的SubCommands字段，即可添加子命令

```go
func main() {
  app := &cli.App{
    Commands: []*cli.Command{
      {
        Name:    "add",
        Aliases: []string{"a"},
        Usage:   "add a task to the list",
        Action: func(c *cli.Context) error {
          fmt.Println("added task: ", c.Args().First())
          return nil
        },
      },
      {
        Name:    "complete",
        Aliases: []string{"c"},
        Usage:   "complete a task on the list",
        Action: func(c *cli.Context) error {
          fmt.Println("completed task: ", c.Args().First())
          return nil
        },
      },
      {
        Name:    "template",
        Aliases: []string{"t"},
        Usage:   "options for task templates",
        Subcommands: []*cli.Command{
          {
            Name:  "add",
            Usage: "add a new template",
            Action: func(c *cli.Context) error {
              fmt.Println("new task template: ", c.Args().First())
              return nil
            },
          },
          {
            Name:  "remove",
            Usage: "remove an existing template",
            Action: func(c *cli.Context) error {
              fmt.Println("removed task template: ", c.Args().First())
              return nil
            },
          },
        },
      },
    },
  }

  err := app.Run(os.Args)
  if err != nil {
    log.Fatal(err)
  }
}
```

### 基本使用

<!-- ### 标志

使用`cli.StringFlag`定义一个字符串类型的标志，并使用`cli.NewApp()`创建一个新的应用程序。然后，在`Action`函数中，使用`c.String("flag-name")`获取标志的值。

```go
package main

import (
 "fmt"
)
``` -->

### 开发一个CLI程序

1. **创建应用**
首先，我们使用 cli.NewApp() 创建 *cli.App 实例，然后分别设置了程序的名称、使用说明、版本，最后使用 cliApp.Run(os.Args) 方法运行程序，传入系统参数并处理错误信息：

```go
package main  
  
import (  
   "fmt"   
   "github.com/urfave/cli/v2"   
   "os"
)
  
func main() {  
   // 创建一个新的CLI应用
   cliApp := cli.NewApp()  
   
   // 设置CLI应用的属性
   cliApp.Name = "demo-cli"  // 应用名称设为 "demo-cli"
   cliApp.Usage = "cli usage demo"  // 使用说明设为 "cli usage demo"
   cliApp.Version = "0.0.1"  // 版本号设为 "0.0.1"
   
   // 运行CLI应用
   err := cliApp.Run(os.Args)  // app退出不会调用 os.Exit，所以默认退出代码都是0，可以通过 cli.Exit方法指定退出信息和退出码  
   
   // 如果运行过程中发生错误，则打印错误信息并退出
   if err != nil {  
      fmt.Printf("demo-cli execute error: %v", err)  // 打印错误信息
      os.Exit(-1)  // 以非零状态码退出程序
   }  
}
```

这样,我们创建好了一个CLI应用，并设置了它的名称,使用说明和版本号。但是这只是一个空程序，运行它除了输出一些帮助信息之外什么也干不了

2.**添加全局选项(Flag)**
我们可以在 cliApp.Flags 中添加全局选项，这样在运行程序时，无论在哪个命令下，都可以使用这些全局选项。例如，我们添加一个 --debug 选项，用于控制程序是否输出调试信息：

添加全局选项也很简单，在运行程序之前添加代码：

```go
var Verbose bool  // 定义一个全局变量Verbose，用于存储命令行参数的值

func main() {
  // ...
  // 全局参数  
  cliApp.Flags = append(cliApp.Flags, []cli.Flag{  
      &cli.BoolFlag{Name: "i", Usage: "show verbose info", Required: false, Destination: &Verbose}, // destination 可以将设置的参数绑定到变量，后续可以直接使用  
  }...)
  // ...
}
```

我们定义了一个名为 Verbose 的全局布尔变量，并将其绑定到 --verbose 选项上。这样，当用户运行程序并使用 --verbose 选项时，Verbose 变量的值将被设置为 true。
这里的`cli.BoolFlag` 是一个布尔类型的标志，`Name` 是标志的名称，`Usage` 是标志的用途描述，`Required` 表示标志是否是必需的，`Destination` 是一个指向布尔值的指针，用于存储标志的值。

&cli.BoolFlag的底层:

```go
// BoolFlag 是一个布尔类型的标志
type BoolFlag struct {
    // 标志的名称，通常是一个简短的字符串，用于在命令行中指定这个标志
    Name string

    // 标志所属的类别，用于在帮助信息中组织标志
    Category string

    // 标志的默认值文本，用于在帮助信息中显示
    DefaultText string

    // 标志的配置文件路径，如果标志的值可以从配置文件中读取，则该字段指定配置文件的路径
    FilePath string

    // 标志的用途描述，用于在帮助信息中显示
    Usage string

    // 标志是否是必需的，如果为 true，则用户必须提供这个标志的值
    Required bool

    // 标志是否在帮助信息中隐藏，如果为 true，则这个标志不会在帮助信息中显示
    Hidden bool

    // 标志的值是否已经被设置，用于内部跟踪标志的状态
    HasBeenSet bool

    // 标志的当前值，类型为布尔值
    Value bool

    // 指向存储标志值的变量的指针，通常用于将标志的值赋给某个变量
    Destination *bool

    // 标志的别名，允许用户使用不同的名称来指定这个标志
    Aliases []string

    // 标志的环境变量，允许用户通过环境变量来设置标志的值
    EnvVars []string

    // 标志的默认值，类型为布尔值
    defaultValue bool

    // 标志的默认值是否已经被设置
    defaultValueSet bool

    // 标志出现的次数，用于处理类似 -vvv 这样的多次出现的情况
    Count *int

    // 是否禁用默认值文本，如果为 true，则默认值文本不会在帮助信息中显示
    DisableDefaultText bool

    // 标志的动作函数，当标志被解析时，会调用这个函数。函数接收两个参数：一个 *Context 对象和一个布尔值，返回一个错误
    Action func(*Context, bool) error
}
```

选项 Flag 有多种类型，包括 StringFlag、BoolFlag、IntFlag 等等，它们都实现了顶层接口 Flag。
再次运行程序，可以通过帮助信息看到全局选项已经添加成功:

```bash
NAME:
   demo-cli - cli usage demo

USAGE:
   demo-cli [global options] command [command options]

VERSION:
   0.0.1

COMMANDS:
   help, h  Shows a list of commands or help for one command

# 全局选项在这里
GLOBAL OPTIONS:
   -i             show verbose info (default: false)
   --help, -h     show help
   --version, -v  print the version

进程 已完成，退出代码为 0
```

3. **添加命令**
现在，我们来添加一个命令，实现问好的功能。

```go
func main() {  
   // ...
   // 系统命令  
   cliApp.Commands = []*cli.Command{sayHelloCmd()}
   // ...
}

func sayHelloCmd() *cli.Command {  
   return &cli.Command{  
      Name:    "hello",        // 命令名称，执行时需要指定  
      Aliases: []string{"ho"}, // 命令别名，简化名称  
      Usage:   "向您问好，-h 查看更多帮助信息", 
      Flags: []cli.Flag{  
        &cli.StringFlag{Name: "n", Aliases: []string{"name"}, Usage: "您的姓名`NAME`", Required: true},  
   }, 
      Action: func(ctx *cli.Context) error { // 具体命令的执行逻辑  
  name := ctx.String("n")  
  fmt.Println("hello,", name, "!")  
  return nil  
   },
 }
}
```

在项目根目录下，通过以下命令编译代码：

```bash
go build -o demo-cli
```

编译成功后，会生成一个名为 cli_demo 的可执行文件。可以通过以下命令运行它：

```bash
./demo-cli 
```

执行 hello 命令并传递一个名字作为参数：

```bash
./cli_demo hello -n hank
```

输出结果应该是：`hello, hank!`

4. **添加子命令**

cli通过设置cli.App的Commands字段添加命令，设置各个命令的SubCommands字段，即可添加子命令

5. **命令分类**

在子命令数量很多的时候，可以设置Category字段为它们分类，在帮助信息中会将相同分类的命令放在一起展示：

```go
package main // 声明包名为main

import (
 "fmt"        // 导入格式化I/O库
 "os"         // 导入操作系统功能库

 "github.com/Weidows/wutils/cmd/wutils/diff" // 导入文件对比工具包
 "github.com/Weidows/wutils/cmd/wutils/keep_runner" // 导入后台任务管理包
 "github.com/Weidows/wutils/cmd/wutils/zip" // 导入压缩文件操作包
 "github.com/Weidows/wutils/utils/log" // 导入日志记录包
 "github.com/urfave/cli/v2" // 导入命令行解析库
)

var (
 logger = log.GetLogger() // 获取全局日志记录器实例
 kr     = keep_runner.NewKeepRunner(logger) // 创建并初始化后台任务管理器
)

var app = &cli.App{ // 定义CLI应用程序
 Name: "wutils", // 设置应用程序名称为wutils
 Authors: []*cli.Author{{ // 设置作者信息
  Name:  "Weidows",
  Email: "ceo@weidows.tech",
 }},
 EnableBashCompletion: true, // 启用Bash自动补全
 Usage: "Documents(使用指南) at here:
https://blog.weidows.tech/post/lang/golang/wutils", // 提供使用文档链接
 Commands: []*cli.Command{ // 定义命令列表
  {
   Name:  "config", // 定义config命令
   Usage: "print config file", // 打印配置文件内容
   Action: func(cCtx *cli.Context) (err error) { // 定义命令执行时的操作
    logger.Println(fmt.Sprintf("%+v", kr.Config)) // 打印当前配置
    return err
   },
  },
  {
   Name:  "diff", // 定义diff命令
   Usage: "diff - 文件对比工具
" +
    "是用来求 '行-差集' 的工具
" +
   "输入为特定的两个文件名称: './inputA.txt', './inputB.txt'", // 描述diff命令的用途和使用方法
   Action: func(cCtx *cli.Context) (err error) { // 定义命令执行时的操作
    missInA, missInB := diff.CheckLinesDiff("./inputA.txt", "./inputB.txt") // 调用CheckLinesDiff函数对比文件内容
    // 输出结果
    fmt.Println("================== Missing in A ==================") // 打印缺失在A文件中的行
    for _, file := range missInA {
     fmt.Println(file) // 逐行打印缺失内容
    }

    fmt.Println("
================== Missing in B:==================") // 打印缺失在B文件中的行
    for _, file := range missInB {
     fmt.Println(file) // 逐行打印缺失内容
   }

   return err
  },
 },
 {
  Name:    "parallel", // 并行执行命令
 Aliases: []string{"pl"}, // 别名为pl
 Usage:   "并行+后台执行任务 (配置取自wutils.yml)", // 描述并行执行任务的用途和配置来源
 Action: func(cCtx *cli.Context) (err error) { // 定义命令执行时的操作
  if kr.Config.Parallel.Dsg { // 如果配置中启用了磁盘睡眠防护
   go kr.Dsg() // 以goroutine方式启动磁盘睡眠防护
  }
 if kr.Config.Parallel.Ol { // 如果配置中启用透明度监听
  kr.Ol() // 启动透明度监听
 }

 return err
},
},
{
Name:      "dsg", // 磁盘睡眠防护命令
UsageText: "",
Usage: "Disk Sleep guard
" +
 "防止硬盘睡眠 (外接 HDD 频繁启停甚，让其怠机跑着，免得增加损坏率)
" +
 "外接 HDD 较常见，请后台怠机跑着，以免增加损坏率", // 描述磁盘睡眠防护的用途和注意事项
Action: func(cCtx *cli.Context) (err error) { // 定义命令执行时的操作
 kr.Dsg() // 启动磁盘睡眠防护
 return err
},
},
{
Name:  "ol", // 透明度监听命令
Usage: "Opacity Listener
" +
 "后台持续运行，并每隔一定时间扫一次运行的窗口
" +
 "把指定窗口设置opacity，使其透明化 (same as blend)", // 描述透明度监听的用途和功能
Action: func(cCtx *cli.Context) (err error) { // 定义命令执行时的操作
 kr.Ol() // 启动透明度监听
 return err
},
Subcommands: []*cli.Command{ // 定义子命令列表
{
Name:  "list", // 列出所有可见窗口命令
Usage: "list all visible windows", // 描述列出所有可见窗口的用途
Action: func(cCtx *cli.Context) (err error) { // 定义命令执行时的操作
kr.OlList() // 调用OlList函数列出所有可见窗口
return err
},
},
},
},
{
Name:    "zip", // zip相关操作命令
Usage:   "some actions to operate on zip/7z files", // 描述zip命令的用途和功能
Subcommands: []*cli.Command{ // 定义子命令列表
{
Name:  "crack", // zip密码破解命令
Usage: "zip crack <path> - Crack the password of zip/7z files", // 描述zip密码破解命令的用途和用法
Action: func(cCtx *cli.Context) error { // 定义命令执行时的操作
if cCtx.Args().Len() < 1 { // 如果未提供路径参数则返回错误
return fmt.Errorf("please provide the path to the archive file")
}
archivePath := cCtx.Args().Get(0) // 获取路径参数
password := zip.CrackPassword(archivePath) // 调用CrackPassword函数破解密码

if password == "" { // 如果未找到密码则返回错误
return fmt.Errorf("no password found")
} else { // 如果找到密码则打印密码
fmt.Printf("Password found: %s
", password)
}

return nil // 返回nil表示成功
},
},
},
},
}

func main() { // 主函数入口
if err := app.Run(os.Args); err != nil { // 运行CLI应用程序并捕获错误
logger.Fatal(err.Error()) // 如果发生错误则记录错误日志并退出程序
}
}
```
