# Ticketmaster Mobile Studio Website

This is the [GitHub Pages](https://pages.github.com/) repo for the [TMS website](http://ticketmastermobilestudio.com/). As with all GitHub Pages sites, it runs on [Jekyll](https://jekyllrb.com/).

## Blog

Contributing to the blog is easy. That said, these instructions assume you're familiar with GitHub and understand how to do things like clone repos and create pull requests. If that sounds foreign, find a nerd and ask them for help.

Posts are written in [Markdown](https://daringfireball.net/projects/markdown/). Here's a couple good resources for that:

* [Writing posts](https://jekyllrb.com/docs/posts/) - from the Jekyll docs
* [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) - a good syntax reference
* [Minimalist Online Markdown Editor](http://markdown.pioul.fr/) - a nice web-based editor

Whatever editor you choose to write your post in, at some point you'll need to add it to this repo for review and posting. Here's how you'd do that:

1. Clone the repo
2. Create a branch for yourself and give it a descriptive name, like `darwin-sketch-tips-post`
3. Locate the blog posts directory in `/_posts/blog/`
4. Add a file for your post, following the guidelines below and referencing other posts
5. Commit and push your branch as needed to track your progress
6. When you're ready to have the post reviewed, create a pull request and inform the **#tms_blog** channel on Slack that your post is up for review
7. Address feedback as needed
8. Merge your pull request to `master` to publish it. Tada! 🎉

### File naming

Posts have filenames like `2017-04-01-sketch-tips-for-leet-hackers.md`. The first part is the publish date in `yyyy-mm-dd` format.

⚠️ **Using a future date will not prevent your post from going live immediately.** If you want to keep it unpublished until a specific date, add `published: false` to the front matter (see below) and remove it on that date.

The rest of the filename becomes the URL slug. Remember to update this if you change the name of your post while writing. After the post is published, avoid making changes to the filename, since that will break inbound links to the old URL and throw a wrench into page analytics.

### Front matter

Jekyll posts have what's called [Front matter](https://jekyllrb.com/docs/frontmatter/) at the top of the file. This is a YAML block of optional variables, and it looks like this:

```yaml
---
title: "Sketch Tips for Leet Hackers"
author: Darwin Campa
categories: Design Tutorials
tags: tips Sketch
---
```

There are lots of other variables you can use, but those are the essentials for TMS blog posts.

#### Categories and tags

The `categories` and `tags` variables are lists and can be space-separated or, if you have spaces within your tags, in bracket form like this:

```yaml
tags: [tips, Sketch, a tag with spaces]
```

To make browsing easier, try to choose one or more categories for your post from the established list below. If you feel that your post needs a new category, go ahead and add it and we'll discuss it during review.

* Android
* Culture
* Development
* Design
* iOS
* Libraries
* Open-Source
* Product
* Testing
* Tutorials

Tags can be anything you want. They are used to associate related content, so if you use the tag "pickles," other posts tagged with "pickles" will show up in the sidebar on your post and vice versa.

Note that categories are title-case, whereas tags should be lowercase except for proper nouns and abbreviations (e.g., Apple, AAPL). This is just a stylistic preference we've set for our blog.

### Headings

If your post has headings, start them at the H3 level, which looks like this in Markdown:

```markdown
### I'm a heading!
```

This gives you 4 levels of headings to work with (H3, H4, H5, and H6). If you need more than that, your post is probably too long.

### Images

If your post has supporting images, you can create a folder in `/assets/` using the name of your post, for example: `/assets/2017-04-01-sketch-tips-for-leet-hackers/` and upload your images there.

#### Featured image

When your post is shared, it appears with a generic TMS graphic. If you want to use a different image (you do!), you can specify it using the `featured-image` front matter variable:

```yaml
featured-image: assets/2017-04-01-sketch-tips-for-leet-hackers/kittens.jpg
```

Facebook has some [sizing guidelines](https://developers.facebook.com/docs/sharing/best-practices#images) for this image.

### Excerpts

The post excerpt is the part that shows in the list, before you click through to the post detail page. By default, this is the first paragraph of your post, but you can specify the break anywhere by adding `<!--more-->` on a new line.

### New authors

If this is your first post, you'll need to add yourself to the `_data/authors.yml` file. Your Ket avatar should already be uploaded, so you just need to specify the path to it. If you'd rather not use your Ket, you can upload another avatar or use your Gravatar.
