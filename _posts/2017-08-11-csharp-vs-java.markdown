---
layout: post
title:  "Clash of the Titans: C# vs Java"
date:   2017-08-11 17:29:00 +0000
categories: thoughts
comments: true
---

### Background
I started Java at university in 2009. It was my first experience of programming and... I hated it. It wasn't until nearly 6 months in that things finally started to click into place whilst I was building a Tetris clone. Throughout university I would compare Java to all the other languages I encountered and it generally came off favourably. A reasonable set of core libraries, great open-source community and cross-platform support all helped it to secure a place in my heart. At the same time I was turned off to C# as it seemed to be tied to the whole Microsoft/Windows ecosystem.

Fast-forward 5 years and I've spent most of that time in professional development with both Java and C#. I feel like I'm just about in a place where I'm able to weigh in on some comparisons between the two languages.

Let me start off by saying that I think both of them are fantastic general purpose object-oriented languages. The post aims to address some of the differences I've encountered but is by no means comprehensive. What follows is an incredibly brief and mainly opinionated view of the world as of 2017.

### Syntax
Whilst the bulk of the syntax is fairly similar between the two (albeit with analogous keywords), this one has to go hands down to C#. Having grown very tired of writing getters and setters in Java, the syntactic sugar of a C# property was a dream come true for my fingers. Throw in string interpolation, collection initialisation and `var` and it's hard to argue that C# isn't a clear winner. This richer vocabulary makes writing C# more pleasant but bear in mind that if you're starting off there is more to learn. I have to say I'm still not a fan of Pascal-case as the language case standard but oh well... it could be worse!

### Language Features
This is where C# really starts to pull away from Java. One of the biggest benefits of C# for me has been the Task Parallel Library and the `async`/`await` keywords. All of this allows you to write some really succinct asynchronous code in a standard format.

The handling of exceptions is probably the most contentious difference between the two languages, but I generally prefer the lack of checked exceptions in C#. Having native constructs for events and operator overloading are also big wins, although I haven't used the later an awful amount.

Extension methods in C# are also a nice feature but I believe they should be used sparingly or they make the code base harder to test and more difficult to traverse. The LINQ library being a good example of how to use extension methods properly.

 In addition, there are a greater number of primitives types available to you in C# and structs can also provide some performance enhancements in certain situations. Being able to pass by reference is also commonly noted as a pro to writing C#, but I rarely find it's required and think it can lead junior devs to write unwieldy code.

 Java's one saving grace is its enum support. Allowing for multiple enum fields rather than the somewhat glorified ints of C# is a plus, but this is simply not enough to usurp C# from the throne!

### Ecosystem
One of the things I initially loved about Java was the huge community; both in terms of open-source and learning resources. I believe the cross-platform nature, accessibility of the language and free tooling around Java really helped to push it to such a high level. When I started learning C# I found online learning resources pretty useful, but a great deal of the open-source software I came across was simply a less well maintained Java port. Saying that, things do seem to have a gotten a little better in recent years and .NET Core will no doubt help with mainstream adoption and bolstering of the community. Back when it launched, the inability to officially run C# on Linux in production was a deal breaker for many people, so it's great to see Microsoft now supporting this with .NET Core.

![StackOverflow feedback events](/img/top-tech-stackoverflow.png)

###### Number of [feedback events on StackOverflow](https://insights.stackoverflow.com/survey/2016#technology-top-tech-on-stack-overflow) in January 2016.

When it comes to tooling, I think Java is quite considerably ahead of C#. I am admittedly a huge JetBrains fanboy, but Visual Studio is nowhere near as good of an IDE as IntelliJ in my opinion (I could probably write a whole post about this). The fact that VS is verging on unusable without ReSharper (a JetBrains plugin) somewhat speaks for itself. With the announcement of .NET Core, JetBrains have decided to build a standalone C# IDE called Project Rider, so there's still hope yet! I imagine as time goes on things will level out between the Java and C# ecosystem, but for now Java wins this one.

### Future
The battle of C# vs Java is a close one and one that is likely to get more interesting in the next few years. In terms of a programming language C# is by far the more fully-featured of the two and is more of a pleasure to write. Java on the other hand has had a big lead in the open-source and cross-platform community but C# looks likely to close that gap with the release of .NET Core. In conclusion then, both of them are great languages, but as of 2017, I think C# just beats Java for me. The battle is far from over though, so lets revisit this in a few years time...

**Disclaimer**: _I've likely missed many valid comparisons in this list but these are just my personal differences that stood out to me when working with them for the last few years._
