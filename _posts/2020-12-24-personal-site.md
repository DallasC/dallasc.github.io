---
title: Personal Website
sample: UI /UX Exploration
date: 2020-12-26 12:00:00 +07:00
category: Design
tags: [design, html, scss, frontend, css, UI, UX]
description: Thoughts and considerations when designing a personal website from UI and UX to the technical implementation.
---
# Overview
Here's the process I went through when designing my personal website. There's no right or wrong way to do it but maybe this process can help others who are considering to make a personal website.

## Purpose
The first thing to do when making any website is to find the purpose. Was I blogging, writing, selling something, using it as a portfolio, a mixture of these. All of these things change how the site would look and function.

For me the purpose is to give me a place to flesh out my ideas and summarize my projects that I've worked on. I do this by writing, it helps me organize my thoughts and present my ideas in a more structured way.

## Audience 
Next you have to consider who is going to be viewing your website or who you want to view it. Customers, readers, recruiters, yourself, and so on. Different types of people expect different things from your website and knowing who is looking at your site allows you to optimize for that group.

To help with this I made a list of who I want to view my site and the priorities.
- Myself 85%   
As stated in the purpose this is mostly for myself. As such I can prioritize things that I want and don't have to include stuff related to converting viewers like site analytics, back linking, ads, etc
- Readers(technical audience) 10%    
I still would like my content to be accessible for other people with similar interests so it can be helpful to them. This means its important to include at least some SEO and discovery options but the site is not going to have a ton of features catering to readers like comments, etc.
- Recruiters 5%   
Not really a priority at all but since I detail a lot of my projects here anyways, the site can double as a portfolio. The only thing extra I really took into consideration was to make sure I contact page with a little of my professional experience listed and details on how to reach me.

As you can see depending on who your audience is can greatly effect your site. Since my site isn't targeting customers I don't have to include store options, and since I'm not fully focused on Viewers I don't really have to include site analytics, crazy SEO optimizations, ads, etc. 

## Hosting
There's a million ways to host a static site both free and paid. Some of the most popular ways are [Netlify](https://netlify.com), [Vercel](https://vercel.com), and [Github Pages](https://pages.github.com). They are all good options for a basic static website. 

Netlify & Vercel are better options than github pages if you are offering a more serious full featured website that customers and users rely on. They offer a better user experience with a custom dashboard and paid plans that have plugins to handle analytics and high workloads. 

Github Pages is bare bones on what it offers but is integrated directly into github itself making it convenient if you are already using github. Another drawback is that it only offers support for Jekyll built in. If you want to use a different static site generator you have build it through a CI setup then you can host just the static site.

I chose to use Github Pages for two reasons. 
1. I don't need the extra features that the other providers offer. If I was doing a commercial website I would value those features but for my purposes I won't be needing them.
2. I already use Github for work and open source contributions. Not having to sign up and manage another service is always nice. I'm a big believer in keeping things as simple as they can be. The less moving parts the better.

## Static site generators
The easiest way to get a site up and running is to use a static site generator. Since I don't need a database or full featured cms for anything this suites my needs perfectly. 

A quick search for "static site generator" and you'll find a bunch of these. They all do pretty much the same thing: convert markdown into a static website. Any of them are good choice and it's fairly easy to switch a different one.  I ended up choosing `Jekyll` since that’s what Github pages supports.

## Theme
Getting a theme is both easy and hard. Jekyll(and other static site generators) support themes and there are a large selection to pick from. This is the easy part. Finding a theme that you like and that fit your needs is the hard part.

What I wanted:
- Basic SEO support
- Minimalistic design
- Article search
- Light/Dark mode support
- responsive for mobile/tablet/desktop

A pretty small requirement list but I had trouble finding one I liked that covered everything I wanted so I ended up building my own which is what you see here on this site.

You can also look at the code on [github](https://github.com/dallasc/dallasc.github.io)

## Design
First thing I did was create the `base.html`. This is the layout that all the pages on the website use. 

At the top the `layout: compress` tells jekyll to run it through `compress.html` which does all the common html compression techniques in order to minimize the size of the static webpage. 

Next you'll see several `include` statements. These are common html components that can be reused. They are stored in the the `_includes` folder. As you can see we have:
1. header.html   
This contains a lot of seo stuff which is automatically generated as long as calling the scripts, css, page title, and all the common header related tags. Since it is dynamically generated by jekyll for each page we can include it and jekyll makes sure that the right title, tags, and seo options are set for each page.
2. themer.html   
This is a small basic js script that enables our theme switch to work. If you notice the `body` part of the html statement includes `data-theme` which we dynamically set based on the system theme. This way when users are in dark mode they aren't blinded when opening the website. We also allow the user to change modes manually to suit their preferences if the system mode isn't what they prefer at the moment.
3. navbar.html   
This is the bar at the top of the page. You want your navbar to be consistent across all pages because you don't want users to have to learn new experiences each page they go to. For my nav bar I keep it simple with links to different pages and the theme switcher. Also, it is dynamic so for mobile phones it'll turn into a hamburger menu on the top right.
4. content   
This is where the main content of the page goes. This can be different depending on the page. The `about` page and `home` pages have slightly different layouts for example.
5. footer.html   
This contains things such as the `footer` at the bottom of the page but also dynamically builds links to last/next articles if viewing an article.


