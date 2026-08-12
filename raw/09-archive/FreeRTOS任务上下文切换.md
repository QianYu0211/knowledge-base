# FreeRTOS任务上下文切换

### 一场精心编排的舞台剧

#### **序幕：舞台设计 \- 职责分离的艺术**

FreeRTOS的任务切换并非一个瞬间动作，而是一场精心编排的戏剧。其设计的核心是**职责分离**：**计时员**只负责报时，**选角导演**只负责选角，而**舞台监督**则负责在幕后将演员和道具（寄存器）精准地搬上搬下。这场戏剧确保了前台表演（任务执行）的流畅，并将复杂的调度工作高效地隐藏在幕后。

#### 第一幕：演员与舞台 \- 核心数据结构

**1\. TCB \(****`tskTCB`****\) \- 演员的“剧本与个人档案”**

每个任务（演员）都有一个TCB，记录了它的一切。

```C
typedef struct tskTaskControlBlock
{
    volatile StackType_t        *pxTopOfStack; /*< 演员下台时，道具存放位置的记录指针。必须是首位。 */
    ListItem_t                 xStateListItem; /*< 一个“挂钩”，决定演员是在“候场区”还是“休息室”。  */
    UBaseType_t                    uxPriority; /*< 演员的“咖位”（优先级）。                      */
    // ... 其他个人信息 ...
} tskTCB;
```

- **源码解读**:

    - `pxTopOfStack`: 档案首页的“道具清单指针”，记录了演员所有个人道具（寄存器上下文）在其后台储物间（私有堆栈）的存放位置。其**首位设计**是为舞台监督（汇编代码）提供最便捷的访问。

    - `xStateListItem`: 演员胸前的“状态牌”。这个“挂钩”挂在哪里，决定了演员的状态。

**2\. 链表 \(****`List_t`**** \& ****`ListItem_t`****\) \- “候场区”与“休息室”**

这是调度剧上演的舞台。

```C++
struct xLIST_ITEM // 状态牌/挂钩
{
    TickType_t xItemValue;                /*< 排序值，如在“休息室”的“唤醒闹钟时间”。        */
    struct xLIST_ITEM * pxNext;           /*< 指向下一个挂钩的指针                         */
    struct xLIST_ITEM * pxPrevious;       /*< 指向前一个挂钩的指针                         */
    void * pvOwner;                       /*< **核心**: 指回持有此状态牌的演员(TCB)。           */
};

typedef struct xLIST // 候场区/休息室
{
    volatile UBaseType_t uxNumberOfItems; /*< 此区域的演员数量。                           */
    ListItem_t * volatile pxIndex;        /*< **核心**: 选角导演的“手指”，指向下一个要考察的演员。*/
    MiniListItem_t xListEnd;              /*< 区域的“边界标志”。                           */
} List_t;
```

- **源码解读**:

    - **嵌入式设计**: `xStateListItem`是演员档案的一部分，而非单独的卡片。演员状态改变时，只需将这个“挂钩”从一个区域（如“休息室”`pxDelayedTaskList`）换到另一个区域（如“候场区”`pxReadyTasksLists`），**零内存开销，极速换场**。

    - `pvOwner`: 确保了从任何一张“状态牌”都能立刻找到它的主人是谁。

    - `pxIndex`: **轮转调度的灵魂**。它不是指向区域的入口，而是导演**上一次选角结束时手指停留的位置**。

#### 第二幕：选角 \- 决策的产生

**1\. ****`xPortSysTickHandler`**** \- 剧院的“计时员”**
每隔一个Tick，准时敲钟，并通知“选角导演”：“时间到，请开始工作！”

**2\. ****`xTaskIncrementTick`**** \- “选角导演”**
导演听到钟声，开始评估所有演员的状态。

