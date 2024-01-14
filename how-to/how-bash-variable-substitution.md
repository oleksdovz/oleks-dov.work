# Bash Parameter Substitution

Table of contents
- [Bash Parameter Substitution](#bash-parameter-substitution)
	- [Source - https://www.cyberciti.biz](#source---httpswwwcybercitibiz)
	- [String Manipulation and Expanding Variables](#string-manipulation-and-expanding-variables)

## [Source - https://www.cyberciti.biz](https://www.cyberciti.biz/tips/bash-shell-parameter-substitution-2.html)


The `$` character is used for parameter expansion, 
arithmetic expansion and command substitution.

You can use it for manipulating and expanding variables on demands 
without using external commands such as perl, python, sed or awk. 

This guide shows you how to use parameter expansion modifiers 
to transform Bash shell variables for your scripting needs.



## String Manipulation and Expanding Variables

| Variable                        |                                                      Description |
| ------------------------------- | ---------------------------------------------------------------: |
| ${parameter:-defaultValue}      |                                Get default shell variables value |
| ${parameter:=defaultValue}      |                                Set default shell variables value |
| ${parameter:?"Error Message"}   |                 Display an error message if parameter is not set |
| ${#var}                         |                                    Find the length of the string |
| ${var%pattern}                  |                          Remove from shortest rear (end) pattern |
| ${var%%pattern}                 |                           Remove from longest rear (end) pattern |
| ${var:num1:num2}                |                                                        Substring |
| ${var#pattern}                  |                               Remove from shortest front pattern |
| ${var##pattern}                 |                                Remove from longest front pattern |
| ${var/pattern/string}           |                 Find and replace (only replace first occurrence) |
| ${var//pattern/string}          |                                 Find and replace all occurrences |
| ${!prefix*}                     | Expands to the names of variables whose names begin with prefix. |
| ${var,}  or ${var,pattern}      |                            Convert first character to lowercase. |
| ${var,,} or ${var,,pattern}     |                             Convert all characters to lowercase. |
| ${var^} or ${var^pattern}       |                            Convert first character to uppercase. |
| ${var^^}    or  ${var^^pattern} |                              Convert all character to uppercase. |