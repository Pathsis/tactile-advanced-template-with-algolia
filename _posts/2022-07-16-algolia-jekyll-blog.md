---
layout: post
title: 为Vercel上的Jekyll网站添加Algolia搜索功能
tags: 博客
excerpt: 在Vercel上设置Algolia的关键的步骤。
render_with_liquid: false
---

## 0.前言

本文旨在帮助那些想要在Vercel上的Jekyll网站使用Algolia功能的人，一旦配置成功，将不再需要依赖本地Jekyll环境。如果你已经探索良久，那么你或许需要[快速抵达关键要点](/algolia-jekyll-blog.html#4结语)。

## 1.为jekyll添加简单的本地搜索功能

如果只是想要拥有一个简单的搜索功能，建议使用[simple-jekyll-search](https://github.com/christian-fei/Simple-Jekyll-Search)这个插件。这个插件可以本地运行，可以搜索标题、标签、时间、网址。Github上虽然也提供了全文搜索的选项，但我没有成功，也不推荐使用，因为可能会导致性能问题。

简单的配置如下：

第一，在根目录新建一个名为search.json的文件，内容如下：

```json
{% raw %}
---
layout: none
---
[
  {% for post in site.posts %}
    {
      "title"    : "{{ post.title | escape }}",
      "category" : "{{ post.category }}",
      "tags"     : "{{ post.tags | join: ', ' }}",
      "url"      : "{{ site.baseurl }}{{ post.url }}",
      "date"     : "{{ post.date }}",
      "excerpt"  : "{{ post.excerpt }}"
    
    } {% unless forloop.last %},{% endunless %}
  {% endfor %}
]
{% endraw %}
```

第二，在根目录新建search.md文件，内容如下：

```html
---
layout: default
title: 搜索
permalink: /search/
---
<!-- HTML elements for search -->
<input type="text" id="search-input" placeholder="搜索：标题、标签、时间、摘要" style="
    transition: box-shadow .4s ease,background .4s ease,-webkit-box-shadow .4s ease;
    display: inline-block;
    margin: 0 12px 12px 0;
    background: #f5f5f500;
    border: 1px solid rgba(0, 0, 0, 0.15);
    border-radius: 6px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    transition: all 0.23s ease-in-out 0s;
    line-height: 1.7;
    color: #202020;
    max-width: 100%;
    margin-bottom: 15px;
    padding: 0.5rem 0.75rem;
    font-size: 1rem;
    line-height: 1.7;
    width: 325px;
    "/>

<ul id="results-container"></ul>

<!-- script pointing to jekyll-search.js -->

<script src="/js/simple-jekyll-search.min.js"></script>

<script>
SimpleJekyllSearch({
    searchInput: document.getElementById('search-input'),
    resultsContainer: document.getElementById('results-container'),
    json: '/search.json',
    searchResultTemplate: '<li><a href="{url}" title="{desc}">{title}</a></li>',
    noResultsText: '没有搜索到文章',
    limit: 20,
    fuzzy: false
  })
</script>
```

当然，您也可以将这个文件下载下来，复制在本地的js文件中（如没有则新建），并将`https://unpkg.com/simple-jekyll-search@1.10.0/dest/simple-jekyll-search.min.js`替换为`/js/simple-jekyll-search.min.js`，以实现完全的本地化。

此时，访问`your_domain_name/search/`，就能看到搜索页面了。

可以将这个页面添加到导航。在_includes中新建一个navigation.html文件，在其中添加内容：

```html
<nav>
  <a class="navi" href="/">主页</a>
  <a class="navi" href="/about/">关于</a>
  <a class="navi" href="/tags/">标签</a>
  <a class="navi" href="/archive/">归档</a>
  <a class="navi" href="/search/">搜索</a>
  </nav>
```

请注意，最后一项导航项才是我们要添加的，其他的项目是通常会有的，如果它们与你的代码冲突或无关，请忽略它们。

以下为演示效果：

![local search](https://res.cloudinary.com/mkyos/image/upload/v1657967848/vercel-jekyll/3_p6qsii.gif)


## 2.在Netlify上使用Algolia

假设你在Netlify上部署了一个网站，由于两者有密切的合作，你会更方面使用Algolia。

先按照[官方文档](https://www.algolia.com/doc/tools/crawler/netlify-plugin/quick-start/)操作。

当你到达[这一步](https://www.algolia.com/doc/tools/crawler/netlify-plugin/quick-start/#:~:text=The%20plugin%20is%20now%20installed%20and%20ready%20to%20index%20your%20site.)的时候，如下图所示，请注意，Algolia所给的代码片段对新手很不友好，因为缺少一个HTML选择器，以至于只把这段代码粘贴到我们的模版文件中是不会显示的。

![algolia html](https://www.algolia.com/doc/assets/images/tools/crawler/guides/netlify/installed-c5b8f621.jpg)

在原有代码的最上方添加选择器代码：

```html
<div id="search"></div>
```
并将全部代码复制到你想显示Algolia的搜索框的任何网页的模版之中。假设你想在你的archive模版中显示，则将其放置在最顶端。假设你想使之单独成页，请在根目录新建search.md文件，将这段代码粘贴进去，并按照之前的方法为之添加导航。

注意，其中的apikey是你的 Search-Only API Key。

其效果如下：

![algolia netlify](https://res.cloudinary.com/mkyos/image/upload/v1657967845/vercel-jekyll/5_j9teoc.gif)

## 3.在Vercel上使用Algolia

假如你像上面那样使用Netlify创建了一个网站，且安装了Algolia的插件，则你将自动获得Algolia的一个免费Application，其配额为20K请求与20K记录。我建议你使用这个Application**再**创建一个index，以使Vercel上的网站享用这个配额。它比不通过Netlify申请的多一倍。

你只需要在Netlifty上随便创建一个网站，并在安装完Algolia的插件后，不再管它。回到Vercel，假设你想为运行在Vercel上的Jekyll添加搜索，你需要调价Jekyll-algolia插件。这个插件与Netlify的不同，其显示效果也不同，看你喜欢哪个了。喜欢Netlify的，则可以满足于上一节的操作。

因为在Netlify比Vercel的访问速度稍微慢些，特别是内地，所以我选择了Vercel。

先从Algolia的[官方文档](https://community.algolia.com/jekyll-algolia/getting-started.html)开始。

### 3.1配置Gemfile

假设你的Jekyll网站本来运行良好，你需要在你的Gemfile文件的末尾添加如下内容：

```ruby
source 'https://rubygems.org'

gem 'jekyll', '~> 3.6'

group :jekyll_plugins do
  gem 'jekyll-algolia'
end
```
请注意，如果你的Gemfile中本来就有`group :jekyll_plugins do`的片段，请将`gem 'jekyll-algolia'`**添加**进去即可。

假如你的你的Gemfile中本来没有`group :jekyll_plugins do`的片段，你需要在**末尾**完整条件以上代码。

比如我的Gemfile中的内容原本是：

```ruby
# frozen_string_literal: true

source "https://rubygems.org"
gemspec
gem "jekyll-feed"
gem "jekyll-paginate"
gem "jekyll-redirect-from"
gem "jekyll-seo-tag"
gem "jekyll-sitemap"
gem "jekyll-tagging"
gem "jekyll-tagging-related_posts"
gem "kramdown-parser-gfm"

gem 'jekyll', '~> 3.6'
```
我一开始只是把`gem 'jekyll-algolia'`添加到这行代码中，导致不起效。`gem 'jekyll-algolia'`必须在`group :jekyll_plugins do`的片段中。而且，为了这样做，你**无需**改动本来工作良好的其他代码。

### 3.2配置_config.yml

添加如下代码：

```yaml
{% raw %}
# _config.yml

algolia:
  application_id: your_application_id
  index_name:     jekyll # 你自己在algolia中创建的index
  search_only_api_key: your_search_only_api_key
{% endraw %}
```

### 3.3配置Algolia前端

在_includes文件夹中新建algolia.html文件，添加以下内容：

```html
{% raw %}
<script src="https://cdn.jsdelivr.net/npm/instantsearch.js@2.10.5/dist/instantsearch.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/moment@2.24.0/moment.min.js"></script>
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.10.5/dist/instantsearch.min.css">
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/instantsearch.js@2.10.5/dist/instantsearch-theme-algolia.min.css">
<script>
const search = instantsearch({
  appId: '{{ site.algolia.application_id }}',
  apiKey: '{{ site.algolia.search_only_api_key }}',
  indexName: '{{ site.algolia.index_name }}',
  routing: true,
  searchFunction(helper) {
    const container = document.querySelector('#search-hits');
    container.style.display = helper.state.query === '' ? 'none' : '';

    helper.search();
  }
});

const hitTemplate = function(hit) {
  let date = '';
  if (hit.date) {
    date = moment.unix(hit.date).format('Y-M-D');
  }

  let url = `{{ site.baseurl }}${hit.url}#${hit.anchor}`;

  const title = hit._highlightResult.title.value;

  let breadcrumbs = '';
  if (hit._highlightResult.headings) {
    breadcrumbs = hit._highlightResult.headings.map(match => {
      return `<span class="post-breadcrumb">${match.value}</span>`
    }).join(' > ')
  }

  const content = hit._highlightResult.html.value;

  return `
    <div class="post-item">
      <span class="post-meta">${date}</span>
      <h2><a class="post-link" href="${url}">${title}</a></h2>
      {{#breadcrumbs}}<a href="${url}" class="post-breadcrumbs">${breadcrumbs}</a>{{/breadcrumbs}}
      <div class="post-snippet">${content}</div>
    </div>
  `;
}

search.addWidget(
  instantsearch.widgets.searchBox({
    container: '#search-searchbar',
    placeholder: 'Search into posts...',
    poweredBy: true // This is required if you're on the free Community plan
  })
);

search.addWidget(
  instantsearch.widgets.hits({
    container: '#search-hits',
    templates: {
      item: hitTemplate
    }
  })
);

search.start();
</script>

<style>
.ais-search-box {
  max-width: 100%;
  margin-bottom: 15px;
  font-size:1em;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  border-radius: 6px;
}
.post-item {
  margin-bottom: 30px;
}
.post-link .ais-Highlight {
  color: #2a7ae2;
  font-style: normal;
}
.post-breadcrumbs {
  color: #424242;
  display: block;
}
.post-breadcrumb {
  font-weight: bold;
  font-size: 18px;
  color: #424242;
}
.post-breadcrumb .ais-Highlight {
  font-weight: bold;
  font-style: normal;
  color: #2a7ae2;
}
.post-snippet .ais-Highlight {
  font-style: normal;
  background: yellow;
}
.post-snippet img {
  display: none;
}
.ais-search-box--input{
  transition: box-shadow .4s ease,background .4s ease,-webkit-box-shadow .4s ease;
    display: inline-block;
    margin: 0 12px 12px 0;
    background: #f5f5f500;
    border: 1px solid rgba(0, 0, 0, 0.15);
    border-radius: 6px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    transition: all 0.23s ease-in-out 0s;
    line-height: 1.7;
    color: #202020;
}

</style>
{% endraw %}
```

你可以自行修改某些样式。

### 3.4引入Algolia搜索部件

方法同简单本地搜索类似，在根目录新建search.md文件，添加如下内容：

```html
{% raw %}
<div id="search-searchbar"></div>

<div class="post-list" id="search-hits">
</div>
{% include algolia.html %}
{% endraw %}
```

当然，你可以将其放置在任何你先要展现搜索的模版文件中，比如在archive.html或tags.html中。不必单独成页。

### 3.5在本地运行测试

执行以下代码：

```bash
ALGOLIA_API_KEY='your_admin_api_key' bundle exec jekyll algolia
```

假设你**本来**在本地运行良好，你将看到官方文档的效果：

![algolia](https://community.algolia.com/jekyll-algolia/assets/images/getting-started-ff88ef93194e04687d7f08079df54ce9.gif)

假设你**本来**就没有本地运行过，你可能需要想办法成功运行，因为这似乎为在Vercel上使用Algolia所必须。

通常，如果你不使用任何像Algolia这类插件，你完全可以在Vercel部署你来自Github仓库的网站，在你提交数据后，Vercel会自动给你建构并发布，并不需要Github建构，后者只需要存储代码。

但是，现在为了使用Algolia，你需要至少上传一次_site文件夹，否则就会出现如下错误提示，而这就需要本地运行jekyll，以生成_site文件夹。

![vercel jekyll](/img/2022-07-16/1.png)


### 3.6在Vercel上配置

假设你在本地成功运行Jekyll。现在请提交一次代码，这将把你在本地构建的网页传送到Vercel。Vercel如果成功建构，则继续在Vercel上的配置。在build command选项中填入如下命令：

```bash
jekyll build & ALGOLIA_API_KEY='your_admin_api' jekyll algolia
```

![Vercel jekyll](/img/2022-07-16/2.png)

这段代码与Algolia官方文档建议的不同，增加了`jekyll build &`；若不增加，每次提交代码，Vercel每次只向Algolia推送记录，而不建构新的代码，导致网站的输出网页会一直是你上一次本地运行后提交的。

**这是关键的一步**。你当然也可以一直使用本地运行，不再使用Vercel的建构功能。

如何判断配置好Vercel的build命令，正常运行呢？

- 改动你的某个post，推送数据，等待Vercel建构结果；
- 查看网页是否发生了该变动；
- 如果有，则运行良好，**此时可以删除_site文件夹内的所有数据**（不包括该文件夹）；
- 如果没有，则需要继续排查；
- 如果始终失败，你可以选择每次本地运行，然后推送给Vercel的方法。

至此，如果顺利，你应该成功在自己部署于Vercel上的Jekyll网站上用上了Algolia。

其效果如下：

![vercl jekyll algolia](https://res.cloudinary.com/mkyos/image/upload/v1657968534/vercel-jekyll/4_ynhp04.gif)


## 4.结语

要点概览：

1. 在Netlify上使用Algolia，关键要在前端代码前[添加选择器代码](/algolia-jekyll-blog.html#2在netlify上使用algolia)。
2. 在Vercel上使用Algolia，执行以下关键步骤：
   
   - 本地运行测试jekyll，建构成功后将数据推送给Vercel；
   - 若Vercel建构成功，则配置[Build Command](/algolia-jekyll-blog.html#36在vercel上配置)；
   - 终止本地jekyll运行，改动任意文章，推送数据给Vercel；
   - 检查Vercel是否成功建构，验证网页是否反映该变动；
   - 若成功，则清空_site文件夹中的数据；
   - 像过去一样正常使用Vercel；
   - 若始终不能反映该变动，则依赖本地运行jekyll，或者选择Netlify。

Netlify上的Algolia显得更现代，显示结果更直接了当；相比之下，Vercel所使用的jekyll-algolia插件显得很传统，显示结果与网站融为一体。前者有利于**访客**快捷查找，后者则特别有利于**读者**慢慢探索。

此外，Netlify上的Algolia只建立很少的链接的索引，而Vercel所使用的jekyll-algolia插件为每一段都建立索引；在我的网站上，前者的索引数量不到100，后者则达2000以上；一个重要的区别就是，前者点击搜索结果只跳到文章，而后者则跳到具体的段落。

如果你不在乎Netlify稍微慢的网速，可以直接使用Netlify的Algolia，这会方便很多。

最后，关于请求的计算，请参考[官方文档](https://www.algolia.com/doc/faq/accounts-billing/what-is-a-usage-unit/?utm_medium=page_link&utm_source=dashboard)。

目前，假设你搜索“博客”，则至少请求5次；如果删除重新输入，则还要浪费2次；如果输入错误了，还会继续浪费请求。每键入一个字母，执行1次请求。所以，如果实在不够用，请回到本地搜索。

如果你遇到了问题，可以在下方给我留言。