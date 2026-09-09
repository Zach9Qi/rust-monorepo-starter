<div align="center">

# 🦀 Rust Monorepo Starter

面向 **crates.io 发布**的 **Rust 多 crate 工作区(monorepo)脚手架与 GitHub 模板仓库**。  
`crates/` 初始为空、不带任何示例代码;工作区级依赖与 lint 统一管理、一条命令生成符合约定的 crate、严苛的多平台 CI 质量门禁(含拼写检查与发布演练)、Keep a Changelog 驱动的一条命令发版,以及推 tag 后自动发布到 crates.io + GitHub Release(可选附带五目标交叉编译的二进制包)。

[![Rust 2024](https://img.shields.io/badge/Rust-Edition_2024_(MSRV_1.85)-DEA584?style=flat-square&logo=rust&logoColor=black)](https://www.rust-lang.org/)
[![Cargo Workspace](https://img.shields.io/badge/Cargo-Workspace-000000?style=flat-square&logo=rust&logoColor=white)](https://doc.rust-lang.org/cargo/reference/workspaces.html)
[![Clippy pedantic](https://img.shields.io/badge/Clippy-pedantic-F74C00?style=flat-square)](https://doc.rust-lang.org/clippy/)
[![cargo-deny](https://img.shields.io/badge/cargo--deny-audited-4B8BBE?style=flat-square)](https://embarkstudios.github.io/cargo-deny/)
[![typos](https://img.shields.io/badge/typos-checked-2E8B57?style=flat-square)](https://github.com/crate-ci/typos)
[![Keep a Changelog](https://img.shields.io/badge/Keep_a_Changelog-1.1.0-E05735?style=flat-square)](https://keepachangelog.com/zh-CN/1.1.0/)
[![CI](https://github.com/your-name/rust-monorepo-starter/actions/workflows/ci.yml/badge.svg)](https://github.com/your-name/rust-monorepo-starter/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](https://opensource.org/licenses/MIT)

[✨ 使用此模板新建项目](#-基于模板新建项目) · [🚀 快速开始](#-快速开始) · [📋 定制清单](#第二步新项目必改清单) · [📐 架构规范](#-工程规范与架构设计) · [📦 发版与 crates.io](#-自动化发版与-cicd)

</div>

---

## 🌟 核心特性 (Features)

不同于 `cargo new` 出来的单 crate Hello World,本脚手架定位为**直接面向生产环境与 crates.io 发布的工程化起步模板**,为你抹平多 crate Rust 库项目初期的大量重复配置——同时**不塞任何示例目录**,你的仓库从第一天起就只有自己的代码:

- 🧱 **工作区级统一管理**:版本号、`edition`、`license`、`rust-version` 全部在根 `Cargo.toml` 的 `[workspace.package]` 声明一处,成员 crate 用 `xxx.workspace = true` 继承;依赖版本表集中在 `[workspace.dependencies]`,升级依赖只改一行。
- 🧩 **一条命令生成合规 crate (`cargo xtask new`)**:`crates/` 初始为空,`cargo xtask new <名字>` 生成**可直接发布到 crates.io 的库 crate**(`description` / `keywords` / `categories` / `readme` / `documentation` / `include` / docs.rs 元数据齐全,自带 `thiserror` 错误枚举与独立 `README.md`,并以 `{ path, version }` 形式登记到工作区依赖表);`--bin` 生成二进制 crate(`clap` + `tracing` + `assert_cmd` 集成测试,`publish = false`)。约定靠生成器落地,而不是靠示例代码。
- 📦 **crates.io 自动发布(Trusted Publishing)**:推 tag 后 `cargo publish --workspace` 按依赖顺序发布全部可发布成员,认证走 crates.io 的 OIDC Trusted Publishing,**仓库里不存长期 API token**;CI 每次 PR 都跑 `cargo publish --dry-run` 演练,漏 `include` 文件、内部依赖没写 `version` 这类「本地能编、发布后编不过」的问题拦在合并前。
- 📝 **Keep a Changelog 驱动发版**:`CHANGELOG.md` 的 `[Unreleased]` 为空时**拒绝发版**;`cargo xtask release` 自动把条目切到 `## [x.y.z] - 日期` 段落、维护底部比较链接,该段落直接成为 GitHub Release 说明。
- 🔤 **拼写检查 (`typos`)**:源码、注释、文档、配置全覆盖,毫秒级;`typos.toml` 管白名单,本机装了 `typos-cli` 时 `cargo xtask ci` 也会跑。
- 🛡️ **强类型错误与日志门面**:库 crate 用 `thiserror` 派生 `Error`、`Display` 直出中文文案;二进制用 `anyhow` 汇总并在 `main` 统一打印。日志只用 `tracing`,`-v` / `RUST_LOG` 控制级别、写 stderr 不污染 stdout。
- 🔒 **工作区级 lint 基线**:`[workspace.lints]` 开启 `clippy::pedantic` + `unwrap_used` / `expect_used` / `print_stdout` / `dbg_macro` 等,`unsafe_code = "forbid"`,`missing_docs = "warn"`;CI 以 `-D warnings` 运行,零警告容忍。
- ✅ **一条命令本地门禁 (`cargo xtask ci`)**:与 CI 一致的 `typos` → `fmt` → `clippy` → `test` → `rustdoc`(警告即失败)串行执行,提交前先在本地过一遍。
- 🚀 **一条命令自动化发版 (`cargo xtask release`)**:工作区洁净度、分支一致性、远端同步、tag 防重、版本回退、更新日志六项安全检查,同步根版本与内部依赖版本、离线刷新 `Cargo.lock`、切更新日志,生成规范提交与附注 tag 并推送;支持 `--dry-run` / `--no-push`。
- 🤖 **工业级 GitHub Actions 流水线**:
  - **CI 门禁 (`ci.yml`)**:typos、fmt / rustdoc、clippy / test 三平台并行(全部 `--locked`)、**MSRV** 按声明的 `rust-version` 实际编译、**cargo-deny** 依赖审计、**crates.io 打包演练**;
  - **发版 (`release.yml`)**:verify(版本 / 日志 / 测试)→ 可选的五目标二进制打包 → 发布 crates.io → 创建带日志说明的 GitHub Release;任一环节失败后续不执行。
- 🔁 **Dependabot 周更**:cargo 依赖与 GitHub Actions 版本(含钉死版本的 typos)分组自动升级。

---

## 🧰 技术栈概览

| 模块 | 技术选型 | 版本 / 说明 |
|---|---|---|
| **语言与工具链** | [Rust](https://www.rust-lang.org/) stable | Edition 2024 (MSRV 1.85),`rust-toolchain.toml` 自动带 `rustfmt` / `clippy` |
| **项目组织** | [Cargo Workspace](https://doc.rust-lang.org/cargo/reference/workspaces.html) | resolver 3,`[workspace.package]` / `[workspace.dependencies]` / `[workspace.lints]` 统一继承 |
| **发布** | [crates.io Trusted Publishing](https://crates.io/docs/trusted-publishing) + `cargo publish --workspace` | OIDC 短期令牌,按依赖顺序发布全部成员;PR 阶段 `--dry-run` 演练 |
| **更新日志** | [Keep a Changelog 1.1.0](https://keepachangelog.com/zh-CN/1.1.0/) | `[Unreleased]` 驱动发版,段落即 Release 说明 |
| **拼写检查** | [typos](https://github.com/crate-ci/typos) | `typos.toml` 白名单;CI 钉死版本,dependabot 升级 |
| **错误处理** | [thiserror](https://docs.rs/thiserror) + [anyhow](https://docs.rs/anyhow) | 库层强类型枚举,二进制层上下文链汇总(预置在依赖表) |
| **命令行解析** | [clap](https://docs.rs/clap) v4 | derive 宏,文档注释即 `--help` 文本(预置在依赖表) |
| **日志** | [tracing](https://docs.rs/tracing) + [tracing-subscriber](https://docs.rs/tracing-subscriber) | `EnvFilter`,`RUST_LOG` 优先,`-v` 兜底(预置在依赖表) |
| **测试** | `cargo test` + [assert_cmd](https://docs.rs/assert_cmd) + [predicates](https://docs.rs/predicates) | 单元测试 + 二进制集成测试(预置在依赖表) |
| **代码质量** | rustfmt + Clippy pedantic + rustdoc `-D warnings` | 格式 / 静态诊断 / 文档三重门禁 |
| **依赖审计** | [cargo-deny](https://embarkstudios.github.io/cargo-deny/) | RustSec 通告、许可证白名单、重复依赖、来源白名单(`deny.toml`) |
| **仓库自动化** | [xtask 模式](https://github.com/matklad/cargo-xtask) | `cargo xtask new / ci / version check / changelog notes / release`,零外部运行时 |
| **持续集成** | [GitHub Actions](https://github.com/features/actions) | 六项 CI 门禁 + 四阶段发版流水线 |

> 「预置在依赖表」指只在根 `Cargo.toml` 的 `[workspace.dependencies]` 登记了版本,没有任何 crate 引用它们时不会进入 `Cargo.lock`,用不上可以直接删。

---

## 🚀 快速开始

### 前置准备

1. **[Rust](https://rustup.rs/)**:进入本仓库时,`rust-toolchain.toml` 会自动切换至 stable(>= 1.85)并补齐 `rustfmt` 与 `clippy`。
2. **(可选)** `cargo install typos-cli cargo-deny --locked`:本地复现 CI 的拼写检查与依赖审计;没装时 `cargo xtask ci` 会跳过 typos 并提示。

### 生成第一个 crate

```bash
# 1. 生成一个库 crate(要发到 crates.io 的那种),自动登记到 [workspace.dependencies]
cargo xtask new my-core

# 2. (可选)生成一个二进制 crate(命令行入口,不发 crates.io),可执行文件名同 crate 名
cargo xtask new my-cli --bin

# 3. 让 my-cli 依赖 my-core:在 crates/my-cli/Cargo.toml 的 [dependencies] 加一行
#    my-core.workspace = true

# 4. 构建一次让 Cargo.lock 记录新成员,然后跑一遍与 CI 一致的门禁
cargo build
cargo xtask ci

# 5. 在 CHANGELOG.md 的 [Unreleased] 下记一笔,然后提交
```

> [!TIP]
> `cargo xtask` 是 `.cargo/config.toml` 里定义的别名,等价于 `cargo run --package xtask --`。首次调用会编译 xtask,之后秒级启动。

---

## 📋 基于模板新建项目

本仓库是标准的 **GitHub Template**。请按照以下步骤将其转化为你的专属生产项目:

### 第一步:创建新仓库

点击本仓库右上角的 **`Use this template`** 按钮(或选择 **`Create a new repository`**),填写你的新仓库名称并克隆到本地。

### 第二步:新项目必改清单

为防止占位符遗留,请依次替换以下位置的标识(代码中均包含 `【新项目必改】` 提示):

| 目标文件 | 需替换字段 | 示例 / 说明 |
|---|---|---|
| `Cargo.toml` | `[workspace.package]` 的 `authors`、`repository` | 如 `authors = ["Your Name <you@example.com>"]`,`repository = "https://github.com/you/my-lib"`。`repository` 还用于生成 CHANGELOG 的比较链接 |
| `CHANGELOG.md` | 底部 `[Unreleased]:` 链接、初始条目 | 换成你的仓库地址;首次发版时脚本会重写该链接 |
| `.github/workflows/release.yml` | 顶部 `env.PKG_NAME`、`env.BIN_NAME`(可选) | 只有要分发安装包的二进制 crate 才填;纯库项目保持为空,自动跳过二进制打包 |
| `README.md` | 项目名、CI 徽章 URL、本节内容 | 换成你自己的项目说明 |
| `LICENSE` | 版权人 | MIT 版权声明 |

> [!NOTE]
> 版本号**不需要**逐个 crate 改:全部成员通过 `version.workspace = true` 继承根 `Cargo.toml` 的 `[workspace.package].version`,`cargo xtask new` 生成的 crate 已经这样写;内部依赖的 `version` 由 `cargo xtask release` 同步。

### 第三步:验证并提交

```bash
cargo xtask version check
cargo xtask ci

git add -A
git commit -m "chore: initialize project from rust-monorepo-starter template"
```

### 第四步:接通 crates.io(首次发版前)

1. 每个库 crate 的**第一个版本**先在本机手动发布一次(crates.io 上得先有这个 crate 才能配置信任发布):`cargo login` 后 `cargo publish -p <名字>`(多个 crate 按依赖顺序,或直接 `cargo publish --workspace`);
2. 到 crates.io → 该 crate → **Settings → Trusted Publishing**,添加 GitHub 发布者:仓库 owner / name、workflow 文件名 `release.yml`,Environment 留空;
3. 之后每次 `cargo xtask release` 推 tag,`release.yml` 会用 OIDC 短期令牌自动发布,无需在仓库里存 API token。

> 不想用 Trusted Publishing:在 `release.yml` 的 `publish-crates` job 里删掉 `crates-io-auth-action` 步骤,把 `CARGO_REGISTRY_TOKEN` 改为 `${{ secrets.CARGO_REGISTRY_TOKEN }}` 并在仓库 Secrets 里配置 token。

---

## 📂 项目结构

```text
.
├── .cargo/
│   └── config.toml             # cargo 别名 `cargo xtask`;Linux ARM64 交叉编译链接器
├── .editorconfig               # 编辑器基础约定:UTF-8 / LF / 缩进
├── .gitattributes              # 文本一律 LF;Cargo.lock 冲突不做逐行合并
├── .github/
│   ├── dependabot.yml          # cargo 依赖与 GitHub Actions 周更,分组降噪
│   └── workflows/
│       ├── ci.yml              # 持续集成:typos + fmt/doc + 三平台 clippy/test + MSRV + cargo-deny + publish --dry-run
│       └── release.yml         # 发版:verify → 可选二进制打包 → crates.io(Trusted Publishing)→ GitHub Release
├── .pi/                        # pi 编码助手的项目级设置
├── .vscode/                    # 编辑器推荐扩展与工作区配置 (rust-analyzer 保存时跑 clippy)
├── crates/                     # 业务 crate 全部放这里;初始为空,用 `cargo xtask new` 生成
│   └── .gitkeep
├── xtask/                      # 仓库自动化(cargo 别名 `cargo xtask`,不发布)
│   └── src/
│       ├── new_crate.rs        # `cargo xtask new`:按约定生成库 / 二进制 crate 并登记依赖
│       ├── changelog.rs        # CHANGELOG.md 读写:Unreleased 校验、切段、提取 Release 说明
│       ├── ci.rs               # `cargo xtask ci`:typos → fmt → clippy → test → doc 门禁
│       ├── git.rs              # git 子进程封装(只读查询 / 写操作)
│       ├── release.rs          # `cargo xtask release`:安全检查 → 版本 → 日志 → commit → tag → push
│       ├── version.rs          # 版本读写(含内部依赖 version)、Cargo.lock 刷新、一致性校验
│       └── main.rs             # 子命令分派
├── AGENTS.md                   # AI 编码助手工作指南(分层约束、lint 策略、发版规则)
├── CHANGELOG.md                # Keep a Changelog;[Unreleased] 为空时拒绝发版
├── Cargo.toml                  # 工作区根清单:版本 / 依赖表 / lint 规则 / release profile
├── Cargo.lock                  # 依赖锁定(应提交;CI 与发版以 --locked 运行)
├── deny.toml                   # cargo-deny 审计规则:通告 / 许可证白名单 / 来源白名单
├── LICENSE                     # MIT
├── rust-toolchain.toml         # 锁定 stable 工具链与 rustfmt / clippy 组件
├── rustfmt.toml                # 格式化配置(仅 stable 选项)
└── typos.toml                  # 拼写检查白名单与排除规则
```

`cargo xtask new my-core` / `cargo xtask new my-cli --bin` 之后会多出:

```text
crates/
├── my-core/                    # 发布到 crates.io 的库
│   ├── Cargo.toml              # description / keywords / categories / readme / documentation / include / docs.rs 元数据
│   ├── README.md               # crates.io 页面内容:安装、用法、MSRV、许可证
│   └── src/
│       ├── lib.rs              # 模块索引与 re-export(设计约束写在文件头)
│       └── error.rs            # thiserror Error 枚举 + Result 别名,中文文案
└── my-cli/                     # 不发布(publish = false),可选地由 release.yml 打成安装包
    ├── Cargo.toml              # [[bin]] name = "my-cli"
    ├── src/
    │   └── main.rs             # clap 参数、tracing 装配、统一错误出口;唯一允许 println! 的地方
    └── tests/
        └── cli.rs              # assert_cmd 子进程集成测试(退出码 / stdout / stderr / --version)
```

---

## 📐 工程规范与架构设计

### 1. 分层与依赖方向

```text
xtask ──(仅 cargo / git 子进程)──▶ 仓库本身
二进制 crate(不发布)──▶ 库 crate(发布)──▶ 更底层的库 crate(发布)
```

- **库 crate 不依赖任何 IO / 终端 / 环境变量**:所有输入通过参数进来、所有结果通过返回值出去,单测无需 mock,使用者接 HTTP / IPC / GUI 都能复用。
- **二进制 crate 不含业务逻辑**:只声明 clap 参数、装配 tracing、分派子命令、打印结果;每个分支只做「整理参数 → 调库 → 打印」。
- **新增领域**:在库 crate 里「一个领域一个文件」,`lib.rs` 里 `pub mod`;需要暴露给用户时在二进制 crate 加子命令与分派。
- **新增 crate**:一律 `cargo xtask new`,不要手写 `Cargo.toml`——生成器保证继承工作区元数据、`[lints] workspace = true`、crates.io 元数据、`{ path, version }` 登记这些约定不被遗漏。

### 2. 面向 crates.io 的约定

- **内部依赖必须带版本**:`[workspace.dependencies]` 里内部 crate 写成 `my-core = { path = "crates/my-core", version = "x.y.z" }`,否则发布后下游解析不到;`version` 由 `cargo xtask release` 与工作区版本一起同步,`cargo xtask version check` 校验一致。
- **发布元数据齐全**:`description`(必填)、`keywords`(≤ 5)、`categories`(用 [官方 slug](https://crates.io/category_slugs))、`readme`、`documentation = "https://docs.rs/<名字>"`、`include` 只带源码 / 清单 / README,`[package.metadata.docs.rs] all-features = true`。
- **不发布的成员显式 `publish = false`**:二进制 crate 与 `xtask`,`cargo publish --workspace` 自动跳过。
- **统一版本(lockstep)**:全部成员共享一个版本号、一起发版——这是 monorepo 库最简单可靠的策略;任何一个 crate 有变更就整体 bump。
- **语义化版本与 MSRV**:破坏性变更 bump major(0.x 阶段 bump minor),CHANGELOG 条目以 **BREAKING** 开头;提升 `rust-version` 记入 `Changed`,CI 的 `msrv` job 保证声明属实。
- **PR 阶段演练**:CI 的 `package` job 跑 `cargo publish --workspace --dry-run --locked`,在隔离目录用打包后的源码重新编译,漏文件、缺元数据当场暴露。

### 3. 错误与日志

- **库层 `Error`**(`thiserror`):`#[non_exhaustive]` 枚举,`#[error("...")]` 文案面向终端用户直接展示;外部输入在公开函数边界校验,不合法返回 `Error::InvalidInput`。
- **二进制层 `anyhow`**:`run` 返回 `anyhow::Result<()>`,`main` 统一 `eprintln!("错误: {err:#}")` 并以 `ExitCode::FAILURE` 退出;`{err:#}` 把 context 链平铺成「做什么: 为什么」一行。
- **禁止 `unwrap` / `expect`**:clippy 已开 `unwrap_used` / `expect_used`,库代码必须显式传播错误;测试模块顶部用 `#[allow(clippy::unwrap_used, clippy::expect_used)]` 放开。
- **日志门面**:业务代码只用 `tracing::debug!` / `info!` / `warn!`,禁止 `println!` / `dbg!` 调试(clippy `print_stdout` / `dbg_macro` 已开)。订阅器只在二进制的 `main.rs` 装配:日志写 **stderr**,`RUST_LOG` 优先,未设置时 `-v` 开 debug、否则 warn。

### 4. 工作区级 lint 基线

根 `Cargo.toml` 的 `[workspace.lints]` 是唯一的规则来源,成员 crate 只写 `[lints] workspace = true`:

| 规则 | 级别 | 意图 |
|---|---|---|
| `rust::unsafe_code` | forbid | 需要 unsafe 时单独评审并在 crate 级显式放开 |
| `rust::missing_docs` | warn | 公开项必须有文档(发到 crates.io 的 API 尤其如此) |
| `clippy::pedantic` | warn(priority -1) | 严格基线,下方单条规则可覆盖 |
| `clippy::unwrap_used` / `expect_used` | warn | 库代码不允许 panic 路径 |
| `clippy::print_stdout` / `print_stderr` / `dbg_macro` / `todo` | warn | 禁止调试残留;正式输出处在文件顶部单独 `#![allow]` |
| `clippy::module_name_repetitions` / `missing_panics_doc` | allow | 小型 crate 噪音大,统一放行 |

> [!IMPORTANT]
> CI 以 `-D warnings` 运行 clippy,以上 **warn 在门禁中等同 deny**;本地开发时仍是 warning,不打断编译。不要为了消警告在业务代码里随手 `#[allow]`,先考虑改写,确需放行时写明原因。

### 5. 更新日志约定

- 每个影响使用者的改动都在 `CHANGELOG.md` 的 `[Unreleased]` 下记一条,归入 `Added` / `Changed` / `Deprecated` / `Removed` / `Fixed` / `Security`,写「对使用者有什么影响」而不是复述提交信息。
- `cargo xtask release` 发现 `[Unreleased]` 为空会**拒绝发版**;发版时自动切成 `## [x.y.z] - YYYY-MM-DD`(UTC),并维护底部 `[Unreleased]` / `[x.y.z]` 比较链接。
- `release.yml` 用 `cargo xtask changelog notes vX.Y.Z` 把该段落作为 GitHub Release 说明;段落缺失时 verify 阶段就失败。

### 6. 版本单一来源

版本号只在根 `Cargo.toml` 的 `[workspace.package].version` 维护,成员全部 `version.workspace = true`,内部依赖的 `version` 由脚本同步。`cargo xtask version check` 通过 `cargo metadata --no-deps` 枚举成员并校验一致;`Cargo.lock` 中的成员条目由 `cargo update --workspace --offline` 刷新,不手写。

---

## 🛠️ 常用开发指令速查

| 指令 | 作用 | 适用场景 |
|---|---|---|
| `cargo xtask new <名字>` | 生成可发布的库 crate 并登记到工作区依赖表 | 新增纯逻辑层 |
| `cargo xtask new <名字> --bin` | 生成二进制 crate(clap + tracing + 集成测试,不发布) | 新增命令行入口 |
| `cargo build` / `cargo test --workspace` | 构建 / 全部单测 + 集成测试 | 日常开发 |
| `cargo run -p <crate> -- <参数>` | 运行某个二进制 crate | 手动验证功能 |
| `cargo xtask ci` | typos / fmt / clippy / test / doc 全量门禁 | 提交前、与 CI 对齐 |
| `cargo xtask ci --keep-going` | 同上,但失败不中断、最后汇总 | 一次看全所有问题 |
| `typos` / `typos -w` | 拼写检查 / 自动修正(需 typos-cli) | 写完文档注释后 |
| `cargo fmt --all` | 格式化全部 crate | 保存 / 提交前 |
| `cargo clippy --workspace --all-targets -- -D warnings` | 静态诊断(含测试代码) | 与 CI 一致的 clippy |
| `cargo doc --workspace --no-deps --open` | 生成并打开文档 | 检查 docs.rs 上会长什么样 |
| `cargo publish --workspace --dry-run` | crates.io 打包演练 | 改了 `include` / 元数据 / 内部依赖后 |
| `cargo deny check` | 依赖审计(需 cargo-deny) | 新增依赖后 |
| `cargo xtask version check [vX.Y.Z]` | 校验工作区 / 内部依赖版本一致,可选与 tag 及 CHANGELOG 比对 | 发版前自检、CI verify |
| `cargo xtask changelog notes vX.Y.Z` | 打印该版本的更新日志段落 | 预览 Release 说明 |
| `cargo xtask release <版本>` | 一键发版 | 见下文 |

---

## 📦 自动化发版与 CI/CD

### 一键发版工作流

无需手动改版本号、切日志、手打 tag,只需执行一条指令:

```bash
# 方式一:直接指定版本号
cargo xtask release 0.2.0

# 方式二:按 semver 规则自动递增 (patch / minor / major)
cargo xtask release patch

# 方式三:预发布版本(Release 会自动打上 prerelease 标记)
cargo xtask release 0.2.0-beta.1

# 只看计划不动文件 / 本地 commit + tag 后不 push
cargo xtask release minor --dry-run
cargo xtask release minor --no-push
```

`xtask/src/release.rs` 会自动执行严密的安全检查,**全部通过才会动文件**:

1. **工作区洁净度检查**:确保没有未提交的改动或未跟踪的文件;
2. **分支与远程同步检查**:确保位于 `main` 分支,且本地已完全同步远端 `origin/main`;
3. **Tag 防重与回退检查**:验证本地与 GitHub 远端均不存在相同的版本 tag;目标版本低于当前版本直接拒绝;
4. **更新日志检查**:`CHANGELOG.md` 的 `[Unreleased]` 必须有条目(已手动切好该版本段落则跳过);
5. **写入版本号**:纯文本替换根 `Cargo.toml` 的 `[workspace.package].version` 与 `[workspace.dependencies]` 里内部 crate 的 `version`(保留注释),并以 `cargo update --workspace --offline` 刷新 `Cargo.lock`——刷新失败即中止并提示如何回滚;
6. **切更新日志**:`[Unreleased]` 条目搬到 `## [x.y.z] - 日期`,更新底部比较链接;
7. **Git Commit & Tag & Push**:只提交 `Cargo.toml` + `Cargo.lock` + `CHANGELOG.md`,提交信息 `chore(release): vX.Y.Z`,创建**附注 tag** 并 `git push --follow-tags`。

> [!NOTE]
> `--dry-run` 下只读的 git 查询照常执行(让检查结果真实),所有写操作改为打印计划。若版本号与日志已全部是目标值(例如手动改过只差打 tag),脚本会跳过 commit 直接打 tag。

### 发版流水线 (`release.yml`)

当 `v*` tag 被推送到 GitHub 后,GitHub Actions 会自动接管:

```text
[ Tag v* ] ──> 1. verify   版本 / 内部依赖 / CHANGELOG 段落一致;cargo test --locked;导出 Release 说明
                   │
                   ├──> 2. build   (仅当 PKG_NAME / BIN_NAME 已填)五目标矩阵编译打包,--locked
                   │        ├── x86_64-pc-windows-msvc     (.zip)
                   │        ├── aarch64-apple-darwin       (.tar.gz, Apple Silicon)
                   │        ├── x86_64-apple-darwin        (.tar.gz, Intel 交叉编译)
                   │        ├── x86_64-unknown-linux-gnu   (.tar.gz)
                   │        └── aarch64-unknown-linux-gnu  (.tar.gz, ARM64 交叉编译)
                   │
                   └──> 3. publish-crates   cargo publish --workspace(Trusted Publishing,按依赖顺序)
                            │
                            └──> 4. publish   GitHub Release:说明 = CHANGELOG 段落,附件 = 二进制包 + SHA256SUMS(若有)
```

- **顺序刻意把 crates.io 放在二进制构建之后**:发布到 crates.io 不可撤销(只能 yank),先让所有可能失败的编译跑完。
- **原子化保障**:任一环节失败后续不执行;修好后删 tag 重推即可。注意已发到 crates.io 的版本无法重发,此时应 bump 一个新版本。
- **纯库项目**:`PKG_NAME` / `BIN_NAME` 留空,`build` 整体跳过,Release 只有说明、没有附件。
- **产物命名**(有二进制时):`<BIN_NAME>-<tag>-<target>.zip|tar.gz`,内含可执行文件、`LICENSE`、`README.md`;附 `SHA256SUMS` 汇总校验和。

| 平台 | 目标三元组 | 包格式 | 构建 runner |
|---|---|---|---|
| **Windows** x64 | `x86_64-pc-windows-msvc` | `.zip` | `windows-latest` |
| **macOS** Apple Silicon | `aarch64-apple-darwin` | `.tar.gz` | `macos-latest` |
| **macOS** Intel | `x86_64-apple-darwin` | `.tar.gz` | `macos-latest`(交叉编译) |
| **Linux** x64 | `x86_64-unknown-linux-gnu` | `.tar.gz` | `ubuntu-24.04` |
| **Linux** ARM64 | `aarch64-unknown-linux-gnu` | `.tar.gz` | `ubuntu-24.04` + `gcc-aarch64-linux-gnu`(交叉编译) |

> [!IMPORTANT]
> **glibc 基线**:Linux 产物在 `ubuntu-24.04` 上编译,动态链接 glibc 2.39,无法在更老的发行版(Debian 12 / Ubuntu 22.04 / RHEL 9 等)运行;需要更广兼容性时把 target 换成 `x86_64-unknown-linux-musl`(静态链接,需 `musl-tools`)。  
> **交叉编译边界**:矩阵默认只适用于纯 Rust 依赖。若引入需要系统库的 crate(如 `openssl-sys`),Linux ARM64 与 macOS Intel 两个交叉目标需要额外配置 sysroot 或改用原生 runner;优先选择 `rustls` 等纯 Rust 替代。  
> **代码签名**:默认产物未签名。macOS 首次运行需在「系统设置 → 隐私与安全性」放行,Windows 会有 SmartScreen 提示;正式分发前请按平台要求配置签名。

### CI 门禁 (`ci.yml`)

| Job | Runner | 内容 |
|---|---|---|
| `typos` | ubuntu | `crate-ci/typos`(钉死版本,dependabot 升级),规则见 `typos.toml` |
| `lint` | ubuntu | `cargo fmt --all --check`、`RUSTDOCFLAGS=-D warnings cargo doc --workspace --no-deps --document-private-items --locked` |
| `test` | ubuntu / windows / macos 并行 | `cargo clippy --workspace --all-targets --locked -- -D warnings`、`cargo test --workspace --locked` |
| `msrv` | ubuntu | 读取 `Cargo.toml` 的 `rust-version`,用该版本工具链 `cargo check --workspace --all-targets --locked` |
| `package` | ubuntu | `cargo publish --workspace --dry-run --locked`:crates.io 打包演练 |
| `deny` | ubuntu | `cargo deny check`(RustSec 通告 / 许可证白名单 / 来源白名单,规则见 `deny.toml`) |

`cargo xtask ci` 在本地复现 `typos` + `lint` + `test` 三组命令(不加 `--locked`,允许边改依赖边跑);改动其中一方的步骤时请同步另一方。CI 加 `--locked` 是为了让过时的 `Cargo.lock` 在 PR 阶段就暴露,而不是拖到发版。

---

## 📄 开源许可证

本项目基于 [MIT License](https://opensource.org/licenses/MIT) 开源,欢迎自由复用、定制与二次分发。
