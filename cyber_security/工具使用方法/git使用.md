# git使用
## 基础使用
 Git 使用教程：

1. 安装 Git：首先，你需要在你的计算机上安装 Git。你可以从 Git 官方网站（https://git-scm.com/）下载适合你操作系统的安装程序，并按照提示进行安装。

2. 配置 Git：安装完成后，你需要进行一些基本的配置，例如设置你的用户名和邮箱地址。你可以通过以下命令进行配置：

   ```
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

3. 创建一个新的仓库：要使用 Git 进行版本控制，你需要在你的项目目录中初始化一个 Git 仓库。你可以使用以下命令来创建一个新的仓库：

   ```
   git init
   ```

4. 添加和提交文件：在你的项目目录中，你可以使用以下命令将文件添加到 Git 仓库中：

   ```
   git add <file>   # 添加指定文件
   git add .        # 添加所有文件
   ```

   然后，你可以使用以下命令将添加的文件提交到仓库中：

   ```
   git commit -m "Commit message"
   ```

5. 查看仓库状态：你可以使用以下命令来查看当前仓库的状态，包括已修改、已添加和已提交的文件：

   ```
   git status
   ```

6. 查看提交历史：你可以使用以下命令查看仓库的提交历史记录：

   ```
   git log
   ```

7. 分支管理：Git 允许你创建和管理多个分支，以便你可以在不同的开发任务之间切换。以下是一些常用的分支管理命令：

   - 创建分支：`git branch <branch-name>`
   - 切换分支：`git checkout <branch-name>`
   - 查看分支：`git branch`
   - 合并分支：`git merge <branch-name>`
   - 删除分支：`git branch -d <branch-name>`

## 远程库建立连接
要将本地仓库与远程仓库建立连接并推送文件，你需要执行以下步骤：
1. 创建远程仓库：首先，在远程仓库托管服务上创建一个新的空仓库。例如，你可以在 GitHub、GitLab 或 Bitbucket 上创建一个新的仓库。

2. 关联远程仓库：在本地仓库中，使用以下命令将本地仓库与远程仓库关联起来：

   ```
   git remote add origin <remote-url>
   ```

   这里的 `<remote-url>` 是你在第一步中创建的远程仓库的 URL。例如，如果你在 GitHub 上创建了一个名为 `my-repo` 的仓库，那么远程仓库的 URL 可能是 `https://github.com/your-username/my-repo.git`。

3. 将本地提交推送到远程仓库：使用以下命令将你的本地提交推送到远程仓库：

   ```
   git push -u origin master
   ```

   这里的 `origin` 是你在第二步中关联的远程仓库的名称，`master` 是你要推送的分支名称。如果你想推送其他分支，只需将 `master` 替换为你想要推送的分支名称。

   `-u` 选项将设置远程仓库为默认的上游仓库，这样以后你只需使用 `git push` 命令即可推送到远程仓库。

4. 输入用户名和密码：在推送过程中，你可能需要输入你的远程仓库的用户名和密码，以进行身份验证。

## 更新远程库
要更新远程库，你需要执行以下步骤：

1. 确保你的本地仓库是最新的：在推送更新到远程库之前，你需要先将本地仓库更新到最新的状态。使用以下命令将远程仓库的最新更改拉取到本地仓库：

   ```
   git pull origin master
   ```

   这里的 `origin` 是你关联的远程仓库的名称，`master` 是你要拉取的分支名称。如果你想拉取其他分支的更新，只需将 `master` 替换为你想要拉取的分支名称。

2. 提交本地更改：在拉取最新更改后，你可以在本地仓库中进行更改和提交。使用以下命令将更改添加到暂存区：

   ```
   git add <file>   # 添加指定文件
   git add .        # 添加所有文件
   ```

   然后，使用以下命令提交更改：

   ```
   git commit -m "Commit message"
   ```

3. 推送更改到远程库：使用以下命令将你的本地更改推送到远程库：

   ```
   git push origin master
   ```

   这里的 `origin` 是你关联的远程仓库的名称，`master` 是你要推送的分支名称。如果你想推送其他分支的更改，只需将 `master` 替换为你想要推送的分支名称。

## 管理远程库关联
```
git remote 查看本地库
git remote -v 查看远程库和本地库的连接状态
git remote remove <remote-name> 删除远程库的关联
```
