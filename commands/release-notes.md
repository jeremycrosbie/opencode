---
description: Generate release notes meant for user consumption
---

$ARGUMENTS contains the starting commit id or tag and optionally the ending tag or commit Id.  If only one argument, assume HEAD is the ending commit.  If no arguments, do not proceed


## Process

1. Run `git diff` with the starting and ending commit
2. Analyze all of the commit messages, looking for information important to end users.
3. Ignore any technical issues not useful to normal end users.
3. The first section highlights new features
4. The next feature highlights bug fixes.
5. The last section highlights miscellaneous fixes that could be important to a user
6. The output must be in Markdown

