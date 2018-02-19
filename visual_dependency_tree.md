
A picture is worth thousand words. *Million* when it comes to finding out maven modules and their interdependencies.

In a drive to shrink our mamoth monolith middleware restful application into microservices, we needed to idenitfy interdependencies between maven modules in the code base.
immagine a mamoth that has 100 legs, all of which have been spaghettied together! Horrible and Scary I know !


`mvn dependency:tree` helps a bit as it shows a tree with all the dependencies.
However it also lists all 3rd party dependencies which is not much useful.  `including` only our application group ids reduced the list
but still it doesnt servers our visual brain.

Digging deeper into the documentation suggested that it can also generate a dot file with `-DoutputType=dot` option.
>See https://maven.apache.org/plugins/maven-dependency-plugin/tree-mojo.html for more info

For those who don't know, `dot` is a format/language (like `json`) used to depict graphs. `.dot` files can then be fed through 
graphviz commands
to generate images in various format like `svg`
>See https://www.graphviz.org/ for more info

this is all good but dependency tree generates one dot file per module, hence it doesnt give a bird's eye view. 
So all these dot files had to be combined to generate an uber dot file

It cries out loud for some clever scripting.


Here is what I did.


~~~~shell

#!/bin/bash

mvn clean -q
ext=dot
includes=my.group.id
excludes="my-artifact:ear|another-artifact:ear|one-more:pom"
mvn -q -e org.apache.maven.plugins:maven-dependency-plugin:3.0.2:tree -Dincludes=${includes} -DappendOutput=true -DoutputType=${ext} -DoutputFile=target/dependency-graph.${ext}

#A3 sfdp settings
#overlap = prism;
#splines = true;
#ratio="fill";
#page="16.5,11.7" ;
#K=5;
#node[shape=box];

echo 'digraph "APP" {
graph[rankdir=LR,ranksep=20,nodesep=2,size=30 ,ratio=expand];
node[shape=box,fontsize=900,penwidth = 50];
edge[arrowsize=20,penwidth = 10];
 ' >app.dot

#aggregate dependencies into a single .dot file
git ls-files | grep pom.xml| sed 's_pom.xml_target/dependency-graph.'${ext}'_'|xargs cat|sed 's_:[0-9.]*-SNAPSHOT__g;
s_${includes}:__g;
s/digraph "/subgraph "cluster_/;
' |grep -v  "subgraph " |
grep -v -E ${excludes} |sort -u >>app.dot
#echo "}" >>app.dot

echo "generating svg.. "
dot -O -Tsvg app.dot
~~~~

##  Pre-Requsites

* mvn
* git (to list all pom.xml files , Can use `find` instead but will be bit slower )
* dot from graphviz
