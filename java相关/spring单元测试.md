## spring 单元测试

### 1.编写主要测试基类

```

//加载配置文件
@ContextConfiguration(locations = { "/spring-mvc.xml", "/applicationContext.xml","/SqlMapConfig.xml" })
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
public class SpringTestBase {
	
	protected MockMvc mockMvc;
	
	@BeforeClass
	public static void beforeClass() throws Exception {
	    //加载数据源
		ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("classpath:InitJndi-test.xml");
		DataSource ds = (DataSource) app.getBean("dataSource");
		SimpleNamingContextBuilder builder = new SimpleNamingContextBuilder();
		builder.bind("java:comp/env/jdbc/attendance", ds);
		
		DataSource ds2 = (DataSource) app.getBean("dataSource2");
		builder.bind("jdbc/gncdb5", ds2);
		builder.activate();
	}
	
	@Before
	public void setUp() throws Exception {
	
		MockitoAnnotations.initMocks( this );

	}
}
```
### 2.数据源配置InitJndi-test.xml


```
<?xml version="1.0" encoding="UTF-8"?>
    <beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
    <beans:bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <beans:property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <beans:property name="url" value="jdbc:mysql:/" />
    <beans:property name="username" value="xxx" />
    <beans:property name="password" value="xxx" />
    </beans:bean>
    
    <beans:bean id="dataSource2" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <beans:property name="driverClassName" value="net.sourceforge.jtds.jdbc.Driver" />
    <beans:property name="url" value="" />
    <beans:property name="username" value="xxx" />
    <beans:property name="password" value="xxx" />
    </beans:bean>
    
    </beans:beans>

```

### 3.编写测试类-继承于基类

```
public class XXXXTest extends SpringTestBase {
	@Autowired
	XXXController  xxxController;
	
	@Before
	public void setUp() throws Exception {
		super.setUp();
		mockMvc = MockMvcBuilders.standaloneSetup(xxxController).build();
	}
	
	//测试类
    @Test
    public void testGetPunchCardData() {
        try {
            String resultString = this.mockMvc
                    .perform(MockMvcRequestBuilders.get("/attendance/XXX").accept(MediaType.APPLICATION_JSON)
                            .param("XX", "XX").param("XX", "XX"))
                    .andExpect(MockMvcResultMatchers.status().isOk()).andReturn().getResponse().getContentAsString();
            System.out.println(resultString);

            String resultString1 = this.mockMvc
                    .perform(MockMvcRequestBuilders.get("/attendance/XX").accept(MediaType.APPLICATION_JSON)
                            .param("", "").param("", "2018--06").param("XXXXXXXXXX", "in"))
                    .andExpect(MockMvcResultMatchers.status().isOk()).andReturn().getResponse().getContentAsString();
            System.out.println(resultString1);

        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    
    
}   
```

### 测试文件下载
> 主要就是增加 ResultHandler

```
@Test
public void testExport() {
    try {
        mockMvc.perform(MockMvcRequestBuilders.get("/XXXX/exportAttendanceRate")
                .characterEncoding("UTF-8").header("User-Agent", "EDGE").param("XX", "2018-01-01")
                .param("XX", "2018-09-01")).andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(new ResultHandler() {
                     
                    @Override
                    public void handle(MvcResult result) throws Exception {
                        result.getResponse().setCharacterEncoding("UTF-8");
                        MockHttpServletResponse contentRespon = result.getResponse();
                        InputStream contentInStream = new ByteArrayInputStream(
                                contentRespon.getContentAsByteArray());
                        XSSFWorkbook resultExcel = new XSSFWorkbook(contentInStream);

                        String filePath = "D:\\data\\jsp\\exportExcel"
                                + DateUtil.getCurrentTime() + ".xls";
                        File file = new File(filePath);
                        FileOutputStream fos = new FileOutputStream(file);
                        resultExcel.write(fos);
                        resultExcel.close();

                    }
                });
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }

}
```

### 测试文件上传
```
 private static final String api = "/api/common";

    @Test
    public void fileUpload() throws Exception {
        String url = api + "/fileUpload";
        String filePath = "D://abc.xlsx";
        FileInputStream fis = new FileInputStream(filePath);
        MockMultipartFile firstFile = new MockMultipartFile("file", filePath, "multipart/form-data", fis);
        String result = mockMvc.perform(MockMvcRequestBuilders.multipart(url)
                .file(firstFile))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn().getResponse().getContentAsString();
        System.out.println(result);
    }
```
