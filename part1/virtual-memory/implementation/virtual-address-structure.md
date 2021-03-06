4.7.1 虚拟地址结构

每一个 64 位的虚拟地址\(e.g.就是在我们程序内使用的地址\)都由一些字段组成，像图 4-1 中展示的一样。



![](/assets/4-1.gif)

图_** 4-1**.虚拟地址结构_

地址自身实际上只有 48 位宽；使用符号位扩展到了 64 位的**权威地址**。这样扩展的特性使得地址的左边 17 位是相等的。如果不满足前述条件，那么使用该地址时会立刻被系统拒绝。

48 位的虚拟地址在一个特殊表格的帮助下被转换为了 52 位的物理地址。

---

■Bus Error 当你使用非权威地址时你将会看到一条错误信息：Bus error

---

物理地址空间被划分为多个槽，这些槽可以被虚拟页所填充。这些槽叫作**页帧**。在页帧之间没有空间间隔，所以页帧地址总是从一个末尾 12 位是零的地址开始的。

虚拟地址的最低 12 位和物理页的最低 12 位对应了页内的地址偏移，所以这 12 位是在两者中是相等的。

虚拟地址的另外四个部分代表了地址翻译表中的索引位置。每一个表负责 4KB，这 4KB 会填满一整个内存的页。翻译表中的每条记录都是 64 位宽；条目中保存了下一个表的起始地址和其它的一些服务 flags。

