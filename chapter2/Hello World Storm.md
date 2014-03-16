###**Hello World**

在这个工程里，我们创建一个简单的topology，数单词数量。我们可以把这个看作Storm的“Hello World”。不过，这是一个非常强大的topology，因为它能够扩展到几乎无限大的规模，而且只需要做一些小修改，就能用它构建一个统计系统。举个例子，我们可以把工程修改一下用来找出Twitter上的热点话题。

要创建这个topology，我们要用一个*spout*读取文本，第一个*bolt*标准化单词，第二个*bolt*，如图2-1所示。

![图2-1 topology入门][1]

你可以从这个网址下载源码压缩包，[ https://github.com/
storm-book/examples-ch02-getting_started/zipball/master][2]。

**NOTE**: 如果你使用[git][3]（一个分布式版本控制与源码管理工具），你可以执行git clone git@github.com:storm-book/examples-ch02-getting_started.git，把源码检出到你指定的目录。

###**Java安装检查**

构建Storm运行环境的第一步是检查你安装的Java版本。打开一个控制台窗口并执行命令：java -version。控制台应该会显示出类似如下的内容：

    java -version

    java version "1.6.0_26"
    Java(TM) SE Runtime Enviroment (build 1.6.0_26-b03)

    Java HotSpot(TM) Server VM (build 20.1-b02, mixed mode)