- **步骤A: 检查“休息室” \(****`pxDelayedTaskList`****\)**
导演查看休息室里所有设了“唤醒闹钟”的演员。若演员`pxTCB`的闹钟响了：

    - 将其“状态牌”从休息室的挂钩上取下。

    - 调用 `prvAddTaskToReadyList( pxTCB )`，将其安排到“候场区”。

    ```C
    #define prvAddTaskToReadyList( pxTCB )  \
        vListInsertEnd( &( pxReadyTasksLists[ ( pxTCB )->uxPriority ] ), &( ( pxTCB )->xStateListItem ) );
    ```

    - **源码解读**: 演员根据自己的“咖位”（优先级）找到对应的候场区，并自觉排到队伍的**末尾**。

- **步骤B: 提出“换角”建议**
当一个新演员`pxTCB`进入候场区，导演会立即评估：

    ```C
    if( pxTCB->uxPriority >= pxCurrentTCB->uxPriority )
    {
        xSwitchRequired = pdTRUE; // 向舞台监督发出“需要换角”的信号
    }
    ```

- **源码解读**: 如果新来的演员`pxTCB`的咖位**不低于**台上正在表演的演员，导演就会举起一面旗子（`xSwitchRequired = true`），通知舞台监督：“有必要进行一次换角，请准备！”。

- 同理，当**时间片轮转**时，导演也会举起这面旗子。

#### 第三幕：换场 \- 上下文切换的执行

当舞台监督看到“需要换角”的信号旗后，一场精密、底层的换场操作开始了。

**1\. ****`xPortPendSVHandler`**** \- “舞台监督”**
这位是全场最关键的幕后人员，直接操作舞台机关（CPU寄存器）。

- **Part 1: 请当前演员下台（保存上下文）**

```Bash
mrs r0, psp                 ; 找到当前演员的后台储物间(PSP)地址
ldr r2, [r3, =pxCurrentTCB] ; 拿到当前演员的档案(TCB)
stmdb r0!, {r4-r11, r14}    ; 将演员的个人道具(寄存器)一件件放入储物间
str r0, [r2]                ; 在演员档案首页，更新“道具清单指针”(pxTopOfStack)
```

- **源码解读**: 监督精准地将台上演员的所有状态保存到其私有堆栈，并更新TCB记录。

- **Part 2: 咨询导演（调用调度器）**

```Bash
stmdb sp!, {r0, r3}         ; 临时保存自己的工作状态
bl vTaskSwitchContext       ; **关键**: 呼叫导演，等待其最终决定
ldmia sp!, {r0, r3}         ; 恢复自己的工作状态
```

- **源码解读**: 监督暂停自己的工作，将控制权交给导演`vTaskSwitchContext`，等待他更新`pxCurrentTCB`（即选出新演员）。

- **Part 3: 请新演员上台（恢复上下文）**

```Bash
ldr r1, [r3, =pxCurrentTCB] ; 拿到**新**演员的档案(TCB)
ldr r0, [r1]                ; 从档案首页拿到**新**演员的“道具清单指针”
ldmia r0!, {r4-r11, r14}    ; 从新演员的储物间，将道具一一恢复到演员身上
msr psp, r0                 ; 将CPU的储物间指针(PSP)指向新演员的储物间
bx r14                      ; 灯光亮起，大幕拉开，新演员开始表演
```

- **源码解读**: 此处代码与`Part 1`完美对称。在导演做出决定后，监督根据**已被更新的****`pxCurrentTCB`**，精准地为新演员恢复所有状态，完成换场。

**2\. ****`vTaskSwitchContext`**** \& ****`taskSELECT_HIGHEST_PRIORITY_TASK`**** \- “导演”的最终决策**

导演接到监督的呼叫后，执行其核心职责：

```C
void vTaskSwitchContext( void )
 {
     taskSELECT_HIGHEST_PRIORITY_TASK(); // **核心**: "现在，我来选角！"
 }
 
 #define taskSELECT_HIGHEST_PRIORITY_TASK()
 {
     // ... 从最高咖位的候场区开始，向下寻找有演员的队伍 ...
     listGET_OWNER_OF_NEXT_ENTRY( pxCurrentTCB, &( pxReadyTasksLists[ uxTopPriority ] ) );
 }
```

**3\. ****`listGET_OWNER_OF_NEXT_ENTRY`**** \- “导演”选角的手指**
这是导演选择下一个演员的具体动作，也是轮转调度的源码实现：

