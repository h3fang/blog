---
layout: post
title: "(Arch) Linux Font Configuration"
date: 2020-05-18
tags: ["linux", "fontconfig"]
draft: true
---

## Introduction

There is a saying that, the more you know, the more you don't know.

It's true. But in general, we don't know that there are a lot of things we do not know. For instance, I think I am pretty good at Linux. Every day I learn a little bit about Linux. Now I have wrote 35 pages of losses and gains about Linux. There are few Linux desktop users, probably about 1% of people are using Linux among all the desktop users. And I think I must be the top 1% of the 1%.

But there is one thing that I know I don't know about Linux, one thing I have tried several times to understand it and still fails, and that thing is **Font Configuration**.

<!--more-->

## The Problem

I actually know little about fonts. I only know there are three types of fonts.

- **Sans-serif**. Font characters don't have curly strokes.
- **Serif**. Font characters have curly strokes.
- **Monospace**. Font characters have the same width. So it's perfect for displaying source code.

Actually there is another type of fonts we usually use - Icon Fonts. You can also call them **emoji**s. But it doesn't have to be emojis, like [Font Awesome](https://fontawesome.com/). It doesn't matter what you call them, you know what I mean.

If you use Arch Linux and you use default (empty) **fontconfig** configuration. Your fonts might look fine. But if you open [getemoji.com](https://getemoji.com/) in your browser, you probably get something like this. And if you open this [GitHub page](https://github.com/h3fang/dotfiles/blob/master/.config/i3blocks/config), you probably see some blocks like this.

## The Battle

### Prepend or Append

### Dejavu/Liberation Fonts

### Android Fonts

### Everything is Sans-serif

## The Lessons We Learned

After all of these, I just want to say.

Don't touch it. Just use the empty configuration. It doesn't worth it. 😂

But I learned a lot from it. 😁

## References
- <https://eev.ee/blog/2015/05/20/i-stared-into-the-fontconfig-and-the-fontconfig-stared-back-at-me/>
