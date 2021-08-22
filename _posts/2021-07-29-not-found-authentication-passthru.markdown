---
layout: post
title: 'Omniauth: Not found. Authentication passthru'
date: 2021-07-29
categories: [Rails]
image: 'facebooklogin.jpg'
author: 'Harry Herskowitz'
author_blurb: 'Harry is a web developer and videographer with a focus on using technology to empower local artists and communities'
author_link: 'https://github.com/roldyclark'
avatar: 'roldy.png'
---

![wordpress.com screenshot]({{ site.baseurl }}/assets/images/posts/facebooklogin.jpg)

Last week, Facebook and Google logins stopped working in a Rails app I developed. Clicking the login links redirected to a blank page that read "Not found. Authentication passthru".

It took me some digging to find an answer, but at the bottom of a stack overflow thread user Washington Botelho had the answer with one upvote. So I thought I'd repost his answer so someone out there will have an easier time finding it:

> It can happen when you're trying to use link_to where the request will be a GET. You need to change it to a button_to where a form will be created. Alternatively, you can use link_to with method: :post if you have the rails-ujs, but I recommend you use the form since it'll have the CSRF on it; You need to add the gem omniauth-rails_csrf_protection to avoid Authenticity Error.

As user Flov points out in the comments: As of v2.0.0, OmniAuth by default allows only POST to its own routes.
