---
title: 云引擎平台功能
sidebar_label: 平台功能
sidebar_position: 2
---

import CloudCustomDomain from '../_partials/cloud-custom-domain.mdx';
import PlatformIntroduction from '../_partials/platform-introduction.mdx';
import PlatformRuntimes from '../_partials/platform-runtimes.mdx';
import EngineRuntimes from '/src/docComponents/MultiLang/engine';
import {BRAND, CLI_BINARY} from '/src/constants/env.ts';
import TabItem from '@theme/TabItem';
import CodeBlock from '@theme/CodeBlock';

<PlatformIntroduction />

:::info
这篇文档会帮助了解云引擎平台提供的功能，有关具体运行环境的详情请查看专门的文档页面：

<PlatformRuntimes />

:::

## 什么可以被部署到云引擎

云引擎是一个进程级别的运行环境，开发者只需关注应用的进程本身（如 `node` 进程或 Go 项目编译出的可执行文件）、专注于实现业务逻辑，而不需要关注操作系统级别的环境。

部署到云引擎的应用只需遵循业界通常的项目结构（例如 Node.js 项目需要有 `package.json`），云引擎就可以自动地为其准备运行环境，可以在 [总览](/sdk/engine/overview) 中查看特定运行环境的文档来了解详情。

云引擎对应用进程内部几乎没有干预，你可以选用你喜欢的开发框架和第三方组件库、自行组织项目的目录结构。云引擎的负载均衡会转发绑定域名下的所有 HTTP 请求，你可以在应用内使用 Web 框架来自行设计 HTTP API 中的路径、请求和响应的格式。

<p>目前云引擎主要为无状态的 HTTP 服务而优化，不支持在文件系统上持久地存储数据，应用可以将数据存储到 <a href={BRAND==='leancloud'?'https://leancloud.cn/docs/storage_overview.html':'/sdk/storage/features'}>数据存储</a> 或 LeanDB 提供的 <a href="/sdk/engine/database/redis/">Redis</a>、<a href="/sdk/engine/database/mongo/">MongoDB</a> 和 <a href="/sdk/engine/database/es/">Elasticsearch</a> 中。</p>

<p>要将一个已有的项目关联到云引擎应用，可以使用 <code>{CLI_BINARY} switch</code>：</p>

<CodeBlock>
{`$ ${CLI_BINARY} switch
[?] Please select an app:
 1) my-engine-app
 => 1
Switching to my-engine-app (group: web)`}
</CodeBlock>

## 查看日志

### 运行日志

在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 云引擎分组 » 日志** 中可以查看云引擎的部署和运行日志，还可以通过环境（预备环境、生产环境）、类型（标准输出、标准错误）、实例、日期时间进行筛选。

![云引擎日志](/img/cloud-engine/engine-logs.png)

