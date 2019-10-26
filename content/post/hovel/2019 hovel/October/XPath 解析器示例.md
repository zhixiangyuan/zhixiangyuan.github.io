---
title: "XPath 解析器示例"
date: 2019-10-22T16:25:33+08:00
keywords: []
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 XPath 解析器示例

```java
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathConstants;
import javax.xml.xpath.XPathExpressionException;
import javax.xml.xpath.XPathFactory;

import org.w3c.dom.Document;
import org.w3c.dom.NodeList;
import org.w3c.dom.Node;
import org.w3c.dom.Element;
import org.xml.sax.SAXException;

public class XPathParserDemo {
  public static void main(String[] args) {
    try {
      String xml =
        "<?xml version=\"1.0\"?>\n" +
          "<class>\n" +
          "   <student rollno=\"393\">\n" +
          "      <firstname>dinkar</firstname>\n" +
          "      <lastname>kad</lastname>\n" +
          "      <nickname>dinkar</nickname>\n" +
          "      <marks>85</marks>\n" +
          "   </student>\n" +
          "   <student rollno=\"493\">\n" +
          "      <firstname>Vaneet</firstname>\n" +
          "      <lastname>Gupta</lastname>\n" +
          "      <nickname>vinni</nickname>\n" +
          "      <marks>95</marks>\n" +
          "   </student>\n" +
          "   <student rollno=\"593\">\n" +
          "      <firstname>jasvir</firstname>\n" +
          "      <lastname>singh</lastname>\n" +
          "      <nickname>jazz</nickname>\n" +
          "      <marks>90</marks>\n" +
          "   </student>\n" +
          "</class>";
      // 创建文件输入流
      ByteArrayInputStream input = new ByteArrayInputStream(xml.getBytes(StandardCharsets.UTF_8));
      // 创建 Document 对象
      DocumentBuilderFactory dbFactory = DocumentBuilderFactory.newInstance();
      DocumentBuilder builder = dbFactory.newDocumentBuilder();
      Document doc = builder.parse(input);
      // 创建 XPath 对象
      XPath xPath = XPathFactory.newInstance().newXPath();

      // 创建表达式
      String expression = "/class/student";
      // 根据表达式找到节点
      NodeList nodeList = (NodeList) xPath.compile(expression)
        .evaluate(doc, XPathConstants.NODESET);
      // 遍历节点
      for (int i = 0; i < nodeList.getLength(); i++) {
        Node nNode = nodeList.item(i);
        System.out.println("\nCurrent Element :" + nNode.getNodeName());
        // 如果是元素节点则取出其中的数据
        if (nNode.getNodeType() == Node.ELEMENT_NODE) {
          Element eElement = (Element) nNode;
          System.out.println("Student roll no : " + eElement.getAttribute("rollno"));
          System.out.println("First Name : " +
            eElement
              .getElementsByTagName("firstname")
              .item(0)
              .getTextContent()
          );
          System.out.println("Last Name : " +
            eElement
              .getElementsByTagName("lastname")
              .item(0)
              .getTextContent()
          );
          System.out.println("Nick Name : " +
            eElement
              .getElementsByTagName("nickname")
              .item(0)
              .getTextContent()
          );
          System.out.println("Marks : " +
            eElement
              .getElementsByTagName("marks")
              .item(0)
              .getTextContent()
          );
        }
      }
    } catch (ParserConfigurationException | SAXException | IOException | XPathExpressionException e) {
      e.printStackTrace();
    }
  }
}
```

# 参考资料

1. [Java XPath 解析器 - 解析 XML 文档](https://www.yiibai.com/java_xml/java_xpath_parse_document.html)