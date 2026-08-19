# 多机协作维护手册

本工程为单文件网页游戏（`fps.html` + `three.min.js`），无构建步骤。多台电脑维护时请遵循以下流程。

## 1. 新机器接入（一次性）

```bash
# 1) 安装 Git for Windows（https://git-scm.com/download/win），一路默认即可
# 2) 克隆仓库（首次推送时会弹浏览器登录 GitHub，凭据由 Git Credential Manager 自动管理）
git clone https://github.com/studyguy/FPS_web.git
# 3) 配置提交身份（与 GitHub 账号一致）
git config --global user.name "Rodger"
git config --global user.email "faith.yrj@gmail.com"
```

本地运行：双击 `fps.html` 即可游玩；调试时用 `python -m http.server 8123`。

## 2. 日常工作流

```bash
git pull --rebase        # 1. 开工前先同步（永远在开始改代码前执行）
# ……修改代码、本地验证……
git status               # 2. 确认改动清单
git add -A
git commit -m "feat: xxx"   # 3. 小步提交，一次只提交一个功能点
git push                 # 4. 推送
```

> ⚠️ 若提示远端有新提交导致 push 被拒：再执行一次 `git pull --rebase` 后重新 push。

## 3. 提交规范（Conventional Commits）

| 前缀 | 用途 | 示例 |
|---|---|---|
| `feat` | 新功能 | `feat: 新增狙击枪武器` |
| `fix` | 修复 bug | `fix: 修复敌人视线检测失效` |
| `docs` | 文档 | `docs: 更新敌人数值表` |
| `chore` | 杂项（资源、配置） | `chore: 升级 three.min.js 至 r152` |
| `refactor` | 重构（不改行为） | `refactor: 抽取碰撞检测工具函数` |
| `test` | 测试 | `test: 增加伤害链路自动化自检` |

## 4. 分支策略

- `main` 分支**始终可运行**，是唯一部署源（GitHub Pages 自动从它发布）
- 大改动（新武器/新敌人类型/地图重做）开分支：

```bash
git checkout -b feature/新功能名
# ……开发、本地验证……
git push -u origin feature/新功能名
# 到 GitHub 网页发起 Pull Request，或本地合并：
git checkout main && git pull --rebase && git merge --no-ff feature/新功能名 && git push
```

## 5. 冲突处理

```bash
git pull --rebase        # 发生冲突时 Git 会暂停并标记文件
# 打开冲突文件，搜索 "<<<<<<<"，手动保留正确内容后删除标记
# 游戏主逻辑都在 fps.html 的 <script> 内，冲突时务必保留双方的最新功能
node --check 提取后的脚本   # 合并后验证语法（见第 7 节）
git add -A && git rebase --continue
git push
```

## 6. 版本发布

```bash
# 功能稳定后打 tag 并更新 CHANGELOG.md
git tag -a v1.1.0 -m "release v1.1.0"
git push origin v1.1.0
```

## 7. 验证方法

- **语法校验**：游戏代码全部在 `fps.html` 中。提取主脚本校验：
  ```bash
  node -e "const s=require('fs').readFileSync('fps.html','utf8');const m=s.match(/<script>([\s\S]*?)<\/script>/g);require('fs').writeFileSync('_check.js',m[m.length-1].replace(/<\/?script>/g,''))"
  node --check _check.js && del _check.js
  ```
  （`_check.js` 等临时文件已在 .gitignore 中，不会入库）
- **功能自检**：F12 控制台使用 `__game.debug.spawnWave(7)`、`__game.debug.teleportEnemyNear('brute', 3)`、`__game.debug.heal()` 快速验证
- **自动化测试**：临时编写 `_selftest*.html`（iframe 加载游戏 + 模拟点击按键），用无头 Chrome 验证后删除

## 8. 注意事项

- **`three.min.js` 是本地兜底库**，必须提交；升级时从 CDN 下载新版（注意保留 UMD 构建 r160 以前的版本，如 r152）后同步提交
- **测试/临时文件不入库**：`_selftest*.html`、`_*.js`、`_shot.png` 已被 .gitignore 覆盖
- **不要在仓库中提交任何密钥**（token、密码）
- **GitHub Pages 自动部署**：push 到 main 后 1~3 分钟生效，无需手动操作；若长期 404 检查 Actions 页面的 pages 工作流
- 多人同时改同一段代码时，尽量提前沟通分工，减少冲突