```C
#define listGET_OWNER_OF_NEXT_ENTRY( pxTCB, pxList )
{
    // 1. 将手指移到下一个演员
    ( pxList )->pxIndex = ( pxList )->pxIndex->pxNext;
    // 2. 如果手指到了队尾标志，就绕回到第一个演员
    if( ( pxList )->pxIndex == &( ( pxList )->xListEnd ) )
    {
        ( pxList )->pxIndex =( pxList )->pxIndex->pxNext;
    }
    // 3. 宣布选择结果
    ( pxTCB ) = ( pxList )->pxIndex->pvOwner;
}
```

- **源码解读**: 导演的手指`pxIndex`在对应优先级的候场队伍中向前移动一步，然后通过`pvOwner`指针指向的演员档案，更新全局的`pxCurrentTCB`。**正是这一步，完成了新旧****`pxCurrentTCB`****的更替**，为舞台监督的后续工作指明了目标。

#### 最终幕：剧终与再开 \- 流程总结

下面以舞台剧的七个连续场景，来总结这场从一个节拍到下一个任务登台的完整流程。

##### 场景一：节拍钟声

剧院的**计时员** \(`SysTick`\) 准时敲响了节拍钟，标志着一个表演时间片的流逝。钟声在整个剧院回响，通知所有幕后人员，时间的齿轮又向前滚动了一格。

##### 场景二：后台甄选

**选角导演** \(`xTaskIncrementTick`\) 闻声而动。他首先检查“休息室”\(`pxDelayedTaskList`\)，将所有“闹钟”响了的演员唤醒。然后，他根据这些演员的“咖位”（优先级），安排他们到各自对应的“候场区”\(`pxReadyTasksLists`\) 的队尾排好。

##### 场景三：换角信号

接着，导演审视全局。如果他发现候场区有演员的咖位**不低于**台上正在表演的演员，或是时候进行公平的轮转，他便会果断地举起一面红色的**“换角”信号旗** \(`xSwitchRequired = true`\)。这是一个明确的信号：舞台需要一位新的主演。

##### 场景四：旧角退场

在舞台最幽暗的角落，一直待命的**舞台监督** \(`PendSV Handler`\) 看到了信号旗。他立刻启动了精密的换场流程。首先，他悄无声息地将台上演员的所有个人道具（寄存器上下文）完整地打包，存入其后台的专属储物间（任务堆栈）。

##### 场景五：导演定角

打包完毕后，舞台监督通过内部通讯系统呼叫导演（`bl vTaskSwitchContext`），请求最终的登场名单。导演迅速地从最高咖位的候场区开始扫视，用他的手指 \(`pxIndex`\) 指向队伍中最靠前的一位，并宣布：“就是他了！”。剧院的**主演名单** \(`pxCurrentTCB`\) 随之更新。

##### 场景六：新角登台

收到新名单后，舞台监督立刻转身，从新主演的储物间中取出所有道具，一丝不苟地为其恢复登场时的状态。一切就绪。

##### 场景七：大幕再起

追光灯亮起，大幕拉开，新的主演（任务）在舞台中央开始了他的表演。一场天衣无缝的切换宣告完成，而台下的观众（用户代码）对此毫无察觉，戏剧仍在流畅地进行。

#### FreeRTOS任务切换流程详解

任务切换触发时机：1\.时间片耗尽（Tick中断）2\.高优先级任务就绪（PendSV中断）3\.主动调用taskYIELD\(\)

关键数据结构：TCB（任务控制块）\-存储任务栈指针、优先级、状态；pxCurrentTCB\-指向当前运行任务的TCB；任务栈\-保存寄存器上下文和局部变量

测试

1\. **保存上下文：** PUSH \{R4\-R11, LR\} // 保存寄存器

2\. **进入临界区：** CPSID I // 关闭中断

5\. **恢复上下文：** POP \{R4\-R11, PC\} // 恢复新任务寄存器

test append

3\. 更新栈指针：保存当前任务SP

4\. 选择下一个任务：加载新任务SP

5\. 恢复上下文：POP \{R4\-R11, PC\}

> （注：部分内容可能由 AI 生成）
