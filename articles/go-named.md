## GO 系列 - 项目命名

### 目录名

关于 Go 目录命名规范，在网上搜索相关资料，基本能找到如下两条共识：

- 全小写单词。例如 `cmd`、`internal`。
- 必要时可以使用中划线分隔。例如 `kube-scheduler`、`kube-controller-manager`。

项目本身也是一个目录，所以项目名也遵循这两条规范。

其实我们可以借鉴流行的 Go 项目，参考这些优秀的项目是如何对目录进行命名的。比如 Kubernetes 项目就非常具有参考价值。

如下是 Kubernetes 项目 `/cmd` 目录部分：

```
.
├── OWNERS
├── clicheck
├── cloud-controller-manager
├── dependencycheck
├── dependencyverifier
├── fieldnamedocscheck
├── gendocs
├── genkubedocs
├── genman
├── genswaggertypedocs
├── genutils
├── genyaml
├── gotemplate
├── import-boss
├── importverifier
├── kube-apiserver
├── kube-controller-manager
├── kube-proxy
├── kube-scheduler
├── kubeadm
├── kubectl
├── kubectl-convert
├── kubelet
├── kubemark
├── preferredimports
├── prune-junit-xml
└── yamlfmt
```

可以发现，Kubernetes 项目目录名都采用全小写单词，并且必要时可以使用中划线分隔。

不过，值得注意的是，有些目录名并没有使用中划线进行分隔，而是直接将多个单词连在了一起。所以何时使用中划线是一个选择问题，而不是固定的强制规范。

我们再看下 Kubernetes 项目 `/pkg` 目录部分:

```
.
├── OWNERS
├── api
├── apis
├── auth
├── capabilities
├── client
├── cluster
├── controller
├── controlplane
├── credentialprovider
├── features
├── fieldpath
├── generated
├── kubeapiserver
├── kubectl
├── kubelet
├── kubemark
├── printers
├── probe
├── proxy
├── quota
├── registry
├── routes
├── scheduler
├── security
├── securitycontext
├── serviceaccount
├── util
├── volume
└── windows
```

可以发现，`/pkg` 目录下所有目录名都没有使用中划线分隔。有意思的是，甚至 `kubeapiserver` 也连在了一起，回看上面 `/cmd` 目录中的写法是 `kube-apiserver`。看来 Kubernetes 项目不同目录有着各自的命名规范。

针对这点，我的理解是：`/cmd/kube-apiserver`，作为项目中组件的入口文件目录，使用中划线分隔可以起到见名知意的效果；而 `/pkg/kubeapiserver` 作为包路径不使用中划线分隔，是为了方便包的导入。

如下列举几个目录名正反示例。

正确示例：

```
cmd
internal
pkg
task
kube-scheduler
kube-controller-manager
```

反向示例：

```
CMD
kubeScheduler
KubeScheduler
kube_scheduler
```

