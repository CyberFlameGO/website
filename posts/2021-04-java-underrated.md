---
title: Java is criminally underhyped
description: The perspective of an ignorant computer science undergrad
date: 2021-04-15
tags:
    - opinion
layout: layouts/post.njk
---

It's likely that you read the title of this post and thought "what is this guy smoking? Java is everywhere!" You're correct, Java still dominates enterprise and runs some of the world's largest mission-critical applications. But Java's adoption isn't what I'm talking about, I'm talking about its *hype*. I spend a lot of time around inexperienced programmers. And what do inexperienced programmers love doing? **Getting excited and opinionated about tools like programming languages**. None of the CS undergrads I meet are hyped about Java but I think they should be.

Young/naive developers (myself included) often fall into the trap of fetishizing new languages and tools at the expense of productivity and sanity. Prior to working at Halp (now owned by $TEAM), I had a nearly romantic relationship with backend TypeScript. I thought the node.js ecosystem was the coolest thing ever: I loved the idea of transpiled code, live debugging, the massive package library, and even the weird and fragmented build systems. When I actually used it in production and spoke to more experienced engineers the magic quickly faded away. 

I had a irrational affinity towards the JS ecosystem because it was the hot new thing; it had *hype*. Reality did not live up to my expectations. Today, the wonderful things I expected from JavaScript I am currently enjoying as I gain experience in Java. **I feel betrayed that hype did not lead me to Java sooner.** Java is fun to write, productive, and gets an unfair reputation among new developers as a dinosaur.

## Ergonomics are what make Java great

This cannot be understated: Java simply feels good to write. A lot of this is due to the craftsmanship JetBrains puts into IntelliJ IDEA. Everything is autocompleted, jump-to-definition is fast, find-usage works well, and refactoring is easy. However, where Java truly shines is the **developer experience with third-party libraries**.

### Dependency heavy workloads and industry trends
My experience is limited, but I feel the winds have shifted in favor towards liberal usage of external dependencies. [Not Invented Here](https://en.wikipedia.org/wiki/Not_invented_here) is out, [Not Invented There](https://en.wikipedia.org/wiki/Invented_here) is in. JavaScript developers in particular are extremely likely to include third-party libraries even for [trivial operations like left-padding a number](https://qz.com/646467/how-one-programmer-broke-the-internet-by-deleting-a-tiny-piece-of-code/). I don't think the current affinity for third-party dependencies is particularly harmful, but upstream API changes can wreak havoc on untyped JS/Python code bases.

When consuming third-party libraries in Java, you always know exactly what types need to be passed to a method. Most importantly, incorrect usage of a function will result in red squigglies in your editor. Given that heavy library usage is in, I think more people should be excited about Java.

### Nominal typing saves time
There are a number of disadvantages with dynamic/duck/weak/whatever typing. When a dependency changes an API method and your application fails at runtime rather than build time that's a problem. When a developer has to refer back to the implementation of a method to figure out which types to pass it, that's a waste of time. TypeScript and Python type hints solve this problem a bit, but they lack the ability to validate passed types at runtime without extra code. 

*Type guards* are my least favorite TypeScript feature. They're essentially duck typing that you have to implement yourself and **trust** that they're implemented correctly. In my opinion, this is the worst of both worlds. Consider the following:
```typescript
interface Dog {
    bark: () => void;
}

/* The developer has to manually implement
a heuristic check for interface adherence!
When they update the interface, they have
to update the type guards too! */
function isDog(pet: object): pet is Dog {
  return (pet as Dog).bark !== undefined;
}
const dog: any = {bark: () => console.log('woof')};

if (isDog(dog)) {
    // TS now knows that objects within this if statement are always type Dog
    // This is because the type guard isDog narrowed down the type to Dog
    dog.bark();
}
```

There's something about declaring a type AND having to write validation logic for said type that really rubs me the wrong way. The code above **[smells](https://en.wikipedia.org/wiki/Code_smell) like someone using the wrong tool**.

Unlike TypeScript definitions, Java's nominal type systems takes load off the programmer's brain by crystallizing type definitions and guaranteeing type guards by default.

### Removal of responsibility for optimization 
Java developers can confidently trust the JVM to do what's best. Whether they're implementing a multithreaded application or storing a large amount of data on the heap, they can be confident they won't shoot themselves in the foot with memory management or data races. This advantage is primarily in comparison to C++, which contains a multitude of footguns.

This is part of Java's ergonomic experience. When a developer has to worry less about technicalities they can focus more on the problem at hand.

### A holy grail of productivity
How many languages can you think of that meet the following conditions?
1. Quality package manager and build system (I 💚 Maven)
2. Nominal typing
3. Large community
4. Hands-off optimization

I think the only qualifying tool is Java, but let me know if there are others!
***edit:*** As [Jwosty pointed out](https://www.reddit.com/r/programming/comments/mrrx9l/java_is_criminally_underhyped/guo7qrl/?utm_source=reddit&utm_medium=web2x&context=3) Microsoft's Java competitor C# has all these characteristics and more/newer language features. I have never used C# outside of the Unity game engine, but I'm going to look into it.



## Surprising absence from university curriculum
I currently attend the University of Colorado Boulder; it's a great school but we're certainly not known for CS. However, the majority of our upper-division computer science curriculum is shamelessly stolen from CMU or Stanford, assignments and all. During my time at CU, I've used the following programming languages:
1. C++. This language was chosen for all the required core courses: *Computer Systems*, *Operating Systems*, *Data Structures*, etc. This language is a reasonable choice as it enables direct memory management, creation of kernel modules, and presents many challenges and learning opportunities.
2. Python and Julia. As you might expect, these languages were the darlings of numerical computation and discrete math professors.
3. Scala. This language was used in *Principles of Programming Languages* instruction, primarily for its functional programming and pattern matching features. While Scala uses the JVM and interops with Java, it has a very different developer experience than Java.
4. Web languages (HTML/CSS/JS). These were only used in a single course called *Software Development Methods and Tools* which focused on industry trends. 

I am graduating this semester and **Java has not made a single appearance**; I think this is a shame.


## Conclusion

There is no One True Way to build applications, but I think that Java doesn't get enough attention particularly among startups and the newbie programming community. Untyped languages are useful tools, but I don't think they should be the default choice for building large applications. If you're a full-stack dev and have never extensively used Java, I think you'll be pleasantly surprised should you try it for your next project.

Java and the JVM were hyped to the moon in the 90's and early 2000's, but I don't think it should have ever died out! The developer experience I've found with IntelliJ and Java is worth getting excited about.

I am curious why Java lost its hype in the first place. Programmer culture history is poorly documented and if you have insight, please <a href="mailto:jacksonroberts25@gmail.com">email me</a> or leave a comment.

<h2 id="comments">Comments (821)</h2>

View comments on [reddit](https://www.reddit.com/r/programming/comments/mrrx9l/java_is_criminally_underhyped/) / [Hacker News](https://news.ycombinator.com/item?id=26827766) or [follow me on Twitter](https://twitter.com/jacksondotsh)