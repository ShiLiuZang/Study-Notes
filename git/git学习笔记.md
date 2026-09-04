# Git 学习教程

## 1. Git 是什么

Git 是一个分布式版本控制系统，用来记录文件修改历史、管理不同版本，并支持多人协作开发。

Git 的基本流程：

```text
工作区 → 暂存区 → 本地仓库 → 远程仓库
```

- **工作区**：正在编辑的项目文件。
- **暂存区**：准备提交的修改。
- **本地仓库**：保存在电脑上的提交记录。
- **远程仓库**：保存在 GitHub、GitLab 等平台上的仓库。

---

## 2. 首次配置

### 设置用户名

```bash
git config --global user.name "你的名字"
```

作用：设置提交记录中显示的用户名。

`global` 表示对当前电脑上的所有项目生效。只对当前项目配置时，可以去掉 `global`。

### 设置邮箱

```bash
git config --global user.email "你的邮箱"
```

作用：设置提交记录中显示的邮箱。通常建议使用 GitHub 或 GitLab 账户关联的邮箱。

### 查看配置

```bash
git config --list
```

作用：查看当前 Git 配置。

---

## 3. 创建或获取项目

### 创建新项目

```bash
mkdir my-project
cd my-project
git init
```

命令说明：

- `mkdir my-project`：创建名为 `my-project` 的文件夹。
- `cd my-project`：进入该文件夹。
- `git init`：在当前文件夹中初始化 Git 仓库。

`git init` 执行后会创建隐藏的 `.git` 文件夹，用来保存版本记录。不要手动删除或修改它。

### 克隆已有项目

```bash
git clone https://github.com/用户名/仓库名.git
cd 仓库名
```

命令说明：

- `git clone`：将远程仓库复制到本地。
- 地址：远程仓库的 URL。
- `cd`：进入克隆后的项目目录。

---

## 4. 查看状态和修改

### 查看当前状态

```bash
git status
```

作用：查看哪些文件新增、修改或删除，以及哪些文件已经进入暂存区。

这是最常用的 Git 命令。遇到不确定的情况时，可以先运行它。

### 查看工作区修改

```bash
git diff
```

作用：查看工作区相对于最近一次提交的具体变化。

### 查看暂存区修改

```bash
git diff --staged
```

作用：查看已经通过 `git add` 放入暂存区的具体变化。

---

## 5. 添加和提交修改

### 添加指定文件

```bash
git add 文件名
```

作用：将指定文件的修改放入暂存区，表示准备在下一次提交中保存它。

例如：

```bash
git add README.md
```

### 添加所有修改

```bash
git add .
```

作用：将当前目录及子目录中的新增、修改和删除操作全部加入暂存区。

注意：使用前最好先运行 `git status`，确认没有把密码、密钥或其他不应提交的文件加入进去。

### 创建提交

```bash
git commit -m "提交说明"
```

作用：将暂存区中的内容保存为一个新的版本。

例如：

```bash
git commit -m "完成用户登录功能"
```

参数说明：

- `git commit`：创建一个提交。
- `-m`：直接在命令行中提供提交说明。
- 提交说明：描述本次提交完成了什么。

提交说明应该简洁明确，例如“修复登录按钮样式”。

### 添加并提交已跟踪文件

```bash
git commit -am "修改提交说明"
```

作用：自动添加已经被 Git 跟踪的已修改文件，然后提交。

注意：不会自动添加新建文件。新文件仍需要先执行 `git add`。

---

## 6. 查看提交历史

### 查看完整历史

```bash
git log
```

作用：查看提交者、提交时间、提交说明和完整提交 ID。

### 简洁查看历史

```bash
git log --oneline
```

作用：每个提交只显示一行，适合快速查看。

### 查看分支关系图

```bash
git log --oneline --graph --all
```

作用：以图形方式查看所有分支的提交关系。

参数说明：

- `--oneline`：每个提交显示一行。
- `--graph`：用字符绘制分支图。
- `--all`：显示所有本地分支。

---

## 7. 分支管理

分支允许你在不影响主代码的情况下开发新功能或修复问题。

### 查看分支

```bash
git branch
```

作用：列出本地分支。当前分支前面会有 `*` 标记。

### 创建分支

```bash
git branch feature-login
```

作用：创建名为 `feature-login` 的分支，但不会自动切换过去。

### 切换分支

```bash
git switch feature-login
```

作用：切换到指定分支。

### 创建并切换分支

```bash
git switch -c feature-login
```

作用：创建新分支并立即切换到该分支，是开发新功能时最常用的写法。

### 合并分支

```bash
git switch main
git merge feature-login
```

作用：

1. `git switch main`：切换到主分支。
2. `git merge feature-login`：将功能分支合并到当前的 `main` 分支。

