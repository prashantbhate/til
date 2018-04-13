````
<%!
    public void visit(StringBuilder sb, int indent, ClassLoader classLoader) {
        if (classLoader == null)
            return;
        String indentStr = new String(new char[indent]).replace("\0", "    ");
        sb.append("\n");
        sb.append(indentStr);
        sb.append(classLoader.getClass().getName());
        sb.append("@");
        sb.append(Integer.toHexString(java.util.Objects.hashCode(classLoader)));
        sb.append(":");
        if (indent > 20){
        sb.append("not visiting any further...\n");
            return;
		}
        if (classLoader instanceof java.net.URLClassLoader) {
            java.net.URL[] urls = ((java.net.URLClassLoader)classLoader).getURLs();
            for (java.net.URL url : urls) {
                sb.append("\n");
                sb.append(indentStr);
                sb.append(url);
            }
        }else if(classLoader instanceof org.glassfish.internal.api.DelegatingClassLoader) {
			org.glassfish.internal.api.DelegatingClassLoader cl = (org.glassfish.internal.api.DelegatingClassLoader)classLoader;
			if(cl.getDelegates().isEmpty()){
			sb.append("\n");
			sb.append(indentStr);
			sb.append("delegate : no delegates");
			}else{
				for (org.glassfish.internal.api.DelegatingClassLoader.ClassFinder d : cl.getDelegates()) {
					sb.append("\n");
					sb.append(indentStr);
					sb.append("delegate :");
					sb.append(d.toString().replace("\n"," "));
					visit(sb, 20, (ClassLoader)d);
				}
			}
		}else if(classLoader.getClass().getName().equals("com.sun.enterprise.v3.server.APIClassLoaderServiceImpl$APIClassLoader")){
			try{
				java.lang.reflect.Field f = classLoader.getClass().getDeclaredField("blacklist");
				f.setAccessible(true);
				for(String s: (java.util.Set<String>)f.get(classLoader)){
					sb.append("\n");
					sb.append(indentStr);
					sb.append(s);
				}
			}catch(Exception e){
				e.printStackTrace();
			}
		}
        sb.append("\n");
        visit(sb, indent + 1, classLoader.getParent());
    }

%>

<%
StringBuilder sb = new StringBuilder();
visit(sb,1,this.getClass().getClassLoader());
%>
<pre>
<%=sb%>
</pre>
````
