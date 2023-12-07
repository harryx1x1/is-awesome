# 如何发布一个 Obsidian 的笔记网站

昨天把 Obsidian 发布网站的事情搞定了，还买了个新的域名，上线了 https://harryx1x1.fun/

这个过程还挺顺畅的，用 obsidian 写，然后发布到 github，自动部署到网站了，不需要太折腾代码的事情，简单，节省时间。另外还能继续用 obsidian 写东西，保留 md 文件和写作习惯。

这个流程比 gatsby 好的一点是，更简单，gatsby 还是需要做相对多的代码方面的工作的，工作流基本没法保持在 obsidian 中完成，麻烦。

能保持用户习惯和工作流的事情更好。

主要有两部分设置：

1. Obsidian 发布可以参考 [这里](https://github.com/jobindjohn/obsidian-publish-mkdocs) 。
2. 域名设置可以参考 [这里](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain)。

域名设置有几点需要注意：

1. github 这边域名的生效可能需要几个小时
2. 记得在 docs 下面加一个 CNAME 文件，内容是自己设置的域名/子域名。要不然每次部署后，在 github 的域名设置就丢失了

附：我在域名服务商 namecheap 的域名设置是这样的:

![[Pasted image 20231207220621.png]]