## java Gson


### 接受数组参数

```

List<String> idList = new Gson().fromJson( httpRequestInfo.getParameter( ID, "[]" ), new TypeToken<List<String>>() {
        }.getType() );

```


### 自定义过滤属性

```

    // 自定义json 过滤不要的属性
            Gson myGson = new GsonBuilder().registerTypeAdapter( Double.class, new DoubleNullAdapter() )
                    .registerTypeAdapter( Double.class, new DoubleTypeAdapterToDouble() )
                    .addSerializationExclusionStrategy( new ExclusionStrategy() {
                @Override
                public boolean shouldSkipField( FieldAttributes arg0 ) {
                            if( arg0.getName().equals( "xxx" ) ) {
                        return true;
                    }
                    return false;
                }

                @Override
                public boolean shouldSkipClass( Class<?> arg0 ) {
                    return false;
                }
            } ).create();


```