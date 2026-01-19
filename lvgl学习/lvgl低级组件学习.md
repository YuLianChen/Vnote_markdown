# lvgl 低级组件实现学习
## 1. line 组件学习
首先line组件非常简单，只有几个基础函数，在lvgl中一个组件最基础的组件，没有动画，没有交互，适合学习最基础的框架

首先在lv_line的头文件里，能够看到line的调用方法和数据类型
```C 
#ifndef LV_LINE_H
#define LV_LINE_H
头文件宏定义，能够确保只会被生成一次，防止循环调用
```

```c
#ifdef __cplusplus
extern "C" {
#endif
...
#ifdef __cplusplus
} /*extern "C"*/
#endif

头部和尾部的宏定义能够确保如果使用C++能够正常调用
```

### (1)看包含的头文件

`#include "../lv_conf_internal.h"`这个头文件很巧妙的将用户的配置文件安全可靠的引入内部,其内部包含了各种宏定义，以后再看吧

```
用户 lv_conf.h
        ↓
lv_conf_internal.h
        ↓
LVGL 内部所有 .c / .h
```

```c
#if LV_USE_LINE != 0
这个定义是为了设置是否使用这个组件的宏开关

#include "../core/lv_obj.h"
lv_obj.h 包含了obj这个对象的所有属性
```
```c
/*Data of line*/
typedef struct {
    lv_obj_t obj;
    const lv_point_t * point_array;     /**< Pointer to an array with the points of the line*/
    uint16_t point_num;                 /**< Number of points in 'point_array'*/
    uint8_t y_inv : 1;                  /**< 1: y == 0 will be on the bottom*/
} lv_line_t;

如注释所说，这个是目前这个组件的数据结构，其在内存空间的样貌
```

`extern const lv_obj_class_t lv_line_class;`这个结构体是组件的“类描述结构体”，描述了line这个对象拥有的最基础内容，用于事件系统和继承系统，这在整个lvgl组件中都是一样的

剩下的头文件函数中，是一些line的方法，包含创建和设置点等...

### (2)C文件本体
先看头文件
```c
#include "lv_line.h"

#if LV_USE_LINE != 0
#include "../misc/lv_assert.h" //用于提供断言，检查是否正确
#include "../draw/lv_draw.h"    //底层图形绘制功能的接口
#include "../misc/lv_math.h"    //lvgl专用的数学工具
#include <stdbool.h>
#include <stdint.h>
#include <string.h>
```
`#define MY_CLASS &lv_line_class` 将line的基础宏定义，用于别的函数调用

较为关键的基础对象结构体
```c
const lv_obj_class_t lv_line_class = {
    .constructor_cb = lv_line_constructor,
    .event_cb = lv_line_event,
    .width_def = LV_SIZE_CONTENT,
    .height_def = LV_SIZE_CONTENT,
    .instance_size = sizeof(lv_line_t),
    .base_class = &lv_obj_class
};
```
| 字段               | 含义                        |
| ---------------- | ------------------------- |
| `constructor_cb` | **对象创建时初始化私有数据**          |
| `event_cb`       | **组件的核心逻辑（绘制 / 布局 / 事件）** |
| `width_def`      | 默认宽度 = 内容大小               |
| `height_def`     | 默认高度 = 内容大小               |
| `instance_size`  | **对象实例内存大小（含私有结构体）**      |
| `base_class`     | 继承自 `lv_obj`              |

👉 **LVGL 8 没有 C++，但用 class struct + instance_size 实现了 OO**

---

# 三、对象创建流程（LVGL 8 核心套路）

```c
lv_obj_t * lv_line_create(lv_obj_t * parent)
{
    lv_obj_t * obj = lv_obj_class_create_obj(MY_CLASS, parent);
    lv_obj_class_init_obj(obj);
    return obj;
}
```

### 这三步是**固定模板**

1️⃣ 分配内存（含 `lv_line_t`）`lv_obj_class_create_obj`函数中会根据lv_line_class结构体内的instance_size大小来申请内存用于保存对象数据结构
2️⃣ 调用构造函数，做一些初始化动作，如初始化刚申请的内存空间
3️⃣ 返回 `lv_obj_t *`

👉 **以后自定义组件，create 函数 100% 长这样**

---

# 四、setter：组件数据写入（安全 + 刷新）

```c
void lv_line_set_points(lv_obj_t * obj, const lv_point_t points[], uint16_t point_num)
{
    LV_ASSERT_OBJ(obj, MY_CLASS);
```

### 为什么第一行是 `LV_ASSERT_OBJ`

* 防止用户传错对象
* 防止强转野指针
* **开发期直接炸，不留隐患**

---

```c
lv_line_t * line = (lv_line_t *)obj;
```

👉 **LVGL 8 的“继承”关键点**

```
lv_obj_t
└── lv_line_t
```

内存是连续的，所以可以直接强转。

---

```c
line->point_array = points;
line->point_num   = point_num;
```

