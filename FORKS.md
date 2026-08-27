# Fork 巡检记录

上游 `public-clis/bilibili-cli` 的最后一次提交是 **2026-03-14 的 `dbe2855`**，此后半年
没有任何动静，open PR 全部无人处理。修复散落在各个 fork 里，这个 fork 的维护方式因此变成
「定期巡检兄弟 fork，把值得的提交搬过来」。

**这个文件是那个循环的状态记录。** 它回答三个问题：上次巡检看到哪、哪些提交已经处理过
（引入或否决）、下次该从哪继续。评估过的提交无论采纳与否都要记在这里 —— 否决记录和引入
记录同样重要，否则下次巡检会把同一批提交重新分析一遍。

---

## 上次巡检

| 项 | 值 |
|---|---|
| 巡检日期 | **2026-08-27** |
| 上游基线 | `dbe2855` (2026-03-14)，`upstream/main` 至今未动 |
| 本 fork HEAD | `bd53faf` (v0.7.0) |
| fork 总数 | 109，其中 **18 个**在上游停摆后有过 push |

---

## 怎么做下一次巡检

不用脚本，三条 `gh` 命令就够，按需改。

**1. 谁还活着** —— GitHub 只在真实 push 时更新 `pushed_at`，所以拿上游的 `pushed_at`
（`2026-03-14T12:23:49Z`）当阈值一卡，91 个从未改动过的镜像 fork 自动消失：

```bash
gh api repos/public-clis/bilibili-cli/forks --paginate \
  --jq '.[] | select(.pushed_at > "2026-03-14T12:23:49Z") | [.full_name, .pushed_at, .default_branch] | @tsv' \
  | sort -k2 -r
```

把结果和下面「活跃 fork」表的 `最后 push` 列比对：**时间变新了的才需要细看**，其余跳过。

**2. 某个 fork 有什么独有提交** —— 注意 `compare` 的基准要用上游而不是本 fork，这样
拿到的是「相对原始基线的全部改动」，和表里记录的口径一致：

```bash
gh api "repos/public-clis/bilibili-cli/compare/main...<owner>:<branch>" \
  --jq '"ahead_by: \(.ahead_by)", (.commits[] | "\(.sha[0:7]) \(.commit.author.date[0:10]) \(.commit.author.name) | \(.commit.message | split("\n")[0])")'
```

**默认分支经常是空的，真东西在特性分支上** —— zwczwczwc 和 HawkW1027 的有用修复都在
非默认分支，只看 `main` 会全部错过。先列分支：

```bash
gh api "repos/<owner>/<repo>/branches?per_page=100" --jq '.[].name'
```

**3. 上游的 open PR 是另一条线索** —— 多数活跃 fork 都提过 PR，PR 标题比 commit message
更能说明意图，而且能看出作者自己后来是否放弃了某个方案：

```bash
gh pr list --repo public-clis/bilibili-cli --state open --limit 40 \
  --json number,title,author,updatedAt \
  --jq '.[] | "#\(.number) \(.updatedAt[0:10]) @\(.author.login) | \(.title)"'
```

**4. 判断某个提交是否已处理过** —— 查本文件，或查已引入提交的溯源行：

```bash
git log --format=%B | rg 'Source:|cherry picked from|Adapted from'
```

**5. 巡检完更新本文件**：改「上次巡检」的日期与 HEAD、更新活跃 fork 表里变化的
`最后 push` 与 `HEAD`、把新评估的提交写进「已引入」或「已否决」。

### 引入提交时的约定

- 优先 `git cherry-pick -x`，保留原作者署名，`-x` 会自动写入 `cherry picked from commit <hash>`。
- 再补一行 `Source: <owner>/<repo>@<短hash>, branch <分支名>`，以及对应的上游 PR/issue 编号。
- 改编而非直接搬运时（比如原提交依赖了我们跳过的前置提交），写 `Adapted from <owner>/<repo>@<hash>`
  并说明为什么不是直接 cherry-pick。
