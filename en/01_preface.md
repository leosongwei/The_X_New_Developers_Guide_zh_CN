The X New Developer's Guide
===========================

Alan Coopersmith
Matt Dew
The X.Org Development Team

*ed. Bart Massey*(?)

Published by the X.Org Foundation
October 2012

Preface
=======

Bart Massey

This is a guide intended to orient new developers in the world of the X Window System.

*That's asking a lot.*(?)

X is big and old. The distribution crossed a hundred thousand lines of C code back when that meant something, before there was an Internet to distribute it on: just reel tapes and dialup modem lines. X version 11, the one that matters, is celebrating its 25th birthday this year.

Big old systems are hard to understand. They grow and evolve. The more used they are, the faster growth and evolution happens. X is the third-most-used desktop environment in the world---and has been for most of its lifetime. There are millions of desktops running X right now, and the system is still adapting and changing to meet new needs of its users.

On the other hand, compared to similar large legacy software systems, there are some things that make getting into X easy. X is dominated by the C programming language. For all its faults, this is barely a legacy language: there are many, many more active C programmers today than when X began. X was exceptionally well-designed, and has been carefully redesigned and reimplemented in a modular and professional fashion as it has grown. The famous split between display and application that allows X to run over a network turns out to have an even more important purpose: it splits the X implementation into two parts connected by a formally-defined and well-understood interface. Within both the server and the client side, there is clear separation of codebases and code responsibilities, and that modularity actually seems to be increasing as the codebase grows.

There's a lot of documentation out there for X. Sadly, much of it is old and stale, or was just not that well written in the first place. *The good news is that many of the people who have worked on the design and implementation of X are still around, and still active in X development.*(?) Even better, they are a substantial fraction of the entire X developer base.

Let me say that again. A very few very smart people, over a 25 year period, have built most of the key X infrastructure, and these same people continue to build a lot of it today.

Keith Packard, "the Linus Torvalds of X" if there is such a thing, *once pointed out to me that there were, in its heyday, many times more active developers of Fetchmail than of X.*(?) In terms of amount of productive work per unit coder, I would put X up against any project, open source or proprietary, that I have ever come across. A few dozen people still do most of the maintenance of the multi-million-line "core X" codebase, many of them working only in their spare time. (X toolkits and applications long ago split off from the core, and formed their own extremely large and vibrant communities. Core X just continued along, mostly unaffected by this sea change.)

If you are thinking of doing X development, here is what you should understand: this disproportion of core developers is a tremendous opportunity for you. There is a huge amount of interesting, exciting work to be done. Because you will be working with well-understood technologies in a well-constructed setting, the work tends to be as much conceptual as it is labor-intensive: there are plenty of places you can design interesting things and then build them quickly. Because there is such a need for developers, and because they are genuinely nice and brilliant people, the founding architects of the systems that you will be working with are excited to help you get started---often on a one-on-one basis. Because the project is so widely deployed, you will have direct, visible impact on a huge userbase all over the world.

A lot of this is what open source in general promises its developers. X, in my experience as a developer and in watching other new X developers, delivers on these promises.

Sure, getting started as an X developer is a little tricky. Hopefully this book can help; certainly the X community can. Let me assure you of this, though: the learning curve is easier than you think, and the payoffs are greater. If you persist, you will befriend some great people and build some things that you will be proud of for the rest of your life.

So yeah, *it's asking a lot.*(?) Try it anyway. Check out the book. Ask questions. Build things.

*Dive in and get started.*(?)

—Bart Massey, Portland Oregon USA, March 2012


