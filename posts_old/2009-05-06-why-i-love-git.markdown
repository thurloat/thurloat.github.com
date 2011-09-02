--------------------------------------------------------------------------------
author: Thurloat
date: 2009-05-06 15:59:20
layout: post
slug: why-i-love-git
status: publish
title: Why I Love Git.
categories:
    - Programming
--------------------------------------------------------------------------------

I made some changes to a production site that i haven’t looked at in a
few days. i knew the designers had changed some stuff, so i had to get
both repos to the same spot. in comes git (copy and pasted from my
history)

> git status\
> git add modules/EComm/plugins/products/ECommProduct.php \
> git add modules/EComm/templates/Product.tpl \
> git commit -m “adding moredetails plugin”\
> git push origin \*\*\*\*\*\*\
> git remote show deploy\
> ssh \*\*\*\*\*\*\*\*@\*\*\*\*\*.norex.ca\
> git status\
> git add images/CarouselBg.png\
> git commit -m “adding justins changes”\
> exit\
> git pull deploy \*\*\*\*\*\*\
> git push deploy
