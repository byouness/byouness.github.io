---
layout: post
title: Your blog with Jekyll and Github-pages
tags: [Jekyll, Github-pages, Markdown]
---

The idea to create a blog where I would share some thoughts, tips and tutorials was in my mind for some time.
After finally getting around to doing it, I thought the first post should, no! **must** be on how to do it to help people in the same situation where I was save some time...

## Prerequisites
 
### Github account
The idea is to host your website or blog on Github. It is free as long as you agree to keep your blog's content public.

First, if you don't have a Github account, start by creating one here for free: https://github.com/ 

Second, from your profile page, at __https://github.com/your_github_username__, click on the **Repositories** tab and clic new to create a new repository. Call it __your_github_username.github.io__.
It is important to stick to this name so that Github can identify this repository as a website and not as an ordinary repository.

Open your repository and add a file called __index.html__ containing the string **"Hello World!"**.

Now, opening the url __your_github_username.github.io__ in your browser will display a blank webpage with **"Hello World!"**. Your blog is up and running!

### Markdown
Of course, a blog with just a "Hello World!" is not very interesting and you want to have more interesting content in your blog.

Writing this content in HTML would be time consuming, especially if you don't have any experience with it. So, the idea is to use Markdown instead.

Markdown is a lightweight markup language with plain text formatting syntax. What makes it very useful is that it could be converted to HTML, LaTeX, doc and other formats, although it is way easier to write than any of those.

A good place to start is one the numerous cheat sheets available online, for example [here](https://www.markdownguide.org/cheat-sheet/). If you're more of a hands-on guy, then [this very simple tutorial](https://www.markdowntutorial.com) will get you up to speed in no time.

Any text editor will do to write Markdown content. However, if you're planning to write your blog posts directly in Github, then [prose.io](https://prose.io/) is a good option. It hooks hooks directly into your Github repository and allows editing, previewing and saving your posts as drafts.

### Git
If, like me, you're more comfortable writing and editing your blog posts locally on your machine, and then pushing them to Github once you're satisfied with the result, then you need a basic knowledge of Git.

Git is a free open source version control system. It tracks all the changes made to your files and enables more than one person to work on the same files.
For the very basic usage we want to make of it here, `clone`, `pull`, `commit` and `push` actions should be enough.

The idea is to clone your repository on some machine, do some changes, commit these changes, and then push them to Github...

This tutorial will get you up to speed in 15 minutes: [try.github.io](https://try.github.io)

## Jekyll

Github pages rely on [Jekyll](https://jekyllrb.com/). It turns the Markdown you wrote into HTML that can be rendered in Web browsers.

Installing Jekyll on your machine is needed only if you want to check your blog locally on your machine before pushing it to Github.

### Local installation (optional)

In this case, the bad news is that Windows is not an officially supported platform for Jekyll. The good news is that starting Windows 10, you can run Linux environments directly on Windows without a virtual machine, via the so-called Windows Subsystem for Linux.

To get started:

 1. Get the Windows Subsystem for Linux [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
 2. Install Jekyll and Ruby following these [instructions](https://jekyllrb.com/docs/windows/#installation-via-bash-on-windows-10)
 3. Create your first blog (empty with default theme), with this [quickstart guide](https://jekyllrb.com/docs/quickstart/)
 4. Choose one of the themes available and use it (see below)
 5. Enjoy :)

### Jekyll tempaltes

Rather than starting from scratch, starting with one of the very nice Jekyll blog templates available online will save you a lot of time:
- on the Jekyll github page: [Jekyll Github page](https://github.com/jekyll/jekyll/wiki/Themes) 
- or elsewhere: [http://themes.jekyllrc.org/](http://themes.jekyllrc.org/), [http://jekyllthemes.org/](http://jekyllthemes.org/), [https://jekyllthemes.io/](https://jekyllthemes.io/)...

Two possibilities, you could download a template, modify it locally then push it to Github, or directly fork the templates Github repository (i.e. copying it directly on Github).

https://blog.github.com/2016-12-09-publishing-with-github-pages-now-as-easy-as-1-2-3/

## Your blog

### Structure of Jekyll website

Usually, the structure of a simple Jekyll blog looks as follows:
```
.
├── _config.yml
├── _data
├── _drafts
|   ├── a-draft.md
|   └── and-another-draft.md
├── _includes
├── _layouts
├── _posts
|   ├── 2018-04-09-my-first-blog-post.md
|   └── 2018-04-10-my-second-blog-post.md
├── _sass
├── _site
├── .jekyll-metadata
└── index.html 

```

Here is a simplified description of the most important folders:

|File or folder | Description |
| --------------|-----------------------------------------------------------------------|
|_config.yml    | [Configuration](https://jekyllrb.com/docs/configuration/) of the blog such as its title, time zone, encoding, theme, plugins, etc.  |
| --------------|-----------------------------------------------------------------------|
|_posts         | This is where the blog posts (markdown files) will live. Their titles should be in yyyy-mm-dd-title-of-the-post.md format      |
| --------------|-----------------------------------------------------------------------|
|_drafts        | You can keep here any drafts in progress and yet to be published      |
| --------------|-----------------------------------------------------------------------|
|_data          | Data in yml, json or csv format used by your blog                     |
| --------------|-----------------------------------------------------------------------|
|_sass          | Files used for the styling of your blog (scss)                        |
| --------------|-----------------------------------------------------------------------|
|_site          | This is where the Jekyll generated site (HTML) will be located        |

Once the blog is set up, you will edit only the content of the **_drafts** and **_posts** folders. The rest will either be handled by Jekyll, or stay static. 


### Format of a Jekyll blog post:

A usual post should look like this:

```
---
layout: post
title: My first blog post
date:  2018-04-09
tags: [ blog, travel ]
---

#  Big Title
## Smaller Title

Post cool content here...

```

The header of the post is the so-called [YAML front matter](https://jekyllrb.com/docs/frontmatter/) of your blog post.
It can be used to set variables that are used by Jekyll or by you in the post or elsewhere...

## More 

### Comments with Disqus
Activating comments using [Disqus](https://disqus.com/) in your blog is very easy, all it takes is to:
 1. create an account on Disqus;
 2. register your website on Disqus;
 2. and add a `comments` variable, set to `true` in of the YAML front matter of the posts for which you want to enable comments like this:

```
---
layout: post
title: My first blog post
(...)
comments: true
---
```

### Domain name, RSS, etc.
Want to get a custom domain name, add RSS feeds, or activate Google Custom Search for your website, check this [excellent blog post](https://deanattali.com/2015/03/12/beautiful-jekyll-how-to-build-a-site-in-minutes/) by Dean Attali.