## FastJson

### 常用方法

1. 实体类转换为字符串

```
Stirng jsonString=JSONObject.toJSONString(attendanceGroup);


```

2.javabean 转换Map
```
//方法 一
Map<String, Object> a = (Map<String, Object>)JSON.toJSON(javabean)

//方法 二
Map<String, Object> a = JSON.parseObject(JSON.toJSONString(javabean))


```
3. 字符串转实体类

```
AttendanceGroup attendanceGroup = JSONObject.parseObject(jsonString,AttendanceClass.class);
```

4.字符串转List
```
List<AttendanceGroup> attendanceGroupList = JSONArray.parseArray(jsonString, AttendanceGroup.class);
```

5.最小范围设定格式化方法

```
JSONObject.DEFFAULT_DATE_FORMAT="yyyy-MM-dd HH:mm:ss";//设置日期格式

String jsonStr=JSONObject.toJSONString(object, SerializerFeature.WriteDateUseDateFormat);

```
### 自定义序列化方法

```

1.SimplePropertyPreFilter:根据propertyName判断是否序列化 
2.NameFilter：修改Key值
3.ValueFilter：修改Value值
4. BeforeFilter：序列化前添加内容
5. AfterFilter：序列化后添加内容
```

1. SimplePropertyPreFilter

```

SimplePropertyPreFilter filter = new SimplePropertyPreFilter(AttendanceGroup.class,
				new String[] { "id", "groupName", "groupMemberCount", "departmentsList", "usersList", "attendanceTimeList",
						"workDaysList", "isAutoRestOfHolidays", "mustPunchCardDay", "noPunchCardDay", "specialWorkgDay",
						"isNaturalMonth" });

```
2. NameFilter

```
NameFilter nameFilter=new NameFilter() {
			
			@Override
			public String process(Object object, String name, Object value) {
				if(name.equals("departmentsList")){ 
					return"departments"; 
				} 
				if(name.equals("usersList")){ 
					return"users"; 
				} 
				if(name.equals("workDaysList")){ 
					return"workDays"; 
				} 
				if(name.equals("attendanceTimeList")){ 
					return"attendanceTime"; 
				} 
				return name;
			}
		};

```
3. ValueFilter

```

	ValueFilter valueFilter = new ValueFilter() {						
		
		@Override			
		public Object process(Object object, String name, Object value) {
	   			if(name.equals("usersList")){	
					return "[]";				
				}				
				return value;			
				}		
			};


```

4. BeforeFilter
5. AfterFilter

```

	BeforeFilter beforeFilter = new BeforeFilter() {
			@Override			
			public void writeBefore(Object object) {
				//改变属性
				AttendanceGroup attendanceGroup = (AttendanceGroup)object;	
				attendanceGroup.setId(5);
			}		
	};


	AfterFilter afterFilter = new AfterFilter() {
			@Override			
			public void writeBefore(Object object) {
				//改变属性
				AttendanceGroup attendanceGroup = (AttendanceGroup)object;	
				attendanceGroup.setId(5);
			}		
	};

```