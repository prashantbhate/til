# Problem

I wanted to search for all fields of type that implemented `java.rmi.Remote` in the current project/repo.

# Background

This was needed to remove/replace all those fields with direct java calls  (to replace SOAP calls with direct java call).
Total Number of fields would then help me to accurately estimate the amount of work that is required.

# Solution

My initial thought was to search by some common pattern in the field name /type. Most of the wsdl2java generated code creates 
Classes which ends with PortType.

## search by variable name
~~~
grep '\w*PortType [^;]*;'  -or * 
~~~

But this did not detect some variable names that did not end with PortType

~~~
grep 'private \w*PortType[^;]*'  -or *
~~~

This fixed the issue, but this also listed fields from commented code !

## Structural Search

Time to do something sophisticated.  Intellij has an easter egg that goes by the name "Structural Search", this allows to 
search in java classers using some pattern that matches java elements

See : https://www.jetbrains.com/help/idea/2018.3/search-templates.html

I used below template
~~~
class $Class$ {   
  @$Annotation$( )
  $FieldType$ $Field$ = $Init$;
}
~~~
This listed all annotated fields from all classes



To narrow down the search I Had to add Filter to some of these variables


Below filter narrowed down to fields annontated with `@Autowired` 
~~~
$Annotation$ : text=.*Autowired, count=[1,∞]
~~~

But some fields could be Autowired through constructurs, so leave the annotation from the search

~~~
class $Class$ {   
  $FieldType$ $Field$ = $Init$;
}
~~~


Added some groovy code to check the the DeclaredType implemented `java.rmi.Remote` , script allows raw access to underlying PSI elements
See http://www.jetbrains.org/intellij/sdk/docs/basics/psi_cookbook.html for more info


~~~groovy
def list = __context__.type.superTypes.findAll {it.name.contains("Remote")}
if(list.size()>0){
def sb = new StringBuilder("")
.append(__context__.parent.parent.qualifiedName)
.append("      ")
.append(__context__.type.resolve().qualifiedName)
__log__.info(sb)
return true;
}
~~~

Or in short
~~~groovy
__context__.type.superTypes.findAll {it.name.contains("Remote")}.size()>0
~~~

**Make sure** to expand the script entry field to allow multiline 

The structural search now listed all fields with DeclaredType that implemented `java.rmi.Remote`


## Write result event log

__log__ prints the output to the event log. It helps to get the search result out in a format that can be used for further analysis See
https://github.com/JetBrains/intellij-community/blob/master/platform/structuralsearch/source/com/intellij/structuralsearch/impl/matcher/predicates/ScriptLog.java



## Write result to a file
~~~groovy
//Build the string str using __context__
new File("~/filetype.txt").append(str); 
~~~

## PSI Viewer

To understand the structure of java file you can configure PSI Viewer. Refer to https://www.jetbrains.com/help/idea/psi-viewer.html

Set this in the platform properties file
~~~
idea.is.internal=true
~~~
Once this is set "View PSI Structure Of Current file..." can be used to see the PSI structure


# More Strucural search Examples

Refer to for more examples

https://github.com/JetBrains/intellij-community/blob/master/platform/structuralsearch/testSource/com/intellij/structuralsearch/StructuralSearchTest.java


## Search Non @Annotated Interfaces

Below snippet search Interfaces that are not Annotations

~~~
class $Class$ {}

~~~
Script: 
~~~groovy
__context__.interface && !__context__.annotationType
~~~
