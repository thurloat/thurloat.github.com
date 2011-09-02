--------------------------------------------------------------------------------
author: Thurloat
date: 2008-08-06 23:34:00
layout: post
slug: tcl-scripting-day-2
status: publish
title: Tcl Scripting - Day 2
categories:
    - eggdrop
    - programming
    - tcl
--------------------------------------------------------------------------------

Well i spent a lot of today getting the bot up and running, actually
responding to commands in the IRC channel. He is finally working well
enough. I’ve run into a few **snags** along the way, for a while i
couldn’t for the life of me figure out why my procs always error’d out
with a “wrong \# of args” when i was clearly passing through the right
amount. This was what i HAD:

> proc pub:setprofile {proService proValue} {

and i passed 2 variables through. easy enough i thought. as it turns
out, when calling an proc through an IRC command, it by default needs a
bunch of arguements (nick, host, handle, and channel) fast forward 30
seconds, and an hour of mysery has disapeared.

Once i got that out of the way, it was clear sailing for a while until i
started using some ifs, whiles, commenting, messing with strings etc.
and when it was time to test, i got an error i can’t make 2 ends of:

> Tcl error [pub:setprofile]: extra characters after close-quote

I’ve been through the code 8 times at least. It doesn’t look like i have
any syntax errors, missing quotes, brackets or anything. So while i’m
waiting for TuxOtaku to setup mysqltcl package on the server, i need to
find out what the hell this error exactly means.

If anyone can enlighten me please hit me up : thurloat at gmail dot com
.
