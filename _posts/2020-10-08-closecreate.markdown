---
layout: post
title: 'Geolocation for Creative Collaboration'
date: 2020-09-04
categories: [React, Node, JS]
image: 'closecreate2.jpeg'
author: 'Harry Herskowitz'
author_blurb: 'Harry is a web developer and videographer with a focus on using technology to empower local artists and communities'
author_link: 'https://github.com/roldyclark'
avatar: 'roldy.png'
featured: true
hidden: true
---

![closecreate screenshot]({{ site.baseurl }}/assets/images/posts/closecreate1.jpeg)

Today I am happy to release Closecreate, an open source web app for bringing together creative collaborators. The app allows users to create a profile which includes a picture, location, bio, and social media links to showcase their work. They can then scroll through nearby profiles to match with in a tinder-like fashion. Once two users match they can instant message each other and discuss future collaborations.

![closecreate screenshot]({{ site.baseurl }}/assets/images/posts/closecreate2.jpeg)

The application is built with React on the client side, Express (Node) on the server side, and a MongoDB database - also known as the MERN Stack. Images are uploaded and served through AWS S3. Each user's current location is obtained at login through the Geolocation API and profiles within 30km are displayed. The app is styled with Halfmoon, a bootstrap alternative with built-in darkmode. Socket.io is used in the chat component for realtime messaging.

![closecreate screenshot]({{ site.baseurl }}/assets/images/posts/closecreate3.jpeg)

I am currently hosting the app live on Heroku at closecreate.com. I'd love to see the creative community get use out of it! I also hope the open source repo is taken advantage of as this code can be reused to create all kinds of interesting matchmaking apps.
