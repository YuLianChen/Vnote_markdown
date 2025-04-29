# x-track UI管理
> `onFocus()`在第一次页面进入显示完成时被`onViewDidAppear()`调用一次，是为了让`group`的第一个`obj`进入`focus`状态并且移动到对应位置。后面的`onFocus()`都是在拨动鼠标滚轮的时候被`group`调用。  
> 感谢！！

最终找到 `group` 回调函数如下：  
[![](https://camo.githubusercontent.com/47cd046b9c5c40fb490b76451230a7dd617ae289e0d2fa25fbec847a7b893d2d/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f7a696c6f6e676d69782f646f632f696d672f32303232303631303133303834302e706e67)](https://camo.githubusercontent.com/47cd046b9c5c40fb490b76451230a7dd617ae289e0d2fa25fbec847a7b893d2d/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f7a696c6f6e676d69782f646f632f696d672f32303232303631303133303834302e706e67)

**页面调度器理解**  
`PageManager` 给用户提供的调度接口一共有三个

1. Pop
2. Push
3. BackHome  
    用户通过手动调用这三个接口，在不同页面之间切换。  
    页面 `Page` 的状态如下

    /* Page state */
    typedef enum
    {
        PAGE_STATE_IDLE,
        PAGE_STATE_LOAD,
        PAGE_STATE_WILL_APPEAR,
        PAGE_STATE_DID_APPEAR,
        PAGE_STATE_ACTIVITY,
        PAGE_STATE_WILL_DISAPPEAR,
        PAGE_STATE_DID_DISAPPEAR,
        PAGE_STATE_UNLOAD,
        _PAGE_STATE_LAST
    } State_t;

不开启页面缓存下的切面调度流程如下图，所有页面都保存在一个 `页面栈` 中，栈顶为当前显示的页面。  
[![](https://camo.githubusercontent.com/962b401bcb1eb78422642dd96ece98f0f3edab36ab295170643b881d24926e5b/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f7a696c6f6e676d69782f646f632f696d672f32303232303631303133353234302e706e67)](https://camo.githubusercontent.com/962b401bcb1eb78422642dd96ece98f0f3edab36ab295170643b881d24926e5b/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f7a696c6f6e676d69782f646f632f696d672f32303232303631303133353234302e706e67)  
如果开启页面缓存，主要是少了 `PAGE_STATE_LOAD` 和 `PAGE_STATE_UNLOAD` 的两个状态，不用每次切换页面时，重新构造组件，但也增加了内存开销。  
这样理解没错吧！