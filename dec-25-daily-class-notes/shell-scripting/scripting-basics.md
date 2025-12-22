# Scripting basics

i have 9 aruments in total<br>

$1 $2 $3 $4 $5 $6 $7 $8 $9

<br>

after 10 th argument we use ${10} ${11} ...

<br>

<br>

$1 -- FIrst Argument

$2 -- Second Argument

$3 -- Third Argument&#x20;

<br>

$0 -- Script Name

$# -- Total Number of Arguments

$\* -- All Arguments as a Single String

$@ -- All Arguments as Separate Strings

\$$ -- Process ID of the Script

$? -- Exit Status of the Last Command

&#x20; 0 -- success

&#x20; non zero -- failure&#x20;

<br>

<br>

<br>

\# Comparison Operators in Bash

-eq -- equals

-ne -- not equals

-gt -- greater than

-lt -- less than

-ge -- greater than or equals

-le -- less than or equals

<br>

\# Logical Operators in Bash

-! -- NOT

-&& -- AND

\-|| -- OR

<br>

\# if statement&#x20;

<br>

if \[ condition ];

then

&#x20;  \# code to be executed if condition is true

&#x20;  else

&#x20;   \# code to be executed if condition is false

fi
