
Below code snippet helps to indentify any issues with JDBC Drivers 
In addition Stack trace helps to indentify how and where the driver manager is accessed in the code

```
		DriverManager.setLogWriter(new PrintWriter(System.out){
			@Override
			public void println(String message) {
				System.out.println("====Not an exception====");
				new Throwable().printStackTrace(System.out);
				super.println(message);
				System.out.println("====Not an exception====");;
			}
		});
```
