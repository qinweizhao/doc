## 1 dom4j介绍

　　dom4j是一个Java的XML API，类似于jdom，用来读写XML文件的。dom4j是一个非常非常优秀的Java XML API，具有性能优异、功能强大和极端易用使用的特点，同时它也是一个开放源代码的软件，可以在SourceForge上找到它。在IBM developerWorks上面可以找到一篇文章，对主流的Java XML API进行的性能、功能和易用性的评测，dom4j无论在那个方面都是非常出色的。如今你可以看到越来越多的Java软件都在使用dom4j来读写XML，特别值得一提的是连Sun的JAXM也在用dom4j。这是必须使用的jar包， Hibernate用它来读写配置文件。

　　dom4j主要接口都在org.dom4j这个包里定义： 　　

　　Attribute Attribute定义了XML的属性

　　Branch Branch为能够包含子节点的节点如XML元素(Element)和文档(Docuemnts)定义了一个公共的行为，

　　CDATA CDATA 定义了XML CDATA 区域

　　CharacterData CharacterData是一个标识借口，标识基于字符的节点。如CDATA，Comment, Text.

　　Comment Comment 定义了XML注释的行为

　　Document 定义了XML文档

　　DocumentType DocumentType 定义XML DOCTYPE声明

　　Element Element定义XML 元素

　　ElementHandler ElementHandler定义了 Element 对象的处理器

　　ElementPath 被 ElementHandler 使用，用于取得当前正在处理的路径层次信息

　　Entity Entity定义 XML entity

　　Node Node为所有的dom4j中XML节点定义了多态行为

　　NodeFilter NodeFilter 定义了在dom4j节点中产生的一个滤镜或谓词的行为（predicate）

　　ProcessingInstruction ProcessingInstruction 定义 XML 处理指令.

　　Text Text 定义XML 文本节点.

　　Visitor Visitor 用于实现Visitor模式.

　　XPath XPath 在分析一个字符串后会提供一个XPath 表达式

## 2 使用dom4j创建xml文档

　　Document document = DocumentHelper.createDocument();

　　通过这句定义一个XML文档对象。

 

　　Element root = document.addElement("根节点名称");

　　通过这句定义一个XML元素，这里添加的是根节点。

 

　　Element有几个重要的方法：

　　　　addComment ： 添加注释

　　　　addAttribute ： 添加属性

　　　　addElement ： 添加子元素

　　最后通过XMLWriter生成物理文件，默认生成的XML文件排版格式比较乱，可以通过OutputFormat类格式化输出，默认采用createCompactFormat()显示比较紧凑，最好使用createPrettyPrint()。

实例代码

```
 1 package cn.mars.app.txn.whpf;
 2 
 3 import java.io.FileOutputStream;
 4 
 5 import org.dom4j.Attribute;
 6 import org.dom4j.Document;
 7 import org.dom4j.DocumentHelper;
 8 import org.dom4j.Element;
 9 import org.dom4j.io.OutputFormat;
10 import org.dom4j.io.XMLWriter;
11 
12 public class Dom4jTest {
13 
14     public static void main(String[] args) {
15         // 创建文档。
16         Document document = DocumentHelper.createDocument();
17         // 设置文档DocType，这里为了举例，添加hibernate的DocType
18         document.addDocType("hibernate-configuration", "-//Hibernate/Hibernate Configuration DTD 3.0//EN",
19                 "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd");
20         // 文档增加节点，即根节点，一个文档只能有一个根节点，多加出错
21         Element root = document.addElement("skills");
22         // 添加注释
23         root.addComment("第一个技能");
24         // 根节点下添加节点
25         Element first = root.addElement("skill");
26         // 节点添加属性
27         first.addAttribute("name", "独孤九剑");
28         // 节点下添加节点
29         Element info = first.addElement("info");
30         // 节点设置内容数据
31         info.setText("为独孤求败所创，变化万千，凌厉无比。其传人主要有风清扬、令狐冲。");
32 
33         // 同理增加其他节点，内容，属性等
34         Element second = root.addElement("skill");
35         second.addAttribute("name", "葵花宝典");
36         Element info2 = second.addElement("info");
37         info2.setText("宦官所创，博大精深，而且凶险至极。练宝典功夫时，首先要自宫净身。");
38 
39         // 创建节点
40         Element third = DocumentHelper.createElement("skill");
41         // 将节点加入到根节点中
42         root.add(third);
43         // 创建属性，第一个参数指定了拥有者，也可以为null，指定拥有者
44         Attribute name = DocumentHelper.createAttribute(third, "name", "北冥神功");
45         // 将属性加入到节点上
46         third.add(name);
47         // 创建子节点并加入到节点中
48         Element info3 = DocumentHelper.createElement("info");
49         info3.setText("逍遥派的顶级内功之一，能吸人内力转化为自己所有，威力无穷。");
50         third.add(info3);
51 
52         try {
53             // 创建格式化类
54             OutputFormat format = OutputFormat.createPrettyPrint();
55             // 设置编码格式，默认UTF-8
56             format.setEncoding("UTF-8");
57             // 创建输出流，此处要使用Writer，需要指定输入编码格式，使用OutputStream则不用
58             FileOutputStream fos = new FileOutputStream("d:/skills.xml");
59             // 创建xml输出流
60             XMLWriter writer = new XMLWriter(fos, format);
61             // 生成xml文件
62             writer.write(document);
63             writer.close();
64         } catch (Exception e) {
65             e.printStackTrace();
66         }
67     }
68 }
```