可以使用命令行工具将运行日志导出至本地，详见 [命令行工具使用指南 § 查看日志](/sdk/engine/cli#查看日志)。

### 访问日志

云引擎的访问日志（Access Log）可以在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 访问日志** 处导出。

## 查看统计数据

**开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 云引擎分组 » 统计** 页面显示出当前应用下所有实例资源的使用情况，开发者由此可以判断实例资源是否即将或已经超限。

![云引擎统计](/img/cloud-engine/engine-metrics.png)

#### 每分钟请求数

一段时间内云引擎每分钟处理的请求数。可以通过左上角的下拉菜单选择按不同请求类型（网站托管、云函数）和不同 HTTP 状态码分别查看。

#### 响应时间

若曲线图升高，说明 CPU 使用量达到或超过限制，实例的 CPU 资源紧张。同样可以按不同请求类型（网站托管、云函数）和不同 HTTP 状态码分别查看。

#### CPU

曲线图展现出在一段时间内应用实际 CPU 的使用量。如果 CPU 接近或达到限制，应用表现为请求响应时间延长。

#### 内存

曲线图展现出在一段时间内应用内存的实际使用量。如果内存使用量达到限制，则相关的业务进程（比如 Node.js 进程或者 Python 进程）会因为内存溢出（OOM）而重启，从而导致业务处理中断，并且该实例在启动期间服务不可用。如果内存曲线图<u>频繁地接近限制然后忽然下降</u>，就说明内存使用超限导致了进程重启。

**显示各实例详情（勾选框）** 勾选后会在 CPU、内存图表中为每个实例单独绘制一条线（不勾选默认显示所有实例合计的数值）。

图表右上角可以切换时间范围：今天、昨天、过去 7 天。

## 部署与发布

### 预备环境和生产环境

标准版云引擎对每个分组提供了「生产环境」和「预备环境」，生产环境设计用来接收和处理线上请求；预备环境则提供了和生产环境几乎相同的环境、访问相同的数据，可以绑定单独的域名供开发者进行测试。LeanCloud 的客户端 SDK 在调用云函数或进行可能触发 Hook 的数据存储操作时，可以设置请求的环境（`X-LC-Prod`）。

在开发过程中，你可以先将改动部署到预备环境，使用线上数据测试通过后再发布到生产环境。如果你希望有一个独立数据源的测试环境，建议单独创建一个应用。

体验版云引擎没有预备环境。

### 命令行部署

在你的项目根目录运行：

<CodeBlock className='sh'>
{`${CLI_BINARY} deploy`}
</CodeBlock>

就会开始上传本地的代码，体验版云引擎会直接部署到生产环境；标准版云引擎则会先部署到一个与生产环境几乎完全相同的预备环境以供测试。

如需直接部署生产环境可以加上 `--prod 1` 这个参数，更多命令行工具的用法请查看 [命令行工具使用指南 § 从本地代码部署](/sdk/engine/cli#从本地代码部署)。

### Git 部署

云引擎还支持从 Git 仓库拉取代码进行部署，包括 [GitHub](https://github.com/) 或 [Gitee](https://gitee.com/) 这样的第三方托管平台，和 [GitLab](https://about.gitlab.com/install/) 这样自己搭建的 Git 托管服务。你可以在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 你的分组 » 部署 » Git 部署** 中设置 Git 仓库的地址。

云引擎支持 SSH 协议的私有仓库（如 `git@github.com:leancloud/node-js-getting-started.git`），你需要在设置 Git 仓库地址后在 Git 托管服务处为云引擎配置 deploy key。如果你的仓库是公开的，则推荐使用 HTTPS 形式的仓库地址。

设置好 Git 仓库后，就可以在部署界面点击「部署」了，默认使用 `master` 分支，你也可以手动填写分支、标签或 commit hash。

如果希望 push 到项目的 Git 仓库的特定分支后自动触发云引擎部署，可以在应用的 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 你的分组 » 部署 » Git 部署 » 自动部署** 中生成一个 deploy token，然后查看页面上显示的 Webhook 地址。当该地址收到任意 POST 请求后，会部署指定分支的代码到指定的环境，通过 GitHub Action 配置自动部署的具体方法请参考控制台上的指引。

### 部署历史与回滚

在**开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 云引擎分组 » 部署**页面可以分别查看预备环境和生产环境的历史部署。
每个历史部署版本会显示简短描述（基于 git 提交日志等信息）、部署版本号、部署时间。
历史部署按部署时间倒序排列，当前部署排在最前。
点击**回滚至该版本**按钮，可以回滚至相应的部署版本。
点击**查看更多历史部署**可以查看最近部署的 10 个版本。
除了按预备环境和生产环境分别查看历史部署外，点击右上角的**查看部署时间线**则可以按照时间顺序（最新部署在前）查看部署到云引擎的各个版本。

除了查看部署历史外，这里还会显示预备环境或生产环境的部署状态（「休眠中」、「部署中」、「运行中」）。
通过右上角的按钮还可以**重启**（重新部署当前版本）或**清除部署**（移除部署，注销相应的云函数和 Hook）。
预备环境还可以点击右上角的**部署到生产环境**按钮将最近部署到预备环境的版本发布到生产环境。
当部署状态为「部署中」时，控制台会显示部署进行中的一些信息，也会显示一个**取消部署**按钮，点击可以取消部署。

### 分组管理

云引擎支持在同一个应用下创建多个「分组」，运行不同的后端程序，在访问同一数据源的情况下，部署多套不同的服务器端业务代码，并对每个分组绑定不同的自定义域名，实现更复杂的业务需求：

- 将用户界面和管理后台拆分为不同的项目，使用不同的域名。
- 单独部署主系统之外的边缘支持系统，以免边缘系统出现问题时影响主系统。
- 使用不同的服务器端语言来编写云函数和网站，例如你可以使用 Node.js 编写云函数，而用 PHP 来实现网站。

每个分组都有独立的预备环境用于测试代码、独立的域名供外部访问，每个分组的环境变量、代码仓库等设置也是独立的，你可以单独对一个组部署代码。

每个分组都可以部署云函数、Hook 和定时任务。但如果当前部署的代码中有和其他组同名的云函数，会中断部署，需勾选[「强制覆盖同名云函数」选项](#部署选项) 强制部署。

每个应用默认有一个 `web` 分组，你可以在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 组管理** 中创建额外的分组。新建的分组默认不包含任何实例（无法响应请求），你需要 [调整实例规格和数量](#调整实例规格和数量)。

### 部署选项

云引擎在进行部署时提供了一些选项：

#### 不使用缓存（`--no-cache`）

在遇到与依赖安装有关的问题时，可以尝试开启该选项，禁用加速构建的 [缓存机制](/sdk/engine/deep-dive#构建)。

#### 打印构建日志（`--options 'printBuildLogs=true'`）

在遇到与构建有关的问题时，可以尝试开启该选项来在日志中打印构建过程的详细日志。

#### 强制覆盖同名云函数（`--overwrite-functions`）

在多个分组的云函数或 Hook 重复时，可以开启该选项强制进行部署（以最后一次部署的分组中的云函数或 Hook 为准），通常用于在分组之间移动云函数或 Hook。

## 管理资源和容量

### 体验版云引擎

云引擎对于每个应用都默认赠送一个 0.5 CPU、256 MB 的体验实例，可以免费使用，供开发者学习和测试云引擎。

**体验实例会执行休眠策略**，没有请求时会休眠，有请求时启动（首次启动可能需要十几秒的时间），每天最多运行 18 个小时。

<details>
<summary>点击展开体验实例休眠详情</summary>

- 如果应用最近一段时间（半小时）没有任何外部请求，则休眠。
- 休眠后如果有新的外部请求实例则马上启动。访问者的体验是第一个请求响应时间是 5 ~ 30 秒（视实例启动时间而定），后续访问响应速度恢复正常。
- 强制休眠：如果最近 24 小时内累计运行超过 18 小时，则强制休眠。此时新的请求会收到 503 的错误响应码，该错误可在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 云引擎分组 » 统计** 中查看。

</details>

### 标准版云引擎

云引擎的计费独立于开发版、商用版方案，开通或取消商用版不影响云引擎的实例和计费。

:::tip
对于商业项目和正式上线的产品，我们建议开发者升级到标准版云引擎，并使用至少两个实例，来保证业务的可用性。
:::

标准实例不会像体验实例一样在没有请求时休眠，会一直保持运行；标准实例配有预备环境方便测试；如果购买了两个或更多的实例，还能进行负载均衡和故障切换，充分保障服务的可用性。

标准版的功能包括：

#### 负载均衡（高可用）

云引擎的网关会将客户端的请求轮流分配给每个实例，随着业务请求量的增加，开发者可以简单地通过增加实例数量来提升处理能力。

在同一分组的同一环境中有两个或更多的实例时，便可实现故障切换。当其中一个实例出现故障无法工作时，云引擎的网关会自动将接下来的请求转发到其他可以正常工作的实例上，等待故障的实例恢复后复原。

#### 预备环境

云引擎会为每个标准版分组赠送一个预备环境，它有着和生产环境几乎完全一样的运行环境。在正式上线前，开发者可以先将代码发布到预备环境，使用线上的环境和数据进行模拟测试。

预备环境规格与生产环境相同，与体验实例一样执行休眠策略。

#### 平滑部署

在部署新版本或其他运维操作时，系统会让新旧版本的实例同时运行一段时间，再关闭旧版本的实例，让服务保持零中断。

#### 多分组

多实例分组可以实现在访问同一数据源的情况下，部署多份云引擎代码，满足不同的业务需求。每个组可以绑定独立域名。详见 [分组管理](#分组管理)。

### 调整实例规格和数量

我们为标准版实例提供了几种不同的规格，差别主要体现在可供使用的内存上：

| 规格          | 内存    | CPU    |
| ------------- | ------- | ------ |
| standard-512  | 512 MB  | 1 Core |
| standard-1024 | 1024 MB | 1 Core |
| standard-2048 | 2048 MB | 1 Core |
| standard-4096 | 4096 MB | 1 Core |

在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 你的分组 » 资源** 页面下，可以修改实例的规格和数量。

:::tip
我们建议根据自己的程序运行时所需要的最大内存来选择实例规格，然后通过调整实例数量来应对请求量的增加。对于商业项目和正式上线的产品，我们建议使用至少两个实例，来保证业务的可用性。
:::

为了防止实例因为资源使用超限而受到影响，我们建议开发者经常 [查看统计数据](#查看统计数据)：

- 一天内平均 **内存** 使用超过可用资源的 **70%**（例如对于 1 个 standard-1024 来说就是 717 MB）就建议提高实例规格。
- 一天内平均 **CPU** 使用超过可用资源的 **30%**（例如对于 1 个 standard-1024 来说就是 30% CPU）就建议增加实例数量。

<details>
<summary>点击展开多实例运行详情</summary>
当一个分组中的一个环境里有多个实例时，我们称之为「多实例运行」。在部署或者运维操作（如重启）期间，也会有多个实例短暂地同时运行。

多个实例的内存和文件系统是独立的，这意味着在一个实例中写入全局变量或文件，其他实例无法读取，建议在首次切换到多实例运行时进行充分的测试。

如果需要在多个实例间低延迟地共享数据，可以使用 LeanCache。
</details>

### 实例计费规则
标准版云引擎按照所选择的规格乘以数量来进行计费，每天会按照前一天最大的实例数量来扣除前一天的费用。各规格的价格可以在当前节点的价格页面查看，扣费记录可在云服务控制台的消费明细中查看。

如果不想继续付费，可以在所有分组中将实例规格修改为「免费版」，标准实例会被删除，并在最后一个分组下赠送一个体验实例。在次日会进行最后一次扣费，之后便不会再产生费用。

## 调整设置

### 环境变量

我们推荐使用环境变量向运行在云引擎上的应用注入配置，你可以在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 你的分组 » 设置 » 自定义环境变量** 中添加自定义环境变量，修改环境变量后会在下一次部署时生效。

勾选了 Secret 的环境变量在保存后便不会在控制台显示，可以用在类似密钥、密码的字段上减少意外泄漏的可能。

云引擎运行环境默认提供的环境变量无法被自定义环境变量覆盖。

你也可以在 **开发者中心 » 你的游戏 » 游戏服务 » 云服务 » 云引擎 » 云引擎分组 » 设置 » 自定义环境变量** 页面中添加自定义的环境变量。其中名字必须是字母、数字、下划线且以字母开头，值必须是字符串，修改环境变量后会在下一次部署时生效。

### 绑定自定义域名

<CloudCustomDomain />