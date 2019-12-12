---
layout: post
title: "HTB - LaCasaDePapel"
author: "mrs3c"
categories: Blog
tags: [hackthebox, htb, pen-testing, certificates, cronjob, nodejs, openssl, otp, php, psysh, ssh rsa auth, ssh, vsftpd]
image: lacasa_logo.png
---

I had trouble with the OTP token on this box: I never figured out why but whenever I scanned the QR code with my Google Authenticator app it would always generate an invalid token. Using a Firefox add-on I was able to properly generate the token to get access to the page. As a nice twist, the login shell was changed to psysh so I couldn’t use the vsftpd exploit to get a full shell on the box. LaCasaDePapel has some typical HTB elements: scavenger hunt for SSH keys, base64 encoding and a cronjob running as root for final priv esc.

## Summary

[Getting Started]({{ site.github.url }}{% post_url 2016-10-10-getting-started %}): getting started with installing Millennial, whether you are completely new to using Jekyll, or simply just migrating to a new Jekyll theme.

## Nmap

[Text and Formatting]({{ site.github.url }}{% post_url 2016-09-09-text-formatting %})

## Questions?

This theme is completely free and open source software. You may use it however you want, as it is distributed under the [MIT License](http://choosealicense.com/licenses/mit/). If you are having any problems, any questions or suggestions, feel free to [tweet at me](https://twitter.com/intent/tweet?text=My%20question%20about%20Millennial;via=paululele), or [file a GitHub issue](https://github.com/lenpaul/Millennial/issues/new).

## More Jekyll!

### Lagrange

Lagrange is a minimalist Jekyll blog theme that I built from scratch. The purpose of this theme is to provide a simple, clean, content-focused blogging platform for your personal site or blog.

Feel free to check out <a href="https://lenpaul.github.io/Lagrange/" target="_blank">the demo</a>, where you’ll also find instructions on <a href="https://lenpaul.github.io/Lagrange/journal/getting-started.html">how to use install</a> and use the theme.

### Portfolio Jekyll Theme

This is a Jekyll theme built using the [DevTips Starter Kit](http://devtipsstarterkit.com/) as a foundation for starting, and following closely the amazing tutorial by [Travis Neilson over at DevTips](https://www.youtube.com/watch?v=T6jKLsxbFg4&list=PL0CB3OvPhDA_STygmp3sDenx3UpdOMk7P). The purpose of this theme is to provide a clean and simple website for your portfolio. Emphasis is placed on your projects, which are shown front and center on the home page.

Everything that you will ever need to know about this Jekyll theme is included in [the repository](https://github.com/LeNPaul/portfolio-jekyll-theme), which you can also find in [the demo site](https://lenpaul.github.io/portfolio-jekyll-theme/).

### Jekyll Starter Kit

The Jekyll Starter Kit is a simple framework for starting your own Jekyll project using all of the best practices that I learned from building my other Jekyll themes.

Feel free to check out <a href="https://github.com/LeNPaul/jekyll-starter-kit" target="_blank">the GitHub repository</a>, where you’ll also find instructions on how to use install and use the theme.
