# Obsidian 插件“我的工作台”环境搭建笔记

> 记录时间：2026-11-10  
> 环境：Windows / Node.js v26.13.0 / npm 11.20.0 / Git 2.56.0.windows.4 / VSCode  
> 状态：插件开发最小闭环已跑通。**当前仅有示例命令，尚未实现“今日打卡”等业务功能。**

---

## 1. 结论速览

- 选用 **Obsidian 官方 Sample Plugin 模板** 初始化项目，插件 ID 为 `my-checkin-workbench`，显示名为 `我的工作台`。
- 代码项目路径：`D:\WorkBuddy_Plugin\Code-Plugin`
- Obsidian 测试库路径：`D:\WorkBuddy_Plugin\WorkTest`
- 测试库插件目录：`D:\WorkBuddy_Plugin\WorkTest\.obsidian\plugins`
- 通过 **Windows 目录联接** 把代码项目链入测试库，实现热加载调试。
- `npm run dev` 已跑通，终端显示 `[watch] build finished, watching for changes...`
- 插件已在 Obsidian 中启用，模板自带示例命令可正常执行，弹出 “woah！”。
- **尚未实现：今日打卡、工作报告、日记辅助、学习笔记整理、AI 接入。**

---

## 2. 路径与命名约定

| 项目 | 路径 / 名称 |
|---|---|
| 总工作文件夹 | `D:\WorkBuddy_Plugin` |
| 代码项目文件夹 | `D:\WorkBuddy_Plugin\Code-Plugin` |
| Obsidian 测试库 | `D:\WorkBuddy_Plugin\WorkTest` |
| 测试库插件目录 | `D:\WorkBuddy_Plugin\WorkTest\.obsidian\plugins` |
| 插件链接目录名 | `Code-Plugin`（与代码项目同名） |
| 插件显示名 | `我的工作台` |
| 插件 ID | `my-checkin-workbench` |

---

## 3. 环境版本

| 工具 | 版本 |
|---|---|
| Node.js | `v26.13.0` |
| npm | `11.20.0` |
| Git | `2.56.0.windows.4` |
| 编辑器 | VSCode |
| Obsidian | 独立测试库 `WorkTest` |

> Node.js 建议 18+ LTS。当前版本较新，已验证可用。

---

## 4. 初始化步骤

### 4.1 用官方模板生成项目

在 `D:\WorkBuddy_Plugin` 下执行：

```bash
git clone https://github.com/obsidianmd/obsidian-sample-plugin.git Code-Plugin
cd Code-Plugin
npm install
```

### 4.2 修改 `manifest.json`

只改两个字段，其余保持模板默认：

```json
{
  "id": "my-checkin-workbench",
  "name": "我的工作台"
}
```

### 4.3 创建 Obsidian 测试库

新建空文件夹并用 Obsidian 打开：

```text
D:\WorkBuddy_Plugin\WorkTest
```

然后在其中创建插件目录：

```text
D:\WorkBuddy_Plugin\WorkTest\.obsidian\plugins
```

### 4.4 目录联接（Windows）

在 `D:\WorkBuddy_Plugin` 下执行：

```bat
mklink /J "D:\WorkBuddy_Plugin\WorkTest\.obsidian\plugins\Code-Plugin" "D:\WorkBuddy_Plugin\Code-Plugin"
```

成功后，`plugins\Code-Plugin` 里可直接看到 `manifest.json`、`src`、`node_modules`。

### 4.5 启动监听编译

在 `D:\WorkBuddy_Plugin\Code-Plugin` 下执行：

```bash
npm run dev
```

终端出现以下内容表示成功：

```text
[watch] build finished, watching for changes...
```

**保持该终端运行**，改代码后自动编译，Obsidian 会热加载。

### 4.6 在 Obsidian 中启用插件

1. 打开 `WorkTest` 测试库
2. 设置 → 第三方插件 → 关闭安全模式
3. 在已安装插件中找到 **我的工作台**
4. 启用
5. `Ctrl + P` 打开命令面板，搜索示例命令，执行后出现 “woah！” 即表示链路打通

---

## 5. 已验证的闭环

```text
VSCode 修改 src/main.ts
        ↓
npm run dev 自动编译生成 main.js
        ↓
Obsidian 热加载插件
        ↓
命令面板执行命令 → 看到效果
```

---

## 6. 遇到的问题与处理

### 6.1 PowerShell 禁止运行 npm.ps1

报错原文：

```text
npm : 无法加载文件 D:\node\npm.ps1，因为在此系统上禁止运行脚本。
```

处理方式（当前用户范围，无需管理员）：

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

临时绕过方案：

```powershell
npm.cmd run dev
```

### 6.2 Obsidian 中看不到插件

排查顺序：

1. `manifest.json` 是否存在且 JSON 格式正确
2. `main.js` 是否已生成
3. 目录联接是否指向正确的 `Code-Plugin`
4. 重启 Obsidian 后重新进入第三方插件列表

---

## 7. 当前仓库文件约定

```text
Code-Plugin/
├─ README.md          # 项目说明
├─ DEVLOG.md          # 开发日志（本文件）
├─ .gitignore         # 忽略 node_modules、main.js、data.json 等
├─ manifest.json      # 插件元信息
├─ package.json
├─ src/
│  └─ main.ts         # 插件逻辑入口
└─ main.js            # 构建产物，不提交
```

`.gitignore` 至少包含：

```gitignore
node_modules/
main.js
*.js.map
data.json
.env
.DS_Store
Thumbs.db
```

---

## 8. 尚未实现 / 下一步

- [ ] 将 `src/main.ts` 示例命令替换为“今日打卡”
- [ ] 确定打卡数据存储方式，候选：
  - 仅 Notice 提示，不持久化
  - 插件 `saveData` 写入 `data.json`
  - 写入指定 Markdown 文件
- [ ] 打卡历史查看
- [ ] 工作报告 / 周报数据汇总
- [ ] 日记辅助功能
- [ ] 学习笔记整理功能
- [ ] 按需接入 AI 能力

---

## 9. 安全与隐私

- 不提交 API Key、Token、个人笔记
- 不提交 `data.json`，除非确认无隐私数据
- 不把正式 Obsidian 笔记库提交到本仓库
- `node_modules/` 不入库
