````
<%@ page language="java"
		 import="java.util.*"
		 import="org.apache.commons.lang.*"
		 import="java.util.concurrent.*"
		 import="java.text.*"
		 import="java.io.*"
		 import="java.security.*"
		 import="java.lang.reflect.*"
		 import="java.lang.ref.*"
%>
asdf
<pre>
<%
Class<?> aClass = Class.forName(request.getParameter("c").replace("/","."));
ClassLoader cl = aClass.getClassLoader();
if(cl!=null){
	out.print("Class loaded using ");
	out.print(aClass.getClassLoader().getClass().getName());
	out.print("@");
	out.println(Integer.toHexString(java.util.Objects.hashCode(aClass.getClassLoader())));
}else{
	out.println("Using system classloader");
}
CodeSource codeSource = aClass.getProtectionDomain().getCodeSource();
if(codeSource!=null){
out.println(codeSource.getLocation().toURI());
}else{
	out.println("codeSource not available");
}
out.println("Location using getResource()");
out.println(this.getClass().getClassLoader().getResource(request.getParameter("c").replace(".","/")+".class"));
%>
</pre>
````
