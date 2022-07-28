



## 创建SpringApplicationRunListener监听器失败

因为我没有构造方法造成的，需要一个有参构造，并且参数固定

```java
@Order(3)
public class AuroraSpringApplicationRunListener implements SpringApplicationRunListener {

    private final SpringApplication application;
    private final String[] args;

    public AuroraSpringApplicationRunListener(SpringApplication sa, String[] args) {
        this.application = sa;
        this.args = args;
    }

    @Override
    public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        SpringApplicationRunListener.super.ready(context,timeTaken);
        LoadRolePermissionInfo loadRolePermissionInfo = context.getBean(LoadRolePermissionInfo.class);
        loadRolePermissionInfo.storagePermissionInfoToRedis();
    }
}

```

