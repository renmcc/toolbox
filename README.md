# mcube
![Build and Test](https://github.com/infraboard/mcube/workflows/Build%20and%20Test/badge.svg)
[![codecov](https://codecov.io/gh/infraboard/mcube/branch/master/graph/badge.svg)](https://codecov.io/gh/infraboard/mcube)
[![Go Report Card](https://goreportcard.com/badge/github.com/infraboard/mcube)](https://goreportcard.com/report/github.com/infraboard/mcube)
[![Release](https://img.shields.io/github/release/infraboard/mcube.svg?style=flat-square)](https://github.com/infraboard/mcube/releases)
[![MIT License](https://img.shields.io/github/license/infraboard/mcube.svg)](https://github.com/infraboard/mcube/blob/master/LICENSE)


微服务工具箱, 构建微服务中使用的工具集

+ http框架: 用于构建领域服务的路由框架, 基于httprouter进行封装
+ 异常处理: 定义API Exception
+ 日志处理: 封装zap, 用于日志处理
+ 加密解密: 封装cbc和ecies
+ 自定义类型: ftime方便控制时间序列化的类型, set集合
+ 服务注册: 服务注册组件
+ 缓存处理: 用于构建多级对象缓存
+ 事件总线: 用于系统事件订阅与发布
+ 链路追踪: mcubte提供的组件都内置了链路追踪

## 快速上手

首先你需要安装mcube, 所有的功能都集成到这个CLI工具上了
```sh
$ go install github.com/infraboard/mcube/cmd/mcube 
```

按照完成后, 通过help指令查看基本使用方法
```
$ mcube -h
mcube ...

Usage:
  mcube [flags]
  mcube [command]

Available Commands:
  enum        枚举生成器
  help        Help about any command
  init        初始化

Flags:
  -h, --help      help for mcube
  -v, --version   the mcube version

Use "mcube [command] --help" for more information about a command.
```

mcube提供项目初始化能力, 利用mcube提供的工具箱, 快速组装出一个接近生产级别的应用(使用请看README):
```sh
$ mkdir demo && cd demo
$ mcube init
? 请输入项目包名称: github.com/infraboard/demo
? 请输入项目描述: 项目描述,会生成到CLI和READMD.md中
项目初始化完成, 项目结构如下: 
├───.gitignore (269b)
├───Makefile (1212b)
├───README.md (894b)
├───api
│       └───api.go (3768b)
├───cmd
│       ├───root.go (888b)
│       └───service.go (4036b)
├───conf
│       ├───config.go (3222b)
│       ├───load.go (720b)
│       └───log.go (365b)
├───etc
│       ├───demo.env (149b)
│       └───demo.toml (237b)
├───go.mod (43b)
├───main.go (90b)
├───pkg
│       ├───auther.go (345b)
│       ├───http.go (1224b)
│       └───service.go (865b)
├───script
│       └───build.sh (3378b)
└───version
        └───version.go (566b)
```

启用看一看
```
$ make run
2020-06-06T20:03:00.328+0800    INFO    [INIT]  cmd/service.go:151      log level: debug
2020-06-06T20:03:00.328+0800    INFO    [CLI]   cmd/service.go:93       loaded services: []
Version   : 
Build Time: 
Git Branch: 
Git Commit: 
Go Version: 

2020-06-06T20:03:00.328+0800    INFO    [API]   api/api.go:66   http endpoint registry success
2020-06-06T20:03:00.328+0800    INFO    [API]   api/api.go:100  HTTP服务启动成功, 监听地址: 0.0.0.0:8050
```

## 解放双手

对于一些可能标准化的代码模块, mcube已经为你准备好了生成器, 用于提升效率



+ 枚举生成器

安装好mcube好后, 编写好基本的枚举, 然后生成器会提取这些信息, 生成序列化方法
```
//go:generate  mcube enum -m
package enum_test

const (
	// Running (running) todo
	Running Status = iota
	// Stopping (stopping) tdo
	Stopping
	// Stopped (stopped) todo
	Stopped
	// Canceled (canceled) todo
	Canceled

	test11
)

const (
	// Running (running) todo
	E1 Enum = iota
	// Running (running) todo
	E2
)

// Status AAA
// BBB
type Status uint

type Enum uint
```

执行生成器
```
go generate ./...
```

基于上面的样例生成如下:
```
// Code generated by github.com/infraboard/mcube
// DO NOT EDIT

package enum_test

import (
	"bytes"
	"fmt"
	"strings"
)

var (
	enumStatusShowMap = map[Status]string{
		Running:  "Running",
		Stopping: "Stopping",
		Stopped:  "Stopped",
		Canceled: "Canceled",
		test11:   "test11",
	}

	enumStatusIDMap = map[string]Status{
		"Running":  Running,
		"Stopping": Stopping,
		"Stopped":  Stopped,
		"Canceled": Canceled,
		"test11":   test11,
	}
)

// ParseStatus Parse Status from string
func ParseStatus(str string) (Status, error) {
	key := strings.Trim(string(str), `"`)
	v, ok := enumStatusIDMap[key]
	if !ok {
		return 0, fmt.Errorf("unknown Status: %s", str)
	}

	return v, nil
}

// Is todo
func (t Status) Is(target Status) bool {
	return t == target
}

// String stringer
func (t Status) String() string {
	v, ok := enumStatusShowMap[t]
	if !ok {
		return "unknown"
	}

	return v
}

// MarshalJSON todo
func (t Status) MarshalJSON() ([]byte, error) {
	b := bytes.NewBufferString(`"`)
	b.WriteString(t.String())
	b.WriteString(`"`)
	return b.Bytes(), nil
}

// UnmarshalJSON todo
func (t *Status) UnmarshalJSON(b []byte) error {
	ins, err := ParseStatus(string(b))
	if err != nil {
		return err
	}
	*t = ins
	return nil
}

var (
	enumEnumShowMap = map[Enum]string{
		E1: "E1",
		E2: "E2",
	}

	enumEnumIDMap = map[string]Enum{
		"E1": E1,
		"E2": E2,
	}
)

// ParseEnum Parse Enum from string
func ParseEnum(str string) (Enum, error) {
	key := strings.Trim(string(str), `"`)
	v, ok := enumEnumIDMap[key]
	if !ok {
		return 0, fmt.Errorf("unknown Status: %s", str)
	}

	return v, nil
}

// Is todo
func (t Enum) Is(target Enum) bool {
	return t == target
}

// String stringer
func (t Enum) String() string {
	v, ok := enumEnumShowMap[t]
	if !ok {
		return "unknown"
	}

	return v
}

// MarshalJSON todo
func (t Enum) MarshalJSON() ([]byte, error) {
	b := bytes.NewBufferString(`"`)
	b.WriteString(t.String())
	b.WriteString(`"`)
	return b.Bytes(), nil
}

// UnmarshalJSON todo
func (t *Enum) UnmarshalJSON(b []byte) error {
	ins, err := ParseEnum(string(b))
	if err != nil {
		return err
	}
	*t = ins
	return nil
}
```