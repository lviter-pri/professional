# java直接调用方法工具

一、参数实体类

```java

import lombok.Data;

import java.io.Serializable;
import java.util.List;

/**
 * @author lviter
 */
@Data
public class InvocationDTO implements Serializable {

    /**
     * beanName
     */
    private String beanName;
    /**
     * 方法名
     */
    private String methodName;
    /**
     * 是否开启事务
     */
    private Boolean openTransaction;
    /**
     * 参数列表
     */
    private List<Parameter> parameterList;

    @Data
    public static class Parameter {
        /**
         * 参数类型 类似: com.omega.api.model.BillInfo
         */
        private String paramType;
        /**
         * 转换类型
         */
        private String convertType;
        /**
         * 值
         */
        private Object value;
    }
}

```

二、SpringUtils工具类

```java

@Configuration
public class SpringUtils implements ApplicationContextAware {
    private static final Logger logger = LoggerFactory.getLogger(SpringUtils.class);

    public static String applicationName = null;
    public static String applicationAbbr = null;

    private static ApplicationContext ctx = null;

    public static Object getBean(String beanName) {
        if (ctx != null) {
            try {
                return ctx.getBean(beanName);
            } catch (Exception e) {
                logger.warn("获取Bean失败！ beanName: " + beanName, e);
                return null;
            }
        }

        return null;
    }

    public static <T> T getBean(String beanName, Class<T> requiredType) {
        T result = null;
        if (ctx != null) {
            try {
                result = ctx.getBean(beanName, requiredType);
            } catch (Exception e) {
                logger.warn("获取Bean失败！ beanName: " + beanName, e);
            }
        }

        return result;
    }

    public static <T> T getBean(Class<T> requiredType) {
        T result = null;
        if (ctx != null) {
            try {
                result = ctx.getBean(requiredType);
            } catch (Exception e) {
                logger.warn("获取Bean失败！", e);
            }
        }

        return result;
    }

    public static void publishEvent(ApplicationEvent event) {
        if (ctx != null) {
            ctx.publishEvent(event);
        }
    }

    public static ApplicationContext getApplicationContext() {
        return ctx;
    }

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        applicationName = applicationContext.getId();
        ctx = applicationContext;
        applicationAbbr = getProperty("spring.application.abbr", applicationName);
    }

    public static String getProperty(String key) {
        return getProperty(key, "");
    }

    public static String getProperty(String key, String defaultValue) {
        String result = defaultValue;
        Environment environment = getBean(Environment.class);
        if (environment != null) {
            result = environment.getProperty(key, defaultValue);
        }

        return result;
    }
}

```

三、Aop工具类

