# AI 助手工作指南

本文件面向在此仓库中工作的 AI 编码助手。改代码前先读完,遵守下列约定;与 README 冲突时以本文件为准。

## 仓库形态

Cargo 工作区(monorepo),根 `Cargo.toml` 统一管理版本、edition、依赖版本表与 lint 规则:

| 路径 | 角色 | 约束 |
|---|---|---|
| `crates/<name>/` | 业务 crate | 全部业务代码放这里;模板初始为空,**用 `cargo xtask new <name> [--bin]` 生成**,不要手写 `Cargo.toml`。库 crate 发布到 crates.io,二进制 crate `publish = false` |
| `xtask/` | 仓库自动化 | `cargo xtask new / ci / version check / changelog notes / release`;不发布 |
| `CHANGELOG.md` | 更新日志 | Keep a Changelog;每个影响使用者的改动都要在 `[Unreleased]` 记一条,为空时无法发版 |

`cargo xtask new` 生成的 crate 已经:继承工作区元数据(`version/authors/.../rust-version.workspace = true`)、`[lints] workspace = true`;库 crate 自带 crates.io 元数据(`description` / `keywords` / `categories` / `readme` / `documentation` / `include` / docs.rs)、独立 `README.md`、`error.rs`(thiserror `Error` + `Result` 别名),并以 `{ path = "crates/<name>", version = "<工作区版本>" }` 登记到根 `[workspace.dependencies]`;二进制 crate 自带 clap + tracing 装配与 `tests/cli.rs` 集成测试。新增 crate 后 `cargo build` 一次让 `Cargo.lock` 记录新成员,并一并提交。

## 分层与依赖方向

- **库 crate**:纯逻辑,不做 IO、不读环境变量、不打印;公开可失败函数返回 `Result<T, Error>`;外部输入在公开函数边界校验,不合法返回 `Error::InvalidInput`。
- **二进制 crate**:只做参数解析、日志装配、子命令分派与结果打印,业务逻辑一律调用库 crate;依赖库 crate 用 `<name>.workspace = true`。
- 依赖方向只能「二进制 → 库」「库 → 更底层的库」,不允许环。

## 编码规范

- **错误**:库用 `thiserror` 在 `error.rs` 扩展 `Error` 枚举,`#[error("...")]` 文案为面向用户的中文;二进制用 `anyhow` 汇总并在 `main` 统一打印 `错误: {err:#}`。库代码禁止 `unwrap` / `expect` / `panic!`(clippy 已开 `unwrap_used` / `expect_used`),测试模块顶部用 `#[allow(clippy::unwrap_used, clippy::expect_used)]` 放开。
- **日志**:只用 `tracing` 宏,禁止 `println!` / `eprintln!` / `dbg!` 调试。正式输出只允许出现在二进制 crate 的 `main.rs` 与 `xtask`(文件顶部已单独放行 lint)。
- **文档**:公开项必须有 `///` 文档(`missing_docs` 已开);可失败函数写 `# Errors` 小节。clap 结构体上的文档注释会成为 `--help` 文本,写给终端用户看。
- **依赖**:新增依赖先加到根 `[workspace.dependencies]` 并写一行中文用途注释,成员 crate 用 `.workspace = true`。许可证须在 `deny.toml` 白名单内。
- **MSRV**:只使用 `Cargo.toml` 声明的 `rust-version` 已稳定的语法与 API(例如 let-chain 需要 1.88,声明 1.85 就不能用);CI 的 `msrv` job 会用该版本实际编译。
- **crates.io 元数据**:库 crate 的 `description` 必填、`keywords` ≤ 5、`categories` 用官方 slug、`include` 只带发布需要的文件;新增需要随包分发的非源码文件(如 `LICENSE-*`、`benches/` 里被 doc 引用的东西)要同步 `include`。内部依赖必须带 `version`,否则发布后下游解析不到。
- **更新日志**:改了公开 API、行为、MSRV、依赖的 feature 等任何使用者可感知的东西,同一次提交里在 `CHANGELOG.md` 的 `[Unreleased]` 下加条目(`Added` / `Changed` / `Deprecated` / `Removed` / `Fixed` / `Security`),写影响而不是复述 diff;破坏性变更以 **BREAKING** 开头。纯内部重构、CI、测试不用记。
- **拼写**:注释、文档、标识符都过 `typos`;确认是专有名词时加到 `typos.toml` 的 `extend-words` / `extend-identifiers`,不要整文件排除。
- **注释与文案**:中文;新项目需替换的占位处以 `【新项目必改】` 标记。
- **子进程**:一律传数组参数,不拼 shell 字符串。

## 质量门禁(提交前必须通过)

```bash
cargo xtask ci
```

等价于依次执行 `typos`(本机装了才跑)、`cargo fmt --all --check`、`cargo clippy --workspace --all-targets -- -D warnings`、`cargo test --workspace`、`RUSTDOCFLAGS="-D warnings" cargo doc --workspace --no-deps --document-private-items`。CI(`.github/workflows/ci.yml`)在三平台跑同一组命令(加 `--locked`),并额外做 MSRV 编译检查、`cargo deny check` 与 `cargo publish --workspace --dry-run --locked` 打包演练。

- clippy 以 `pedantic` 为基线,CI 下 warning 即失败;不要为了消警告在业务代码里随手 `#[allow]`,先考虑改写,确需放行时写明原因。
- 改了 `xtask/src/ci.rs` 的步骤要同步改 `ci.yml`,反之亦然。
- 新增/升级依赖或新增 crate 后提交 `Cargo.lock`;CI 与发版都以 `--locked` 运行,锁文件过时直接失败。

## 版本与发版

- 版本号**只在**根 `Cargo.toml` 的 `[workspace.package].version` 维护,成员全部继承,`[workspace.dependencies]` 里内部 crate 的 `version` 由脚本同步;不要手改成员 crate 或 `Cargo.lock` 里的版本。全部成员统一版本、一起发(lockstep)。
- 发版用 `cargo xtask release <x.y.z | patch | minor | major>`,它负责安全检查、写版本、刷新 `Cargo.lock`、把 `CHANGELOG.md` 的 `[Unreleased]` 切成版本段落、commit、附注 tag 与 push;推送 tag 后 `release.yml` 依次:verify → (可选)二进制打包 → `cargo publish --workspace`(crates.io Trusted Publishing)→ 带 CHANGELOG 段落说明的 GitHub Release。
- 发到 crates.io 的版本不可撤销:发版前确认 `cargo publish --workspace --dry-run` 通过;发错只能 yank 再 bump 新版本。
- `release.yml` 顶部的 `PKG_NAME` / `BIN_NAME` 指定要打成安装包的二进制 crate,纯库项目保持为空(自动跳过打包);有了二进制 crate 后填上,改 `[[bin]] name` 时同步它与 `tests/cli.rs` 的 `cargo_bin(...)`。

## 提交信息

Conventional Commits,中文描述:`feat(<crate>): ...`、`fix(<crate>): ...`、`chore(xtask): ...`、`docs: ...`、`ci: ...`。scope 用 crate 名去掉公共前缀后的短名。
