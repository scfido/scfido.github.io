---
   layout: default
   title: 使用Git Submodule来管理文档
   tags: 
     - Git
---

# {{ page.title }}
{{ page.date | date: "%Y-%m-%d" }}

# 前言
软件开发过程中有许多配套的文档，最开始我们使用Word来编写，一段时间后发现Word管理文档的缺陷还是相当明显，项目人员找不到文档位置、打开速度慢、无版本管理等。现在我们改为Markdown格式来编写开发文档，将文档与代码放到Git仓库中，在Gitlab网站就能直接查看内容，有了版本管理后对比版本之间变更也非常方便，更新文档时还可以通过Merge Request审核。

但随着Markdown格式文档使用范围扩大，带来了一个新的问题。文档阅读人员范围要比源代码大得多，文档和源代码仓库合在一起后就只有开发团队成员才能阅读文档了。当然可以将文档和代码仓库分离出来，但这样又造成开发人员读写文档不方便。有没有两全其美的方案呢？答案就是Git Submodule。

# Submodule
Submodule简单来说就是用来解决一个大项目需要多个子项目组成这种问题，既能保证大项目引用子项目的版本一致，又便于在大项目中修改子项目的内容。我们就用这个特性来管理项目文档。

一个项目拆分`docs`和`source`两个git仓库，并将`docs`作为`source`仓库的Submodule加到`source`仓库的`/docs`目录下，这样访问`/docs`目录其实就是访问的`docs`仓库。

# 操作方法
以TortoiseGit为例讲解如何实现上述目的。

## 创建仓库
创建名为`source`和`docs`两个git仓库。创建仓库的过程就不具体讲了，当然也可以使用已有的仓库来进行试验。

## 添加Submodule
在`source`仓库中使用`Submodule Add`命令添加。  
**注意**：红圈中的内容一定要填  

![](/assets/misc/GitSubmodule/添加子模块.png)

参数说明：
- Repository 填写`docs`项目的地址，例如：https://github.com/scfido/docs.git
- Path 填写文档仓库映射到`source`仓库的路径，例如：docs
- Branch 文档项目的分支，注意这个选项默认是没有勾选的，不勾选后续的文档更新和同步非常麻烦，一定记得勾上并填写分支名称。分支名称例如：master

## 获取Submodule内容
克隆包含Submodule的仓库时，父仓库只会为Submodule创建一个空目录，不会同时克隆到工作目录中。需要在父仓库中使用`Submodule Update`功能来获取submodule的内容。

如图所示：  
![](/assets/misc/GitSubmodule/更新子模块.png)  

**注意**：如果这步获取到的Submodule文件不是最新的，原因是添加Submodule这一步没有设置branch导致。

## 提交Submodule更改

![](/assets/misc/GitSubmodule/提交子模块1.png)  
提交父仓库时如果submodule有改动也会在变更列表中列出。

![](/assets/misc/GitSubmodule/提交子模块2.png)  
这时点提交按钮会提示是否要同时提交submodule。  

   
![](/assets/misc/GitSubmodule/提交子模块3.png)  
选择Commit会自动弹出提交submodule的对话框，注意看窗口标题栏的路径是submodule的仓库。