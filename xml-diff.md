#Comparing xml files


I am currently building a docker container to replicate existing glassfish server running in dev environment. 
To make sure that I am building exactly same glassfish domain using docker , I needed to compare new xml with  existing.

- domain.xml within glassfish docker container (This glassfish domain.xml is what I am currently building issuing sequence of asadmin command as part of docker build )
- with handcrafted domain.xml that is version controlled int the git repository (this glassfish domain.xml is build over the time and over multiple versions)


As the handcoded domain.xml does not have any strict ordering of elements and attributes which made it impossible to compare two xmls with regular text diff .


## xml diff technique

xslt to the resque:

So here is mini algorithm,
create an xslt that 
- sorts each attribute of every element by its name
- sorts each element by 
 - element name
 - when two element names match, compare attribute in following order
  - `@id`
  - `@name`
  - `@value`
  - `@description`
  - `@ref`
  - if all above are same match it with element's content  `text()`

Once I have this sort.xslt create a shell script that can
- apply sort.xslt to new.xml
- apply sort.xslt to old.xml
- do a diff (with your favorite text diff app )


## xml aware diff  
~~~~xml
<xsl:stylesheet version="1.0"
 xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
 <xsl:output omit-xml-declaration="yes" indent="yes"/>
 <xsl:strip-space elements="*"/>

 <xsl:template match="node()|@*">
  <xsl:copy>

   <xsl:apply-templates select="@*">
    <xsl:sort select="name()"/>
   </xsl:apply-templates>

   <xsl:apply-templates select="node()">
    <xsl:sort select="name()"/>
    <xsl:sort select="@id"/>
    <xsl:sort select="@name"/>
    <xsl:sort select="@value"/>
    <xsl:sort select="@description"/>
    <xsl:sort select="@ref"/>
    <xsl:sort select="text()"/>
   </xsl:apply-templates>

  </xsl:copy>
 </xsl:template>
</xsl:stylesheet>
~~~~

## xml diff script

~~~~shell
#!/bin/bash

# domaindiff.sh

# Change this to your taste
NEW_XML=devbox_app_1:/usr/local/glassfish3/glassfish/domains/domain1/config/domain.xml
docker cp $NEW_XML new.xml
NEW_XML=new.xml
OLD_XML=${SOURCE_DIR}/src/main/glassfish3x/domain1/domain.xml
$DIFF=opendiff
$SORT_XSLT=sort.xslt

cd /tmp
xsltproc -o new_domain.xml $SORT_XSLT $NEW_XML
xsltproc -o old_domain.xml $SORT_XSLT $OLD_XML

$DIFF old_domain.xml new_domain.xml

rm new_domain.xml old_domain.xml

~~~~

This way I can repeat 

* docker build
* run domaindiff.sh and compare
* correct Dockerfile

again and again ....

##conclusion

I agree there could be may other and better way to do the same thing,

But this was good enough for me ,
and something that I have learnt today

>24 Aug 2016

