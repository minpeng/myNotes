## 优先使用Collections.EMPTY_MAP

```

/**
 * Collections.emptyMap() 和 new HashMap() 对比 <br>
 * 结论:优先使用Collections.emptyMap()
 * 
 * @author pengm
 *
 */
public class EmptyMap {
    public static void main( String[] args ) {
        long startTime = System.nanoTime();
        long endTime;

        for( int i = 0; i < 100000000; i++ ) {
            Map<String, String> map = Collections.EMPTY_MAP;
        }
        endTime = System.nanoTime();
        System.out.println( "Collections.EMPTY_MAP:  " + (endTime - startTime) );

        startTime = System.nanoTime();
        for( int i = 0; i < 100000000; i++ ) {
            Map<String, String> map = new HashMap<>();
        }
        endTime = System.nanoTime();
        System.out.println( "new HashMap:  " + (endTime - startTime) );
    }

}
```

结果:
```
Collections.EMPTY_MAP:  3826772
new HashMap:  179010057

```


类似

```
Collections.EMPTY_LIST:  453739326
new ArrayList:  583200178
```