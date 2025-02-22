---
slug: 2022-learn-from-scale-practice
title: 从规模化平台工程实践，我们学到了什么
authors:
  name: 朵晓东
  title: Kusion Creator
tags: [KusionStack, Kusion]
---

**摘要**：本文尝试从平台工程、专用语言、分治、建模、自动化和协同文化等几个角度阐述规模化平台工程实践中的挑战和最佳实践。希望通过把我们平台工程的理念和实践分享给更多企业和团队，一起让一些有意思的变化发生。

本文基于 [KusionStack](https://kusionstack.io/docs/user_docs/intro/kusion-intro) 技术栈在蚂蚁平台工程及自动化中的实践总结而成。


## 1. 平台工程：让企业级 DevOps 发生

DevOps 理念在 10 多年前被提出，从 KVM 到容器再到云原生时代，大量企业投入 DevOps 运动以期望解决内部规模化运维效率和平台建设效率的困境。其中大部分陷入过某种基于对 DevOps 朴素认知的 Anti-Pattern，同时也有部分公司探索出自己的路径。我经历过如下图简示的 Anti-Patterns，Dev 与 Ops 团队各行其是，或者简单的强制 Dev 团队独立完成 Ops 工作。在[这里](https://web.devopstopologies.com/#anti-types)可以找到更多更典型分类。

![](/img/blog/2022-09-16-learn-from-scale-practice/devops-anti-pattern.png)

企业内规模化 DevOps 难以推行的原因多种多样，特别是在企业内自持基础设施、同时采用云上技术平台的公司阻力最大。其中以这几种情况尤为常见：

- 研发团队和运维团队由于部门墙、领导者缺少洞察等等原因各自为政，难以达成一致意见
- 研发团队低估了基础设施技术、运维、稳定性工作的专业性、复杂性和快速变化，以朴素的 DevOps 理解强制应用研发者成为专家
- 领导者建立了专职的 DevOps 团队，但沦为中间的执行者，没能让 Dev 和 Ops 团队各自向前一步，紧密协同
- 平台研发团队对规模化带来的业务复杂性以及技术演进带来的技术复杂性应对不足，无法对应用研发者提供有效的技术支撑
- ...

不同于面向云上托管基础设施服务和 DevOps-as-a-Service 产品工作的小型团队，中大型企业往往需要根据自身团队架构和文化建立适当的 DevOps 体系。从成功案例看，无论是 Meta 公司由 Dev 完全承担 Ops 职能，还是 Google 公司引入 SRE 团队作为中间层，平台工程（[Platform Engineering](https://platformengineering.org/blog/what-is-platform-engineering)）都扮演了非常重要的角色。平台工程旨在帮助企业构建面向应用研发者的自服务运维体系，尝试通过工程化的技术手段和工作流程解决以下关键问题：

- 设计合理的抽象层次，帮助应用研发者降低对 Infra、platform 等技术以及运维、稳定性工作的认知负担
- 为应用研发者提供统一的工作界面和空间，避免用户陷入割裂的平台产品界面、复杂的工作流中
- 帮助研发者通过有效的工作流程和推荐路径基于 [内部工程平台](https://internaldeveloperplatform.org/what-is-an-internal-developer-platform/) 快速开展工作
- 帮助研发者通过配套的 CI、CD、CDRA 等产品自服务管理应用生命周期
- 帮助平台产品研发团队简单、高效、一致的开放其平台基础能力
- 通过培训、布道、运营等手段营造协同工作和分享的文化

事实上，不是所有人都应该或者能够成为这个领域的专家，这非常困难！实际上平台技术团队的专家通常也仅擅长自己的专业领域而已，特别是在云原生理念和技术广泛应用的今天，面向大量高度开放、可配置的平台技术带来的成百上千的应用配置，PaaS 领域的业务复杂性，以及高稳定性和统一治理的要求，而平台工程的目的正是为了让应用研发者尽可能简单无痛的参与到这样规模化的 DevOps 工作中。在蚂蚁的实践中，我们更趋向于以下这种合作状态，在团队架构和工作模式上更靠近 Google 的最佳实践。平台研发者及 SRE 成为 “Enabler” 支持应用研发者自服务的完成研发及交付运维，同时应用研发者使其应用可交付运维的工作结果也成为运维人员可以接手应用运维工作的基础。最终 SRE、应用研发及运维人员把工作过程中的问题和痛点反馈给平台研发者形成正向循环。

![](/img/blog/2022-09-16-learn-from-scale-practice/devops-cycle.png)


## 2. 专用语言：工程化方式的一极

有什么比一种专用语言更适合开放的、自服务的、面向领域业务的问题定义，同时需要满足自动化、低安全风险、低噪音、易治理的企业内部要求吗？正如记录音乐有五线谱，存储时间序列数据有时序数据库一样，在平台工程的特定问题域内，一批配置和策略语言用于编写和管理规模化复杂配置及策略。不同于混合编写范式、混合工程能力的高级通用语言，这类专用语言的核心逻辑是以收敛的有限的语法、语义集合解决领域问题近乎无限的变化和复杂性，将规模化复杂配置和策略编写思路和方式沉淀到语言特性中。

在蚂蚁的平台工程实践中，我们强化了客户端的工作方式，将围绕应用运维生命周期的模型、编排、约束和策略稳定、可扩展的通过专用语言  [KCL](https://github.com/KusionStack/KCLVM)  编写维护在共享仓库 [Konfig](https://github.com/KusionStack/konfig) 中。 KCL 是一种面向有编程能力的应用研发者的静态强类型语言，提供现代高级语言的编写体验和围绕领域目的有限功能。在平台工程实践中 KCL 不是一种仅用于编写 K-V 对的语言，而是一种面向平台工程领域的专用语言。应用研发者、SRE、平台研发者面向 Konfig 协同研发，通过 KCL 原生功能编写应用配置，以及在 PaaS 领域更为高频和复杂的[模型](https://kusionstack.io/docs/reference/lang/lang/tour/#schema)抽象、[功能函数](https://kusionstack.io/docs/reference/lang/lang/tour/#function)和[约束](https://kusionstack.io/docs/reference/lang/lang/tour/#validation)[规则](https://kusionstack.io/docs/reference/lang/lang/tour/#rule)，即编写稳定、可扩展的业务模型、业务逻辑、防错约束和环境规则。Konfig 仓库则成为统一的编程界面，工作空间和业务层载体，而基于 KCL 的安全、低噪音、低副作用、一致的编写范式更有利于长期管理和治理。

![](/img/blog/2022-09-16-learn-from-scale-practice/kcl-dev.png)


## 3. 分治：解构规模化问题

分治思路是解决规模化问题的钥匙，从 MapReduce 到 Kubernetes 无不体现其功效。在规模化交付运维领域，经典运维平台试图在统一的黑盒平台产品中，以内置的统一模型、编排、provision 技术来应对全量业务场景。这样的实践可以快速启动，在小范围内奏效，但随着不同业务主体采用率提升引入差异化需求，同时随着持续变化的平台技术逐渐进入疲态。

![](/img/blog/2022-09-16-learn-from-scale-practice/classic-plats.png)

在蚂蚁的实践中，Konfig  monorepo 是内部工程平台向研发者开放的编程界面和工作空间，帮助应用研发者以统一的编程界面编写围绕应用运维生命周期的配置和策略，从而编排和使用存量和新增的平台基础设施，按需创建管理云原生环境以及基于 RBAC 的权限，并通过 GitOps 方式管理交付过程。Konfig monorepo 为不同场景、项目、应用提供了独立的白盒的编程空间，其内生的扩展性来源于：

- 灵活、可扩展、独立的客户端的 [工程结构设计](https://kusionstack.io/docs/user_docs/concepts/konfig)
- 独立配置块 [自动合并技术](https://kusionstack.io/docs/reference/lang/lang/tour/#-operators-1)支持任意分块、可扩展的配置块组织
- [静态类型系统](https://kusionstack.io/docs/reference/lang/lang/tour/#type-system)技术提供现代编程语言可复用、可扩展的类型化建模和约束功能
- 项目粒度的 GitOps CI 工作流程定义支持
- 基于 [Kusion](https://github.com/KusionStack/kusion) 引擎的 provision 技术选择

Konfig monorepo 提供了分治的、可组合的工程结构设计、代码组织、建模方式、工作流程定义和 provision 技术选择支持，同时又以一致的研发模式和工作流承载了可扩展的业务需求。这样客户端的工作方式在保证灵活性、可扩展性、可移植性的同时也降低了对服务端扩展机制，如 Kubernetes API Machinery，持续增长的压力。

下图示意了一种 Konfig  monorepo 中 GitOps 方式的典型的自动化工作流程，从面向应用的代码变更开始，通过可配置的 CI、CD 过程触达运行时，这样的机制相比中心化的黑盒产品方式更加开放、可定制、可扩展，也免去了针对不同业务场景、不同项目、应用设计笨拙的配置文件管理 portal 的必要。

![](/img/blog/2022-09-16-learn-from-scale-practice/d-c-overview.png)


## 4. 建模：边际收益和长尾问题

有了分治的白盒化的工程结构设计、代码组织方式、建模方式、工作流程定义和 provision 技术选择，以怎么的策略面向平台 API 工作是另一个需要考虑的问题。在企业内典型的争议在于直面平台细节还是设计一种抽象，或者上升到显式（explicit）和隐式（implict）的理念的争议。

抽象的隐式的方式是运维平台工程师们面向非专家型终端用户的普遍选择，他们希望能设计出易于理解、便于使用的应用模型或 Spec 抽象，与具体的平台技术细节隔离，降低用户认知负担，并通过降低细节感知防错。但大部分运维平台的研发者倾向于设计一种强大的、统一的应用模型或 Spec 抽象，在实践中往往会遇到这些阻碍：

- 随着企业内不同业务主体采用率的提升，统一建模难以落地。在蚂蚁内部最典型的案例是 Infra 基础技术类组件和 SaaS 应用间的巨大差异性，SaaS 应用便于统一，Infra 应用往往需要单独设计。
- 面向企业内大量的平台技术，统一模型自身难以稳定，特别是应对持续变化的业务需求和平台技术驱动的需求增长。在蚂蚁的实践中，交付运维受多种因素影响有较强的不稳定性，同时围绕应用的 deliverable、runtime、security、instrumentation 的业务需求也在增长。以 instrumentation 为例，近两年对应用运行时可观察性、SLO 定义的需求快速增长直接驱动了终端用户使用的变化。
- 抽象模型的共性问题是需要面向用户设计出合理的模型，面向平台 API 细节保持同步。

在蚂蚁的实践中，面向终端用户即应用研发者我们采用了抽象模型的方式，通过如下思路解决几个关键问题：

- 面向典型应用场景（如蚂蚁的 Sofa 应用）建模，这些模型由平台研发者、平台 SRE 主导开发，与应用研发者共同维护，达到用户体验、成本和标准兼容的平衡，在蚂蚁的实践中抽象模型的信息熵收敛比约为 1：5，通过广泛的高频使用保证建模投入的边际收益。
- 对于非典型用户场景或应用，由平台研发者、平台 SRE 支持应用研发者完成针对应用的模型设计。KCL [schema](https://kusionstack.io/docs/reference/lang/lang/tour/#schema) 和 [mixin](https://kusionstack.io/docs/reference/lang/lang/tour#protocol--mixin) 等机制帮助用户建模、抽象、继承、组合、复用，减少重复代码，事实上这样的建模设计工作也是应用 PaaS 领域的重点之一，但对于这样的场景我们需要更合理的分工。最终大量 “非标” 平台技术在蚂蚁内部首次以一致的方式被纳管，有效解决了长尾问题。在典型协同模式下，平台研发者、平台 SRE 编写平台能力基础组件成为 “Enabler”，帮助应用研发者使用平台能力基础组件快速“搭积木”，完成其应用模型的研发工作。
- 面向平台技术，我们提供了平台 API Spec 到 KCL 类型代码的[生成技术](https://kusionstack.io/docs/reference/cli/openapi/)，并通过[组合编译技术](https://kusionstack.io/docs/reference/lang/lang/tour/#multi-file-compilation)原生支持对不同 Kubernetes API 版本的编译时选择，在内部实践中解决了应用抽象模型面向不同版本 Kubernetes 集群工作的灵活需求。同时，KCL 支持 [in-schema 约束](https://kusionstack.io/docs/reference/lang/lang/tour/#validation)和独立环境[规则](https://kusionstack.io/docs/reference/lang/lang/tour/#rule)的编写。此外，KCL 还提供了 [deprecated 装饰器](https://kusionstack.io/docs/reference/lang/lang/tour/#decorators)支持对已下线模型或模型属性的标注。通过在客户端健壮的、完备的模型和约束机制，在编译时暴露如配置错误、类型漂移等常见问题。相对于运行时左移的发现问题，避免推进到集群时发生运行时错误或故障，这也是企业内，特别是高风险等级企业，对生产环境稳定性的必须要求。

对于基础平台技术的专家型用户，他们通常非常熟悉特定的技术领域，更希望以直面平台细节的显式的方式工作，语言提供必要的动态性和模块化支持，通过类型和约束机制保证稳定性。但这种显式的方式无法解决专家用户不熟悉跨领域平台技术使用细节的问题，也不能解决面向平台技术的扩展性和复杂性叠加的问题。在蚂蚁内部小范围基于 YAML 的显式的工程实践中，面向大量高度开放、可配置的平台技术，复杂性随着平台技术使用率持续叠加，最终陷入难以阅读、编写、约束、测试及维护的僵化状态。

## 5. 自动化：新的挑战

运维自动化是基础设施运维领域的经典技术范畴，随着云原生理念及技术的推波助澜，可以被自动化集成成为企业运维实践的基本要求，开源开放、高度可配置的 CI、CD 技术逐步被企业采纳，黑盒的、无法被集成的 “产品” 方式逐步被灵活的可编排方式弱化并替代。这种实践的主要优势在于其强大的自定义编排和链接能力，高度的可扩展性和良好的可移植性。特别是在 Kubernetes 生态，GitOps 方式有更高的采用率，与可配置的 CI、CD 技术有天然的亲和性。这样的变化也在推进以工单和运维产品为中心的工作流逐步转变为以工程效率平台为中心的自服务工作流，而生产环境的运维能力则成为了工作流中面向生产自动运维的一个重要环节。在开源社区，面向不同研发效率平台的抽象层技术创新也在活跃进行中，平台侧研发者希望通过最短的认知和实践路径打通应用到云环境的 CI、CD 过程。

在蚂蚁的工程实践中，工程效率平台深度参与了 Konfig monorepo 的开放自动化实践，我们的实践方向也与工程效率平台技术演进方向高度一致。在从几人到几十人再到几百人的协同工作中，面向运维场景的工作流设计，高频的代码提交和 pipelines 执行，实时自动化测试和部署过程，这些对服务于单库的工程效率平台造成了很多的挑战。特别是 monorepo 中多样化的业务需要独立且强大的工作流自定义和操作支持，也需要高实时性、强 SLO 保障的并行的工作流执行能力，这些需求与单库模式的需求有巨大的差异，也给我们制造了很多麻烦。大部分配置语言是解释型语言，而 KCL 被设计为一种编译型语言，由 Rust、C、LLVM 优化器实现，以达到对规模化 KCL 文件提供[高性能](https://kcl-lang.io/blog/2022-declarative-config-overview#35-performance)编译和运行时执行的目标，同时支持编译到本地码和 wasm 以满足不同运行时的执行要求。另外 Git 的存储及架构设计不同于 [Citc/Piper](https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext) 架构，不适用于规模化代码的 monorepo，所幸今天我们的代码量还没有遇到很大的问题。我们正在一起工作解决这些问题，希望随着实践的深入逐步解决他们。

## 6. 协同和文化：更重要的事

以上的技术、工具、机制都非常重要，但我必须要说，对于工程化、Devops 更重要的是团体与团队的协同、合作和分享的文化，因为这是一种由人组成的工作，人和文化是其中的关键。在企业内，如果部门墙、团队壁垒丛生，流行封闭糟糕的工程文化，我们通常会看到大量私有的代码库和私有文档，小群体的判断和工作方式，本该紧密合作的团队以各自目标为导向各行其是，在这样的文化下我认为一切规模化工作都会非常困难。所以如果你所在的公司或团队想采纳规模化 Devops，我认为最重要的是做好广泛的沟通并开始文化的建设，因为这绝对不只是几个人的事，并且这很有难度且不可控。

在蚂蚁的实践中，初期总有各种各样的困难，大家对自服务机制和协同文化的担心尤为突出，例如 “我居然要写代码？” “我的代码居然跟其他团队在一个仓库里？” ，“我负责的工作可不简单，这种方式行不通” 都是很典型的担忧。所幸我们最终建立了一个面向共同目标的虚拟组织，合作方和领导者给予了充分的支持，我们在理念和工作方式上达成一致并协同工作。在实践过程中，大多数工程师并不是障碍，当然他们会吐槽技术、流程和机制还不够完善，希望获得更好的体验，这无可厚非。真正的障碍首先来自于运维平台研发团队自身，我看到一些公司的 Devops 理想最终回归到运维平台团队替代应用研发者做掉所有工作，甚至不让用户接触到代码和工具链这些生产工具，急于隐藏于已有的 GUI 产品界面，我认为这跑偏了，也低估了用户自身的能力和创造力。另外障碍也来自于部分平台技术团队的技术负责人，他们很难放下持续多年的已有工作，难以接受转向到新的用户服务模式。可行的办法是让他们明白这项工作的意义和远景，逐步的分阶段的影响他们。

## 7. 小结

经过一年多的实践，有 400+ 研发者直接研发参与了 Konfig monorepo 的代码贡献，管理了超过 1500 个 projects，其中平台研发者及平台 SRE 与应用研发者比例不到 1：9，这些应用研发者有些是应用 owner 本人，有些是应用研发团队的代表，这由应用团队自己决定。通过持续的自动化能力搭建，基于 Konfig monorepo 每天发生 200-300 次 commits，其中大部分是自动化的代码修改，以及大约 1k pipeline 任务执行和近 10k KCL 编译执行。在今天如果将 Konfig 中全量代码编译一次并输出会产生 300W+ 行 YAML 文本，事实上一次发布运维过程中需要多次不同参数组合的编译过程。通过轻量化，便于移植的代码库和工具链，我们完成了一次意义重大的外部专有云交付，免去了改造、移植输出一系列老旧运维平台的痛苦。在蚂蚁内部我们服务了几种不同的运维场景，正在扩大应用规模并探索更多的可能性。

最后我想说一说下一步的计划，我们的技术和工具在易用性和体验上还有很大的提升空间，需要更多的用户反馈和持续的改进，用户体验工作没有快速路径。在测试方面，我们提供了简单的集成测试手段，起到了冒烟测试的作用，但这还不够，我们正在尝试基于约束、规则而非测试的方式保证正确性。在工作界面方面，我们希望构建基于 IDE 的线下工作空间，持续规约、优化内部线上产品体验和工作流程。同时我们希望持续提升覆盖范围和技术能力。另外我们也希望将实践方式更广泛的应用在 CI 构建，自动化运维等场景，缩短终端用户的信息感知和端到端工作流程。目前KusionStack还处于刚刚开源的非常早期的阶段，在未来还有大量的工作要做。最重要的是我们希望把我们平台工程的理念和实践分享给更多企业和团队，一起推动并见证一些有意思的变化发生。

## 8. 引用

- [https://kusionstack.io/docs/user_docs/intro/kusion-intro](https://kusionstack.io/docs/user_docs/intro/kusion-intro)
- [https://platformengineering.org/blog/what-is-platform-engineering](https://platformengineering.org/blog/what-is-platform-engineering)
- [https://internaldeveloperplatform.org/what-is-an-internal-developer-platform/](https://internaldeveloperplatform.org/what-is-an-internal-developer-platform/)
- [https://web.devopstopologies.com/#anti-types](https://web.devopstopologies.com/#anti-types)
- [https://github.com/KusionStack/kusion](https://github.com/KusionStack/kusion)
- [https://github.com/KusionStack/KCLVM](https://github.com/KusionStack/KCLVM)
- [https://kusionstack.io/docs/reference/lang/lang/tour](https://kusionstack.io/docs/reference/lang/lang/tour/#%E9%85%8D%E7%BD%AE%E6%93%8D%E4%BD%9C)
- [https://kusionstack.io/docs/user_docs/concepts/konfig](https://kusionstack.io/docs/user_docs/concepts/konfig)
- [https://kcl-lang.io/blog/2022-declarative-config-overview#35-performance](https://kcl-lang.io/blog/2022-declarative-config-overview#35-performance)
- [https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext](https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext)

