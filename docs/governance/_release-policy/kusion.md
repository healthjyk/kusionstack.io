# kusionctl Release Policy

kusionctl 是基于 KCL 的 DevOps 工具链，开源之后开发团队希望采用 [语义化版本](https://semver.org/lang/zh-CN/) 来简化管理。版本格式：主版本号.次版本号.修订号。版本号递增规则如下：主版本号对应不兼容的 API 修改，次版本号对应向下兼容的功能性新增，修订号对应向下兼容的问题修正。其中主版本号和次版本号均包含了不一样的特性统一称之为大版本，补丁修复称之为小版本。

总体目标是每个季度发布一个特性增强的大版本，并支持最近发布的两个大版本，根据需要不定期发布其他版本的修订。

## 1. 发布流程

发布流程如下：

- master 主干开发，每日产出一个 Nightly 版本，CI 系统进行测试
- beta 测试分支，经过 3 周后从 Nightly 版本产出一个 Beta 版本
- stable 稳定分支，经过 3 周后从 Beta 版本产出一个 Stable 版本
- rc 发布分支，每个季度从 Stable 版本产出一个 rc 候选版本，并最终发布

如果本次发布失败，则顺延到下个发布周期。

## 2. 发布维护

发布次要版本以解决一个或多个没有解决方法的关键问题（通常与稳定性或安全性有关）。版本中包含的唯一代码更改是针对特定关键问题的修复。重要的仅文档更改和安全测试更新也可能包括在内，但仅此而已。一旦 Kusion 1.x+2 发布，解决 Kusion 1.x 的非安全问题的次要版本就会停止更新。解决 Kusion 1.x 安全问题的次要版本在 Kusion 1.x+2 发布后停止。
