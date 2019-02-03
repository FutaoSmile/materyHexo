---
title: springboot 国际化
date: 2019-02-03 19:45:15
tags:
---

### # 定义国际化资源
* resources下新建i18n文件夹
* 新建xx.properties文件
* 中文：新建xx_zh_CN.properties文件存放对应的中文
* 英文：新建xx_en_US.properties文件存放对应的英文
效果是这样的:
![image.png](https://upload-images.jianshu.io/upload_images/1846623-8b85b05fa6735c41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 定义需要国际化的内容

![image.png](https://upload-images.jianshu.io/upload_images/1846623-3dc555615998a052.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 在application.yml中配置

```yml
spring:
  messages:
    # 定义国际化文件的文件地址，读取的原则是顺序读取如果存在同名的则读取第一个
    basename: i18n/login,i18n/errorMessage
```

* 定义国际化解析器与拦截器

```java
package com.futao.springmvcdemo.foundation.configuration;

import org.springframework.context.annotation.Bean;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;
import org.springframework.web.servlet.i18n.SessionLocaleResolver;

import java.util.Locale;

/**
 * 通用bean注册管理中心
 *
 * @author futao
 * Created on 2019-01-15.
 */
@org.springframework.context.annotation.Configuration
public class Configuration {

    /**
     * 国际化，设置默认的语言为中文
     * 将用户的区域信息存在session
     * 也可以存在cookie,由客户端保存   {@link org.springframework.web.servlet.i18n.CookieLocaleResolver}
     *
     * @return
     */
    @Bean
    public LocaleResolver localeResolver() {
        SessionLocaleResolver slr = new SessionLocaleResolver();
        slr.setDefaultLocale(Locale.SIMPLIFIED_CHINESE);
        return slr;
    }

    /**
     * 国际化，设置url识别参数
     *
     * @return
     */
    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor lci = new LocaleChangeInterceptor();
        lci.setParamName("lang");
        return lci;
    }
}
```

* 国际化配置文件会被spring加载，所以可以通过spring容器获取到国际化文件里面的内容
  * 获取spring容器
  
```java
package com.futao.springmvcdemo.utils;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Service;

/**
 * Spring容器
 *
 * @author futao
 * Created on 2019-01-15.
 */
@Service
public class SpringUtils implements ApplicationContextAware {

    private static ApplicationContext context = null;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if (context == null) {
            context = applicationContext;
        }
    }

    /**
     * 获取容器
     *
     * @return 容器
     */
    public static ApplicationContext getContext() {
        return context;
    }
}
```

* 在程序中使用

```java
//通过spring容器读取
SpringUtils.getContext().getMessage(code, null, LocaleContextHolder.getLocale())

//或者
    @Resource
     private MessageSource messageSource;
     return messageSource.getMessage(code, null, LocaleContextHolder.getLocale());
```

* 写个工具类

```java

package com.futao.springmvcdemo.service.notbusiness;

import com.futao.springmvcdemo.foundation.LogicException;
import com.futao.springmvcdemo.model.system.ErrorMessage;
import com.futao.springmvcdemo.utils.SpringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.NoSuchMessageException;
import org.springframework.context.i18n.LocaleContextHolder;

/**
 * 国际化
 *
 * @author futao
 * Created on 2019-01-15.
 */
public class I18nService {

    private static final Logger LOGGER = LoggerFactory.getLogger(I18nService.class);


    /**
     * 获取消息
     * 也可以使用下面的方式：
     * Resource private MessageSource messageSource;
     * return messageSource.getMessage(code, null, LocaleContextHolder.getLocale());
     *
     * @param code k
     * @return v
     */
    public static String getMessage(String code) {
        try {
            return SpringUtils.getContext().getMessage(code, null, LocaleContextHolder.getLocale());
        } catch (NoSuchMessageException e) {
            LOGGER.error("获取国际化资源{}失败" + e.getMessage(), code, e);
            throw LogicException.le(ErrorMessage.I18N_RESOURCE_NOT_FOUND, new String[]{code});
        }
    }
}
```

* 使用

```java
I18nService.getMessage("welcome")
```

* 前端在接口后面加上local信息`?lang=zh_CN`或者`?lang=en_US`即可


源代码: [https://github.com/FutaoSmile/springbootFramework](https://github.com/FutaoSmile/springbootFramework)
或者: [https://gitee.com/FutaoSmile/springboot_framework](https://gitee.com/FutaoSmile/springboot_framework)
欢迎star~