输出文件内容：

```
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <!DOCTYPE hibernate-configuration PUBLIC "-//Hibernate/Hibernate Configuration DTD 3.0//EN" "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
 3 
 4 <skills>
 5   <!--第一个技能-->
 6   <skill name="独孤九剑">
 7     <info>为独孤求败所创，变化万千，凌厉无比。其传人主要有风清扬、令狐冲。</info>
 8   </skill>
 9   <skill name="葵花宝典">
10     <info>宦官所创，博大精深，而且凶险至极。练宝典功夫时，首先要自宫净身。</info>
11   </skill>
12   <skill name="北冥神功">
13     <info>逍遥派的顶级内功之一，能吸人内力转化为自己所有，威力无穷。</info>
14   </skill>
15 </skills>
```

## 3 使用dom4j解析xml文件

### 3.1 构建dom4j树

　　org.dom4j.io提供了两个类：SAXReader和DOMReader，DOMReader只能一个现有的w3c DOM树构建dom4j树，即只能从一个org.w3c.dom.Document中构建org.dom4j.Document树，而SAXReader则使用SAX解析器，从不同的输入源构建dom4j树，如可以从xml文件中读取并构建dom4j树。

　　**实例代码：使用SAXReader解析**

```
1 SAXReader reader = new SAXReader();
2 Document document = reader.read(new File("d:/skills.xml"));
```

　　**实例代码：使用DOMReader解析**

```
1 DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
2 DocumentBuilder db = dbf.newDocumentBuilder();
3 File file = new File("d:/skills.xml");
4 org.w3c.dom.Document domDocument = db.parse(file);
5 DOMReader reader = new DOMReader();
6 org.dom4j.Document document = reader.read(domDocument);
```

### 3.2 获取节点

　　获得dom4j树之后，可以根据dom4j树获取节点。首先获取根节点，然后根据根节点获取其子节点。

　　**实例代码：访问根节点**

```
1 Element root = document.getRootElement();
```

　　**实例代码：访问所有子节点**

```
List skills = root.elements();
for (Iterator<?> it = skills.iterator(); it.hasNext();) {
Element e = (Element) it.next();

}
```

　　**实例代码：访问指定名称的节点，如访问名称为“skill”的全部节点**

```
1 List skills = root.elements("skill");
2 for (Iterator<?> it = skills.iterator(); it.hasNext();) {
3 Element e = (Element) it.next();
4 //TODO
5 }
```

　　**实例代码：访问指定名称的第一个节点**

```
1 Element skill = root.element("skill");
```

　　**实例代码：迭代某个元素的所有子元素，如迭代root**

```
1 for(Iterator<Element> it = root.elementIterator();it.hasNext();){
2 Element e = it.next();
3 //TODO
4 }
```

### 3.3 获取属性

　　获取节点后，可以根据节点获取属性，获得方式与获取节点类似。

　　**实例代码：获取指定名称的元素**

```
1 Element skill = root.element("skill");
2 Attribute attr1 = skill.attribute("name");
```

　　**实例代码：按照属性顺序获取属性**

```
1 Element skill = root.element("skill");
2 Attribute attr2 = skill.attribute(0);
```

　　**实例代码：获取某节点下全部属性1**

```
1 Element skill = root.element("skill");
2 List<Attribute> list = skill.attributes();
3 for (Iterator<Attribute> it = list.iterator(); it.hasNext();) {
4 Attribute attr = it.next();
5 // TODO
6 }
```

　　**实例代码：获取某节点下全部属性2**

```
1 Element skill = root.element("skill");
2 for (Iterator<Attribute> it = skill.attributeIterator(); it.hasNext();) {
3 Attribute attr = it.next();
4 // TODO
5 }
```

 

### 3.4 使用XPath获取节点和属性

　　Dom4j 中集成了对XPath的支持。在选择节点时，可以直接使用XPath 表达式。这种方式更加方便，简洁，官方文档中推荐使用该种方式。

　　**实例代码：要选择所有的<skill>元素的name属性**

```
1 List list = document.selectNodes("//skills/skill/@name");
2 for (Iterator it = list.iterator(); it.hasNext();) {
3 Attribute attr = (Attribute) it.next();
4 //TODO
5 }
```

　　注意:为了能够编译执行上述使用XPath表达式的代码，需要配置dom4j 安装包中自带的jaxen包，你也可以从http://sourceforge.net/products/jaxen/上下载jaxen。jaxen是 一个用java开发的XPath引擎，用于配合各种基于XML的对象模型，如DOM,dom4j和JDOM。在dom4-1.6.1 目录下，有一个lib 子目录，其中有个jaxen-1.1-beta-6.jar文件，需要在classpath环境变量中配置该文件的全路径名。

