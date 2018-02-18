---
layout: post
title:  "My wordpress blog"
date:   2014-04-06
excerpt: "word press blog built on google cloud"
image: "/images/wordpress/users-vs-latency.png"
---
(NOTE THIS REFERS TO AN OLD VERSION OF THIS BLOG)

## About this blog

After months of putting off this project I’ve finally decided to wrap up all of my writings in one place. Here. While deciding what the best approach to setting up my new blog, I decided to branch out from my normal approach and in the end I decided to use Google’s App Engine in combination with word press.

Google made it easy enough to get set up with a a ready-to-go fork of word press specifically designed for Google app engine. It always convenient when software comes ready to go for the cloud. Often times people underestimate the amount of effort that can go into porting a web app to a cloud based infrastructure.

![users vs latency]({{ "/images/wordpress/users-vs-latency.png" | absolute_url }})

Once the site was up and running I wanted to test how well the site would scale. One of the greatest allures of the cloud is the dream of easy scaling to millions of users (this is often a difficult to reach goal, more on this later). This blog utilizes the batcache plugin to speed up the rate at which pages were rendered and served up. Normally I would use JMeter, an open source testing website performance testing tool, but in the sake of time I decided to use a web service called BlazeMeter to automate the process. So far I’ve been very happy with the results. I’m always in search of tools that make my life easier and blaze meter seems to fit the bill in this particular case.


![analytics]({{ "/images/wordpress/wordpress-blog-analytics.png" | absolute_url }})