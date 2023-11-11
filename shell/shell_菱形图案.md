#  shell 菱形图案
tags: 艺术

 - `special-pattern.sh`

```bash
#!/bin/bash

MAX_NO=0

echo -n "Enter Number between (5 to 9) : "
read MAX_NO

if ! [ $MAX_NO -ge 5 -a $MAX_NO -le 9 ]; then
	echo "WTF... I ask to enter number between 5 and 9, Try Again"
	exit 1
fi

clear

for ((i = 1; i <= MAX_NO; i++)); do
	for ((s = MAX_NO; s >= i; s--)); do
		echo -n " "
	done
	for ((j = 1; j <= i; j++)); do
		echo -n " ."
	done
	echo ""
done
###### Second stage ######################
for ((i = MAX_NO; i >= 1; i--)); do
	for ((s = i; s <= MAX_NO; s++)); do
		echo -n " "
	done
	for ((j = 1; j <= i; j++)); do
		echo -n " ."
	done
	echo ""
done

echo -e "\n\n\t\t\t Whenever you need help, Tecmint.com is always there"
```
执行：

```bash
bash special-pattern.sh
Enter Number between (5 to 9) : 5
      .
     . .
    . . .
   . . . .
  . . . . .
  . . . . .
   . . . .
    . . .
     . .
      .


                         Whenever you need help, Tecmint.com is always there

bash special-pattern.sh
Enter Number between (5 to 9) : 8
         .
        . .
       . . .
      . . . .
     . . . . .
    . . . . . .
   . . . . . . .
  . . . . . . . .
  . . . . . . . .
   . . . . . . .
    . . . . . .
     . . . . .
      . . . .
       . . .
        . .
         .


                         Whenever you need help, Tecmint.com is always there


```


