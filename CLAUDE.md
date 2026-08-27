# bilibili-cli（MurphyLo fork）

## 这是一个维护中的 fork，不是上游

上游 `public-clis/bilibili-cli` 自 **2026-03-14（`dbe2855`）** 起停止维护，open PR 无人处理。
本 fork 的 `main` 在该基线上追加了来自多个兄弟 fork 的修复，直接在 `main` 上开发。

**动手前先读 [`FORKS.md`](FORKS.md)。** 它记录了：

- 哪些 fork 还活着、上次巡检时各自的水位线（HEAD），以及怎么查出「上次之后有什么新提交」
- 已引入的 10 个提交及其来源，其中**哪三条没有实测证据** —— 在这些区域 debug 时先怀疑它们
- 已否决的提交及原因，**不要重复评估**
- 待评估队列，以及几条方向性约束（比如「不走浏览器 cookie 提取」直接否掉了两个 fork 的大部分提交）

遇到 bug 时，除了自己排查，先去 `FORKS.md` 的活跃 fork 里找找有没有人已经修过。

## 约定

- **引入他人提交**：`git cherry-pick -x` 保留原作者署名，再补 `Source: <owner>/<repo>@<短hash>, branch <分支>`
  和上游 PR/issue 编号。改编而非直接搬运时写 `Adapted from ...` 并说明原因。详见 `FORKS.md`。
- **commit message 不写 `Claude-Session` 之类的 trailer** —— 这是公开仓库。
- **慎跑 `uv lock`**：若本地 uv 配置了自定义 index，重新生成会把该 index 的地址写进 `uv.lock`
  里的每一条依赖 URL（并丢掉 `upload-time` 字段）。用
  `UV_DEFAULT_INDEX=https://pypi.org/simple uv lock` 规避；只改版本号则直接手工编辑那一行。
  副作用：这种情况下 `uv lock --check` 会一直报「需要更新」，属正常，CI 用官方源不受影响。
- **改输出字段要同步 `SCHEMA.md`**，YAML/JSON 输出是对外契约。
- 版本号在四处：`pyproject.toml`、`SKILL.md` frontmatter、`uv.lock` 本包条目、`CHANGELOG.md`。
  `bili_cli/__init__.py` 从包元数据读取，不用改。

## 本地安装

```bash
uv tool install --force git+https://github.com/MurphyLo/bilibili-cli.git@main
```
