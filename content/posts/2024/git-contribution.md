---
title: "如何参与贡献开源项目的代码"
date: 2024-06-05T20:14:08+08:00
draft: false

categories:
- 开发技能

tags:
- 开源
- Github
---

# 如何参与贡献开源项目的代码

这里以github上的操作为例

假设下图就是要贡献代码的开源仓库
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515114141.png)
## 步骤

1. 第一步就是需要先fork开源项目
在要贡献代码的仓库页面点击`fork`按钮
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515114226.png)
`fork`完成后就可以在自己的仓库中看到这个
仓库
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515114431.png)
2. 克隆仓库
下一步就是我们需要将`fork`的仓库克隆到自己的本地电脑上，下面假设你已经安装好了`git工具`。

首先我们在`github`页面上复制我们刚才`fork`后的仓库的地址:
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515115248.png)

找一个合适的位置，打开命令行,输入以下命令
```bash
git clone 你仓库的url链接
```
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515115801.png)
之后切换到克隆下来的仓库目录下
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515115939.png)



3. 准备工作

添加上游仓库：使用以下命令将原始项目(就是你要要贡献代码的仓库)仓库作为上游仓库添加到本地仓库：

```bash
git remote add upstream <upstream_repository_url>
```
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515131405.png)
查看远程仓库列表：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515131444.png)
在上游仓库发生更改变化的时候，本地可能需要拉取同步更新，使用以下命令拉取上游仓库的更改，并将其合并到您的本地分支中（由于刚从上游仓库克隆下来所以并没有什么更改）：
```bash
git fetch upstream
git merge upstream/main
```
上面的命令也可以合为：
```
git pull upstream main
```
下面是我执行的操作：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515133019.png)
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515133039.png)

创建开发分支：分支是你的更新的副本。使用分支，您可以在不影响主分支的情况下，对代码进行更改。使用以下命令在本地创建一个新分支并切换到该分支上：
```bash
git checkout -b <branch_name>
```
我这里创建并切换到了`dev`分支（名字你可以随便起，不过可以写和这个分支要做的事情相关的描述或许更好）：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515133143.png)


4. 进行代码贡献  

在本地计算机上进行所需的更改。确保您的更改符合项目的贡献指南，并且经过了充分的测试和代码审查。

这里我作为测试对`README.md`进行修改，添加一段语句如下：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515170738.png)
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515172113.png)

5. 添加更改 
 
使用以下命令将更改添加到暂存区中：
```bash
git add <file_name>
```
其中 <file_name> 是您想要添加的文件名称。如果想要添加所有更改，可以使用以下命令：
```bash
git add .
```
下面是我执行的操作：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515171856.png)

6. 提交更改  

使用以下命令将更改提交到本地分支中：
```bash
git commit -m "Commit message"
```
其中 "Commit message" 是对更改的简短描述，可以根据对应的项目贡献指南按规范来描述你的更改。

下面是我执行的操作：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515172404.png)

查看提交记录：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515172757.png)

7. 推送更改  

使用以下命令将更改推送到您的远程分支中：
```bash
git push origin <branch_name>
```
其中 <branch_name> 是您创建的分支名称。

下面是我执行的操作：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515173010.png)

8. 创建拉取请求
在你Fork的项目仓库页面上，点击`Pull requests`，接着单击“New pull request”按钮。选择你刚才的开发分支，添加说明和描述，然后提交拉取请求。

下面是我执行的操作：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515173331.png)
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515173531.png)
选择刚才的开发分支
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515173642.png)
可以看到我们更改，我们点击绿色按钮`Create pull request`
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515174114.png)
之后可以填写标题和修改的内容的描述
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515174710.png)
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515185957.png)
填写完成后，点击绿色按钮`Create pull request`创建拉取请求。
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515190147.png)

9. 等待审核  

等待项目维护者审核您的拉取请求。他们可能会要求您进行更改或修复某些问题。

10. 合并更改

如果您的更改被接受，项目维护者将合并您的分支到主分支中。合并后，你会收到一封电子邮件通知。
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515194100.png)

11. 删除开发分支

在代码合并之后，你就可以在本地和远程仓库删除这个开发分支了
```
git branch -d <branch_name>
git push origin --delete <branch_name>
```

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230515195102.png)