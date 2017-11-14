## java 代码审查

1.工具类或者常量类里面的方法都是静态的，建议直接用类名调用，不用创建对象，所以将构造方法私有化，禁止创建对象

>error example:

```
public  class StringUtils{
    /**
     * 判断一个字符串是否为英文中文或数字
     * 
     * @param str
     * @return
     */
    public static boolean isLetterDigitOrChinese( String str ) {
        String regex = "^[a-z0-9A-Z\u4e00-\u9fa5]+$";
        return str.matches( regex );
    }
}
```

>right example:

```
 public  class StringUtils{
  //添加构造私有构造方法
    private StringUtils{
    }
    /**
     * 判断一个字符串是否为英文中文或数字
     * 
     * @param str
     * @return
     */
    public static boolean isLetterDigitOrChinese( String str ) {
        String regex = "^[a-z0-9A-Z\u4e00-\u9fa5]+$";
        return str.matches( regex );
    }
}
```

2.NPE(空指针异常)
>使用equal 方法

```
//error 
String testString="abc";
if(testString.equals("abc")){
}else{
}
//right
String testString="abc";
if("abc".eqauls(testString)){
}else{
}
```

3.map list 新建 (java 1.7 )


```
// Noncompliant
List<String> strings = new ArrayList<String>();  
Map<String,List<Integer>> map = new HashMap<String,List<Integer>>();  

//Compliant Solution
List<String> strings = new ArrayList<>();
Map<String,List<Integer>> map = new HashMap<>();

```


4.使用isEmpty 来判断集合是否为空
```
//Noncompliant
if(data.size==0){

}

//compliant
if(data.isEmpty){

}
```
 
5.数组定义 ("[]" 在变量名字前面)
```
//Noncompliant
int arr[]={};

//compliant
int[] arr={};
```


6.字符串转换为数字
```
//Noncompliant
String numString="12222222";
long numLong=Long.valueOf(numString);
//compliant
String numString="12222222";
long posNum=Long.parseLong( posMap.get( "num" ).toString());
```


7.公共静态成员应该加上final，也就是public static final 一般不分家


8.switch cese : 相同的代码可以放在一个case里
```
//Noncompliant
switch (i) {
  case 1:
    doSomething();
    break;
  case 2:
    doSomethingDifferent();
    break;
  case 3:  // Noncompliant;
    doSomething();
    break;
  default:
    doTheRest();
```

```
//compliant
switch (i) {
  case 1:
  case 3:  
    doSomething();
    break;
  case 2:
    doSomethingDifferent();
    break;
 
  default:
    doTheRest();
```


9.float double 不能用==
```
float f = 0.1; // 0.100000001490116119384765625
double d = 0.1; // 0.1000000000000000055511151231257827021181583404541015625

//精度问题 不会等于0.1
if (f==0.1){
   
}

//right
if(f>=0.1){

}
```

10.常量命名

```
//Noncompliant
public static final String week="week";

//需要满足^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$'
//compliant
public static final String WEEK="week";
```