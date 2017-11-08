---
layout: post
---

### 步骤
1. 在github新建项目，项目的名字命名为：'maoji.github.io'
2. 在<https://rubyinstaller.org/>下载一个可用的ruby安装包
3. 设置[RubyGems淘宝镜像]<https://gems.ruby-china.org/>，这样安装包时会快很多
4. 安装jekyll和bundler
  ```
    gem install jekyll bundler
  ```
5. 在<https://rubygems.org/search?utf8=%E2%9C%93&query=jekyll-theme>上下载一个模板，本博客采用的是[minima](https://rubygems.org/gems/minima)模板
6. 把模板源代码解压到一个空文件夹，把gemfile里的`gemspec`一行删掉，并添加：`gem 'github-pages', group: :jekyll_plugins`
7. 运行 `bundle install` 安装需要的依赖
8. 运行 `bundle exec jekyll serve` 可开启一个本地服务器，访问 <http://127.0.0.1:4000/> 可查看效果。启动服务器时可能会提示 `seo` 错误，把 `head.html` 里面的 `<% seo %>` 删掉即可
9. 上传到github后，浏览器里访问 <https://maoji.github.io> 可在线查看效果
```
git init
git remote add origin git@github.com:maoji/maoji.github.io.git
git add *
git commit -m 第一次提交
git push origin master
```

### 参考
1. [github-pages和jekyll配置](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/#requirements)
2. [Jekyll文档](https://jekyllrb.com/docs/quickstart/)
