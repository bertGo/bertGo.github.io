---
layout: page
title : create-react-app创建项目并用git上传至GitHub及展示预览效果
subtitle : 
author : Bert
header-img: img/post-bg-alitrip.jpg
catalog : true
date : 2016-03-17 18:00:00
tags :
 - Git
 - React
---

# create-react-app创建项目并用git上传至GitHub及展示预览效果

1. 在本地中创建一个项目所在的文件夹

2. npm -g create-react-app

3. 在此文件夹下 create-react-app react-demo (项目名)

4. cd react-demo

5. npm start（等待一会浏览器自动开启）

6. 开始噼里啪啦写你需要的代码

7. （重点）在package. json配置文件中加一句：　“homepage”: “. /” （为下面打包做准备，如果不加这句话，那么在预览时开启的页面空白，原因路径不对）

8. npm run build（打包）
   (在这里有可能)

9. 在你的GitHub上新建个仓库，把地址复制下来备用

10. 项目所在文件夹下git init

11. git add .  (点的意思是所有文件，把所有文件添加上去)
    在这里有可能打包的文件上传不完整，用git add -f <文件名>将未上传的文件标记

12. git commit -m "xxxxxxxxxx"（提交信息）

13.  git remote add origin https://github. com/xxx/xxx（刚才你在GitHub上保存的地址）

14. git pull origin master（上传之前先拉一下，第一次不拉也行，但是之后提交最好想成这个习惯）

15. git push -u origin master（把你的代码提到GitHub上）

16. 此时，在GitHub对应的仓库上，就可以看到刚才提交的代码了。

　　接着，点击“setting”；

　　找到GitHub Pages，source中点击下面按钮切换到master branch，点击save；

　　就可以看到一个链接了，点击链接，发现出现的是你项目中的README. md；

　　在链接后面加上   /build/#    回车后，即可看到预览效果。 

之后的修改代码后，重新npm run build ，重复11~15步骤即可。

 