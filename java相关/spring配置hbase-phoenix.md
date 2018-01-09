

### spring配置hbase-phoenix

#### 1.增加相关jar包
```
    <dependency>
      <groupId>org.apache.phoenix</groupId>
      <artifactId>phoenix-core</artifactId>
      <version>4.5.2-HBase-1.0</version>
    </dependency>
```


#### 2.配置phoenix
```
<bean id="phoenixJdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
            <constructor-arg ref="phoenixDataSource"/>
            <qualifier value="phoenixJdbcTemplate"></qualifier>
        </bean>
        
        <bean id="namedParameterJdbcTemplate"
          class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
            <constructor-arg ref="phoenixDataSource"/>
            <qualifier value="namedParameterJdbcTemplate"></qualifier> 
        </bean>

        <bean id="phoenixDataSource" class="org.apache.commons.dbcp.BasicDataSource">
            <property name="driverClassName" value="org.apache.phoenix.jdbc.PhoenixDriver"/>
           <!-- ip地址-->
           <property name="url" value="jdbc:phoenix:xxx1,xxx2"/>

            <!-- 因为Phoenix进行数据更改时不会自动的commit,必须要添加defaultAutoCommit属性,否则会导致数据无法提交的情况 -->
            <property name="defaultAutoCommit" value="true"/>
        </bean>
```

#### 3.使用

##### jdbc.query查询

```
@Autowired
private NamedParameterJdbcTemplate namedParameterJDBCTemplate;

Map<String, Object> paramMap = new HashMap<String, Object>();

StringBuffer sb = new StringBuffer();        
sb.append( " select REGEXP_SPLIT(\"key\",'\\\\+')[2] date ," )
                .append( " \"type\" type,sum(\"triggerCount\") triggerTimes, sum(\"uniqueUserCount\") triggerUserTimes " );
sb.append( " from " );
sb.append( "\"" + TableName.T_HTML_EVENT_ALL_DAY + "\" " );
sb.append( " where  " );
sb.append( "( \"key\" between " ).append( ":startRow" ).append( " and " ).append( ":endRow ) " );
sb.append( " group by \"type\",REGEXP_SPLIT(\"key\",'\\\\+')[2]" );

paramMap.put( "startRow", productId + "+" + startDate );
paramMap.put( "endRow", productId + "+" + endDate + "," );

String sql = sb.toString();


//构造返回对象
private RowMapper<BehaviorEventAnaly> getEventAnalyRowMapper(){
        return new RowMapper<BehaviorEventAnaly>(){
            public BehaviorEventAnaly mapRow(ResultSet rs, int i) throws SQLException {
                BehaviorEventAnaly obj = new BehaviorEventAnaly();
               
                obj.setProductId(rs.getInt("APPID"));
                obj.setClientType(rs.getString("OS_TYPE") );
                String keys[] = rs.getString("key").split("\\+");
                obj.setVersion(keys[3]);
                obj.setChannel(keys[4]);
                obj.setEventId(keys[5]);
                obj.setTrigNum(rs.getString("EVENTIME")!=null?Integer.valueOf(rs.getString("EVENTIME")):0);
                obj.setTrigUserNum(rs.getString("UTOTAL")!=null?Integer.valueOf(rs.getString("UTOTAL")):0);
                
                return obj;
            }
        };
    }

//jdbc查询
public List<BehaviorEventAnaly> queryEventAnaly( String sql, Map<String, Object> paramMap ) {
        return namedParameterJDBCTemplate.query( sql, paramMap, getEventAnalyRowMapper() );
    }

```


##### jdbc.query查询(2)

