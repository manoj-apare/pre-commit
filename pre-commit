#!/bin/bash

# Redirect output to stderr.
exec 1>&2

# Color codes
red=`tput setaf 1`
green=`tput setaf 2`
blue=`tput setaf 4`
reset=`tput sgr0`

# Notification
echo  "${blue}"
echo "# Running pre-commit git hook."
echo "==============================${reset}"

# Debugging functions to be checked.
keywords=(dpm dpq dvm ddebug_backtrace print_r var_dump debug_backtrace console\.log)

# Join keywords array into a single string for grep and remove the separator from the start of the string.
keywords_for_grep=$(printf "|%s" "${keywords[@]}")
keywords_for_grep=${keywords_for_grep:1}

# Directories and files to be excluded during search.
exclude_dir_and_ext='\/features\/|\/contrib\/|\/devel\/|\/libraries\/|\/vendor\/|\.info$|\.png$|\.gif$|\.jpg$|\.ico$|\.patch$|\.htaccess$|\.sh$|\.ttf$|\.woff$|\.eot$|\.svg$'

# Flag counter
synatx_error_found=0
debugging_function_found=0
merge_conflict=0
coding_standard_error=0

# Check for PHP syntax
# List php files added to commit
php_files_changed=`git diff-index --diff-filter=ACMRT --cached --name-only HEAD -- | egrep '\.php$|\.inc$|\.module|\.theme$'`
if [ -n "$php_files_changed" ]
then
  for FILE in $php_files_changed; do
  php -l $FILE > /dev/null 2>&1
  compiler_result=$?
  if [ $compiler_result -eq 255 ]
  then
    if [ $synatx_error_found -eq 0 ]
    then
      echo "${red}"
      echo "# Compilation error(s):"
      echo "-----------------------${reset}"
    fi
    synatx_error_found=1
    `php -l $FILE > /dev/null`
  fi
  done
fi

# Check for debugging functions
# List all files added to commit excluding the exceptions
files_changed=`git diff-index --diff-filter=ACMRT --cached --name-only HEAD -- | egrep -v $exclude_dir_and_ext`
if [ -n "$files_changed" ]
then
  for FILE in $files_changed ; do
    for keyword in "${keywords[@]}" ; do
      # Find debugging function exists in file diff one by one.
      pattern="^\+(.*)?$keyword(.*)?"
      result_for_file=`git diff --cached $FILE | egrep -x "$pattern"`
      if [ ! -z "$result_for_file" ]
      then
        if [ $debugging_function_found -eq 0 ]
        then
          echo "${red}"
          echo "# Debugging function(s):"
          echo "------------------------${reset}"
        fi
        debugging_function_found=1
        echo "Debugging function" $keyword
        git grep -n $keyword $FILE | awk '{split($0,a,":");
          printf "\tfound in " a[1] " on line " a[2] "\n";
        }'
      fi
    done
  done
fi

# Check for Drupal coding standards
# List php files added to commit
php_files_changed=`git diff-index --diff-filter=ACMRT --cached --name-only HEAD -- | egrep -v $exclude_dir_and_ext | egrep '\.php$|\.module$|\.inc$|\.install$|\.test$|\.profile$|\.theme$|\.js$|\.css$|\.info$|\.txt$'`
if [ -n "$php_files_changed" ]
then
  phpcs_result=`phpcs --standard=Drupal --report=csv $php_files_changed`
  if [ "$phpcs_result" != "File,Line,Column,Type,Message,Source,Severity,Fixable" ]
  then
    echo "${red}"
    echo "# Coding standard issue(s):"
    echo "---------------------------${reset}"
    phpcs --standard=Drupal $php_files_changed
    coding_standard_error=1
  fi
fi

# Check for merge conflict markers
# List all files added to commit
files_changed=`git diff-index --diff-filter=ACMRT --cached --name-only HEAD --`
if [ -n "$files_changed" ]
then
  for FILE in $files_changed; do
    # Find debugging function exists in file diff one by one.
    pattern="(<<<<|====|>>>>)+.*(\n)?"
    result_for_file=`egrep -in "$pattern" $FILE`
    if [ ! -z "$result_for_file" ]
    then
      if [ $merge_conflict -eq 0 ]
      then
        echo "${red}"
        echo "# Merge confict marker(s) found in:"
        echo "-----------------------------------${reset}"
      fi
      merge_conflict=1
      echo $FILE
    fi
  done
fi

# Decision maker
errors_found=$((synatx_error_found+debugging_function_found+merge_conflict+coding_standard_error))
if [ $errors_found -eq 0 ]
then
  echo "${green}"
  echo "Clean code :), your code now been committed!"
  echo "${reset}"
else
  echo "${red}"
  echo "Clean listed file(s). Aborting git commit."
  echo "${reset}"
  exit 1
fi
