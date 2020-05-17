## jackJson 工具类

```java

public class JsonUtils {

    /**
     * 默认 ObjectMapper
     */
    private static ObjectMapper defaultObjectMapper = buildDefaultObjectMapper();

    protected static ObjectMapper buildDefaultObjectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();

        // 将Json字符串转为对象时,允许字符串包含不存在字段
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // 忽略空Bean转json的错误
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        // 设置时间格式
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        // 当属性值为NULL时,不参与序列化
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

        return objectMapper;
    }

    /**
     * 序列化对象,默认设置: 忽略null/不存在字段的报错,null值不参与序列化,时间格式为:yyyy-MM-dd HH:mm:ss
     *
     * @param object
     * @return
     */
    public static <T> String toJson(T object) {
        return toJson(object, null);
    }

    /**
     * 序列化对象,使用自定义ObjectMapper
     *
     * @param object       序列化对象
     * @param objectMapper 自定义ObjectMapper
     * @return
     */
    public static <T> String toJson(T object, ObjectMapper objectMapper) {
        if (null == objectMapper) {
            objectMapper = defaultObjectMapper;
        }
        try {
            return objectMapper.writeValueAsString(object);
        } catch (JsonProcessingException e) {
            throw new IllegalArgumentException(e.getMessage());
        }
    }

    public static <T> T toObject(String json, Class<T> valueType) {
        return toObject(json, valueType, null);
    }

    public static <T> T toObject(String json, TypeReference<T> type) {
        return toObject(json, null, type);
    }

    /**
     * @param json
     * @param clazz
     * @param type
     * @param <T>
     * @return
     */
    private static <T> T toObject(String json, Class<T> clazz, TypeReference<T> type) {
        T obj = null;
        if (!StringUtils.isEmpty(json)) {
            try {
                if (clazz != null) {
                    obj = defaultObjectMapper.readValue(json, clazz);
                } else {
                    obj = defaultObjectMapper.readValue(json, type);
                }
            } catch (IOException e) {
                throw new IllegalArgumentException(e.getMessage());
            }
        }
        return obj;
    }
}
```