# RTOS-5
**嵌入式实时操作系统5——就绪表**

**就绪表作用**

操作系统内核会将就绪的任务存放在就绪表，内核总是从就绪表中找出最高优先级任务，并执行该任务。
内核调度任务时只用关注就绪表，从就绪表中最高优先级项中选择任务并执行。

就绪表是存放就绪任务的列表，就绪表通常有两个项目：优先级，任务成员。**同一个优先级的任务放在就绪表中的同一个优先级列表项中**,就绪表框架如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2f4dafac53c84d96a63fe7b32e5cdee2.png)
**构建就绪表**
使用**静态数组**的方式可以构建一个就绪表，代码实现如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a1fdf496377d46aab9c3caa8903200f7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_18,color_FFFFFF,t_70,g_se,x_16)

其中tcb_item_t为 TCB项 ，list_item_t列表项，ready_list为就绪表。
就绪表中包含了10个列表，每个列表对应一个优先级，ready_list[0]表示优先级0的任务列表，ready_list[9]表示优先级9的任务列表。
ready_list中的每个列表包含10个TCB项，itme[0]表示该优先级下的任务0，itme[9]表示该优先级下的任务9。
每个TCB项中包含一个标志位和TCB指针，TCB指针指向任务的TCB数据结构。数据结构图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e48173b0c9a44c28a80d230b5bac8de2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)

使用**静态数组**方式的优点是：结构简单，使用方便。但是使用静态数组的缺点非常明显：
1、每个一优先级容纳的任务数量是固定的，一旦需要增加某个优先级任务数量，整个列表大小将增加。
2、在同一个优先级任务中间插入一个任务，需要移动多个TCB项。
3、存在多个优先级未用的情况，导致内存浪费严重。
综合上述问题，因此使用静态数组的方式是不明智的选择。
但是这种数据结构能让我们认清就就绪表的本质：**就绪表分为多个优先级，就绪表的每个优先级可以容纳多个任务。**

构建就绪表可以使用**双向链表**的方式，代码实现如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/da35ac25e7c64ea681ece9cddc9150ca.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

list_item_t列表项，ready_list为就绪表。
就绪表中包含了10个列表，每个列表对应一个优先级，ready_list[0]表示优先级0的任务列表，ready_list[9]表示优先级9的任务列表。
每个list_item_t列表中包含一个TCB指针，下一个列表项指针和上一个列表项指针。数据结构图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/1d3b6904abb345e8a2d1b7398852bc71.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

> 使用双向链表的方式有以下优点：
1、每一个链表可以连接任意数量的链表项，长度不受限制。
2、链表的每一项，都是有用项，不存在内存浪费
3、在链表中间插入一项，操作效率较高。
**使用双向链表构建就绪表是很好的选择**。



**就绪表源码分析**
接下来我们一起来分析FreeRTOS就绪表的源码，源码如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/0e9e9365d4334bdeaf902692c4399590.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

根据源码可知：**FreeRTOS就绪表由链表组成**。
pxReadyTasksLists为就绪表，就绪表中包含10个列表，pxReadyTasksLists[0]表示优先级0的任务列表，pxReadyTasksListst[9]表示优先级9的任务列表。
每个列表包含一个xListEnd项，xListEnd中包含下一个列表项指针和上一个列表项指针。