## 4 使用dom4j修改xml文件

　　修改xml文件，需要先获取dom4j树（即Document），通常欲修改节点需要先获得该节点或其父节点，欲修改属性，需要先获得该属性所在的节点和该属性。

　　增加操作：参照前文。

　　删除操作：

　　**实例代码：删除某节点**

```
1 Element root = document.getRootElement();
2 Element skill = root.element("skill");
3 root.remove(skill);
```

　　**实例代码：删除指定名称的属性**

```
1 Element skill = root.element("skill");
2 skill.remove(skill.attribute("name"));
```

　　修改操作：

　　**实例代码：修改节点名称和节点值**

```
1 Element skill = root.element("skill");
2 skill.setName("new_skill");
3 skill.setText("你好");
```

　　**实例代码：修改属性值**

```
1 Attribute attr = skill.attribute("name");
2 attr.setText("newName");
```

　　属性名称无法修改，欲修改属性名称，可以先删除旧属性，再增加新属性

## 5 常用方法

### 5.1 Element元素API

　　getQName() 元素的QName对象

　　getNamespace() 元素所属的Namespace对象

　　getNamespacePrefix() 元素所属的Namespace对象的prefix

　　getNamespaceURI() 元素所属的Namespace对象的URI

　　getName() 元素的local name

　　getQualifiedName() 元素的qualified name

　　getText() 元素所含有的text内容，如果内容为空则返回一个空字符串而不是null

　　getTextTrim() 元素所含有的text内容，其中连续的空格被转化为单个空格，该方法不会返回null

　　attributeIterator() 元素属性的iterator，其中每个元素都是Attribute对象

　　attributeValue() 元素的某个指定属性所含的值

　　elementIterator() 元素的子元素的iterator，其中每个元素都是Element对象

　　element() 元素的某个指定（qualified name或者local name）的子元素

　　elementText() 元素的某个指定（qualified name或者local name）的子元素中的text信息

　　getParent 元素的父元素

　　getPath() 元素的XPath表达式，其中父元素的qualified name和子元素的qualified name之间使用"/"分隔

　　isTextOnly() 是否该元素只含有text或是空元素

　　isRootElement() 是否该元素是XML树的根节点

### 5.2 Attribute属性API

　　getQName() 属性的QName对象

　　getNamespace() 属性所属的Namespace对象

　　getNamespacePrefix() 属性所属的Namespace对象的prefix

　　getNamespaceURI() 属性所属的Namespace对象的URI

　　getName() 属性的local name

　　getQualifiedName() 属性的qualified name

　　getValue() 属性的值

### 5.3 字符串转化

　　**实例代码：把节点，属性，文档等转化成字符串，使用asXML()方法。**

```
1 String docXmlText = document.asXML();
2 String rootXmlText = root.asXML();
```

　　**实例代码：把字符串转换为文档，注意引号需要转义**

```
1 String skillString = "<skill name="xxx">神龙摆尾</skill>";
2 Document d = DocumentHelper.parseText(skillString);
```

### 5.4 命名空间（Namespace）操作

　　dom4j的名称空间信息api常用的方法有8个。

　　dom4j在Element和Attribute 接口中定义了获取名称空间信息的方法，这些方法和JDOM中的方法相同。如下所示：

　　public java.lang.String getNamespacePrefix()该方法返回元素（属性）的名称空间前缀

　　public java.lang.String getNamespaceURI()该方法返回元素（属性）的名称空间

　　URIpublic java.lang.String getName()该方法返回元素（属性）的本地名

　　public java.lang.String getQualifiedName()该方法返回元素（属性）的限定名

　　public Namespace getNamespace()该方法返回元素本身的名称空间

　　public java.util.List additionalNamespaces()返回某元素上附加的名称空间声明列表，列表中的每一个对象都是Namespace类型。

　　这个类的方法提供了两个方法分别获得名称空间前缀和本地名。如下：

　　public java.lang.String getPrefix()该方法返回名称空间前缀。

　　public java.lang.String getURI()该方法返回名称空间的URI。

## 6 Qname介绍

　　Qname在使用dom4j的时候，经常见到，一般自己解析的xml很少使用这种复杂格式。

　　1. 来历：qname是qualified name的简写

　　2. 构成：由名字空间(namespace)前缀(prefix)以及冒号(:)，还有一个元素名称构成

　　3. 举例：

```
1 <xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
2 xmlns="http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"
3 version="1.0">
4 <xsl:template match="foo">
5 <hr/>
6 </xsl:template>
7 </xsl:stylesheet>
```

　　xsl是名字空间前缀，template是元素名称，xsl:template 就是一个qname。

　　4.总结：qname无非是有着特定格式的xml元素，其作用主要是增加了名字空间，比如有同样的元素名称，而名字空间不同的情况。

## 7 Visitor模式

　　dom4j对visitor的支持可以大大缩减代码量，并且清楚易懂。这个模式很有用、很实用！

　　自己的类需要继承VisitorSupport，由于VisitorSupport使用适配器模式，可以覆写需要的方法，不需要的方法VisitorSupport提供了空实现。

