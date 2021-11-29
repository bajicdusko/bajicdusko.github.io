---
layout: "post"
title: "How to setup git author details for different clients"
date: "2020-07-07 00:00"
categories: [tips]
tags: [git]
description: "Setting different author info for different clients and its repositories requires three steps only. While it might seems like unimportant at first, improper setup can have legal consequences."
redirecturl: "https://www.crispy-engineering.com/setup-git-author-details-for-different-clients/"
---

You've definitelly been in a situation where you'd like to commit changes to repository, but to use your
personal email instead of company email for commit author details.  
Even more important, if you are working with different clients, with a couple of repositories each, you really want to make sure that your commits use proper author details for respective repo.  
From first hand experience, improper author info in client code base can have legal consequences. Since then, I'm extra careful with it. This is how to set it up easily.

On repository level, it's easy to set it up in `--local` config. However, this is repetitive and you'll forget it at some point.
What we need is more general and once per client set, approach, without tackling each repository individually.

## 1. Organize repositories on disk

Whether you're setting git configs or not, having directory structure like this helps with other stuff as well:

```
- repositories
  - clientA
    + longterm_repo
    + new_proj
  - clientB
    + boring_project
    + mainenance_repo
```

## 2. Edit git global config

Open `.gitconfig` file in edit mode and add following blocks. Location of this file depends of your git installation. On MacOS, it's in the `/Users/youruser/.gitconfig`.

```
[user]
        name = Dusko Bajic
        email = dusko@mail.com

[includeIf "gitdir:~/repositories/clientA/"]
        path = .gitconfig-clientA

[includeIf "gitdir:~/repositories/clientB/"]
        path = .gitconfig-clientB
```

With `name` and `email`, we've defined default author setup for all repositories.  
`[includeIf]` blocks are the key and they match directory with repositories on the disk, with partial config file.
As we haven't created these partial config files yet, let's do it in the next step.

## 3. git-config per client

Let's create client related git config files, in the same directory where `.gitconfig` resides, with the following content.

`.gitconfig-clientA`
```
[user]
        name = Dusko Bajic
        email = dusko@clientA.net
```

`.gitconfig-clientB`
```
[user]
        name = Dusko Bajic
        email = dusko@clientB.org
```

And that's it. On the file system you'll have these three files one besides other:

```
- users
  - youruser
    .gitconfig
    .gitconfig-companyA
    .gitcontig-companyB
```

Now, each repository will have correct author info, per client. You can check it out by navigating to `/repositories/clientA/longtermRepo` and executing `git config user.email`. With the setup example from above, it will print `dusko@clientA.net` in the console.