如果不是上述内容，检查你的Java安装情况。（参考[http://www.java.com/download/][4]

###**创建工程**

开始之前，先为这个应用建一个目录（就像你平常为Java应用做的那样）。这个目录用来存放工程源码。

接下来我们要下载Storm依赖包，这是一些jar包，我们要把它们添加到应用类路径中。你可以采用如下两种方式之一完成这一步：

 - 下载所有依赖，解压缩它们，把它 们添加到类路径
 - 使用*[Apache Maven][5]*

**NOTE**: Maven是一个软件项目管理的综合工具。它 
可以用来管理项目的开发周期的许多方面， 
从包依赖到版本发布过程。在这本书中，我们将广泛使用它 
。如果要检查是否已经安装了maven，在命令行运行mvn。如果没有安装 
你可以从[http://maven.apache.org/download.html][6]下载。

没有必要先成为一个Maven专家才能使用Storm，不过了解一下关于Maven工作方式的基础知识仍然会对你有所帮助。你可以在 
在Apache Maven的网站上找到更多的信息（[http://maven.apache.org/][7]）。

**NOTE:** Storm的Maven依赖引用了运行Storm本地模式的所有库。

要运行我们的topology，我们可以编写一个包含基本组件的pom.xml文件。

    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
             http://maven.apache.org/xsd/maven-4.0.0.xsd">
             <modelVersion>4.0.0</modelVersion>
             <groupId>storm.book</groupId>
             <artifactId>Getting-Started</artifactId>
             <version>0.0.1-SNAPSHOT</version>
             <build>
                 <plugins>
                     <plugin>
                         <groupId>org.apache.maven.plugins</groupId>
                         <artifactId>maven-compiler-plugin</artifactId>
                         <version>2.3.2</version>
                         <configuration>
                             <source>1.6</source>
                             <target>1.6</target>
                             <compilerVersion>1.6</compilerVersion>
                         </configuration>
                     </plugin>
                 </plugins>
             </build>
             <repositories>
                 <!-- Repository where we can found the storm dependencies -->
                 <repository>
                     <id>clojars.org</id>
                     <url>http://clojars.org/repo</url>
                 </repository>
             </repositories>
             <dependencies>
                 <!-- Storm Dependency -->
                 <dependency>
                     <groupId>storm</groupId>
                     <artifactId>storm</artifactId>
                     <version>0.6.0</version>
                 </dependency>
             </dependencies>
    </project>

开头几行指定了工程名称和版本号。然后我们添加了一个编译器插件，告知Maven我们的代码要用Java1.6编译。接下来我们定义了Maven仓库（Maven支持为同一个工程指定多个仓库）。clojars是存放Storm依赖的仓库。Maven会为运行本地模式自动下载必要的所有子包依赖。

一个典型的Maven Java工程会拥有如下结构：

    我们的应用目录/
             ├── pom.xml
             └── src
                   └── main
                      └── java
                   |  ├── spouts
                   |  └── bolts
                   └── resources

java目录下的子目录包含我们的代码，我们把要统计单词数的文件保存在resource目录下。

**NOTE**：命令mkdir -p 会创建所有需要的父目录。

###**创建我们的第一个Topology**

我们将为运行单词计数创建所有必要的类。可能这个例子中的某些部分，现在无法讲的很清楚，不过我们会在随后的章节做进一步的讲解。

###***Spout***

*spout* WordReader类实现了IRichSpout接口。我们将在[第四章][8]看到更多细节。WordReader负责从文件按行读取文本，并把文本行提供给第一个*bolt*。

**NOTE:** 一个*spout*发布一个定义域列表。这个架构允许你使用不同的*bolts*从同一个*spout*流读取数据，它们的输出也可作为其它*bolts*的定义域，以此类推。

例2-1包含WordRead类的完整代码（我们将会分配下述代码的每一部分）。

`
/**
 *  例2-1.src/main/java/spouts/WordReader.java
 */
`

    package spouts;
    import java.io.BufferedReader;
    import java.io.FileNotFoundException;
    import java.io.FileReader;
    import java.util.Map;
    import backtype.storm.spout.SpoutOutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.IRichSpout;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Values;
    public class WordReader implements IRichSpout {
        private SpoutOutputCollector collector;
        private FileReader fileReader;
        private boolean completed = false;
        private TopologyContext context;
        public boolean isDistributed() {return false;}
        public void ack(Object msgId) {
                System.out.println("OK:"+msgId);
        }
        public void close() {}
        public void fail(Object msgId) {
             System.out.println("FAIL:"+msgId);
        }
        /**
         * The only thing that the methods will do It is emit each
         * file line
         */
        public void nextTuple() {
        /**
         * The nextuple it is called forever, so if we have been readed the file
         * we will wait and then return
         */
             if(completed){
                 try {
                     Thread.sleep(1000);
                 } catch (InterruptedException e) {
                     //Do nothing
                 }
                return;
             }
             String str;
             //Open the reader
             BufferedReader reader = new BufferedReader(fileReader);
             try{
                 //Read all lines
                while((str = reader.readLine()) != null){
                 /**
                  * By each line emmit a new value with the line as a their
                  */
                     this.collector.emit(new Values(str),str);
                 }
             }catch(Exception e){
                 throw new RuntimeException("Error reading tuple",e);
             }finally{
                 completed = true;
             }
         }
         /**
          * We will create the file and get the collector object
          */
         public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
                 try {
                     this.context = context;
                     this.fileReader = new FileReader(conf.get("wordsFile").toString());
                 } catch (FileNotFoundException e) {
                     throw new RuntimeException("Error reading file ["+conf.get("wordFile")+"]");
                 }
                 this.collector = collector;
         }
         /**
          * Declare the output field "word"
          */
         public void declareOutputFields(OutputFieldsDeclarer declarer) {
             declarer.declare(new Fields("line"));
         }
    }

  [1]: https://github.com/runfriends/GettingStartedWithStorm-cn/blob/master/chapter2/Figure%202-1.%20Getting%20started%20topology.png
  [2]: https://github.com/%20storm-book/examples-ch02-getting_started/zipball/master
  [3]: http://git-scm.com/
  [4]: e%20http://www.java.com/download/
  [5]: http://maven.apache.org/
  [6]: http://maven.apache.org/download.html
  [7]: http://maven.apache.org/
  [8]: https://github.com/runfriends/GettingStartedWithStorm-cn/blob/master/chapter4/Spouts.md
