This is a quick zsh snippet that I used to generate funny phonics for my 4 year old

```
printAndSay()
{
echo "" && figlet -w 400 -f banner3 $1 && say $1
}

BOUNCY=(b c d g h j p qu t w x y ch)
STRETCHY=(f l m n r s v z sh th ng)
for i in $BOUNCY
 for j in $STRETCHY 
    do
      WORD=$( echo $i{a,e,i,o,u}$j )
      DROW=$( echo $j{a,e,i,o,u}$i )
      WOORD=$( echo $i{ay,ee,igh,ow,oo}$j )
      DROOW=$( echo $j{ay,ee,igh,ow,oo}$i )

      printAndSay $WORD
      printAndSay $WOORD

      [[ "$j" != "ng" ]] && printAndSay $DROW && printAndSay $DROOW
    done
```
