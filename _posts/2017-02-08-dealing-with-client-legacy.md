---
title: Embrace legacy with constant change
description: The rules after which we decide when to refactor.
tags: legacy, refactoring, how-to
layout: article
---

# Introduction

- I'm Philipp, Product Manager Workflow and part-time JavaScript developer in the Wolf-Pack
- I want my product to be top-notch with an up-to-date stack so that:
    - my developers are happy
    - we can deliver the best quality possible to our customers
- We faced problems when facing legacy code
    - Refactorings could stall feature development
    - This is a no-go
    - We needed to find a solution.

# What is legacy

- Every code base that lives long enough is going to have some portion of legacy code
(Meme: Live long enough to see yourself become the villain)
    - Dead code
    - "Finished" code: features that change so little that they are not being refactored in a long time
    - Too complex code that is very hard to change and therefore isn't changed
    - Good, refactored code that needs to change because you
        - Changed your coding style
        - Changed something in the technology stack

## What were the hardest parts for us?

- We encountered a couple of these changes
    - migration from all-globals `Signavio.components.Panel` to require `var Panel = require('signavio/components/panel')`
    - migration from [require](http://requirejs.org/) to [es6 modules](http://www.2ality.com/2014/09/es6-modules-final.html) `import { Panel } from 'signavio/components'`
    - migration from [Backbone](http://backbonejs.org/) views to [React](https://facebook.github.io/react/)
        - some more work besides the regular rewrite was necessary (optional: don't know yet)
            - [backbone-rel](https://github.com/effektif/backbone-rel) -> @jfschwarz
            - re-render when changes through backbone relations were propagated
    - migration from Backbone models to Redux
- Method shown here somewhat applicable to all of these problem, but we're going to focus on Backbone views to React

# Think of your application as a tree

- Add tree image with components and how they are composed of other components
- The pain lies in the people who start to refactor too high in the tree ("It's just a small change...")
- Try to start at the bottom of the tree and work your way up

## Come up with a simple rule of thumb

> Does any of the components, the current component uses still rely on Backbone or our `BaseMixin`?

If yes, leave the component as-is. If no, refactor the component.

- Every engineer must be able to answer the question "Should I refactor that part of the application, or should I leave it as is?"
- If you have too many excuses, you're probabyl doing it wrong
- Tackle problems when they arise, not before
    - up-front "what-ifs" can kill this approach without there being any real problems

## Pros

- If you add something at the bottom of the tree you can already use the new stuff
- Once some leaves have been refactored, refactoring the bigger node above is easy and just feels nice
- You can estimate the overall time needed, once you've finished some leaves.
    - Since you only refactor in small portions, your refactorings tend to be fast
- With each component you also only need to refactor the tests for that compoennt
- Feature development can still take place while refactoring

## Cons

- It can take a significant amount of time until the refactoring is done
    - You need too make progress visible and tell new employees about it
    - New refactorings can start while you are still working to get the first one going
        - Still I would say this is no problem
- You need to restrain yourself from starting "that big refactoring your always wanted to do"

# Conclusion

- Helped us migrate Backbone views to React
- Is currently helping us, to move from Backbone models to Redux
- Is also currently helping to move from a big code base to a mono-repo with a lot of smaller self-contained packages
- Since it worked so good, we try to come up with simple rules for every change we're making
    - If we cannot come up with a simple rule, we question the initial task
- Helps the developers to work
- Helps the product manager to push progress
- Might not help in every use-case, but helped us

# Related Work

- [Ryan Florence - Don't Rewrite, React!](https://www.youtube.com/watch?v=BF58ZJ1ZQxY)
