---
layout: post
title: "Is Your Website Quietly Paying for Bots?"
description: "Bots now make up a big share of website traffic, and some of them cost you money. How to tell the helpful ones from the expensive ones, and what to do about it this week."
date: 2026-10-23 09:00:00 -0700
published: true
image: /assets/blog/is-your-website-paying-for-bots.webp
tags: [tech-training, security, web-dev, wordpress, small-business, beginner]
---

If your website has felt slower lately, or your hosting company keeps nudging you toward a bigger plan, there's a decent chance the extra visitors aren't people. They're bots, automated programs that visit websites all day long. Some of them help you. A lot of them don't. And every visit, helpful or not, burns a little of the server you're paying for.

This isn't a fringe problem anymore. The web host Kinsta [analyzed more than 10 billion requests](https://kinsta.com/ai-bot-traffic/) and found traffic from AI bots grew roughly 300% in a single year. Your site doesn't have to be famous to get crawled. It just has to exist.

## Not all bots are bad

Before you go blocking everything in sight, it helps to know who's knocking:

- **Search engines** like Google send bots to read your pages so you show up in search results. Block these and you disappear from Google. Leave them alone.
- **AI crawlers** read your site to train AI tools or to answer people's questions. Some send you visitors and mentions in AI answers. Others take your content and send nothing back.
- **Monitoring tools** check that your site is up. Usually yours, usually fine.
- **Scrapers and attackers** copy your content, guess passwords on your login page, or stuff junk into your contact form. These are the gremlins costing you money.

The goal isn't "no bots." It's the right bots, in the right places.

## Where bots hurt the most

Most of your website is cheap to serve. A blog post or an "About Us" page gets saved ahead of time (that's called caching) and handed out fast, so a bot reading it barely registers.

The expensive pages are the ones your server has to build fresh every time: the shopping cart, checkout, site search, the login page and your forms. When a bot hammers those, the server does real work for every single request, and your actual customers end up waiting in line behind it. Picture a guy at the deli counter ordering one slice of turkey at a time, four hundred times in a row. If you run an online store, this is usually where the slowdowns come from.

## Signs your site has a bot problem

You don't need special tools to spot the symptoms:

- Pages load slower than they used to, or you see the occasional error
- Your host warns you about high resource usage or suggests an upgrade
- Your traffic numbers jump, but sales and contact form leads don't
- Your login page or contact form gets a steady stream of junk

If two or more of those sound familiar, it's worth a look.

## Allow, challenge or block

Once you know what's hitting your site, every kind of bot gets one of three answers:

- **Allow** the ones that help you, like Google and the AI tools that send you visitors.
- **Challenge** the ones you're unsure about. A challenge is a quick check (usually invisible to real people) that a simple bot can't pass.
- **Block** the ones that are clearly abusive or expensive and give you nothing back.

You can mix and match by page, too. It's perfectly reasonable to let a crawler read your blog while keeping it far away from your checkout.

## Build it yourself: what to do this week

1. **Ask your host what they offer.** A lot of hosts now include bot protection and reports showing which bots visit your site. You might already be paying for it.
2. **Put a free firewall in front of your site.** [Cloudflare's free plan](https://www.cloudflare.com/plans/free/) has a bot protection toggle and a one-click option to block AI scrapers. It also caches your pages, which makes the cheap pages even cheaper.
3. **Protect the expensive pages first.** If you're on WordPress, a security plugin like [Wordfence](https://www.wordfence.com/) can limit how fast any one visitor hits your login page and forms.
4. **Do NOT block Google.** If you take only one thing from this post, make it this one.
5. **Check again in two weeks.** Is the site faster? Did your real traffic and search rankings hold steady? Loosen any rule that caught something it shouldn't have.

None of this is a one-and-done fix. Bots change, new ones show up, and the rules need an occasional tune-up, the same way you'd check the locks on the shop every so often.

The bots will be back next week. They always are.

---

*Not sure what's hitting your website, or whether it's costing you? [Get in touch](/contact/) and I'll take a look and tell you what to allow, what to block and what to leave alone.*
