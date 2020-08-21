## 1. jetty 8 配置

进入jetty安装主目录下子目录contexts中，创建resource.xml(文件名可自行设置)，添加如下内容：

```xml
<?xml version="1.0"  encoding="ISO-8859-1"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">
<Configure class="org.eclipse.jetty.server.handler.ContextHandler">
  <Call class="org.eclipse.jetty.util.log.Log" name="debug">
      <Arg>Configure resource.xml</Arg>
  </Call>
  <!--根据实际情况配置-->
  <Set name="contextPath">/static</Set>
  <!--根据实际情况配置-->
  <Set name="resourceBase">file:/home/app/public/static</Set>
  <Set name="handler">
    <New class="org.eclipse.jetty.server.handler.ResourceHandler">
      <Set name="welcomeFiles">
        <Array type="String">
          <Item>index.html</Item>
        </Array>
      </Set>
      <!-- <Set name="cacheControl">max-age=3600,public</Set> -->
          <Set name="directoriesListed">true</Set>
    </New>
  </Set>
</Configure>
```

## 2. jett9配置

在jetty主目录下子目录webapps中，创建resource.xml(文件名可自行设置)，添加如下内容：

```xml
<?xml version="1.0"  encoding="UTF-8"?>
<!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN"  "http://www.eclipse.org/jetty/configure_9_0.dtd">
<Configure class="org.eclipse.jetty.server.handler.ContextHandler">
    <!--根据实际情况配置-->
    <Set name="contextPath">/static</Set>
    <Set name="handler">
        <New class="org.eclipse.jetty.server.handler.ResourceHandler">
            	<!--根据实际情况配置-->
                <Set name="resourceBase">file:/home/app/public/static</Set>
                <Set name="directoriesListed">true</Set>
        </New>
    </Set>
</Configure>
```

