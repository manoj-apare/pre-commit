# Git pre-commit hook script

## Overview

Git pre-commit hook script is been built specifically for Drupal 7 git
repository.

## Installation

For using this hook script just copy/download the script file to your git
repository hooks folder that is in `.git/hooks/`. Don't forget to give
executable permission for the script file.

## Working

Whenever a git commit is been performed, pre-commit hook script will be executed
by git inbuilt hook support. The above script does syntax check as well as looks
for any debugging function in the changes staged to commit (which is a bad
practice). In both case, the commit operation will be aborted and can be
committed after doing corrections or removing the debugging function from
the code.

In case you want to force commit with any debugging functions (not recommended)
you can do it by using the git command option `-n OR --no-verify`.

### Syntax check

Syntax check is been performed using `php -l` command for each php files (file
with extensions 'php, inc and module').

### Check for debugging functions

Checking for debugging function is been performed using `git diff` command only
to files staged to commit and in turn only to added/modified changes.

List of debugging functions included for checking are: dpm, dpq, dvm,
ddebug_backtrace, print_r, var_dump, debug_backtrace, console.log. First 4
functions are of Drupal 7 devel module, later 3 are of php and last one is of
javascript.

For better performance of the script, we have excluded the checking of files
with following extensions: 'info, png, gif, jpg, ico, patch, htaccess, sh, ttf,
woff, eot, svg'. Also excluded files residing in the following directories:
'features, contrib, devel, libraries'.
