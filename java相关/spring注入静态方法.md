## spring注入静态方法

###1.使用@PostConstruct初始化
```

@Component
public class ProductConfigItemTagUtil {
    @Autowired
    ProductConfigItemService productConfigItemService;

    private static ProductConfigItemTagUtil productConfigItemTagUtil;
    
    //初始化
    @PostConstruct
    public void init() {
        productConfigItemTagUtil = this;
        productConfigItemTagUtil.productConfigItemService = this.productConfigItemService;

    }
    
    //静态方法
    public static void myWork() {
       //这样就可以使用productConfigItemService 里面的方法
       productConfigItemService.demo();
    }
}
```