```
@Autowired
private NamedParameterJdbcTemplate namedParameterJDBCTemplate;

//传参返回数据
public List<Map<String, Object>> getFeedBackTrendData( final String productId, final String startTime, final String endTime,
            final String[] channels ) {
       //结果集
        ResultSetExtractor rs = new ResultSetExtractor() {
            public List<Map<String, Object>> extractData( ResultSet rs ) throws SQLException, DataAccessException {
                List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
                while( rs.next() ) {
                    Map<String, Object> map = new HashMap<String, Object>();
                    // 加判断
                    if( rs.getString( "dateString" ) != null || rs.getString( "complaintCount" ) != null
                            || rs.getString( "consultCount" ) != null || rs.getString( "suggestCount" ) != null
                            || rs.getString( "otherCount" ) != null ) {
                        map.put( "complaintCount", rs.getLong( "complaintCount" ) );
                        map.put( "consultCount", rs.getLong( "consultCount" ) );
                        map.put( "suggestCount", rs.getLong( "suggestCount" ) );
                        map.put( "otherCount", rs.getLong( "otherCount" ) );
                        map.put( "date", rs.getString( "dateString" ) );
                        list.add( map );
                    }
                }
                return list;
            }
        };
        

        //sql
        StringBuilder sb = new StringBuilder();

        sb.append( " select " );
        sb.append( " \"Date\" as dateString, " );
        sb.append(
            " sum( (case  when \"ComplainNum\" is null then (cast(0 as UNSIGNED_LONG)) else \"ComplainNum\" end ) ) as complaintCount, " );
        sb.append(
            " sum( (case when \"ConsultNum\" is null then (cast(0 as UNSIGNED_LONG)) else \"ConsultNum\" end) ) as consultCount, " );
        sb.append(
            " sum( (case when \"AdviceNum\" is null then (cast(0 as UNSIGNED_LONG)) else \"AdviceNum\" end) ) as suggestCount," );
        sb.append(
            " sum( (case when \"OtherNum\" is null then (cast(0 as UNSIGNED_LONG)) else \"OtherNum\" end) ) as otherCount " );
        sb.append( " from " );
        sb.append( "\"" + TableName.TV_T_CUSTOMER_NUM + "\" " );
        sb.append( " where ( \"key\" between " ).append( ":startRow" ).append( " and " ).append( ":endRow )" );

        sb.append( "and \"CustomerId\" in " );
        sb.append( "(" + getChannels( channels ) + ")" );
        sb.append( "  group by dateString " );
        final String sql = sb.toString();

        Map<String, Object> paramMap = new HashMap<String, Object>();

        paramMap.put( "startRow", productId + "+" + startTime + "+" );
        paramMap.put( "endRow", productId + "+" + endTime + "," );
        

        List<Map<String, Object>> list = namedParameterJdbcTemplate.query( sql, paramMap, rs );

        return list;
```


##### mybatis 映射查询

######  1.增加phoenix -mybatis 配置

```
<!-- MyBatis配置 -->
    <bean id="phoenixSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="phoenixDataSource" />
        <property name="configLocation" value="classpath:mybatis-config.xml"></property>
        <!-- 自动扫描entity目录, 省掉Configuration.xml里的手工配置 -->
        <property name="typeAliasesPackage" value="com.xxx" />
        <!-- MyBatis的实体类 -->
        <!-- 显式指定Mapper文件位置 -->
        <property name="mapperLocations"
            value="classpath:/phoenix/*/*Mapper.xml" />
    </bean>
    <!-- 扫描basePackage下所有以@PhoenixRepository 接口 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.xxx" />
        <property name="annotationClass"
            value="com.xxx.PhoenixRepository" />
        <property name="sqlSessionFactoryBeanName" value="phoenixSqlSessionFactory"></property>
    </bean>

```

######  2.增加PhoenixRepository注解

```
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.stereotype.Component;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Component
public @interface PhoenixRepository {
    String value() default "";
}
```

######  3.增加实现接口注解
```
@PhoenixRepository
public interface PhoenixDaoMapper {
    List<Map<String,Object>> getIncomeTrend(Map<String,Object> param);
    List<IncomeEntity> getIncomeTrend(Map<String,Object> param);
}
```

######  4.编写mapper sql 

```
<select id="getIncomeTrend" resultType="map">
       select * from xxx
</select>
```