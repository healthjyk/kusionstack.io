---
sidebar_position: 3
---
# Validate

校验当前目录的 KCL 配置是否可以编译

### Synopsis

验证当前目录中的 KCL 配置是否可以编译， 并且不与任何 backend/state/provider 交互。

```
kusion validate [flags]
```

### Examples

```
  # 验证 main.k 中的配置
  kusion validate main.k
  
  # 使用参数验证 main.k
  kusion validate main.k -D name=test -D age=18
  
  # 使用来自 settings.yaml 的参数验证 main.k
  kusion validate main.k -Y settings.yaml
  
  # 使用工作目录验证 main.k
  kusion validate main.k -w Konfig/appops/demo/dev
```

### Options

```
  -D, --argument strings    指定顶级参数
  -n, --disable-none        禁用转储 None 值
  -h, --help                help for validate
  -a, --override-AST        指定覆盖选项
  -O, --overrides strings   指定配置覆盖路径和值
  -Y, --setting strings     指定命令行配置文件
  -w, --workdir string      指定工作目录
```

### Options inherited from parent commands

```
      --log-level string        设置 kusion 开发日志级别，默认为 INFO，所有选项：DEBUG、INFO、ERROR、WARN、FATAL (default "INFO")
      --profile string          要捕获的档案名称。none、cpu、heap、goroutine、threadcreate、block 和 mutex 之一 (default "none")
      --profile-output string   档案写入的文件名 (default "profile.pprof")
```

### SEE ALSO

* [kusion](./overview.md)	 - kusion 通过代码管理 Kubernetes

###### Auto generated by spf13/cobra on 21-Jan-2022