- **不要写 `Claude-Session` 之类的 trailer** —— 这是公开仓库。

### 加远端（可选，本地便于 `git log`/`git diff` 直接看代码）

远端配置不入库，换机器要重建：

```bash
for o in cestivan ZeroMarker annoft Gqingbo n1qzhao wjjsn Chesszyh Hi-Zi-Li \
         someblue JupiterTheWarlock Pigletzzz guowenfh HawkW1027 Aki894; do
  git remote add "$o" "https://github.com/$o/bilibili-cli.git" 2>/dev/null
done
git remote add zwczwczwc https://github.com/zwczwczwc/zwczwczwc-bilibili-cli.git 2>/dev/null
git remote add upstream https://github.com/public-clis/bilibili-cli.git 2>/dev/null
git fetch --all
```

---

## 活跃 fork（截至 2026-08-27）

`HEAD` 是巡检时该分支的顶端提交，就是**水位线**：下次巡检时它没变，说明这个 fork 没有新东西。

| Fork | 最后 push | 分支 | ahead | HEAD | 状态 |
|---|---|---|---|---|---|
| [cestivan](https://github.com/cestivan/bilibili-cli) | 2026-08-25 | `fix/qr-login-tv-channel` | +3 | `9e99be3` | 待评估（PR #29） |
| [ZeroMarker](https://github.com/ZeroMarker/bilibili-cli) | 2026-08-17 | `main` | +1 | `6962d5b` | 待评估（PR #27） |
| [annoft](https://github.com/annoft/bilibili-cli) | 2026-08-08 | `main` | +7 | `bb60136` | 部分引入，其余否决 |
| [Gqingbo](https://github.com/Gqingbo/bilibili-cli) | 2026-08-05 | `main` | +21 | `809967d` | 部分引入，多数不适用 |
| [n1qzhao](https://github.com/n1qzhao/bilibili-cli) | 2026-07-15 | `feature/video-pages` | +2 | `9e3ed7a` | 待评估 |
| [wjjsn](https://github.com/wjjsn/bilibili-cli) | 2026-06-20 | `main` | +3 | `26d76e0` | ✅ 全部引入 |
| [Chesszyh](https://github.com/Chesszyh/bilibili-cli) | 2026-06-13 | `feature-video-download` | +3 | `d281658` | 待评估 |
| [Hi-Zi-Li](https://github.com/Hi-Zi-Li/bilibili-cli) | 2026-06-12 | `main` | +1 | `8913d53` | 待评估 |
| [zwczwczwc](https://github.com/zwczwczwc/zwczwczwc-bilibili-cli) | 2026-05-28 | `fix/watch-later-empty-list` | +1 | `fd28f79` | ✅ 已引入 |
| [someblue](https://github.com/someblue/bilibili-cli) | 2026-05-25 | `feat/user-dynamics` | +2 | `348b8ba` | 待评估（PR #17） |
| [JupiterTheWarlock](https://github.com/JupiterTheWarlock/bilibili-cli) | 2026-05-13 | `feat/dynamic-post-image` | +1 | `b83abd9` | 待评估（PR #19） |
| [Pigletzzz](https://github.com/Pigletzzz/bilibili-cli) | 2026-05-13 | `main` | +4 | `06ff1fc` | 待评估 |
| [guowenfh](https://github.com/guowenfh/bilibili-cli) | 2026-05-13 | `main` / `fix/subtitle-output` | +2 / +1 | `0ba64b3` / `d0f6595` | 待评估（PR #9 / #10） |
| [HawkW1027](https://github.com/HawkW1027/bilibili-cli) | 2026-04-27 | `feat/add-chromium-support` | +3 | `10d8aa9` | 部分引入，其余否决 |
| [Aki894](https://github.com/Aki894/bilibili-cli) | 2026-03-18 | `main` | +3 | `efa9812` | 待评估 |
| [forechoandlook](https://github.com/forechoandlook/bilibili-cli) | 2026-04-16 | `main` | +2 | `4b0a7e1` | ❌ 不适用（Go 重写） |
| [WhizZest](https://github.com/WhizZest/bilibili-cli) | 2026-03-23 | `dev`（默认分支） | +4 | `7cc35c1` | ❌ 否决（Selenium 登录） |
| [nan1888](https://github.com/nan1888/bilibili-cli) | 2026-04-04 | `main` | 0 | — | 无独有提交 |

其余 91 个 fork 的 `pushed_at` ≤ 上游，是纯镜像，无需查看。

---

## 已引入（2026-08-27，v0.7.0）

10 个提交，线性追加在 `dbe2855` 之后。每条的完整溯源在 commit message 里。

| 本仓库 | 来源 | 上游 PR/issue | 验证情况 |
|---|---|---|---|
| `86c84aa` | 自研 | — | ✅ `user-videos` 412 消失 |
| `8719e9b` | 自研 | — | ✅ 充电字段 0/1/2 全命中 |
| `0b1a48d` | `zwczwczwc@fd28f79` | PR #22 | ⚠️ 本账号未复现原 bug，改动价值是换用专用端点 `x/v2/history/toview` |
| `9835196` | `HawkW1027@db323d3` | PR #13 | ⚠️ 未实测（需特定网络环境才能触发） |
| `65dd7a3` | `HawkW1027@10d8aa9` | PR #13 | ⚠️ 未实测（同上） |
| `200947a` | `wjjsn@e878ace` | PR #25 | ✅ 实测 `_64K` → `_192K` |
| `c259dc7` | `Gqingbo@b075011` | issue #23 | ⚠️ 未实测（找不到含 `VideoCodecs.UNKNOWN` 的样本视频） |
| `810c2ec` | `wjjsn@1116f11` | PR #25 | ✅ 封面下载实测通过 |
| `926e496` | `wjjsn@26d76e0` | PR #25 | ✅ 随上一条 |
| `893a2f8` | 改编自 `annoft@c150f96` | — | ✅ mock 验证四条路径 |

三条标 ⚠️ 的**没有实测证据**，只是代码逻辑上说得通。如果日后在这些区域 debug，先怀疑它们。

---

## 已否决（不要重复评估）

| 提交 | 来源 | 否决原因 |
|---|---|---|
| `a71660e` | annoft | TV QR 登录通道。cestivan 在 PR #29 独立尝试过同一思路，随后自己用 `8111162` 回退成 web QR Set-Cookie —— 说明这条路走不通 |
| `8d24636` `5d05661` `118b647` `70cbbc8` | annoft | 浏览器凭据刷新的一系列改进（含 Thorium 支持）。**与本 fork 方向冲突**：`893a2f8` 已经把浏览器 cookie 扫描整个从默认路径摘掉了，这几条是在优化一条我们不再走的路 |
| `bb60136` | annoft | 只是 annoft 自己的 release v0.6.3 版本号提交，与我们的版本线无关 |
| `c53daa0` | HawkW1027 | Chromium 支持 + 登录流程改动，同样属于浏览器凭据提取方向，与 `893a2f8` 冲突 |
| `5a31025` `a53d777` `7cc35c1` | WhizZest | Selenium 浏览器登录。引入一个重量级依赖来做 QR 登录已经能做的事，且方向与摘除浏览器依赖相反 |
| `3d4891f` | WhizZest | 把 `SKILL.md` 挪进独立目录（PR #8）。纯目录结构偏好，无功能收益 |
| `33f3539` `4b0a7e1` | forechoandlook | 用 Go 重写整个项目，与本仓库无共同代码 |
| `bd6cf43` 起共 16 条 | Gqingbo | `bili-track` 个人工作流脚本 + 本地 faster-whisper 转写 + DeepSeek 润色 + 飞书推送。是作者自己的私有流水线，不是通用 CLI 功能；转写需求本地已有 `qwen-filetrans-asr` 覆盖 |

---

## 待评估队列

按「预计价值 / 引入成本」大致排序。**每条都还没细看过 diff**，下次巡检从这里挑。

### 值得优先看

| 提交 | 来源 | 内容 | 备注 |
|---|---|---|---|
| `6962d5b` | ZeroMarker `main` | QR 登录支持 B 站 crossDomain 响应格式 | PR #27。登录目前没坏，属于「B 站改接口后会需要」的预备件 —— 真出问题时第一个来这里拿 |
| `5faf032` `8111162` `9e99be3` | cestivan `fix/qr-login-tv-channel` | web QR Set-Cookie 登录策略 | PR #29。注意读提交顺序：作者先试 TV QR（`5faf032`）再自己回退（`8111162`），**最终方案是第二个**，别只看第一条 |
| `29bfef9` | guowenfh `main` | `feed` 命令补全动态详情 | PR #9 |
| `d0f6595` | guowenfh `fix/subtitle-output` | 字幕结构化输出格式修正，去掉冗余 `items` 字段 | PR #10。会改输出 schema，注意 `SCHEMA.md` 同步 |

### 新功能（都是增量，互不冲突）

| 提交 | 来源 | 内容 |
|---|---|---|
| `7846a68` `9e3ed7a` | n1qzhao `feature/video-pages` | 多 P 视频支持 + 番剧订阅与分集详情 |
| `ad7ea59` `efa9812` | Aki894 `main` | 合集与系列（Series & Seasons）管理命令 |
| `8f41a91` `348b8ba` | someblue `feat/user-dynamics` | `user-dynamics` 命令，后一条改用 web feed API |
| `b83abd9` | JupiterTheWarlock `feat/dynamic-post-image` | `dynamic-post` 支持 `--image` 发图 |
| `a210308` | Pigletzzz `main` | 评论楼中楼（完整回复树） |
| `4829d5b` | Pigletzzz `main` | 纯视频下载命令 |
| `26888fb` `d281658` | Chesszyh `feature-video-download` | 视频下载 + 原始流解析。与 Pigletzzz 的下载功能**二选一**，需要先比对两套实现 |
| `9c6c5aa` `ef5e569` `974ac81` | Gqingbo `main` | YAML 持久化配置模块。基础设施类改动，会影响面较大，引入前想清楚是否需要 |
| `8913d53` | Hi-Zi-Li `main` | stream-curator 集成 |
| `8e0f0a1` | Gqingbo `main` | QR 登录 SSO ticket 交换 + 多页字幕。**和 cestivan/ZeroMarker 的登录改动是同一问题域**，三者应该放在一起比较后择一 |

---

## 已知的方向性约束

引入任何提交前先对照这几条，能省掉大量分析：

1. **不走浏览器 cookie 提取。** `893a2f8` 之后默认凭据路径是「已保存凭据 → QR 登录」。任何
   基于 browser-cookie3 / Selenium / Chromium 的凭据方案都与此冲突（这条直接否掉了 annoft 和
   WhizZest 的大部分提交）。`_extract_browser_credential()` 和 `_is_credential_stale()` 作为
   休眠代码保留在 `bili_cli/auth.py` 里，日后若要做成 opt-in 可以复用。
2. **输出 schema 是对外契约。** 改动 YAML/JSON 字段要同步 `SCHEMA.md`，并留意本仓库自己加的
   `charging_exclusive` / `charging_type` / `charging_badge` 三个字段。
3. **登录相关的三个 fork 互斥。** ZeroMarker、cestivan、Gqingbo 各有一套 QR 登录修复，解决的是
   同一类问题。要动登录，一次性比完三家再决定，不要逐个 cherry-pick。
4. **依赖要克制。** 这是给 agent 用的 CLI，装机成本敏感。Selenium、faster-whisper 这类重依赖
   不引入。
