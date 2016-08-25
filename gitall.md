# gitall

We have about more than 20 git repositories, of which only a handful changes each day.

Its really a pain to update each, one by one manually so 
I use below script that goes through each git repo and selectively updates it with `gitall sp`. 

[ **sp** *is my favorite alias more on it some other day*]

Well infact it takes any git operation as argument and runs same command on each repo

## gitall script

~~~~shell
#!/bin/bash
for i in $(ls -1d */.git | cut -d / -f 1)
do
echo -e "$i>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>";
git -C $i "$@"
done
~~~~

Place it in a location accessible by $PATH, I keep such scripts under $HOME/bin

~~~~
gitall status -sb

repo1234>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
AM scripts/sort.xslt
A  share/file.sh
?? devops/.gradle/
?? devops/build.gradle
?? devops/glassfish-deployer/.gradle/
?? devops/glassfish-deployer/build/
?? devops/glassfish-deployer/deploy.sh
?? devops/gradle/
?? devops/gradlew
?? devops/gradlew.bat
?? devops/jbehave/
?? devops/liquibase/
?? devops/settings.gradle
?? scripts/domaindiff.sh
repo12345>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master [ahead 1]
repo1234678>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master
repo1234345>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master
repo12341324>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master
repo3453245>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master
repo2342345234>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master
repo111345>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master
repo11111324>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master
repo1213512351>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master [ahead 1]
repo1123412>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
## master...origin/master [ahead 1]
~~~~
*it looks more colorful than above in a zsh window*

This could have been extended to finding all .git directories,  
But as I have all my git repo at the same level `ls` works for now 

>25 Aug 2016
