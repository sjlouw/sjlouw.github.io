---
layout: post
title: "Octopress blogging engine on GitHub Pages"
date: 2013-12-23 00:44:45 +0200
comments: true
categories: macosx other
---
{% img left /images/blog_posts/octopress.png %}

This is a guide to install Octopress blogging framework on Mac OS X Mavericks. This was done and tested on OS X Mavericks version 10.9.1. You will need to have a GitHub account with SSH keys properly [configured](https://help.github.com/articles/generating-ssh-keys) to sucessfully host the Octopress blog on GitHub Pages.
<!--more-->
<br>
### Getting and installing Octopress locally.

{% codeblock lang:bash %}
cd ~/
git clone git://github.com/imathis/octopress.git octopress
cd octopress

gem install bundler
bundle install
rake install
{% endcodeblock %}

Installing the [octostrap3](https://github.com/kAworu/octostrap3) theme:

{% codeblock lang:bash %}
cd ~/octopress/
git clone https://github.com/kAworu/octostrap3 .themes/octostrap3
rake 'install[octostrap3]'
{% endcodeblock %}

Edit `_config.yml` with your parameters.

{% codeblock lang:bash %}
vi ~/octopress/_config.yml
{% endcodeblock %}

{% codeblock lang:bash %}
rake generate
{% endcodeblock %}


### Getting ready for deployment to GitHub Pages:

Follow this guide to setup SSH key based authentication to Github: [(Reference)](https://help.github.com/articles/generating-ssh-keys)

Create a new Github repository and name the repository with the format `username.github.io`, where username is your GitHub user name.

Github Pages for users uses the master branch like the public directory on a web server, serving the files at your Pages url `http://username.github.io`. As a result, you'll want to work on the source for your blog in the source branch and commit the generated content to the master branch. Octopress has a configuration task that helps you set all this up.


### Prepare Octopress and deploy to GitHub Pages: [(Reference)](http://octopress.org/docs/deploying/github/)

{% codeblock lang:bash %}
cd ~/octopress/
rake setup_github_pages
{% endcodeblock %}

Enter the following where `username` is your username.

{% codeblock lang:bash %}
git@github.com:username/username.github.io.git
{% endcodeblock %}

This will generate the blog, copy the generated files into `_deploy/`, add them to git, commit and push them to the master branch.

{% codeblock lang:bash %}
rake generate
rake deploy
{% endcodeblock %}

Commit the source for your blog.
{% codeblock lang:bash %}
git add .
git commit -m 'source commit'
git push origin source
{% endcodeblock %}


### Using Bootstrap labels for categories:

{% codeblock lang:bash %}
cd ~/octopress/
curl -s -S http://kaworu.github.io/octopress/downloads/code/category-with-bs-label.patch | patch -p1
curl -s -S http://kaworu.github.io/octopress/downloads/code/category_list.html > source/_includes/custom/asides/category_list.html
{% endcodeblock %}

{% codeblock lang:bash %}
vi _config.yml
{% endcodeblock %}

In section `default_asides:`, add: `custom/asides/category_list.html`


### Replacing Google search with lunr.js javascript site-search:

Download some javascript files and move them to the correct path.
{% codeblock lang:bash %}
cd ~/octopress/
curl -s -S -L http://raw.github.com/olivernn/lunr.js/master/lunr.min.js > source/javascripts/lunr.min.js
curl -s -S -L http://github.com/janl/mustache.js/raw/master/mustache.js > source/javascripts/mustache.js
curl -s -S -L http://github.com/medialize/URI.js/raw/gh-pages/src/URI.min.js > source/javascripts/URI.min.js
{% endcodeblock %}

Create a `search.json` file.

**NOTE:** The `content` line can cause problems and make the search not working. If this is the case, remove the whole `content...` line from below. This will make only post titles searchable.

`vi source/search.json`
{% codeblock lang:json %}{% raw %}
---
---
[
{% for post in site.posts %}
    {
        "url": "{{ "{{" }} post.url }}",
        "title": "{{ "{{" }} post.title }}",
        "content": "{{ "{{" }} post.content | strip_html | strip_newlines | remove:'"' }}",
        "date": "{{ "{{" }} post.date | date:'%d %B %Y' }}"
    }{%  unless forloop.last %},{% endunless %}
{% endfor %}
]
{% endraw %}{% endcodeblock %}

Create a `search.js` file.
`vi source/javascripts/search.js`
{% codeblock lang:javascript %}
$(document).ready(function() {
    var index = lunr(function () {
        this.field('title', {boost: 10});
        this.field('content');
        this.ref('url');
    });
    
    $.getJSON('/search.json', function(json) {
        for(var post in json) {
            index.add(json[post]);
        }
        
        var uri = new URI(window.location.search.toString());
        var queryString = uri.search(true);
        if(queryString.hasOwnProperty('q')) {
            var results = index.search(queryString.q.toString()).map(function(result) {
                return json.filter(function(p) { return p.url == result.ref; })[0];
            });
            if(results.length > 0) {
                $('#search-results').html(Mustache.to_html($("#search-results-template").text(), {posts: results}));
            }else if(results.length = 1){
                $('#no-results').html(Mustache.to_html($("#no-results-template").text(), {noposts: results}));
            }
        }
    });
});
{% endcodeblock %}

Create new search page.
{% codeblock lang:bash %}
rake new_page['search']
{% endcodeblock %}

Edit the newly created search page.
`vi source/search/index.markdown`
{% codeblock lang:html %}
---
layout: default
title: Search
footer: true
---

<script src="/javascripts/libs/jquery.min.js"></script>
<script src="/javascripts/lunr.min.js"></script>
<script src="/javascripts/URI.min.js"></script>
<script src="/javascripts/mustache.js"></script>
<script src="/javascripts/search.js"></script>

<div class="container">
    <div class="page-header">
         <h1>Search Results</h1>
    </div>

<script id="search-results-template" type="text/mustache">
<ul>
{{ "{% raw " }}%}{% raw %}
{{ #posts }}
    <li><a href="{{ url }}">{{ title }}</a> - {{ date }}</li>
{{ /posts }}
{% endraw %}{{ "{% endraw " }}%}
</ul>
</script>

<script id="no-results-template" type="text/mustache">
<ul>
{{ "{% raw " }}%}{% raw %}
{{ #noposts }}
    <p>No search results found.</p>
{{ /noposts }}
{% endraw %}{{ "{% endraw " }}%}
</ul>
</script>

<div id="no-results">
</div>

<div id="search-results">
</div>

</div>
{% endcodeblock %}

Modify the `navigation.html` file.
`vi source/_includes/custom/navigation.html` and replace the line:
{% codeblock lang:html %}
<input type="hidden" name="q" value="site:{{ "{{" }} site.url | shorthand_url }}">
{% endcodeblock %}
with
{% codeblock lang:html %}
<input type="hidden">
{% endcodeblock %}

`vi _config.yml` and replace the line:
{% codeblock %}
simple_search: http://google.com/search
{% endcodeblock %}
with
{% codeblock %}
simple_search: http://username.github.io/search
{% endcodeblock %}


### Hiding the `username.github.io` repo from the GitHub aside: [(Reference)](http://blog.codebykat.com/2013/05/20/three-octopress-tweaks/)

{% codeblock lang:bash %}
cd ~/octopress/
{% endcodeblock %}
`vi _config.yml`

Add the following below the `github_skip_forks:` tag:
{% codeblock %}
github_skip_repos: [username.github.io]
{% endcodeblock %}
If you want to hide more than your GitHub page repo, you can specify all repos that you want to hide like below.
{% codeblock %}
github_skip_repos: [username.github.io, another_repo, whatever_repo]
{% endcodeblock %}

`vi source/_includes/asides/github.html`
Add the following line below `skip_forks: {{ "{{" }}site.github_skip_forks}},`
{% codeblock lang:javascript %}
skip_repos: [ {{ "{%" }} for repo in site.github_skip_repos %}"{{ "{{" }}repo}}"{{ "{%" }} unless forloop.last %}, {{ "{%" }} endunless %}{{ "{%" }} endfor %} ],
{% endcodeblock %}

`vi source/javascripts/github.js`
Add the following line below `if (options.skip_forks && data.data[i].fork) { continue; }`
{% codeblock lang:javascript %}
if ( jQuery.inArray( data.data[i].name, options.skip_repos ) != -1) { continue; }
{% endcodeblock %}


### Custom 404 error page:
{% codeblock lang:bash %}
cd ~/octopress/
{% endcodeblock %}

`vi source/404.markdown`
{% codeblock lang:html %}
---
layout: page
title: "404 Error"
comments: true
sharing: false
footer: true
---

<p>Sorry, the page you&#8217;re looking for cannot be found.</p>

<p>Please use the search function in the top navigation bar if you&#8217;re looking for something specific.</p>
{% endcodeblock %}


### Custom Domain:
{% codeblock lang:bash %}
echo 'your-domain.com' >> source/CNAME
{% endcodeblock %}

Visit your domain registrar or DNS host and add a record for your domain name.
For a sub-domain like `www.example.com` you would create a `CNAME` record pointing to `username.github.io`.
If you are using a top-level domain like `example.com`, you must use an `A` record pointing to `204.232.175.78`.


### Easy deploy/upload new pages script:
{% codeblock lang:bash %}
sudo vi /usr/local/bin/octoupload
{% endcodeblock %}

{% codeblock lang:bash %}
#!/bin/bash

cd ~/octopress/

rake generate
rake deploy
git add .
git commit -m 'source commit'
git push origin source
{% endcodeblock %}

{% codeblock lang:bash %}
sudo chmod +x /usr/local/bin/octoupload
{% endcodeblock %}


### New posts:

{% codeblock lang:bash %}
cd ~/octopress/
rake new_post["My New Post"]
{% endcodeblock %}

Edit the `source/_posts/my-new-post.markdown` with any text editor.

{% codeblock lang:bash %}
rake generate
rake preview
{% endcodeblock %}

Open `http://localhost:4000` to see a preview of the blog. When satisfied, the previously created, `/usr/local/bin/octoupload`, script can be executed to automatically commit all changes and upload to GitHub Pages.

{% codeblock lang:bash %}
octoupload
{% endcodeblock %}


### Free mardown text editors for Mac OS X:

1. [Mou](http://mouapp.com/)
2. [LightPaper](http://clockworkengine.com/lightpaper-mac/)
