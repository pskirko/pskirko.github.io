---
layout: post
title:  "Hello Dev Log!"
date:   2023-11-14 18:00:00 -0800
categories: javascript
---
I’ve been meaning to start a dev log for [a while](https://www.merriam-webster.com/grammar/awhile-usage).
Well, no day like today to finally do that. My regular website, [www.pskirko.com](https://www.pskirko.com),
is for polished writing, but I only irregularly finish polished pieces. This dev log is more a raw
feed.

It’s been a few years since I switched from being a software engineer to being a software
engineering manager. I don’t really have time to code in my current job; being a manager means doing
a lot of things, other than coding, to ensure that my team accomplishes great things.

However, code is special. You can create something from almost nothing; that’s very cool. So I do
believe, even as a manager, it’s important to stay in-the-loop (pun intended) with code. It’s just a
fundamental association.

My current hypothesis (subject to change) is that, since I don’t code in my job, I really only have
space in my brain for one programming language right now. So the most appropriate language to focus
on is Javascript, because it reasonably supports both client and server programming. But, if I had
more time, I’d be doing some Python too, so maybe I’ll sneak that in here and there.

My goal is to focus on fluency: being able to express ideas in code without looking everything up.
Right now, being rusty, I need to look up all sorts of things. So the goal is to get away from that.

The other goal is immersion. Coding shouldn’t be something I think about or do only here or there.
It should be daily. Code shouldn’t be associated with just work. It should be a creative act, like
writing a story or drawing a picture. I don’t mean that for everyone, but for me, that’s my take.

I’m in no rush; going to take it nice and easy and have fun along the way.

To get started, I picked a simple toy problem: guessing game. Here is my code:

{% highlight javascript %}
const assert = require('node:assert/strict');
const readline = require('node:readline/promises');
const { stdin: input, stdout: output, exit } = require('node:process');

const rl = readline.createInterface({ input, output });

async function playGame() {
 let theNum = Math.floor(Math.random() * 21) + 1
 assert.ok(theNum >= 1 && theNum <= 21, 'Oops, I made a maths error.');

 let correctGuess = false;
 let prompt = 'I have picked a number between 1 and 21. Please try to guess it.\n';

 while (!correctGuess) {
   const guess = await rl.question(prompt);
   const guessNum = parseInt(guess, 10);

   if (isNaN(guessNum)) {
     prompt = 'Sorry, that was not a number. Please try again.\n';
   } else if (guessNum < 1 || guessNum > 21) {
     prompt = 'Please guess a number between 1 and 21.\n';
   } else if (guessNum < theNum) {
     prompt = 'Too low. Please try again.\n';
   } else if (guessNum > theNum) {
     prompt = 'Too high. Please try again.\n';
   } else {
     console.log(`Correct! The number I picked is ${theNum}. Great guessing.\n`);
     correctGuess = true;
   }
 }

 exit(0);
}

(async () => await playGame())();
{% endhighlight javascript %}

I’m not a fan of callback-based Javascript, so I’ll be using async/await and promises wherever I
can. Having not used Node in a while, I had to look up the various APIs, though.

Note the use of asserts. Back when I wrote C++ code at Adobe, I really relied on asserts to express
invariants and avoid footguns. I’m not entirely sure why they aren’t more popular nowadays.

Anyways, that’s it for today.