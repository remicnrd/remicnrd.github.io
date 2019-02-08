---
layout: post
title: Blogging with Jekyll and GitHub Pages
date: 2018-01-14 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: post_1.jpg # Add image post (optional)
tags: [Blogging, Jekyll, Themes, GitHub Pages]
author: # Add name author (optional)
---
**I recently switched my blog to Jekyll and GitHub pages.** 
I started blogging last year and went for WordPress without thinking too much about it. But I'm happy with the change and wanted to share the setup as my first article here.

**GitHub Pages**  is a static site hosting service, designed to host pages directly from a GitHub repository. It is based on Jekyll.

**Jekyll**  is an open-platform, blog-aware, static site generator written in Ruby. It basically transforms plain-text files into a website, interpreting [Markdown][markdown-def] and [Liquid][liquid-def]


Blogging with Jekyll and GitHub Pages is nice because:
- It's free
- It's open-source
- Markdown is fantastic for text editing
- My notebooks,  scripts and  blog articles are now all on GitHub
- It only takes a few minutes to create a blog, and modifications are easy

# Create your first Blog using Jekyll
To begin with, Jekyll and its prerequisites needs to be installed on your computer. If you are using Windows, Jekyll is not officially supported but you can make it work following the instructions [here][jekyllwin]. Otherwise:
- [Install Ruby][install-ruby] or check your version with `ruby -v` in a terminal
- [Install RubyGems][install-rubygems], the Ruby package manager, or check your version with `gem -v`
- [Install GCC][install-gcc] and [Make][install-make], or check you version with `gcc -v`and `make -v`
- Now you can install Jekyll  and Bundler by running `gem install jekyll bundler`

Once you are all set up, it is time to actually create the blog. Open your terminal, go the the folder of your choice and run:
```
# Create the Jekyll site in current folder
jekyll new .

# Build the site on the preview server
jekyll serve
```

Here is how the folder and the terminal should look like:

![Jekyll Screenshot 1]({{site.baseurl}}/assets/img/jekyll_screenshot_1.jpg)


As you can see, Jekyll created the needed files in your folder. Going to `http://127.0.0.1:4000/`on your browser should show the following:

![Jekyll Screenshot 2]({{site.baseurl}}/assets/img/jekyll_screenshot_2.jpg)

This is it! **You created your first blog, congratulations!**
It is a simple, un-themed version only hosted locally for now. The goal is to understand the main architecture and important files here.
 
By modifying `config.yml`, you can edit the principal information of the newly created site, such as adding your name and the name of the blog, or create a description. 
The `_posts`directory is for all the articles. You want to publish a new blog post? Just create your Markdown file and put it here.


# Make it shine 

You now have the basics of Jekyll, including Blog creation, local rendering and file browsing. Let's see how to obtain something even more pleasant to the eyes of your future readers.

An very interesting feature of Jekyll is the themes. A theme can change the fonts, the main page layout, the navigation menu of the blog, etc. A quick google search should help you find the perfect theme. Here are some availables at [Jekyllthemes][jekyllthemes]:

![Jekyll Screenshot 3]({{site.baseurl}}/assets/img/jekyll_screenshot_3.jpg)

Once you found the theme you like, check its documentation on GitHub. Sometimes it can be installed in a few seconds and two lines of codes using Gem:
- Add `gem  YOUR_THEME_NAME` to `Gemfile``
- Add `theme: YOUR_THEME_NAME` to `_config.yml`
- Run `bundle` in your terminal

Otherwise, you can go on the GitHub repository of the theme and clone it locally. In that case, the content of the theme will constitute your blog's base. Meaning you have to clone the repository in a new, empty folder with the following code on your terminal:
```
# Go to the folder you want to clone the theme in
cd blog

# Clone the git repository
git clone https://github.com/YOUR-THEME-REPOSITORY
```
From here, you can do a bit of cleaning by removing the example markdown files of the articles in `_posts`, and modify the `config.yml`. I encourage you to read the configuration file and GitHub documentation entirely, as some themes provides different functionalities.







# Publish it with GitHub Pages
Time to show your creation to the world!  This is where GitHub Pages comes in.

Go to your GitHub profile (or [create one][join-github]), and create a new repository called `YOUR_USERNAME.github.io`.  This repository will be linked to your local blog file.

Then ([install git][install-git] and) open the terminal in your blog directory and type:
```
# Create a git repository from your folder
git init

# Add all the files to the git repo
git add .

# Commit the changes to the repo
# git commit -m "first commit"

# Add your GitHub repository as remote origin 
git remote add origin remote repository git@github:YOUR_USERNAME/YOUR_USERNAME.github.io

# Push your local repository content to GitHub
git push -u origin master
```

From here, all the content of your blog folder will be uploaded to your GitHub repository. You have nothing else to do! Github Pages will render your blog at the address `YOUR_USERNAME.github.io`.

To dig deeper or if you encounter any problem during the process, you can check the official documentations of [Jekyll][jekyll-doc], [GitHub Pages][githubpages-doc] and [Git][git-doc]

[markdown-def]: https://blog.ghost.org/markdown/
[liquid-def]: http://shopify.github.io/liquid/](http://shopify.github.io/liquid/
[jekyll-win]: https://jekyllrb.com/docs/windows/
[install-ruby]: https://www.ruby-lang.org/en/downloads/
[install-rubygems]: https://rubygems.org/pages/download
[install-gcc]: https://gcc.gnu.org/install/
[install-make]: https://www.gnu.org/software/make/
[jekyllthemes]: http://jekyllthemes.org/
[join-github]: https://github.com/join?source=header-home
[install-git]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
[jekyll-doc]: https://jekyllrb.com/docs/home
[githubpages-doc]: https://help.github.com/categories/github-pages-basics/
[git-doc]: https://git-scm.com/doc