### 删除本地分支

```bash
git branch -d feature-login
```

作用：删除已经合并的本地分支。

强制删除未合并分支：

```bash
git branch -D feature-login
```

注意：`-D` 可能导致未合并的提交丢失，应谨慎使用。

---

## 8. 远程仓库操作

### 查看远程地址

```bash
git remote -v
```

作用：查看当前项目连接的远程仓库，以及下载和上传地址。

### 添加远程仓库

```bash
git remote add origin https://github.com/用户名/仓库名.git
```

作用：为当前本地仓库添加远程仓库。

`origin` 是远程仓库的常用名称，也可以使用其他名称。

### 获取远程更新并合并

```bash
git pull
```

作用：下载远程仓库的新提交，并自动合并到当前分支。

指定远程仓库和分支：

```bash
git pull origin main
```

表示从 `origin` 的 `main` 分支获取更新。

### 只获取远程更新

```bash
git fetch
```

作用：下载远程仓库的新提交，但不修改当前分支。

它适合在合并前先检查远程发生了什么变化。

### 推送到远程仓库

```bash
git push origin main
```

作用：将本地 `main` 分支的提交上传到远程仓库 `origin`。

第一次推送某个分支时：

```bash
git push -u origin main
```

`-u` 会建立本地分支和远程分支的关联。之后可以直接使用：

```bash
git push
git pull
```

---

## 9. 撤销操作

### 撤销工作区修改

```bash
git restore 文件名
```

作用：放弃指定文件自上次提交以来的修改，恢复到最近一次提交的状态。

注意：未提交的修改可能会丢失，执行前请确认不再需要这些内容。

### 取消暂存

```bash
git restore --staged 文件名
```

作用：将文件从暂存区移出，但保留工作区中的修改。

### 修改最近一次提交

```bash
git commit --amend -m "新的提交说明"
```

作用：修改最近一次提交的说明。

如果先执行 `git add`，还可以将遗漏的文件补充到最近一次提交中。

### 撤销某次提交

```bash
git revert 提交ID
```

作用：创建一个新的提交，用来抵消指定提交的修改。

这种方式不会删除历史，适合已经推送到远程仓库的提交。

### 使用 reset 回退版本

```bash
git reset --soft 提交ID
```

作用：回退提交记录，但保留修改，并将修改放在暂存区。

```bash
git reset --mixed 提交ID
```

作用：回退提交记录，保留工作区修改，但取消暂存。`mixed` 是默认模式。

```bash
git reset --hard 提交ID
```

作用：回退提交记录，并删除之后的工作区和暂存区修改。

注意：`--hard` 可能造成数据丢失。除非确定不需要这些修改，否则不要使用。

---

## 10. 忽略文件 .gitignore

在项目根目录创建名为 `.gitignore` 的文件，可以指定不提交到 Git 的文件和文件夹：

```gitignore
node_modules/
.env
*.log
__pycache__/
.vscode/
```

示例说明：

- `node_modules/`：忽略 Node.js 依赖目录。
- `.env`：忽略环境变量文件，避免提交密码和密钥。
- `*.log`：忽略所有日志文件。
- `__pycache__/`：忽略 Python 缓存目录。
- `.vscode/`：忽略 VS Code 的个人配置目录。

---

## 11. 常用工作流程

开发一个新功能时，可以按照下面的流程操作：

```bash
# 1. 切换到主分支并获取最新代码
git switch main
git pull

# 2. 创建功能分支
git switch -c feature-login

# 3. 修改代码后查看状态
git status

# 4. 查看具体修改
git diff

# 5. 添加修改
git add .

# 6. 提交修改
git commit -m "完成登录功能"

# 7. 推送功能分支
git push -u origin feature-login

# 8. 功能完成后合并回主分支
git switch main
git pull
git merge feature-login
git push
```

---

## 12. 常用命令速查表

| 命令 | 作用 |
|---|---|
| `git init` | 初始化 Git 仓库 |
| `git clone 地址` | 克隆远程仓库 |
| `git status` | 查看项目状态 |
| `git diff` | 查看未暂存的修改 |
| `git add 文件` | 将文件加入暂存区 |
| `git add .` | 将所有修改加入暂存区 |
| `git commit -m "说明"` | 创建提交 |
| `git log` | 查看提交历史 |
| `git branch` | 查看分支 |
| `git switch 分支` | 切换分支 |
| `git switch -c 分支` | 创建并切换分支 |
| `git merge 分支` | 合并分支 |
| `git pull` | 获取并合并远程更新 |
| `git fetch` | 获取远程更新但不合并 |
| `git push` | 推送本地提交 |
| `git restore 文件` | 撤销工作区修改 |
| `git revert 提交ID` | 创建新提交来撤销旧提交 |

---



