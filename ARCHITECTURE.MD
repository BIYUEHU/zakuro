# Architecture

快速管理 Monorepo 下的各包版本、自动构建、自动格式化、自动 publish、自动 tag、自动 push、自动 release

- lifecycle-ready:

zakuro.json：

- $schema
- root 根目录，相对于 zakuro.json 所在目录 ./
- registry 发布地址 npmjs.org
- branch 允许的分支，string 或 string[] master
- packageMange 包管理工具 pnpm
- workspace 相关配置，无则代表不是工作区，有则是工作区 null
- workspace.main 项目主包的包名
- options.version 版本号格式 v%major%.%minor%.%%
- options.versionPre 预发布版本号格式 v%major%.%minor%.%%-pre-%order%\*
- options.versionBeta 测试版本号格式 v%major%.%minor%.%%-beta-%order%\*
- options.checkCommand 检查指令可用性 true
- options.checkBranch 检查分支 true
- options.checkChance 检查仓库变化 true
- options.lint eslint 指令，不填则不运行 pnpm lint
- options.format prettier 指令，不填则不运行 pnpm format

- 先获取工作目录： process.cwd()，判断./zakuro.json 是否存在否则返回 null，有则读取并 JSON.parse 不合法 JSON 则抛错。经由 Tsukiko.parse 检查格式是否合法并处理默认值，不合法则抛错。保存 config 数据于全局中。
- 通过 config.workspace 判断是否为 monorepo 项目，是则使用 manypkgs 包读取所有包数据并保存于全局。
- checkCommand：获取 config 的 packageMange 与钩子里用的指令的前缀，使用 shelljs 包依次检查 git 及 获取内容指令是否有效，无效则抛错。

beforeCommit：

- checkBranch：使用 git checkout 指令判断当前分支是否允许，不允许则抛错。
- checkChance：使用 git 指令获取最新的 tag 距今为止的提交发生变化文件清单。非
- checkTags：判断本地和远程 tag 是否已存在。workspace：过滤掉所有包的根目录没有处于变化文件清单中的包。不开启该检查项则会为所有包。
- workspace：列出过滤后包列表，可供选择，没有选择任何包或无变化则运行完毕

- 1.询问版本号：正式 pre beta
- 2.询问 major,minor 增加意愿
- 3.遍历过滤包的当前版本号，然后解析版本号，根据意愿变动版本号（正式则在相应位置＋ 1，beta 和 pre 则判断当前版本号与当前语义版本号是否一致，是则 beta/pre 后缀+1，否则替换为新语义版本号且 beta/pre 设为 1）
- 5.更新过滤包的 pkg.json 版本号

- 如果有，执行 lint，format...
- workspace：判断主包版本号是否有更新，是则以此作为基准，否则跳转至 add。
- 根据当前版本号加对应标签(如果有 checkTags 则先检查是否已存在，是则抛错)
