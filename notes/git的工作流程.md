## Git的工作流程

Git作为一个源码管理系统，不可避免涉及到多人协作。

写作必须有一个规范的工作流程，让大家有效地合作，使得项目井井有条地发展下去。“工作流程”在英语里，叫做“workflow”或“flow”，比喻项目像水流那样，顺畅、自然地向前流动，不会发生冲击、对撞、甚至漩涡。

![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015122301.png)

本文介绍三种广泛使用地工作流程：

- Git flow
- Github flow
- Gitlab flow



### 一、功能驱动

本文的三种工作流程，有一个共同点：都采用“功能驱动式开发”（Feature-driven development，简称FDD）。

它指的是，需求是开发的起点，先有需求再有功能分支（feature branch）或者补丁分支（hotfix branch）。完成开发后，该分支就合并到主分支，然后被删除。





















