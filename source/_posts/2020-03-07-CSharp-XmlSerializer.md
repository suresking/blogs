---
title: .NET序列化及反序列化
date: 2020-03-07 17:56:39
tags: CSharp
---

## 前言

序列化是指一个对象的实例可以被保存，保存成一个二进制串，当然，一旦被保存成二进制串，那么也可以保存成文本串了。

比如，一个计数器，数值为2，我们可以用字符串“2”表示。

如果有个对象，叫做`connter`，当前值为2，那么可以序列化成“2”，反向的，也可以从“2”得到值为2的计数器实例。

这样，关机时序列化它，开机时反序列化它，每次开机都是延续的。不会都是从头开始。

序列化概念的提出和实现，可以使我们的应用程序的设置信息保存和读取更加方便。

序列化有很多好处，比如，在一台机器上产生一个实例，初始化完毕，然后可以序列化，通过网络传送到另一台机器，然后反序列化，得到对象实例，之后再执行某些业务逻辑，得到结果，再序列化，返回第一台机器，第一台机器得到对象实例，得到结果。

这个例子是目前比较先进的“智能代理”的原理。

当前比较热火的`web services`使用`soap`协议，`soap`协议也是以对象的可序列化为基础的。

<!-- more -->

## 概述

.NET Framework为处理XML数据提供了许多不同的类库。`XmlDocument`类能让你像处理文件一样处理xml数据，而`XmlReader`、`XmlWriter`和它们的派生类使你能够将`xml`数据作为数据流处理。

`XmlSerializer`则提供了另外的方法，它使你能够将自己的对象串行化和反串行化为`xml`。串行化数据既能够让你像处理文件一样对数据进行随机处理，同时又能跳过你不感兴趣的数据。

## 主要类库介绍

.NET 支持对象`xml`序列化和反序列化的类库主要位于命名空间`System.Xml.Serialization`中。

### XmlSerializer 类

该类用一种高度松散耦合的方式提供串行化服务。你的类不需要继承特别的基类，而且它们也不需要实现特别的接口。相反，你只需在你的类或者这些类的公共域以及读/写属性里加上自定义的特性。`XmlSerializer`通过反射机制读取这些特性并用它们将你的类和类成员映射到`xml`元素和属性。

### XmlAttribute 类

指定类的公共域或读/写属性对应`xml`文件的`Attribute`。

例：`[XmlAttribute(“type”)] `or` [XmlAttribute(AttributeName=”type”)]`

### XmlElementAttribute类

指定类的公共域或读/写属性对应`xml`文件的`Element`。

例：`[XmlElement(“Maufacturer”)] `or`[XmlElement(ElementName=”Manufacturer”)]`

### XmlRootAttribute类

`Xml`序列化时，由该特性指定的元素将被序列化成`xml`的根元素。

例：`[XmlRoot(“RootElement”)] `or` [XmlRoot(ElementName = “RootElements”)]`

### XmlTextAttribute类

`Xml`序列化时，由该特性指定的元素值将被序列化成`xml`元素的值。一个类只允许拥有一个该特性类的实例，因为`xml`元素只能有一个值。

### XmlIgnoreAttribute类

`Xml`序列化时不会序列化该特性指定的元素。

## 实例

下面例子中的`xml schema `描述了一个简单的人力资源信息，其中包含了`xml`的大部分格式，如`xml `元素相互嵌套， `xml`元素既有元素值，又有属性值。

### 待序列化的类层次结构

``` c#
[XmlRoot("humanResource")]
public class HumanResource
{
    #region private data.
    private int m_record = 0;
    private Worker[] m_workers = null;
    #endregion
    [XmlAttribute(AttributeName = "record")]
    public int Record
    {
        get { return m_record; }
        set { m_record = value; }
    }
    [XmlElement(ElementName = "worker")]
    public Worker[] Workers
    {
        get { return m_workers; }
        set { m_workers = value; }
    }
}
public class Worker
{
    #region private data.
    private string m_number = null;
    private InformationItem[] m_infoItems = null;
    #endregion
    [XmlAttribute("number")]
    public string Number
    {
        get { return m_number; }
        set { m_number = value; }
    }
    [XmlElement("infoItem")]
    public InformationItem[] InfoItems
    {
        get { return m_infoItems; }
        set { m_infoItems = value; }
    }
}
public class InformationItem
{
    #region private data.
    private string m_name = null;
    private string m_value = null;
    #endregion
    [XmlAttribute(AttributeName = "name")]
    public string Name
    {
        get { return m_name; }
        set { m_name = value; }
    }
    [XmlText]
    public string Value
    {
        get { return m_value; }
        set { m_value = value; }
    }
}

```

### 序列化生成的xml结构

```xml
<?xml version="1.0" encoding="utf-8"?>
<humanResource xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" record="2">
 <worker number="001">
  <infoItem name="name">Michale</infoItem>
  <infoItem name="sex">male</infoItem>
  <infoItem name="age">25</infoItem>
 </worker>
 <worker number="002">
  <infoItem name="name">Surce</infoItem>
  <infoItem name="sex">male</infoItem>
  <infoItem name="age">28</infoItem>
 </worker>
</humanResource>

```

### 利用XmlSerializer类进行序列化和反序列化的实现（一般利用缓存机制实现xml文件只解析一次。）