```java

/**
 * AopTargetUtils
 * 用于获取Spring AOP代理背后的真实目标对象（支持JDK动态代理和CGLIB代理）
 *
 * @author Lv.
 * @date 2025/7/7 14:49
 */
public class AopTargetUtils {

    /**
     * CGLIB代理中回调字段名称
     */
    private static final String CGLIB_CALLBACK_FIELD = "CGLIB$CALLBACK_0";

    /**
     * JDK动态代理中绑定的InvocationHandler字段名称
     */
    private static final String JDK_PROXY_HANDLER_FIELD = "h";

    /**
     * 获取代理对象背后的真实目标对象
     *
     * @param proxy 代理对象
     * @return 真实目标对象，如果非代理则返回自身
     * @throws RuntimeException 如果获取过程中发生异常
     */
    public static Object getTarget(Object proxy) {
        if (!AopUtils.isAopProxy(proxy)) {
            return proxy;
        }

        try {
            if (AopUtils.isJdkDynamicProxy(proxy)) {
                return getJdkDynamicProxyTargetObject(proxy);
            } else {
                return getCglibDynamicProxyTargetObject(proxy);
            }
        } catch (Exception e) {
            throw new RuntimeException("获取代理目标对象失败", e);
        }
    }

    /**
     * 获取JDK动态代理的目标对象
     */
    private static Object getJdkDynamicProxyTargetObject(Object proxy) throws Exception {
        Object invocationHandler = getFieldValue(proxy, JDK_PROXY_HANDLER_FIELD);
        Object advised = getFieldValue(invocationHandler, "advised");
        return ((AdvisedSupport) advised).getTargetSource().getTarget();
    }

    /**
     * 获取CGLIB代理的目标对象
     */
    private static Object getCglibDynamicProxyTargetObject(Object proxy) throws Exception {
        Object dynamicAdvisedInterceptor = getFieldValue(proxy, CGLIB_CALLBACK_FIELD);
        Object advised = getFieldValue(dynamicAdvisedInterceptor, "advised");
        return ((AdvisedSupport) advised).getTargetSource().getTarget();
    }

    /**
     * 使用反射获取指定字段的值
     *
     * @param obj       要获取字段的对象
     * @param fieldName 字段名称
     * @return 字段值
     * @throws Exception 反射异常
     */
    private static Object getFieldValue(Object obj, String fieldName) throws Exception {
        Field field = obj.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);
        return field.get(obj);
    }

    /**
     * 泛型方法：获取代理对象背后的真实目标对象并转换为目标类型
     *
     * @param proxy      代理对象
     * @param targetType 目标类型
     * @param <T>        泛型
     * @return 转换后的目标对象
     * @throws RuntimeException 如果类型不匹配或获取失败
     */
    public static <T> T getTarget(Object proxy, Class<T> targetType) {
        Object target = getTarget(proxy);
        if (!targetType.isInstance(target)) {
            throw new IllegalArgumentException("目标对象不是期望的类型：" + targetType.getName());
        }
        return targetType.cast(target);
    }
}

```

四、方法调用工具类

```java

/**
 * 调用方法工具类
 *
 * @author Lv.
 * @date 2025/7/8 9:20
 */
@Slf4j
public class InvocationUtil {
    /**
     * 调用指定Bean的方法（支持事务控制）
     *
     * @param dto 包含调用参数的 InvocationDTO
     */
    public void invocation(InvocationDTO dto) {
        if (dto == null || dto.getParameterList() == null) {
            log.warn("调用参数为空");
            return;
        }
        try {
            // 获取Bean和实际目标对象
            Object bean = SpringUtils.getBean(dto.getBeanName());
            Object target = AopTargetUtils.getTarget(bean);
            Class<?> clz = target.getClass();

            // 构建参数类型和值
            int length = dto.getParameterList().size();
            Class<?>[] parameterTypes = new Class[length];
            Object[] params = new Object[length];

            for (int i = 0; i < length; i++) {
                InvocationDTO.Parameter param = dto.getParameterList().get(i);
                parameterTypes[i] = Class.forName(param.getParamType());

                if ("object".equals(param.getConvertType())) {
                    String json = JSON.toJSONString(param.getValue());
                    params[i] = JsonUtils.serializable(json, parameterTypes[i]);
                } else {
                    params[i] = param.getValue();
                }
            }

            // 获取方法
            Method method = clz.getMethod(dto.getMethodName(), parameterTypes);
            ReflectionUtils.makeAccessible(method);

            // 是否启用事务
            if (Boolean.TRUE.equals(dto.getOpenTransaction())) {
                TransactionTemplate transactionTemplate = SpringUtils.getBean(TransactionTemplate.class);
                transactionTemplate.execute(action -> {
                    try {
                        method.invoke(target, params);
                    } catch (Exception e) {
                        log.error("事务内调用 {}#{} 异常", dto.getBeanName(), dto.getMethodName(), e);
                        throw new ApplicationException(ResponseCode.ONLY_THROW_MSG, "方法调用失败: " + dto.getMethodName(), e);
                    }
                    return null;
                });
            } else {
                method.invoke(target, params);
            }

        } catch (ClassNotFoundException e) {
            log.error("类未找到: {}", dto.getBeanName(), e);
        } catch (NoSuchMethodException e) {
            log.error("方法不存在: {}", dto.getMethodName(), e);
        } catch (Exception e) {
            log.error("调用 {}#{} 发生未知异常", dto.getBeanName(), dto.getMethodName(), e);
        }
    }
}

```