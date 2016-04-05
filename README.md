# Trace Flight 

This is my personal blog modified from [Microdust](https://github.com/Azeril/azeril.github.io) which is based on  [Clean Blog theme by Start Bootstrap](https://github.com/BlackrockDigital/startbootstrap-clean-blog-jekyll). The theme is designed for [Jekyll](https://jekyllrb.com).

## The main changes are shown as follows.

* Reorganize the folders structure for more elegent look. Such as move *`Tools.md, About.md, Lists.md, tags.html`* to folder *`pages`*, merge folder *`img`* and *`images`*.
* Redesign same pages' logic. Some buttons on top right of the tab bar can not just display a static markdown page but a list of blogs classified by category. So it is fair to these buttons.
* Remove some useless functions in my opinion. For example, remove tags above title. And in [Microdust](https://github.com/Azeril/azeril.github.io) a click on these tags will led to 404 page.
* ...

## How to use it as your own blog?

1. First, you should know some basics. Here is a guide [Creating and Hosting a Personal Site on GitHub](http://jmcglone.com/guides/github-pages/). It shows brief description of [Git](git-scm.com), [GitHub](github.com), [GitHub Pages](https://pages.github.com/) and [Jekyll](jekyllrb.com).
2. Then follow the guide to creat your first blog. Though simple, it is really useful for you to understand Jekyll.
3. After that, you have known how to creat a site on github online, but this is not enough. Then you need to [install Jekyll](https://jekyllrb.com/docs/quickstart/) on your pc in order to view your blog in real-time. To perform this blog on pc you also need to install *jekyll-gallery-generator* and *jekyll-paginate* by gem. Before that you should install *libmagickwand-dev* and *imagemagick* by apt-get or yum depending on you linux distribution.

Or, you can use commands as follows:
	`sudo apt-get install ruby ruby-dev libmagickwand-dev imagemagick`
	`gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/` 
		* *This command is needed for Chinese users.*
	`sudo gem install jekyll jekyll-gallery-generator jekyll-paginate`
4. Finally, delete the repository you created for test, then fork this repository and rename to your own username. Clone and modify it as you want. Enjoy it!
