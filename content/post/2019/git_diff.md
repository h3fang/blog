---
title: git diff 101
date: 2019-06-20
tags: ["git"]
toc: false
---

To use git diff more efficiently, we actually have to understand different stages of git files.

<!--more-->

{{% center %}}
![](/image/stages_of_git_files.png)
{{% /center %}}

Then we can use git diff to show changes between different stages. Here, **HEAD** corresponds to **Committed** tip, **Index** corresponds to **Staged**, **Modified** corresponds to **Working tree**.

{{% figure class="center" src="/image/git_diff.png" title="center" alt="img" %}}

### References

* <https://backlog.com/git-tutorial/git-workflow/>
* <https://stackoverflow.com/questions/1587846/how-do-i-show-the-changes-which-have-been-staged>
