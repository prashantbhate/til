# Problem

I wanted to search for all `@Autowired` fields of type that implemented `java.rmi.Remote` in the current project/repo.

# Background

This was needed to remove/replace all those fields with direct java calls  (to replace SOAP calls with direct java call).
Total Number of fields would then help me to accurately estimate the amount of work that is required.

# Solution

My initial thought was to search by some common pattern in the field name /type. Most of the wsdl2java generated code creates 
Classes which ends with PortType.

# search by variable name
~~~
grep '\w*PortType [^;]*;'  -or * 
~~~

But this did not detect some variable names that did not end with PortType

~~~
grep 'private \w*PortType[^;]*'  -or *
~~~

This fixed the issue, but this also listed fields from commented code !

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

Added some groovy code to check the the DeclaredType implemented `java.rmi.Remote` , script allows raw access to underlying PSI elements
See http://www.jetbrains.org/intellij/sdk/docs/basics/psi_cookbook.html for more info
~~~
$FieldType$ : script=
def t = __context__.type.resolve();
def list = t.interfaces.toList().collect{it.qualifiedName};
def s = list.join(", ");
def str = t.qualifiedName+':'+s+'\n';
s.contains("java.rmi.Remote")
~~~
**Make sure** to expand the script entry field to allow multiline 

The structural search now listed all `@Autowired` fields with DeclaredType that implemented `java.rmi.Remote`
