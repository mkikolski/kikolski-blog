---
layout: post
title: "Short: git kurwa - unleash your anger"
subtitle: "A short info on an unhinged git config"
date: 2026-05-04 19:00:00 +0200
tags: [short, coding, git, funny]
excerpt: "This is for the readers from Poland, Lithuania, Hungary and Brazil as these..."
---

This is for the readers from Poland, Lithuania, Hungary and Brazil as these are the translations available. It's a really old config file which maps most popular git commands and options to some combinations of simple words characteristic for certain commands with common swear words. I've been using it for years, probably started somewhere on my first year of university so I got used to it and only recently my friend standing behind me started laughing when I've been pushing some commits for our project. I really couldn't believe that it was his first encounter with this config file and he was mesmerized.

There's not much more to say - it's a fun way to break your routine and unload your anger from fixing bugs. Overall, this config brings some perverted joy to otherwise mundane tasks and the sole introduction of simple to remember aliases, makes the git operation a bit easier, because instead of typing:

```bash
git log --graph --date=relative \
        --format=format:'%C(auto)%h %C(bold blue)%an%C(auto)%d %C(green)%ad%C(reset)%n%w(80,8,8)%s'
```

you can just type:

```bash
git drzewokurwa
```

which translates to:

```bash
git fuckingtree
```

Check out the config by [jakubnabrdalik](https://github.com/jakubnabrdalik/gitkurwa/blob/master/configNSFW_PL) and unleash the beast when the CI/CD pipeline fails for the third time in a row.
