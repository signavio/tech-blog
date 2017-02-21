# Signavio Tech Blog

This README provides a selection of tips, guidelines and instructions for bloggers.

## Planning a blog post

### What to write about

The best blog post to write is something that only you could write, however obscure the topic.
In practice, this is likely to be one of:

* A technical problem you solved, like a StackOverflow question and answer
* A tutorial for something you learned how to do - a ‘How to’
* What you wish everyone did differently - a lesson learned
* Review/comparison of technologies that you have chosen between
* Fun tech ideas you talked about over beers
* A demo of something cool, however pointless

Everyone has something to write about!
See [How To Brainstorm Talk Ideas](http://missgeeky.com/2016/11/21/how-to-brainstorm-talk-ideas/), which mostly applies to blogging as well as presenting.

### How to develop content ideas

All writers develop their ideas differently, but here’s a standard process that you can use.

1. Keep a list of every idea that makes you think _that would make a good blog post_
2. Test each idea at the coffee machine or over a beer by starting a conversation about it
3. Use other people’s questions about the idea to learn what would be interesting to write about
4. Practice arguing for different points of view to discover what you want to convince people about
5. Make notes for any points you make that sound particularly intelligent or funny
6. Try writing up your notes to see what you can write a paragraph about
7. When you have a number of paragraphs, trying organising them into a few sections, with headings
8. If you don’t know what to do next, ask someone for input on what you have so far

## Writing a blog post

### Content guidelines

These content guidelines should give writers and reviewers confidence that we can publish a blog post.

* Be nice
* Keep blog posts short: 400-500 words is plenty; if you have more to say, save it for another post
* Write for a developer audience: current and future colleagues, and your future self
* Celebrate work you did, while giving credit where credit is due
* Don’t write ‘this is the first post in a series’ unless the whole series is already _published_
* Don’t write anything that would violate our code of conduct
* Don’t make Signavio look bad
* Spellcheck your text
* No stock photos

### Blog post outline

You may find it easier to write certain kinds of blog post if you follow a standard outline.
In general:

* Start by explicitly stating the goal of the blog post, e.g. to _introduce_ or _teach_ some topic, so the reader knows what to expect
* Describe concrete problems and concrete solutions before describing general techniques, to make it easier to follow
* Finish with a conclusion about what you’ve written, or suggest next steps, so the reader feels like they’ve finished

A typical blog post might use some or all of the following outline.

1. Introduction - one or two sentences on each of:
   * Purpose - the blog post’s goal
   * Scope - what you’ll cover and who it’s for
   * Summary - what you’ll be able to do by the end
2. Specific problem
   * Describe a specific problem, ideally one you’ve actually faced
   * Why did you want to solve this?
3. Specific solution
   * Solve one version of the problem, ignoring variations
   * Break the solution into steps
4. General solution
   * Describe the rationale for your solution, e.g. _In general, …_
   * How broadly could you use this approach?
   * How might you approach variations on this problem?
   * How could you improve on this solution?
   * What other approaches did you consider?
5. Conclusion
   * What did you learn?
   * Did you like doing it this way?
   * Would you try it a different way next time?
   * What are the next steps?

Using an outline can help you avoid getting stuck.
First, pick a section and write one sentence.
If you have more to write, keep going until you run out.
When you don’t know what to write, pick a different section.

## Publishing a blog post

### Review guidelines

Blog posts require review before publication,

### Content schedule

To start with, we’ll publish posts no more frequently than once per week.
A reasonable expectation is that every developer in the company will publish one blog post in 2017.
We should only increase this if we get a backlog of more than four or five unpublished posts.

### How to publish a blog post

1. Pick a URL-friendly ‘short title’ (a.k.a. a [slug](https://john.do/post-slugs/)) to identify the post; you can change this later
1. Pick a planned publication date at least one week after the previous post’s date
1. Create a new branch in the repo, named `post/short-title`
1. Add a file `_posts/yyyy-mm-dd-short-title.md` with the planned publication date
1. Add a [YAML front matter](https://jekyllrb.com/docs/frontmatter/) block, with a `title`, `description` and `tags`. Also, set the `author` variable to your full name. Wrap the `title` and `description` values in quotes.
1. Write the blog post in Markdown
1. Build the site by [running Jekyll locally](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)
1. Commit, push to your branch, and create a pull request
1. Merge to master once you’ve had at least one review and resolved any comments
1. Contribute an improvement to these instructions, to make it easier for the next person to publish their first post

### Adding images and files to posts

Post URL paths start with a year, e.g. `/2017/`, so put image files in the repo’s `/2017` folder.
To link to an image, use a relative URL starting with `../2017/` so that image paths work in both the GitHub file view as well as the rendered site.
See existing blog posts for examples.
