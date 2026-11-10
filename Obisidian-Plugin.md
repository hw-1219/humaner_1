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
