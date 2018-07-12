## sql模糊搜索特殊字符转义

```

	/**
	 * 转义用户输入的搜索词
	 * 
	 * @param str
	 * @return
	 */
	public static String transferSearchWord( String str ) {
		str = str.replace( "%", "\\%" );
		str = str.replace( "_", "\\_" );
		return str;
	}
	
```