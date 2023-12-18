---
title: "Learning Hugo(1)"
author: "Haobo Gu"
tags: [Hugo]
date: 2019-08-23T15:35:00+08:00
---



<!--more-->

## Content Source Organization

The top level of content folder is `./content/`. Under this folder, the contents can be organized and nested at any level. The following is an example:

![image-20190828171512051](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-08-28-091512.png)

The page origanization conforms to the content source organization. However, each type of pages has an ***index page***, which is the front page when you goes to this category. For instance, the index page of `tags` looks like: ![image-20190823154917184](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-08-23-074917.png)

The correspondence between source file paths and site urls is:

```sh
# Content file path
.         url
.       ⊢--^-⊣
.        path    slug
.       ⊢--^-⊣⊢---^---⊣
.           filepath
.       ⊢------^------⊣
content/posts/_index.md


# Site urls
                     url ("/posts/")
                    ⊢-^-⊣
       baseurl      section ("posts")
⊢--------^---------⊣⊢-^-⊣
        permalink
⊢----------^-------------⊣
https://example.com/posts/index.html
```

### Override Destination Paths in Front Matter

In front matter, the `slug`, `url`, and `layour` can be overridden. For example, if a post's path is `content/posts/old-post.md`, by default, its url would be `https://example.com/posts/old-post/`. We can set the slug in front matter to change the url:

```yaml
title: New Post
slug: "new-post"
```

 Then the url is changed to `https://example.com/posts/new-post/`

The url can be also overridden by `url` option in front matter:

```yaml
title: New Post
url: https://example.com/blog/new-url/
# OR
url: /blog/new-url/
```

In the second case, the url will be changed to `baseUrl + /blog/new-url/`.

For more information about url configuration, see [URL Management](https://gohugo.io/content-management/urls/)

## Section

A section is a collection of pages. Sections are organized under the `content/` folder. By default, all **first-level** folder under `content/` are sections. In deeper level, we can create a section by creating a folder with a `_index.md` file, which indicates that this folder is a section. 

>  A **section** cannot be defined or overridden by a front matter parameter – it is strictly derived from the content organization structure.

Hugo provides page variables:

- `.CurrentSection`
- `.FirstSection`
- `.Parent`
- `.Section`: First path element in the directory
- `.Sections`: Sections below this content

### Content Section vs Content Type

Content section is a organization-level concept, while content type is the template of a page, such as draft, post, etc. 

## Config Markdown Renderer

The default markdown renderer in Hugo is [Blackfriday](https://github.com/russross/blackfriday). There are lots of configurations in `config.toml` about Blackfriday:

| Config name     | Default value | Explanation                                                  |
| --------------- | ------------- | ------------------------------------------------------------ |
| taskLists       | true          | Markdown TO-DO List                                          |
| smartypants     | true          | Smart punctuation subsitutions, such as `(c)`-> ©            |
| fractions       | true          | Render fractions in a collapsed format                       |
| extensions      | []            | Enable [Blackfriday's Markdown extensions](https://gohugo.io/content-management/formats/#blackfriday-extensions) |
| HrefTargetBlank | false         | Open **absolute** links in a new window or tab               |