不过凡事总有特例，我之前写过一篇文章[《如何设计一个优秀的 Go Web 项目目录结构》](http://mp.weixin.qq.com/s?__biz=MzkzMjQ1NjkyNw==&mid=2247484601&idx=1&sn=2c11bb090ef151d41b841e1541634145&chksm=c25a3409f52dbd1f455366dce3d2b14925a07218183958b64a2432d946e19a4c956251fbfadc&scene=21#wechat_redirect)，里面介绍了一个 Go 项目的标准布局。其中包含一个外部辅助工具目录 `third_party`，用来存放 fork 的代码和其他第三方工具。这里 `third_party` 就采用了下划线命名，这种算是约定俗成的命名，就无需刻意遵循目录命名规范了。

另外，关于项目本身的的命名，其实还有一些额外的规范：

- 可以是项目功能的描述。例如 `userapi`、`redis-go`、`go-swagger`。
- 也可以是一个代号（如神话人物的名字、希腊语、游戏角色等）（公司的基础组件、开源项目，都是比较适合采用代号命名的项目）。例如 `kratos`、`kubernetes`。
- 项目名要尽量避免重复，如果可能重复要添加必要的前缀或者后缀做区分。例如 `ai-user`、`ai-infra`。

如下列举几个项目名正反示例。

正确示例：

```
user
userapi
redis-go
kubernetes
```

反向示例：

```
user_api
Product
AIInfra
AiInfra
```

以下是我个人的一些建议：

- 尽量使用单数命名（不过对于约定俗称的名称比如 `configs` 可以存在特例）。
- 尽量不要使用中划线，而采用简短的单词。
- 单词长度尽量控制在 3 个单词以内。

### 包名

接下来，我们再一起探究下 Go 项目中包名的命名规范是什么。

幸运的是，对于包名，Go 官方博客给出了参考建议，也是最为权威的规范。

在 Package names 这篇 Go 官方博文中，给出了几条好的包命名原则：

- 好的包名应该简短而清晰的，具有描述性。例如 `bufio`（带缓冲的 `I/O`）比 `buf` 或 `buffer` 更好，因为它既简短又具有描述性。
- 小写单一单词，简短名词，避免冗余。避免蛇形命名（`under_scores`）或驼峰命名（`mixedCaps`）。例如 `time`、`list`、`http`。
- 明智而谨慎的使用缩写，如果缩写包名称会使其产生歧义或不清楚，请不要这样做。例如 `strconv`(string conversion)、`syscall`(system call)、`fmt`(formatted I/O)。
- 不要从用户那里窃取好名字，避免给包起一个在客户端代码中常用的名字。例如，带缓冲的 `I/O` 包名叫 `bufio`，而不是 `buf`，因为 `buf` 是表示缓冲区的一个好的变量名，常会出现在客户端程序代码中。
- 包名以及包所在的目录名，不要使用复数。例如可以是 `net/url`，而不应是 `net/urls`。
- 不要用 `common`、`util`、`shared` 或者 `lib` 这类过于宽泛且无意义的包名。记住，一个包应该只有一个目的，只有一个责任。
- 避免不必要的包名冲突，不使用常用名或标准库作为包名。
- 不同目录中的包也可以具有相同的名称。例如 `runtime/pprof` 和 `net/http/pprof` 不会产生歧义，客户端代码在导入包时可以重命名（当重命名导入的包时，本地名称同样应遵循与包名称相同的指导原则(lower case, no under_scores or mixedCaps)）。
- 按照惯例，包路径的最后一个元素是包名，如果不一致，会对代码阅读者产生困惑。例如 `import golang.org/x/time/rate`，包名为 `rate`。

Go Team 成员 David Crawshaw 在 2014 Google I/O talk 中也对包命名规范给出了建议：

- 保持包名简短而有意义。
- 不要使用下划线，它们会使包名变长。例如采用 `suffixarray` 而不是 `suffix_array`。
- 包的名称是它的类型名和函数名的一部分。例如 `bytes` 包下存在 `Buffer` 结构体，本身而言 `Buffer` 存在二义性，但客户端用户看到的是 `buf := new(bytes.Buffer)`，即包名 + 类型名，解决了二义性的问题。

同目录名规范一样，包名也存在例外的情况：

- 例如使用代码生成工具生成的代码包名称可能会存在下划线，个人建议是在导入时将其重命名为适合在 Go 代码中使用的名称。
- 以 `_test` 结尾的包名是测试代码包。

如下列举几个包名正反示例。

正确示例：

```
controller
stringset
tabwriter
```

反例：

```
MyUtil
util
time // 与标准库重名
tabWriter
TabWriter
tab_writer
```

### 文件名

前文分别介绍了 Go 项目中目录名和包名的命名规范，现在终于可以探讨 Go 文件的命名规范了，这也是本文的重头戏。

不过，对于 Go 文件的命名规范，即没有 Go 团队的官方博客作为参考，也没有著名的 Gopher 站出来分享  Go 文件到底改如何命名。

这是一个少有人特意提及的规范项，不过我在 Go 项目的 GitHub 仓库中找到了一条讨论这个问题的 `issue` doc: filename conventions。

虽然不少人参与了讨论，但遗憾的是，这个问题现在仍然没有定论。不过，这也正是此问题值得探讨的原因，也是本文的意义所在。

其实遇到规范相关问题，我们最先想到的，应该就是参考 Go 源码本身。很不巧，针对 Go 文件的命名规范问题，Go 源码做的也不够好。

正如 doc: filename conventions 问题 `issue` 提问者列举的几个 Go 文件名示例中，存在很大差异：

- `file.extension.go` 风格。例如 `cmd/internal/obj/arm/a.out.go`、`cmd/internal/obj/arm64/a.out.go`。
- `snake_case.go` 风格。例如 `cmd/internal/obj/abi_string.go`、`internal/syscall/unix/at_sysnum_linux.go`。
- `lowercase.go` 风格。例如 `cmd/compile/internal/ssa/loopreschedchecks.go`、`runtime/mksizeclasses.go`。
- `CamelCase.go` 风格。例如 `cmd/compile/internal/ssa/gen/AMD64Ops.go`、`cmd/compile/internal/ssa/gen/WasmOps.go`。

这就比较有意思了 😁。

在 `issue` 中点赞最多的是 `peterbourgon` 的评论:

> There is a de facto standard for Go source file names: all lowercase with underscore separation when necessary, i.e. snake case. (The exceptions are exceptions.) I'd appreciate having it as a documented standard, too, to answer questions like @carnott-snap notes, especially among junior Gophers. In my opinion it needn't be so formal as entering Effective Go, a short new section on CodeReviewComments is more than sufficient.

这个回答的核心是：`all lowercase with underscore separation when necessary, i.e. snake case. (The exceptions are exceptions.) `。

即大家达成了三点共识：

- 所有文件名采用小写单词 `lowercase.go`。例如 `mksizeclasses.go`。
- 必要时用下划线分隔，即蛇形命名法 `snake_case.go`。例如 `abi_string.go`。
- 例外情况除外。

前面两条好理解，例外情况都有哪些？我总结大概有如下几种：

- 以 `_test.go` 结尾的测试文件（仅供 `go test` 编译和运行）。
- 以 `.` 或 `_` 开头的“隐藏文件”（将被 `go tool` 忽略，如果你使用 GoLand 创建一个 `.a.go` 文件，GoLand 会提示 `'.a.go' is ignored by the build tool since its name starts with '.'`）。
- 具有 `OS` 和 `ARCH` 特定后缀的文件会遵守特定的约束。例如 `name_linux.go` 仅在 Linux 上构建，`name_amd64.go` 仅在 `amd64` 上构建（这与在文件顶部的 `//+build amd64` 具有相同作用）。

虽然很多人达成了上面三点共识，但其实文件命名和目录命名规范一样，存在“灰色地带” —— `必要时`用下划线分隔。

到底何时是**必要的时候**？

这也是最不统一的一点规范，我发现很多人推荐，在名称出现多个单词时采用下划线分隔。例如 `task_status.go`、`web_shell.go`。

但还有一小部分人，更推崇不采用下划线的命名方式，例如 `taskstatus.go`、`webshell.go`。

而我个人的偏好则更倾向于后者。如果单词数量为 2 个，我会倾向于不使用下划线分隔，例如 `taskstatus.go`；如果单词数量为 3～4 个，则倾向于使用下划线分隔，例如 `import_known_versions.go`、`default_storage_factory_builder.go`。

但最好的方案，仍然是尽量用简短有意义的单个单词或缩写作为文件名。

> NOTE: 我倾向于不使用下划线分隔的几点理由：
>
> - **与包名对齐**。
> - `kubernetes` 也经常这么干。
> - 你看，`testdata` 就没有使用下划线分隔 😄。
> - ... 想到了再说 :)。

如下列举几个文件名正反示例。

正确示例：

```
router.go
middleware.go
webshell.go
```

反面示例：

```
routers.go // 复数
fooBar.go
Service.go
```