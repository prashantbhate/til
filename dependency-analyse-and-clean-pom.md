
I wanted to remove all unused dependencies from pom.xml

dependency:analyse plugin did what was needed but only partially !
It just prints the result, its painstaking task to search for those <dependency> manually  and remove them from pom.xml

So I Cooked up below script to do the magic, 


````
mv -f pom.xml p1.xml
mvn org.apache.maven.plugins:maven-dependency-plugin:3.1.1:analyze -f p1.xml |
grep WARNING |
awk '/Unused declared dependencies found/,/Missing dependencies/{print $0}' |
grep -v Unused |
awk -F ' ' '{print $2}' |
awk -F : '{print "/x:project/x:dependencies/x:dependency[x:groupId=\""$1"\" and x:artifactId=\""$2"\"]"}' |
while read LINE
do
xmlstarlet ed -P -O -N x="http://maven.apache.org/POM/4.0.0" -d $LINE p1.xml > p2.xml
mv -f p2.xml p1.xml
done
mv -f p1.xml pom.xml
````

Howevever note that dependency:analyse lists false positives, so you may have to restore some of those dependencies back!
but that's easy with the help from version control and the IDE
