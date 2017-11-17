## jdbc原生api批量插入数据库


```

@Autowired
JdbcTemplate jdbcTemplate;

public int[] batchInsertOrUpdate( final List<InsertEntity> InsertEntiyList ){
    StringBuilder stringBuilder = new StringBuilder();
    //写sql
    stringBuilder.append( " insert into XXX " )
                .append( "( xx,xx,xx,xx...." )
                .append( " values (" )
                .append( " ?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)" )
                .append( " on duplicate key update " )
                .append( " xx =?, x=?" );
    String sql = stringBuilder.toString();
    int[] updateCounts = jdbcTemplate.batchUpdate( sql, new BatchPreparedStatementSetter() {
        @Override
        public void setValues( PreparedStatement ps, int i ) throws SQLException {
                InsertEntity insertEntity = InsertEntiyList.get( i );
                ps.setString( 1, xxx);
                //省略其他
            }
            @Override
            public int getBatchSize() {
                return InsertEntiyList.size();
            }
        } );    
        return updateCounts;
    }


```