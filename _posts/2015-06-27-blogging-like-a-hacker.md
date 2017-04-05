---
layout: post
section-type: post
title: Blogging Like a Hacker
category: tech
tags: [ 'jekyll' ]
---
I don't remember exactly how many blogs I created the past years, but this one feels like the one.
I had one blog on Tumblr and three or four on Wordpress.
Tumblr's brand didn't align with my vision of my blog.
I wanted a truly personal blog.
A blog where I could share my thoughts mainly on technology but at the same time, keep it open to explore other areas.
For example, at some point I might wanted to blog about music or personal thoughts on a viral and important matter, you name it. Tumblr wasn't the right place for this purpose, because its brand is purely recreational. On the other hand, Wordpress has a more elastic brand. You can use it for whatever purpose you want, but it was too complicated for me and of course the produced code was too heavy, given how little control you have on the generated code. Moreover, migrating a blog was a pain in the ass. After creating and deleting several blogs, I gave up Wordpress. One week ago a friend told me about [Jekyll](https://www.jekyllrb.com) and I decided to give it a try.
An easy decision, given my zero options at the time.

First of all, its origins are in GitHub, a company that I really appreciate and support.
My personal opinion about GitHub is that they changed our field, by transforming the day-to-day interaction with the best version control system into a UX.
Git is complicated but powerful and GitHub's client Apps for [Mac](https://mac.github.com) and [Windows](https://windows.github.com) made it easy and playful to use.
Also their [web access](https://github.com) promoted coding as a social network, which was ingenious because it worked as a motivation for people to open-source their work and help others.
Back to Jekyll.
Yes, reading that it was created in GitHub was more than enough for me to invest my time on it.
And yes, Jekyll needs some time invested on it, given that it's not a CMS, which means that you have to get your hands dirty.

I started from the official [documentation](https://jekyllrb.com/docs/home),
but soon I realized that it would be more fun to not reinvent the wheel,
but fork an existing open-source blog that I liked.
I searched a few themes, chose the [Timeline](https://kirbyt.github.io/timeline-jekyll-theme) and started playing with it.
Timeline is a mashup of the popular [Agency](https://y7kim.github.io/agency-jekyll-theme/) and [Grayscale](https://jeromelachaud.github.io/grayscale-theme/) themes.
Most Jekyll themes are sleek and <strike>mobile first</strike> responsive,
but Timeline won me because of the <strike>Career page</strike> Timeline (duh).
Also I loved the white on black text, because it improves significantly the reading experience and my vision was to create a website/blog that people would enjoy to spend a few minutes on it without getting tired.
The first day that I spent on Jekyll was to understand how it works and the result was to just replace existing placeholders and variables to create my Timeline. But that wasn't enough, I needed a blog.
I couldn't spend more than one day at the time, so I just published it for free on [GitHub Pages](https://pages.github.com).
I hadn't used GitHub Pages for a while, so I had forgotten how awesome it is to just push your changes to origin/master and get your website deployed with them out of the box. Also I was thrilled that I was able to use Emacs as my editor. My geek inside me was overwhelmed. The only downside with GitHub Pages, is that they don't allow dynamic pages (yes, Jekyll is producing static pages!), for security reasons.

One week later, I was able to spend one more day on it. The goal was to somehow integrate the Blogging feature in the Timeline theme, but pretty soon I realized that Timeline's functionality was hijacking the Blogging capabilities of Jekyll, because it was rendering the posts as timeline events. Bummer. This feature was one of the two main reasons that I picked Timeline, but lacking the Blog feature was not an option, I wanted it badly. I tried to see if I could manipulate Jekyll in order to support both the Timeline and Blog features, but in order to manage that I had to hack Jekyll and I had only one day to spend on this. So I ripped off Timeline's generated HTML code into a file and started reading how to use Jekyll's API to create a Blog. This [post](https://erjjones.github.io/blog/How-I-built-my-blog-in-one-day) was my most valuable resource. After a while and a [bug fix](https://github.com/kirbyt/timeline-jekyll-theme/pull/2) on Timeline I was there.
I opened the website from my phone to see how it renders and it was rendering as expected (thanks to Bootstrap).
By the way, I fixed a bug on my navigation bar from my bed, using my phone, by editing a source file on GitHub.
How awesome is that?

Writing blog posts using [Markdown](https://daringfireball.net/projects/markdown) and Emacs is exciting and keeping my draft posts on git [branches](https://github.com/PanosSakkos/panossakkos.github.io/tree/blogging-like-a-hacker) is kick-ass.
The next logical step is to make the ripped off Timeline HTML autogenerated, by defining custom Jekyll variables and then open-source it.

<pre><code data-trim class="bash">
git commit -a -m "Finished post"
git checkout master
git merge blogging-like-a-hacker
git push origin master:master
</code></pre>

:smile:

Edit: I tend to [keep](https://panossakkos.github.io/tech/2015/07/05/personal-jekyll-theme.html) my promises
