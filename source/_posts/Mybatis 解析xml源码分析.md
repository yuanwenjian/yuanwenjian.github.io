---
title: Mybatis 解析xml源码
description:  Mybatis通过xml来配置数据源，别名，环境等一系列配置，再初始化时，会加载配置文件并解析，xml配置如何转换为我们需要的值
date: 2018-11-14 20:40:00
comments: true
tags: 
    - Java  
categories:
    - Mybatis
---

# mybatis-config xml配置及java解析

xml配置
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="config/generator.properties"/>

    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    

    <typeAliases>
        <package name="com.yuanwj.mybatisdemo.model"/>
    </typeAliases>


    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
                <property name="driver" value="${jdbc.driver}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mybatis/mapper/PhoneImeiMapper.xml"/>
    </mappers>


</configuration>
```

java解析核心代码

```java
    // 通过配置解析各个节点，每个节点生成XNode,并通过XNode获取其对应的配置属性
    private void parseConfiguration(XNode root) {
        try {
            Properties settings = this.settingsAsPropertiess(root.evalNode("settings"));
            this.propertiesElement(root.evalNode("properties")); 
            this.loadCustomVfs(settings);
            this.typeAliasesElement(root.evalNode("typeAliases"));
            this.pluginElement(root.evalNode("plugins"));
            this.objectFactoryElement(root.evalNode("objectFactory"));
            this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
            this.reflectionFactoryElement(root.evalNode("reflectionFactory"));
            this.settingsElement(settings);
            this.environmentsElement(root.evalNode("environments"));
            this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
            this.typeHandlerElement(root.evalNode("typeHandlers"));
            this.mapperElement(root.evalNode("mappers"));
        } catch (Exception var3) {
            throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
        }
    }
```

# 解析配置文件

mybatis解析核心位于parsing包，如下图
![parsing][parsing]

1. XNode类：每个标签就是一节点

2. XPathParser：  XPath 类的一个包装，主要的逻辑就是该类中实现的。

3. PropertyParser ： 属性解析器

4. TokenHandler ： 占位符解析器，是一个接口，由子类自己实现解析规则

5. GenericTokenParser ： 通用的占位符解析器，用来处理 #{} 和 ${} 参数

通过上述代码可知XNode为解析结果，下面代码分析解析过程及上述各个类作用

```java
    //上述代码调用方法  位于XNode
    public XNode evalNode(String expression) {//通过节点名解析
        return this.xpathParser.evalNode(this.node, expression);
    }

    // XPathParser.class
    public XNode evalNode(Object root, String expression) {
        Node node = (Node)this.evaluate(expression, root, XPathConstants.NODE);
        return node == null ? null : new XNode(this, node, this.variables);//生成XNode
    }

    // XPathParser.class 新建XNode
    public XNode(XPathParser xpathParser, Node node, Properties variables) {
        this.xpathParser = xpathParser;
        this.node = node;
        this.name = node.getNodeName();
        this.variables = variables;
        this.attributes = this.parseAttributes(node); //解析属性,关键方法
        this.body = this.parseBody(node); //子节点
    }    

    // XPathParser.class 解析属性
    private Properties parseAttributes(Node n) {
        Properties attributes = new Properties();
        NamedNodeMap attributeNodes = n.getAttributes();
        if (attributeNodes != null) {
            for(int i = 0; i < attributeNodes.getLength(); ++i) { //遍历属性
                Node attribute = attributeNodes.item(i);
                String value = PropertyParser.parse(attribute.getNodeValue(), this.variables); //通过PropertyParser类解析属性值
                attributes.put(attribute.getNodeName(), value); //获取属性值，放入properties中
            }
        }
        return attributes;
    }
// PropertyParser.class
public class PropertyParser {
    private PropertyParser() {
    }

    public static String parse(String string, Properties variables) {
        PropertyParser.VariableTokenHandler handler = new PropertyParser.VariableTokenHandler(variables);
        GenericTokenParser parser = new GenericTokenParser("${", "}", handler);// 通过GenericTokenParser解析
        return parser.parse(string);
    }

    private static class VariableTokenHandler implements TokenHandler {
        private Properties variables;

        public VariableTokenHandler(Properties variables) {
            this.variables = variables;
        }

        //获取${} 配置属性
        public String handleToken(String content) {
            return this.variables != null && this.variables.containsKey(content) ? this.variables.getProperty(content) : "${" + content + "}";
        }
    }
}    
    // GenericTokenParser.class 核心，读取配置属性值，如果是${},则读取配置，否则直接访问value值
    public String parse(String text) {
        StringBuilder builder = new StringBuilder();
        StringBuilder expression = new StringBuilder();
        if (text != null && text.length() > 0) {
            char[] src = text.toCharArray();
            int offset = 0;

            for(int start = text.indexOf(this.openToken, offset); start > -1; start = text.indexOf(this.openToken, offset)) {
                if (start > 0 && src[start - 1] == '\\') {
                    builder.append(src, offset, start - offset - 1).append(this.openToken);
                    offset = start + this.openToken.length();
                } else {
                    expression.setLength(0);
                    builder.append(src, offset, start - offset);
                    offset = start + this.openToken.length();

                    int end;
                    for(end = text.indexOf(this.closeToken, offset); end > -1; end = text.indexOf(this.closeToken, offset)) {
                        if (end <= offset || src[end - 1] != '\\') {
                            expression.append(src, offset, end - offset);
                            int var10000 = end + this.closeToken.length();
                            break;
                        }

                        expression.append(src, offset, end - offset - 1).append(this.closeToken);
                        offset = end + this.closeToken.length();
                    }

                    if (end == -1) {
                        builder.append(src, start, src.length - start);
                        offset = src.length;
                    } else {
                        builder.append(this.handler.handleToken(expression.toString()));
                        offset = end + this.closeToken.length();
                    }
                }
            }

            if (offset < src.length) {
                builder.append(src, offset, src.length - offset);
            }
        }

        return builder.toString();
    }
```

[parsing]:../images/mybatis/parsing.png