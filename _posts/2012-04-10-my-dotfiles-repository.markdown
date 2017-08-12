---
layout: post
title: My Dotfiles Repository
date: '2012-04-10 04:25:00'
tags:
- bash
---

<p>Added my first attempt at a &ldquo;dotfiles&rdquo; directory today. The results are available from my Github account <a href="https://github.com/mspaulding06/dotfiles">here</a>. I got this awesome idea from a fellow Githubber and thought it was the best thing since aliases. You can check out his work <a href="https://github.com/plu/dotfiles">here</a>. Having a single git repository that travels with me wherever I go will probably increase my productivity. I can&rsquo;t tell you how many times I&rsquo;ve been on a new Linux box and wanted my aliases, vim config, etc., only to &ldquo;scp&rdquo; the stuff from another box or try to rewrite my configs the best I could from memory. Usually this is just me trying to remember which vim command tells my editor to use spaces instead of tabs, the end result being that every system I use has some mishmash of half-baked scripts with no consistency.</p>
<p>So, now I can pull my Git repository, run the &ldquo;install.sh&rdquo; script, and be on my way. The install script loops over the files in the &ldquo;dotfiles&rdquo; repository and symlinks them as dot files into the current user&rsquo;s home directory. Since there&rsquo;s no copying of the files, its easier to edit them as needed and push the changes back to Github. Nice.</p>