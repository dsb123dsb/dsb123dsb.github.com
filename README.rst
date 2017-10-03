[一、官方hexo文档](https://hexo.io/zh-cn/)   [二、官方next主题文档](http://theme-next.iissnan.com/)

三、官方文档没有的配置
# 1.hexo添加多个tag #
	方式一：仿照Hexo配置文件中的写法
	tag:
	  - 前端
	  - Hexo
	  - HTML
	  - JavaScripto
	方式二：伪JavaScript数组写法
	tag: [前端,Hexo,HTML,JavaScript]

#2.转动头像
:C:\Users\zxw\Desktop\blog\themes\hexo-theme-next\source\css\_common\components\sidebar:这个路径下sidebar-author.styl这个文件，打开:

	.site-author-image {
	  display: block;
	  margin: 0 auto;
	  max-width: 96px;
	  height: auto;
	  border: 2px solid #333;
	  padding: 2px;
	  border-radius: 50px;
	  -webkit-border-radius: 50px;
	  -moz-border-radius: 50px;
	  box-shadow: inset 0 -1px 0 #333sf;
	  -webkit-transition: 0.4s;
	  -webkit-transition: -webkit-transform 0.4s ease-out;
	  transition: transform 0.4s ease-out;
	  -moz-transition: -moz-transform 0.4s ease-out;
	}
	img:hover { 
	    transform: rotateZ(360deg);
	    -webkit-transform: rotateZ(360deg);   
	    -moz-transform: rotateZ(360deg);   
	  }
# 3.设置页面宽度 #

默认情况下Next主题宽度比较窄，在使用浏览器访问的时候两端留白很多，且标题过长的话会出现自动换行的情况。编辑    

    HEXO_ROOT/source/css/_schemes/Picses/_layout.styl
    header{ width: 90%; }
    .container .main-inner { width: 90%; }
    .content-wrap { width: calc(100% - 260px); }

# 4.生成 post 时默认生成 categories 配置项 #

    在 scaffolds/post.md 中添加
    categories: