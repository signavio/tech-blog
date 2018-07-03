---
title: "Integrating Flow type into a large JavaScript project"
description: "Lessons learned on our journey towards type safety"
author: Philipp Giese
tags: React, Flow, Mono-Repo
layout: article
image:
  feature: /2018/river.jpg
  alt: A river has a complex flow
---

For over two years our team has followed the development of the flow ecosystem very closely.
A couple of weeks ago we made the decision to go all-in.
This is a summary of the lessons we've learned along the way.

## Some context

My team is working on a product called "Signavio Workflow Accelerator".
The whole application is written with React and mostly uses Redux for state handling (we also have some backbone models left over).

In the beginning we added a lot of PropType checks to our UI components.
However, it was clear quite fast that this wouldn't scale as the type checks during runtime were slowing down the application way too much.
Luckily for us this realization came at around the same time when flow was initially introduced.

There was only one catch.
The actual type check didn't work for our large code base for various reasons.
We decided to give flow a go nonetheless hoping that the ecosystem would evolve eventually.

**TL;DR** we started adding types all over the place without ever checking them.

## Pulling the trigger

A couple of weeks ago I was once again looking at an error that could have been prevented by having proper type checks.
It was that moment, when I realized that we should finally start enforcing the code to type check on each PR.
But since we never did this before, but had added the `// @flow` pragma to over 800 files, we were left with a couple thousand violations.
Hurray!
Since we've learned that having optional and failing checks on each PR won't get us anywhere we had to come up with a different strategy.
This strategy should meet the following conditions:

- The PR check has to be mandatory
- The burden for individual developers must be low
- The types that have already been defined should not be lost

## How we pulled it off

The most drastic decision was the first one.
In for the type check and therefore the PR check to succeed we removed the `// @flow` pragma from **all files**.
We realized that we cannot fix all errors in all files in one PR.
And since we didn't enforce type checking before we would also not loose any confidence in our code by turning it off initially.

After that we had a new mandatory PR check that would now be applied to all new PRs.
At this point we added a new rule to our development Process "In every file you touch, add the `// @flow` pragma again".
With this kind of "fix what you touch" rule we already fixed over 16.000 lint errors over the course of 2 years so we knew that it worked (even though its a long term road to success).
After a day we realized that something was not quite right.

### Flow isn't so good a reporting uncertainty

Everything seemed fine until we realized that flow wasn't reporting obvious errors.
It turns out that if you import a type from a file that doesn't have the `// @flow` pragama flow simply ignores it completely.
A warning would have been nice.
The fix for this is rather easy.
We shouldn't have removed **all** the `// @flow` pragmas.
Here are the two types of files that should have that pragma form the beginning.

- All the files containing type definitions (duh!)
- All `index.js` files

The second point is obvious once you encounter the problem.
When flow encounters any file without `// @flow` at the top it just stops because form there on it cannot make any proper assumptions anymore.
So having all `index.js` files marked as "please proceed flow" is a rather smart thing to do.

The PR that activated all `index.js` already was not so much fun.
We had to fix all false positives from earlier PRs.
Luckily for us we resolved that issue in the first week.

### Enabled packages for flow

The `index.js` fiasco had made us look out for false negatives.
So shortly after that we realized that there is another category where `flow` seemed to be not working correctly.

Our application is managed inside a mono-repo.
This means that we're actually managing a couple of different `npm` packages that make up the whole of the application.
We defined types also within the different packages and then imported them in other packages as we would import components.
And once again `flow` wouldn't complain but also wouldn't quite work correctly.
The problem this time was that (of course) the type annotations would be stripped out by our `babel` transform so that the code could actually work in production.
However, when looking for types in `npm` packages `flow` also uses the entry point you define in your `package.json`.
In other words: `flow` was looking for types in files where no types were defined anymore.

Good news though!
Other people have already solved this problem for us.
There is a tool called [`flow-copy-source`](https://github.com/AgentME/flow-copy-source) which simply copies all your source files and adds a `.flow` type ending.
Flow in turn can pick these files up instead of the transpiled versions.
Voilà, typing now works across packages.

## Conclusion

The whole team is still a big fan of `flow` and after we have cleared away the initial obstacles the errors we get from `flow` are pretty sweet.
It would have been great though, if `flow` would be a bit more expressive when it comes to uncertainty.
For instance if `flow` would actually tell us, when it doesn't know a type instead of just pretending that everything is ok we would have found some issues earlier.

If we would start over again, we would make sure the following conditions are met:

- Have the `// @flow` pragma in all type definition files

This is also great, because this way you know that you don't have any obvious errors in your types.

- Add `// @flow` to all `index.js` files
- Setup `flow-copy-source` if you're dealing with `npm` packages

This is probably true in any case.
When you're developing a library you want to enable your users to have type safety for the components you export.
When you have mono-repo setup you want to make sure that all packages can access their shared type information.

<a style="background-color:#555;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@nathananderson?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" rel="noopener noreferrer" title="Nathan Anderson’s photos"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Nathan Anderson</span></a>
