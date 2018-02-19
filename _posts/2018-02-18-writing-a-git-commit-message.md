---
layout: post
title: Writing a Git Commit Message
date:   2018-02-18
categories: software
tags: git
excerpt_separator: <!--more-->
---

This is a summary of Chris Beams' post on ["How to Write a Git Commit Message"](https://chris.beams.io/posts/git-commit/) for the purpose of being my quick reference. (As Chris mentions: "This has all been said before" - see his references.)

<br>
### Guidelines
1. [Separate subject from body with a blank line](#separate)
2. [Limit the subject line to 50 characters, with hard limit at 72](#limit)
3. [Capitalize the subject line](#capitalize)
4. [Do not end the subject line with a period](#period)
5. [Use the imperative mood in the subject line](#imperative)
6. [Wrap the body at 72 characters](#wrap)
7. [Use the body to explain _what and why_ vs. _how_](#body)
<!--more-->

<br>
#### 1. Separate subject from body with a blank line
{: #separate}
- Blank line separation is needed for some Git commands to work better in short form
- Use a text editor instead of the CLI -m option. To setup an editor for use with Git CLI, see [this section of Pro Git](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration).

<br>
#### 2. Limit the subject line to 50 characters, with hard limit at 72
{: #limit}
- GitHub's UI truncates any subject line longer than 72 characters with an ellipsis
- Git log one-liner or shortlog works better with approx 50 chars

<br>
#### 3. Capitalize the subject line
{: #capitalize}

<br>
#### 4. Do not end the subject line with a period
{: #period}

<br>
#### 5. Use the imperative mood in the subject line
{: #imperative}
- Imperative mood means “spoken or written as if giving a command or instruction” e.g. clean your room, or close the door
- A Git commit subject line should be able to complete the following sentence:
  - If applied, this commit will **_your subject line here_**
  - If applied, this commit will _refactor subsystem X for readability_

<br>
#### 6. Wrap the body at 72 characters
{: #wrap}

<br>
#### 7. Use the body to explain _what and why_ vs. _how_
{: #body}
- Focus on clarifying the reasons why you made the change in the first place — _the way things worked before the change and what was wrong with that_, _the way they work now_, and _why you decided to solve it the way you did_.

<br>
#### Commit template

{% highlight shell %}
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here is the place to explain them.

Further paragraphs come after blank lines.

 - Bullet points are okay, too

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789
{% endhighlight %}