⚠️ **注意：这里不拷贝数据**

* points 的生命周期由用户负责
* 这是一个 **零拷贝设计**

👉 你自己写组件时要非常清楚这一点

---

```c
lv_obj_refresh_self_size(obj);
lv_obj_invalidate(obj);
```

### 两个动作的意义（非常重要）

| 函数                  | 含义                          |
| ------------------- | --------------------------- |
| `refresh_self_size` | 触发 `LV_EVENT_GET_SELF_SIZE`，刷新布局 |
| `invalidate`        | 触发重绘                        |

---

# 五、构造函数（只做初始化）

```c
static void lv_line_constructor(...)
{
    lv_line_t * line = (lv_line_t *)obj;

    line->point_num   = 0;
    line->point_array = NULL;
    line->y_inv       = 0;
```

👉 构造函数原则：

* **只初始化**
* **不画图**
* **不发事件**

---

```c
lv_obj_clear_flag(obj, LV_OBJ_FLAG_CLICKABLE);
```

👉 `lv_line` 是纯显示组件，不可点击
（减少事件分发开销）

---

# 六、事件函数：组件的“灵魂”

```c
static void lv_line_event(...)
{
    res = lv_obj_event_base(MY_CLASS, e);
    if(res != LV_RES_OK) return;
```

### 这是 **继承链的关键**

* 先让父类处理事件
* 再处理自己的逻辑

👉 **你自己写组件必须照抄**

---

## 1️⃣ `LV_EVENT_REFR_EXT_DRAW_SIZE`

```c
if(code == LV_EVENT_REFR_EXT_DRAW_SIZE) {
```

### 作用

* 告诉 LVGL：**我画的内容可能超出自身区域**

```c
lv_coord_t line_width = lv_obj_get_style_line_width(obj, LV_PART_MAIN);
if(*s < line_width) *s = line_width;
```

👉 防止线条边缘被裁剪

---

## 2️⃣ `LV_EVENT_GET_SELF_SIZE`

```c
else if(code == LV_EVENT_GET_SELF_SIZE) {
```

### 核心作用

👉 **“内容自适应尺寸”的实现**

```c
for(i = 0; i < line->point_num; i++) {
    w = LV_MAX(line->point_array[i].x, w);
    h = LV_MAX(line->point_array[i].y, h);
}
```

* 扫描所有点
* 找最大 x / y

```c
w += line_width;
h += line_width;
```

👉 考虑线宽

---

## 3️⃣ `LV_EVENT_DRAW_MAIN`（最关键）

```c
else if(code == LV_EVENT_DRAW_MAIN) {
```

### 你画 UI 的地方 **只应该在这里**

---

```c
lv_draw_ctx_t * draw_ctx = lv_event_get_draw_ctx(e);
```

👉 **抽象绘图上下文**

* 软件渲染
* DMA
* GPU
* 都在这里统一

---

```c
lv_obj_get_coords(obj, &area);
lv_coord_t x_ofs = area.x1 - lv_obj_get_scroll_x(obj);
lv_coord_t y_ofs = area.y1 - lv_obj_get_scroll_y(obj);
```

👉 正确处理滚动 & 父对象偏移

---

```c
lv_draw_line_dsc_init(&line_dsc);
lv_obj_init_draw_line_dsc(obj, LV_PART_MAIN, &line_dsc);
```

👉 把 style（颜色 / 线宽 / 圆角）转成 draw 参数

---

```c
for(i = 0; i < line->point_num - 1; i++) {
```

👉 折线 = N-1 条线段

---

```c
if(line->y_inv == 0) {
    p1.y = ...
}
else {
    p1.y = h - line->point_array[i].y + y_ofs;
}
```

👉 **Y 轴反转（典型 UI 坐标技巧）**

---

```c
lv_draw_line(draw_ctx, &line_dsc, &p1, &p2);
```

👉 **真正画线的地方**

---

```c
line_dsc.round_start = 0;
```

👉 只在第一段画起点圆角
（细节设计，非常 LVGL）

---

# 七、你应该从这个组件学到什么？

### ✅ 写组件的固定骨架

```
class
 ├─ constructor
 ├─ setter / getter
 └─ event_cb
     ├─ size
     ├─ draw
     └─ extend draw size
```

---

### ✅ 绝对不要犯的错误

❌ 在 setter 里画图
❌ 直接访问 framebuffer
❌ 不调 `lv_obj_event_base()`
❌ 忘记 invalidate / refresh size

---

## 下一步（强烈建议）

如果你愿意，我可以：

1️⃣ **对比 `lv_line` vs `lv_arc`（引入 lv_math）**
2️⃣ **手把手带你写一个最小自定义组件（30 行能跑）**
3️⃣ **专门讲 LV_EVENT_DRAW_MAIN 的性能优化套路**

你现在已经在 **LVGL 进阶门槛之上了**，接下来就是“会不会写好组件”的阶段 👍