```c#
public sealed class ConfigurationManager
{
    private static HumanResource m_humanResource = null;
    private ConfigurationManager() { }
    public static HumanResource Get(string path)
    {
        if (m_humanResource == null)
        {
            FileStream fs = null;
            try
            {
                XmlSerializer xs = new XmlSerializer(typeof(HumanResource));
                fs = new FileStream(path, FileMode.Open, FileAccess.Read);
                m_humanResource = (HumanResource)xs.Deserialize(fs);
                fs.Close();
                return m_humanResource;
            }
            catch
            {
                if (fs != null)
                    fs.Close();
                throw new Exception("Xml deserialization failed!");
            }
        }
        else
        {
            return m_humanResource;
        }
    }
    public static void Set(string path, HumanResource humanResource)
    {
        if (humanResource == null)
            throw new Exception("Parameter humanResource is null!");
        FileStream fs = null;
        try
        {
            XmlSerializer xs = new XmlSerializer(typeof(HumanResource));
            fs = new FileStream(path, FileMode.Create, FileAccess.Write);
            xs.Serialize(fs, humanResource);
            m_humanResource = null;
            fs.Close();
        }
        catch
        {
            if (fs != null)
                fs.Close();
            throw new Exception("Xml serialization failed!");
        }
    }
}

```

## 说明

- 需要序列化为`xml`元素的属性必须为读/写属性；

- 注意为类成员设定缺省值，尽管这并不是必须的。




## 有关.NET中序列化的一些知识

“序列化”可被定义为将对象的状态存储到存储媒介中的过程。在此过程中，对象的公共字段和私有字段以及类的名称（包括包含该类的程序集）都被转换为字节流，然后写入数据流。在以后“反序列化”该对象时，创建原始对象的精确复本。

### 为什么要选择序列化

- 一个原因是将对象的状态保持在存储媒体中，以便可以在以后**重新创建精确的副本**。

- 另一个原因是通过值**将对象从一个应用程序域发送到另一个应用程序域中**。

例如，序列化可用于在 ASP.NET 中保存会话状态并将对象复制到 Windows 窗体的剪贴板中。远程处理还可以使用序列化通过值将对象从一个应用程序域传递到另一个应用程序域中。

### 如何实现对象的序列化及反序列化
要实现对象的序列化，首先要保证该对象可以序列化。而且，序列化只是将对象的属性进行有效的保存，对于对象的一些方法则无法实现序列化的。

实现一个类可序列化的最简便的方法就是增加`Serializable`属性标记类。如：

```c#
[Serializable()]
public class MEABlock
{
    private int m_ID;
    public string Caption;

    public MEABlock()
    {
        //构造函数
    }
}

```

即可实现该类的可序列化。

要将该类的实例序列化为到文件中？`.NET FrameWork`提供了两种方法：

#### XML序列化
使用 **`XmLSerializer`** 类，可将下列项序列化。

- 公共类的公共读／写属性和字段
- 实现 **`ICollection`** 或 **`IEnumerable`** 的类。（注意只有集合会被序列化，而公共属性却不会。）
- **`XmlElement`** 对象。
- **`XmlNode`** 对象。
- **`DataSet`** 对象。

要实现上述类的实例的序列化，可参照如下例子：

```c#
MEABlock myBlock = new MEABlock();
// Insert code to set properties and fields of the object.
XmlSerializer mySerializer = new XmlSerializer(typeof(MEABlock));
// To write to a file, create a StreamWriter object.
StreamWriter myWriter = new StreamWriter("myFileName.xml");
mySerializer.Serialize(myWriter, MEABlock);
```

需要注意的是XML序列化只会将public的字段保存，对于私有字段不予于保存。

生成的XML文件格式如下：

```xml
<MEABlock>
  <Caption>Test</Caption>
</MEABlock>
```

对于对象的反序列化，则如下：

```c#
MEABlock myBlock;
// Constructs an instance of the XmlSerializer with the type
// of object that is being deserialized.
XmlSerializer mySerializer = new XmlSerializer(typeof(MEABlock));
// To read the file, creates a FileStream.
FileStream myFileStream = new FileStream("myFileName.xml", FileMode.Open);
// Calls the Deserialize method and casts to the object type.
myBlock = (MEABlock) mySerializer.Deserialize(myFileStream);

```

#### 二进制序列化

与XML序列化不同的是，二进制序列化可以将类的实例中所有字段（包括私有和公有）都进行序列化操作。这就更方便、更准确的还原了对象的副本。

要实现上述类的实例的序列化，可参照如下例子：

```c#
MEABlock myBlock = new MEABlock();
// Insert code to set properties and fields of the object.
IFormatter formatter = new BinaryFormatter();
Stream stream = new FileStream("MyFile.bin",FileMode.Create,FileAccess.Write, FileShare.None);
formatter.Serialize(stream, myBlock);
stream.Close();
```

对于对象的反序列化，则如下：

```c#
IFormatter formatter = new BinaryFormatter();
Stream stream = new FileStream("MyFile.bin", FileMode.Open,FileAccess.Read, FileShare.Read);
MEABlock myBlock = (MEABlock) formatter.Deserialize(stream);
stream.Close();
```

### 如何变相实现自定义可视化控件的序列化、反序列化

对于`WinForm`中自定义控件，由于继承于`System.Windows.Form`类，而`Form`类又是从`MarshalByRefObject`继承的，窗体本身无法做到序列化，窗体的实现基于`Win32下GUI`资源，不能脱离当前上下文存在。

当然可以采用变通的方法实现控件的序列化。这里采用的是**记忆类模型**。

定义记忆类（其实就是一个可序列化的实体类）用于记录控件的有效属性，需要序列化控件的时候，只需要将该控件的实例Copy到记忆类，演变成序列化保存该记忆类的操作。

反序列化是一个逆过程。将数据流反序列化成为该记忆类，再根据该记忆类的属性生成控件实例。而对于控件的一些事件、方法则可以继续使用。