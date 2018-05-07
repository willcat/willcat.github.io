> 作者：CrazyMilk \
> 链接：https://www.zhihu.com/question/21193762/answer/79109280 \
> 来源：知乎 \
>著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 一、关于搭建的流程
1. 创建仓库，http://xxx.github.io；
2. 创建两个分支：master 与 hexo；
3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；
4. 使用`git clone git@github.com:xxx/xxx.github.io.git`拷贝仓库；
5. 在本地http://CrazyMilk.github.io文件夹下通过Git bash依次执行`npm install hexo`、`hexo init`、`npm install` 和 `npm install hexo-deployer-git`（此时当前分支应显示为hexo）;
6. 修改_config.yml中的deploy参数，分支应为master；
7. 依次执行`git add` .、`git commit -m "..."`、`git push origin hexo`提交网站相关的文件；
8. 执行`hexo g -d`生成网站并部署到GitHub上。这样一来，在GitHub上的http://xxx.github.io仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。完美( •̀ ω •́ )y！
### 二、关于日常的改动流程在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理。
1. 依次执行`git add .`、
`git commit -m "..."`、
`git push origin hexo`
指令将改动推送到GitHub（此时当前分支应为hexo）；
2. 然后才执行`hexo g -d`发布网站到master分支上。
虽然两个过程顺序调转一般不会有问题，不过逻辑上这样的顺序是绝对没问题的
（例如突然死机要重装了，悲催....的情况，调转顺序就有问题了）。
### 三、本地资料丢失后的流程当重装电脑之后，或者想在其他电脑上修改博客，可以使用下列步骤：
1. 使用`git clone git@github.com:xxx/xxx.github.io.git`拷贝仓库（默认分支为hexo）；
2. 在本地新拷贝的 xxx.github.io 文件夹下通过Git bash指令：`npm update`
