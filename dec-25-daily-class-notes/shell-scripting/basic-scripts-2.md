# Basic scripts -2

Shell Script:

echo $SHELL --> to check default shell

\#!/bin/bash --> shebang --> invokes bash shell

exp1.sh

\#!/bin/bash\
echo "Hi \nThis is devops class"\
echo "Started on Nov"

execution shell script:

./exp1.sh\
bash exp1.sh\
sh exp1.sh

\================================================

exp2.sh

\#!/bin/bash\
echo "This is $1 class"\
echo "Started on $2"

execution:\
./exp2.sh Devops Nov

\================================================

Input arguments

if the argument from 1 to 9

```
we used as $1, $2, $3 ... $9
```

if the argument from 10th onwards\
we used as ${10}, ${11}....

$1 --> first input argument\
$2 --> Second input argument\
$0 --> Shell script name

\================================================

exp3.sh

\#!/bin/bash\
name="Test"\
month="Nov"\
num="2022"\
echo "This is $name platform and month is $month"\
echo "The number will be $num"

\================================================

<br>

<br>

If statement:

<br>

if \[ condition ]

then

execution

else

execution

fi

<br>

\======================================

<br>

exp4.sh

<br>

\#!/bin/bash

if \[ $1 -eq 5 ]

then

echo "$1 is Five"

else

echo "$1 is not Five"

fi

<br>

\=========================================

<br>

parameters:

<br>

-eq --> equals

-lt --> less than

-le --> less than or equal

-ge --> greater than or equal

-gt --> greater than

-ne --> not equal

<br>

<br>

\=========================================

exp5.sh

<br>

write a shell script to find biggest of two numbers.

<br>

2 3

<br>

\#!/bin/bash

if \[ $1 -gt $2 ]

then

echo "$1 is Big"

else

echo "$2 is Big"

fi

<br>

\===========================================

exp6.sh

<br>

\#!/bin/bash

if \[ $# -ne 2 ]

then

echo "Enter only two arguments"

exit

fi

if \[ $1 -gt $2 ]

then

echo "$1 is Big"

else

echo "$2 is Big"

fi

<br>

\===========================================

<br>

<br>

special characters of global variables:

<br>

$? --> status of last executed command&#x20;

0 --> means success

non-zero --> failure &#x20;

<br>

\$$ --> process ID of current running process

<br>

$! --> process ID of last command went into background

<br>

$\* --> All the arguments pass to a shell script

<br>

$@ --> All the arguments pass to a shell script stored in array format \[ 1 23 55 66 ]

<br>

$# --> Total number of arguments pass to a shell script

<br>

\======================================================================================

<br>

syntax for "for" loop

<br>

for i in List

do

commands/execution

done

<br>

\======================================================================================

<br>

<br>

Syntax for "While" lopp

<br>

<br>

while \[ condition]

do&#x20;

commands/execution

done

<br>

\======================================================================================

<br>

Write a shell script to check whether given name is a file or directory or link or doesnot exist?

<br>

\#!/bin/bash

echo "Enter the name to check:"

read name

if \[ -f $name ]

then

echo "$name is file"

elif \[ -d $name ]

then

echo "$name is a directory"

elif \[ -l $name ]

then

echo "$name is a link"

else

echo "$name doesnot exit"

fi

<br>

\======================================================================================

<br>

<br>

Assignments:



1\. Biggest 3 no's

2\. Write a shell script to check whether given name is a file or directory or link or doesnot exist,

\* If its a file display the content, count how many lines content is present in that file.

\* if its a directory display the file&#x20;

3\. Write a shell script to add, subtract, multiply and divide the two numbers

4\. Write a script to find factorial of a given number

5\. Write a script to add given set of numbers



\======================================================================================



