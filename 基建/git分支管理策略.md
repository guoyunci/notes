#### GitFlow介绍

Git Flow定义了一个项目发布的分支模型，为管理具有预定发布周期的大型项目提供了一个健壮的框架，是由 Vincent Driessen 提出的一个 git操作流程标准、解决当分支过多时 , 如何有效快速管理这些分支。

***

#### 分支分类及用途

简单来说, Git Flow将branch分成2个主要分支和3个临时的辅助分支.

###### 主要分支

> master: 永远处在即将发布(production-ready)状态；
>
> develop: 最新的开发状态；

###### 辅助分支

> feature: 开发新功能的分支, 基于develop, 完成后merge回develop；
>
> release: 准备要发布版本的分支, 用来修复bug. 基于develop, 完成后merge回develop和master；
>
> hotfix: 修复master上的问题, 等不及release版本就必须马上上线. 基于master, 完成后merge回master和develop；

***

#### 分支管理流程及规范

1. 最稳定的代码放在master分支上，不要直接在master分支上提交代码，只能在该分支上进行代码合并操作，例如将其它分支的代码合并到 master 分支上。

2. 从master分支拉一条develop分支出来，该分支所有人都能访问，但一般情况下，我们也不会直接在该分支上提交代码，代码同样是从其它分支合并到develop分支上去。

3. 需要开发某个特性时，从develop分支拉出一条feature分支，例如feature-1与feature-2，在这些分支上并行地开发具体特性。

4. 当特性开发完毕后，我们决定需要发布某个版本了，此时需要从 develop 分支上拉出一条release分支，例如release-1.0.0，并将需要发布的特性从相关feature分支一同合并到release
   分支上，随后将针对release分支部署测试环境，测试工程师在该分支上做功能测试，开发工程师在该分支上修改bug。

6. 待该release分支无bug后，将release分支上的代码同时合并到develop分支与master分支，并在master分支上打一个tag，例如 v1.0.0。

7. 当生产环境发现bug时，我们 需要从对应的tag上（例如 v1.0.0）拉出一条hotfix分支（例如hotfix-1.0.1），并在该分支上做bug修复。待bug完全修复后，需将hotfix
   分支上的代码同时合并到develop分支与master分支。

对于版本号的要求，格式为：x.y.z，其中，x用于有重大重构时才会升级，y用于有新的特性发布时才会升级，z用于修改了某个bug后才会升级。针对每个微服务，我们都需要严格按照以上开发模式来执行。

***

> [git flow推荐工具使用](https://github.com/nvie/gitflow)