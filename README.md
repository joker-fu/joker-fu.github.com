
#### 分支说明（切记这两个分支不能merge rebase）
	master：博客文章发布分支
	hexo: hexo备份分支

#### 使用说明
	- 1 搭建hexo环境
	- 2 拉取master分支
	- 3 拉取hexo分支
	
#### 博客发布与分支push

	- 每次写博客前先pull hexo分支
	- hexo new "博客名" 
	- 使用markdown编辑博客 放入source post文件夹下对应md文件
	- hexo clean -> hexo g -> gulp(压缩优化) -> hexo d 发布博客
	- 本地hexo分支提交到远程hexo分支	


