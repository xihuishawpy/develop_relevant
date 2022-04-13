<!--
 * @Description: 
 * @Version: 1.0
 * @Autor: xihuishaw
 * @Date: 2022-04-13 14:14:30
 * @LastEditors: xihuishaw
 * @LastEditTime: 2022-04-13 14:15:41
-->

## 一、推送

1. 本地初始化

   ```shell
   git init
   ```

2. 文件添加、提交

   添加（暂存状态staged state）

   ```shell
   git add .
   ```

   提交（提交状态committed state）

   ```shell
   git commit -m "first commit"
   ```

   查看当前状态：

   ```shell
   git status
   ```

3. 推送到github

    在本地 repo 和 Github 上的远程 repo 之间建立连接：

   ```shell
   git remote add origin https://github.com/ihechikara/git-and-github-tutorial.git
   ```

   更改主分支名（默认是master），但`一般为main`:

   ```shell
   git branch -M main
   ```

   将repo 从本地设备推送到 GitHub:

   ```shell
   git push -u origin main
   ```

   如果本地修改了文件，还是重复以上2、3步操作

## 二、创建、合并分支

1. 创建并切换到新分支

    `b`告诉 Git 创建一个新的分支，test是要创建和切换到的分支的名字

   ```shell
   git checkout -b test
   ```

   查看分支

   ```shell
   git branch
   ```

2. 切回主分支、合并分支

   切回主分支

   ```shell
   git checkout main
   ```

    合并分支

    ```shell
    git merge test

    git push -u origin main 
    ```

3. 删除远程分支(这里删除远程分支master)
   
   ```shell
   git push origin -d master
   ```
   

---

links：

1. [Git 和 GitHub 教程——版本控制入门](https://mp.weixin.qq.com/s/e0nEJFwiZQqxupSJVvouxg)

2. [git合并分支](https://www.jianshu.com/p/26d050497abb)
