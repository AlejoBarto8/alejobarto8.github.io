---
layout: post
title:  "Validation Writeup - Hack The Box"
date:   2023-08-23
desc: "Abusing Printer, Abusing Server Operators Group, Service Configuration Manipulation"
keywords: "HTB,eJPT,OSCP,Easy"
categories: [HTB]
tags: [HTB,eJPT,OSCP,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.



1. Fork [this project -- Jalpc](https://github.com/jarrekk/Jalpc) at [GitHub](https://github.com). If you want to edit website at github, do it as following gif or clone forked repository. `git clone git@github.com:github_username/Jalpc.git`.

	<!-- ![edit]({{ site.img_path }}/returnwriteup/Return.png) -->
	<img src="{{ site.img_path }}/validation_writeup/Validation.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">

2. Enter into repository directory and edit following file list:

	* **_config.yml**: edit 'Website settings', 'author', 'comment' and 'analytics' items.

	* **_data/landing.yml**: custom sections of index page.

	* **_data/index/**: edit sections' data to yours at index page, please notice comment at each file.

	* **_data/blog.yml**: edit navbar(categories) of blog page, if you have different/more blog page, copy `blog/python.html` and change it to your category HTML file, and edit **Python**, **/python/** to your category name at items **title** and **permalink**, make sure title is the same as permalink but capitalized first letter(except HTML).

	* **CNAME**: If you wanna release website at your own domain name: edit it and create `gh-pages` branch; if you want to use *github_username.github.io*: leave it blank.

	* Go to repo's settings panel, config **GitHub Pages** section to make sure website is released.

3. Push changes to your github repository and view your website, done!

From now on, you can post your blog to this website by creating md files at `post/` directory and push it to GitHub, you can clear files at this directory before you post blogs.

If you like this repository, I appreciate you star this repository. Please don't hesitate to mail me or post issues on GitHub if you have any questions. Hope you have a happy blog time!😊
