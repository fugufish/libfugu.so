---
layout: post
title: "The Wrong and Right of Rewrite"
date: 2012-01-05 11:18
comments: true
categories: software engineering, project management
---

Almost every developer has faced it. Staring down code with frustration building until something in side you just snaps. Suddenly you see the light. This code is crap. Useless. There is too much duplication. It's too hard to understand. There's no documentation and the guy who wrote went missing in a jungle Safari in the Congo. It's time to rewrite. Or is it? 

The rewrite is an easy trap to fall into. You have to ask yourself though, am I considering a rewrite for the right reasons. The answer lies in the ability for you to be honest with yourself. First, a rewrite is NEVER a snap decision. If it took you less than a few days to determine you need to rewrite then you are %99.99 likely to be doing it for the wrong reasons. Having recently found myself considering a rewrite, I decided to stop myself and think it through. Here is a small list of various reasons for a rewrite. Some good, some bad.

## The Wrong Reasons (or the most commonly used excuses)
* I hate PHP. Or whatever language it is you find a particular distaste for. If this is your reason, you have probably spoiled yourself (I'm talking to most Ruby developers, including myself). Changing languages should only be a factor if you already have legitimate reasons for doing a rewrite.
* I don't want to learn the code. The actual typical excuse is that it was just "done wrong". I think this is the most common excuse for a rewrite. Face it, working with other peoples code is a part of being a Software Engineer.
* The person that wrote the original code sucks. This is often closely linked to the above. They suck because I cannot understand it.
* It's not the way I would have done it. Well of course it isn't. Code style is almost as unique to a Software Engineer as a fingerprint.

## The Right Reasons
* Unusable in it's current state. If the codebase is unusable a rewrite is justified. Keep in mind that unusable code doesn't mean unreadable code, or code you don't understand, or code you don't want to debug.
* NOBODY can read it. Okay, if it is that unreadable then it is likely that a rewrite is justified.
* The codebase cannot support the current requirements of the software, either due to language limitations or the software was created in a way that prevents th requirements being met. A good albeit controversial example is Twitter's move from Ruby to Java.
