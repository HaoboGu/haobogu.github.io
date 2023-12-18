---
title: "Get Started with Hugo"
author: "Haobo Gu"
tags: [hugo]
date: 2019-08-14T19:57:40+08:00
summary: Hugo installation and Github Pages configuration
---

## What's Hugo?

Hugo is a popular open-source static site generator. After my painful journey on trying Hexo and Jekyll, Hugo makes me happy.

## Get Started

### Install Hugo on Mac OS.

   ```sh
   brew install hugo
   ```

### Initialize the first website.

   ```sh
   hugo new site your-folder-name
   ```

### Set Github repo

   I'm using two repos for my blog: one stores all source files of hugo and the other stores generated websites.

   First, create a repo for source files in Github. 

   For example, my source repo is [github.com/HaoboGu/source-haobogu.github.io](https://github.com/HaoboGu/source-haobogu.github.io).

   After the repo for source files is created, set local hugo folder to track the repo.

   ```sh
   cd your-folder-name
   
   # Git initialization
   git init
   # Commit hugo files
   git add .
   git commit -m "first commit"
   # Track your remote repo
   git remote add origin git@github.com:HaoboGu/source-haobogu.github.io.git
   git push -u origin master
   ```

### Add a theme

   Hugo doesn't have a built-in theme, so I decided to use [hugo-nuo](https://github.com/HaoboGu/hugo-nuo)(my edited version). You can choose one from [themes.gohugo.io/](https://themes.gohugo.io/).

   ```sh
   # Import the hugo-nuo theme as a git submodule
   git submodule add git@github.com:HaoboGu/hugo-nuo.git themes/hugo-nuo
   
   # Add theme to config file
   echo 'theme = "hugo-nuo"' >> config.toml
   ```

### Add first page

   ```sh
   hugo new posts/Hello-Hugo.md
   ```

   You may notice that there is a generated header in the markdown file. This header is called [front matter](https://gohugo.io/content-management/front-matter/). By default, this page will not be publised because `draft = true` option in front matter template.

### Test your site locally

   Run

   ```sh
   # -D means draft will be publised
   hugo server -D
   ```

   Then navigate to your site at http://localhost:1313/.

### Deploy site to [Github Pages](https://help.github.com/articles/what-is-github-pages/)

   Create your Github Pages repo for hosting generated site.

   Suppose the site repo is `github.com/HaoboGu/haobogu.github.io`, then run 

   ```sh
   # Delete generated public folder
   rm -rf public 
   # Set public folder as a submodule which tracks Github Pages repo
   git submodule add -b master git@github.com:<USERNAME>/<USERNAME>.github.io.git public
   ```

   Create script [deploy.sh](https://github.com/HaoboGu/source-haobogu.github.io/blob/master/deploy.sh), then run this script to deploy the website

   ```sh
   chmod +x deploy.sh
   
   ./deploy.sh
   ```

   DONE!