![在这里插入图片描述](https://img-blog.csdnimg.cn/52aa7426c68742b1aa72b55fc566cf36.png)
FreeRTOS中的TCB结构，根据源码可知TCB内部有名为xStateListItem的列表项（ListItem_t），就绪表中的列表项xListEnd指针指向TCB结构中的列表项。

FreeRTOS的就绪表和任务TCB关系框图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/6f1f685f29854ae889d2467f722b43a6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

**就绪表操作**
操作系统内核总是选择最高优先级的任务运行，相同优先级的任务轮流运行。
针对这种调度原则，内核只需要从就绪表中找出最高优先级任务列表pxReadyTasksLists[n]，运行该列表中第一个列表项指向的任务，任务运行完毕后内核将任务插入列表的尾部，然后内核再运行该任务列表中第一个列表项指向的任务，从而实现轮流运行。新任务加入就绪表时，总是插入到对应优先级列表的尾部。
根据应用要求，内核还可以移除列表中任意位置的列表项。

> 内核操作就绪表有三个基本操作：
1、头部取出，内核总是从最高优先级列表中取出第一个列表项任务运行。
2、尾部插入，任务运行完毕或者新任务加入就绪表时，总是插入到对应优先级列表的尾部。
3、任意位置移除，内核可以移除列表中任意位置的列表项。

FreeRTOS的就绪表操作源码：

![在这里插入图片描述](https://img-blog.csdnimg.cn/02976103403d4f159e970e57127b9da2.png)

> prvAddTaskToReadyList 的功能是将任务加入就绪表， vListInsertEnd 的功能是将任务插入列表尾部。


![在这里插入图片描述](https://img-blog.csdnimg.cn/db0da61caa8e4213b71343dea980623b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)


> taskSELECT_HIGHEST_PRIORITY_TASK的功能是选择最高优先级任务，listGET_OWNER_OF_NEXT_ENTRY 的功能是从该优先级列表中取出第一个列表项。

![在这里插入图片描述](https://img-blog.csdnimg.cn/25b1221a7d42474c8673ec5deb861623.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)

> uxListRemove的功能是在列表中移除指定的一个列表项。
> 
就绪表更新
上一节描述了内核操作就绪表的三个基本操作：**头部取出，尾部插入，任意位置移除**。那么哪些位置会对就绪表进行操作？操作系统会在如下位置更新就绪表（参考FreeRTOS）：
> **xTaskCreate**
xTaskCreate的作用是创建一个任务，该系统函数的调用流程如下：
xTaskCreate -> prvAddNewTaskToReadyList  -> prvAddTaskToReadyList  -> vListInsertEnd 
就绪表变化：任务插入到就绪表对应优先级列表的尾部。
> 

> **vTaskDelete**
vTaskDelete的作用是删除一个任务，该系统函数的调用流程如下：
vTaskDelete -> uxListRemove
就绪表变化：任务从就绪表对应优先级列表中移除



> **vTaskSuspend**
vTaskSuspend的作用是暂停一个任务，该系统函数的调用流程如下：
vTaskSuspend -> uxListRemove
就绪表变化：任务从对应优先级列表中移除

> **vTaskResume**
vTaskResume的作用是恢复一个任务，该系统函数的调用流程如下：
vTaskResume -> prvAddTaskToReadyList -> vListInsertEnd
就绪表变化：任务插入到就绪表中对应优先级列表的尾部。

> **vTaskPrioritySet**
vTaskPrioritySet的作用是改变一个任务优先级，该系统函数的调用流程如下：
vTaskPrioritySet ->uxListRemove -> prvAddTaskToReadyList -> vListInsertEnd
就绪表变化：任务从就绪表中的当前优先级列表中移除，并将任务插入到设定的优先级列表的尾部。

> **vTaskDelay**
vTaskDelay的作用是将当前任务从就绪表中移动到等待表中，该系统函数的调用流程如下：
vTaskDelay -> uxListRemove ->  vListInsert
就绪表变化：任务从就绪表中移除，并将任务插入到等待表中。

> **xQueueSemaphoreTake**
xQueueSemaphoreTake的作用是当前任务等待一个信号，该系统函数的调用流程如下：
xQueueSemaphoreTake ->  vListInsert -> prvAddCurrentTaskToDelayedList -> uxListRemove -> vListInsert
就绪表变化：将当前任务插入到挂起中，并当前任务从就绪表中移除，最后将当前任务插入等待表中。

> **xQueueGenericSend**
xQueueSemaphoreTake的作用是发生一个信号，该系统函数的调用流程如下：
xQueueGenericSend -> xTaskRemoveFromEventList-> uxListRemove -> prvAddTaskToReadyList -> vListInsertEnd
就绪表变化：将移除挂起表中的第一个任务，并该任务插入到就绪表中中对应优先级列表的尾部。

**就绪表框图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/3e32285f008346839b9621ceddf8da6b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_18,color_FFFFFF,t_70,g_se,x_16)


**TCB设计**

增加任务的TCB结构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/057296f37e4a41cfbbdaeb038db2aed3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_19,color_FFFFFF,t_70,g_se,x_16)
>TCB中包含了：
1、栈指针。
2、任务状态列表。
3、任务事件列表。
4、优先级。
5、栈起始地址
6、任务名

TCB结构，寄存器堆和任务栈的关系图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a4498d68a6294dc99c0f2227e5f6c760.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_14,color_FFFFFF,t_70,g_se,x_16)
静态区定义的任务TCB对象中的栈指针指向任务栈区的栈顶，任务栈区保存着寄存器堆的数据，寄存器堆中的PC值指向任务的代码区，任务状态列表和任务事件列表与就绪表，等待表和挂起表关联。

><font color=red>**未完待续…
实时操作系统系列将持续更新
创作不易希望朋友们点赞，转发，评论，关注。
您的点赞，转发，评论，关注将是我持续更新的动力
作者：李巍
Github：liyinuoman2017**
