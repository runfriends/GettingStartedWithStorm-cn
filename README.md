GettingStartedWithStorm-cn
==========================

翻译Getting Started with Storm
所有Storm相关术语都用斜体英文表示。
这些术语的字面意义翻译如下，由于这个工具的名字叫Storm，这些术语一律按照气象名词解释
Storm    暴风雨

*spout*  龙卷，读取原始数据为*bolt*提供数据

*bolt*   雷电，从*spout*或其它*bolt*接收数据，并处理数据，处理结果可作为其它*bolt*的数据源或最终结果

*nimbus* 雨云，主节点的守护进程，负责为工作节点分发任务。

下面是一些

*topology* Storm的一个任务单元

*define field(s)* 定义域，由*spout*或*bolt*提供，被*bolt*接收
