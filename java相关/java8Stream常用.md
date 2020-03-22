## java8Stream常用


```java


//1.stream某一个元素计数
long count = myInfoList.stream().mapToInt(item -> item.getAge()).sum();
System.out.println(count);

//2.stream过滤后某一个元素计数
long count2 = myInfoList.stream().filter(item -> item.getAge() > 21).mapToInt(item -> item.getAge()).sum();
System.out.println(count2);



//3.过滤后统计集合大小
long count1 = myInfoList.stream().filter(item -> item.getAge() > 21).count();
System.out.println(count1);

//4.过滤后收集
List<MyInfo> collect = myInfoList.stream().filter(item -> item.getAge() > 21).collect(Collectors.toList());
System.out.println(collect);

//5.判断是否有该元素
boolean flag = myInfoList.stream().anyMatch(item -> item.getName().equals("pengm1"));
System.out.println(flag);

//6.分组
Map<Integer, List<MyInfo>> collect1 = myInfoList.stream().collect(Collectors.groupingBy(item -> item.getAge()));
System.out.println(collect1);
Map<Integer, Map<String, List<MyInfo>>> collect2 = myInfoList.stream().collect(Collectors.groupingBy(item -> item.getAge(), Collectors.groupingBy(item -> item.getName())));
System.out.println(collect2);


//7.list转map(要唯一)
Map<String, MyInfo> myInfoMap = myInfoList.stream().collect(
        Collectors.toMap(item -> String.valueOf(item.getAge()),
                item -> item));


```
