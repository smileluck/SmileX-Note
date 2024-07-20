[TOC]

---

# 前言

该文章基于我之前写的另一篇`关于Druid多数据源的配置`，若没有使用多数据，可以借鉴一下，主要是思路是创建新的连接池，然后替换掉原先的连接池，因为 `Druid` 初始化后不能更换连接再次初始化。

# 实现代码

```java
@Slf4j
@RefreshScope
@Configuration
public class NacosListener implements InitializingBean {

    @Value("${smilex.nacos.listener.dataId}")
    private String dataId;

    @Resource
    private DataSourceProperties dataSourceProperties;

    @Autowired
    private NacosConfigManager nacosConfigManager;

    @Resource
    private DynamicDataSource dynamicDataSource;

    @Override
    public void afterPropertiesSet() throws Exception {
        NacosConfigProperties nacosConfigProperties = nacosConfigManager.getNacosConfigProperties();
        log.info(nacosConfigProperties.toString());

        ConfigService configService = nacosConfigManager.getConfigService();
        NacosConfigProperties.Config config = nacosConfigProperties.getSharedConfigs().get(0);
        configService.addListener(dataId, config.getGroup(), new Listener() {
            @Override
            public Executor getExecutor() {
                return null;
            }

            @Override
            public void receiveConfigInfo(String configInfo) {
                log.info(configInfo);
                Yaml yaml = new Yaml();
                JSONObject infoJson = new JSONObject(yaml.load(configInfo));
                refreshDataSource(infoJson);
            }
        });
    }

    /**
     * 刷新数据库
     */
    private void refreshDataSource(JSONObject infoJson) {
        JSONObject druid = getProperties(infoJson, "spring.datasource.druid");
        log.info(druid.toJSONString());
        if (druid != null) {
            DataSourceProperties dataSourceProperties = druid.toJavaObject(DataSourceProperties.class);
//            DataSource dataSource = (DataSource) dynamicDataSource.getDataSource(DynamicDataSourceConfig.MASTER);
//            DataSourceFactory.createDataSource(dataSource, dataSourceProperties);
            dynamicDataSource.replaceDataSource(DynamicDataSourceConfig.MASTER, dataSourceProperties);

        }
    }

    private JSONObject getProperties(JSONObject infoJson, String properties) {
        if (StringUtils.isNotBlank(properties)) {
            return loopProperties(infoJson, properties.split("\\."), 0);
        }
        return null;
    }

    private JSONObject loopProperties(JSONObject infoJson, String[] properties, int index) {
        if (infoJson.containsKey(properties[index])) {
            if (index < properties.length - 1) {
                return loopProperties(infoJson.getJSONObject(properties[index]), properties, index + 1);
            } else {
                return infoJson.getJSONObject(properties[index]);
            }
        }
        return null;
    }
}
```
