# PPT Master 本地部署与 DSH 接入笔记

> 记录时间：2026-10-26  
> 环境：Windows / Python 3.13.11 / DeepSeek Harness 桌面客户端  
> 状态：已跑通，可生成原生可编辑 PPTX。**本机导出必须走 `run_export.py`。**

---

## 1. 结论速览

- 选用 GitHub 项目 **PPT Master**（原生 PPTX 路线），导出的 `.pptx` 中文本框、形状、图表均为 PowerPoint 原生对象，可逐元素编辑。
- 本地项目路径：`D:\AI-skills\PPT-master\ppt-master-main`
- Skill 本体：`skills\ppt-master`（含 `SKILL.md`、`requirements.txt`、`scripts`、`templates`、`workflows`）
- 已接入 **DeepSeek Harness 桌面客户端**，全局技能目录：`C:\Users\xhw\.dsh\skills\ppt-master`
- 依赖已装齐，Python 环境验证通过。
- **本机导出约定：必须使用 `run_export.py` 包装脚本，不要直接调用原始导出器。**

---

## 2. 环境信息

| 项目 | 值 |
| :--- | :--- |
| 操作系统 | Windows |
| Python | 3.13.11 |
| 项目路径 | `D:\AI-skills\PPT-master\ppt-master-main` |
| Skill 路径 | `D:\AI-skills\PPT-master\ppt-master-main\skills\ppt-master` |
| DSH 安装路径 | `D:\DSh\DeepSeek Harness.exe` |
| DSH 全局技能目录 | `C:\Users\xhw\.dsh\skills\` |
| 关键依赖 | `python-pptx`、`PyMuPDF`、`edge-tts`、`aiohttp`、`PyYAML` |

---

## 3. 部署过程

### 3.1 路线选择

从 SaaS 工具（WPS AI、Gamma、豆包）转向 GitHub Skill，原因是 Skill 把生成与编辑权保留在本地，适合需要交付**可编辑 PPTX** 的正式场景。PPT Master 属于原生 PPTX 派，代价是需要本地 Python 环境、需在 Agent 工具中驱动、逐页串行生成较慢、无在线协作。

### 3.2 项目结构与关键文件

GitHub ZIP 解压后实际路径多套一层 `ppt-master-main`。关键文件两处：

- 根目录 `requirements.txt`：入口 / 汇总
- `skills\ppt-master\requirements.txt` 与 `SKILL.md`：**Skill 本体**，Agent 实际读取这一层

### 3.3 依赖安装问题与解决

**现象**：`pip install -r requirements.txt` 报 `ResolutionImpossible`，并伴随 SSL 错误：

```text
SSL: UNEXPECTED_EOF_WHILE_READING
edge-tts depends on aiohttp<4.0.0 and >=3.8.0
no matching distributions available for your environment: aiohttp
```

**根因**：

- `requirements.txt` 中只有 `edge-tts>=7.2.8`，未显式声明 `aiohttp`
- `edge-tts` 自身依赖 `aiohttp>=3.8.0,<4.0.0`
- 本机 Python 3.13.11 且 `aiohttp` 未安装，pip 解析时无法匹配可用 wheel，叠加 SSL 连接问题，导致整体失败

**解决**：不改 `requirements.txt`，先手动安装兼容版本 `aiohttp`，再安装其余依赖。

```cmd
pip install "aiohttp>=3.9.0,<4.0.0" --trusted-host pypi.org --trusted-host files.pythonhosted.org
```

```cmd
pip install -r requirements.txt --trusted-host pypi.org --trusted-host files.pythonhosted.org
```

安装成功后验证：

```cmd
python -c "import pptx; import fitz; import yaml; print('OK')"
```

输出 `OK`。`fitz` 的 `FutureWarning` 是 PyMuPDF 弃用提示，不影响使用。

### 3.4 Skill 接入 DeepSeek Harness

DSH 桌面版读取全局技能目录 `%USERPROFILE%\.dsh\skills\`。复制 Skill：

```cmd
mkdir "%USERPROFILE%\.dsh\skills" 2>nul
xcopy "D:\AI-skills\PPT-master\ppt-master-main\skills\ppt-master" "%USERPROFILE%\.dsh\skills\ppt-master\" /E /I
```

实际目标：`C:\Users\xhw\.dsh\skills\ppt-master`。复制约 13030 个文件，主要为模板与参考素材，结构正常。

验证目录：

```cmd
dir "%USERPROFILE%\.dsh\skills\ppt-master"
```

应能看到 `SKILL.md`、`requirements.txt`、`scripts`、`templates`、`workflows`、`references`。

### 3.5 导出问题与关键约定

**问题**：本机 Python 使用 `mkdir(mode=0o700)` 创建临时目录时，会把当前用户挡在外面，导出器的 `.write-probe` 写入触发 `PermissionError`。

**绕过方案**：使用项目内 `run_export.py` 包一层，将目录创建时的 `mode` 归一化后重跑，导出成功，**产物本身未做任何妥协**。

**约定**：以后在本机重新导出，一律走 `run_export.py` 包装脚本，不要直接调用原始导出器。

**清理状态**：残留的被锁空临时目录已用完全权限删除，工作区只剩正常的 `projects/`。

---

## 4. cd 命令速查（Windows cmd）

| 命令 | 作用 |
| :--- | :--- |
| `cd` | 显示当前目录 |
| `cd 子目录名` | 进入当前目录下的子目录 |
| `cd ..` | 返回上一级 |
| `cd \` | 回到当前盘符根目录 |
| `cd \xxx` | 从**当前盘符根目录**查找 `xxx` |
| `cd /d D:\完整路径` | 跨盘符切换并进入目录（最稳） |
| `D:` | 只切换盘符，不改变盘内目录 |
| `cd /d "D:\路径 带空格"` | 路径含空格时加引号 |

### 4.1 本次踩坑记录

```cmd
C:\Users\xhw>cd D:\PPT-master
系统找不到指定的路径。
```

原因：`cmd` 中 `cd` 跨盘符不会自动切换驱动器，且路径不存在时直接报错。正确写法：

```cmd
cd /d D:\PPT-master
```

或：

```cmd
D:
cd \PPT-master
```

```cmd
C:\Users\xhw>cd D
系统找不到指定的路径。
```

原因：`D` 不是目录，盘符应写 `D:`，且单纯 `D:` 仅切盘。

```cmd
D:\AI-skills>cd \PPT-master
系统找不到指定的路径。
```

原因：`\` 表示**当前盘根目录**，该命令查找的是 `D:\PPT-master`，而非当前目录下的子目录。正确写法：

```cmd
cd PPT-master
```

### 4.2 实用习惯

1. 不确定路径时先 `dir` 看当前目录内容。
2. 跨盘切换优先用 `cd /d 完整路径`。
3. 当前目录下的子目录直接 `cd 子目录名`，不要加前导 `\`。
4. PowerShell 中 `cd D:\path` 可直接跨盘，但 `2>nul` 等写法仅适用于 cmd。

---

## 5. 后续操作清单

- [x] 安装 Python 依赖并验证
- [x] 复制 Skill 到 `~/.dsh/skills/`
- [x] DSH 客户端识别 Skill
- [x] 配置 DeepSeek API Key，选用 Flash 模型控制成本
- [x] 跑通测试 PPT
- [ ] 正式任务导出时使用 `run_export.py`
- [ ] 如重新导出报 `PermissionError`，先检查是否绕过了包装脚本
