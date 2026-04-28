---
layout: post
title: Markdown Viewer
date:   2026-04-27
categories: software
tags: html app
excerpt_separator: <!--more-->
---

## Problem

When working with Markdown files, I encounter the following challenges:

1. rendering it in a similar or better style than Github's Markdown style
2. printing it with a normal font size and spacings (can be PDF or paper)

## Solution

I vibe coded with Claude AI a self-contained, local HTML file that will render a Markdown file with Github styling and better print formatting. I extended the viewer to render code files with syntax highlighting and print in color.

<!--more-->

## Features

- GitHub-style rendering
- Sidebar table of contents — auto-generated from headings, indented by level (h1/h2/h3)
- Collapsible header sections
- Light / dark mode themes
- Code file support — view source code files with syntax highlighting applied
- Print-ready — optimized font sizes and margins, color highlighted code

Download `markdown-viewer.html` from [Markdown Viewer Repo][markdown-viewer], open it in any browser, drop in a `.md` or code file.

## Screenshots

![Markdown in light mode and code in dark mode][markdownCodeScreenshot]

[markdown-viewer]:(https://github.com/hmistry/markdown-viewer)
[markdownCodeScreenshot]: <{{ site.baseurl }}/assets/images/md-viewer-sm.png>