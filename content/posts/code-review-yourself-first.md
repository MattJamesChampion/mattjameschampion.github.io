+++
title = 'Code Review Yourself First'
date = '2025-07-16T08:00:00+01:00'
draft = false
aliases = ['/2025/07/16/code-review-yourself-first']
+++

One piece of advice that I like to give to any developer looking to improve is this: you should be reviewing your own
code before you submit it anywhere, whether you’re committing to a branch or opening the final pull request. I’ve found
few pieces of advice that can so quickly help improve someone’s code quality and, crucially, their *understanding* of
what they’re writing.

Now, your first thought might be "but I don’t know *how* to code review" (hopefully your organisation includes everyone
in code reviews, not just seniors!) or "I’ve just written this code, what is there to review?" which are fair questions,
but ultimately they’re both quite simple to answer.

In regards to knowing *how* to review code, it’s a skill that you get better at over time (especially with practice
*hint hint*) but ultimately code reviews can be summarised as "look over the code and keep an eye out for issues or
improvements". For example, here are some questions you could ask yourself to start out:

- Does the code actually solve the problem/add the new functionality etc., or at least move me in the correct direction?
- Is it clear what the code is actually doing? (Sometimes the few minutes between writing something and reading it again
  later can make badly named variables/functions or unclear logic much easier to spot!)
- Is there any unnecessary code that needs removing, or any sections I was meant to implement that I didn’t (i.e. have I
  left something like `//TODO: Implement` anywhere in my code)?
- Have I formatted the code correctly? Is it consistent with other similar code? Are there any spelling issues?

As to the second point for the folks wondering what there is to review if they’ve only just written the code, the whole
joke of "which idiot wrote this code? oh wait it was me" outlines the idea that the act of *writing* something and
*reading* it are fundamentally different. Hell, I finished the first draft of this post, went straight back up to the
first paragraph and started spotting things that needed changing! Unless it’s only a handful of lines in a single file,
you’ll likely benefit from looping back to see your earlier changes with fresh eyes, even if only minutes later.

Now to be absolutely clear about all of this, "code reviewing yourself first" doesn’t mean waiting until you’re about to
merge into `develop` with dozens of commits; it should be a continuous/ongoing process. I tend to (or at least *try* to)
keep commits quite small, so they’re a pretty logical place where I can stop and have a quick spin through the files
I’ve made changes to before firing them off. You can also have a look at the entirety of an overall change (if made up
of lots of commits) towards the end, but at least you’ll know that you’ve already had a more ground level check!

The whole process is usually very quick; depending on the number of changes, it could be 20-30 seconds for a couple of
files, or a few minutes if I’ve had to make a lot of changes before getting to a logical point to commit. If you *do*
find issues that need resolving it can obviously end up taking longer, but that’s fine; the code review has done its
job!

I cannot tell you how many times I’ve caught issues with my own code before sending it down the wire. Simple (or
complex!) bugs that I’d introduced, spelling mistakes, leftover test code, you name it! Sometimes taking a look at a
change more holistically makes me question my approach or realise that there’s a much easier way to tackle things which
only aids the overall code quality.

The process of writing code is rarely a straight line from beginning to end, instead taking a bit of a winding path down
unexpected routes where you try out different things until you find what works. While you get to the right destination
in the end, sometimes the code we leave behind shows signs of these changes in approach and direction. As an example, we
might have added an extra property to a class, only to realise that the value in question needed to be stored in a
separate object. If you forgot to undo that change there and then you might end up leaving it behind as a strange
vestigial appendage that other developers down the line may not feel comfortable removing, especially if you also wired
up other parts of the process!

The whole "code review yourself first" ethos is intended to act as a process to clean up after yourself before you
submit to the next step, which may be a more formal code review by someone else, a round of testing or maybe even an
automatic deployment straight to production (hey, I’m not here to judge (what am I saying, of course I am, don’t do
that)).

And remember, if your code is going to be reviewed by someone else it’s still not wasted effort! The earlier in the
process a change is caught and resolved, the cheaper it is (if Code Complete ever taught me anything that is!). It also
saves someone else having to point out simple fixes.

This process shouldn’t be burdensome or an unnecessary polishing step, it’s the act of spending a tiny percentage of the
overall effort that you’ve *already put in* to double check that everything looks as you intended. Look at it like a
quick proof-read of an essay you’ve written, or checking the screws are tight after building something; you’ve already
put in the work, make sure that it really shines!
