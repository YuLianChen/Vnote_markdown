# LVGL 配色方案设计

**User:**   
**Created:** 4/9/2026 15:15:23  
**Updated:** 4/9/2026 17:25:24  
**Exported:** 4/9/2026 17:29:58  
**Link:** [https://chatgpt.com/c/69d751dd-fc1c-83a6-b377-0e808b9c1f98](https://chatgpt.com/c/69d751dd-fc1c-83a6-b377-0e808b9c1f98)  

## Prompt:
2026/4/9 15:20:20

我的背景是纯黑色的，不要只局限于图片的描述，重新按照美学给我生成一份我使用的色调图

## Response:
2026/4/9 15:20:24

> 已思考若干秒

重新按 **纯黑背景 UI** 的思路给你做了一版，更适合 LVGL 这种嵌入式界面，不再被原图的紫雾氛围绑住：

下载新的色调图 PNG

这版思路是：

*   背景真正用 **纯黑 `#000000`**
*   卡片不用灰太浅，避免发脏
*   高亮色控制在 **紫 / 蓝 / 青 / 品红 / 橙**
*   文字分三级，不用一把全白
*   适合 OLED，也更适合 LVGL 做“少层级、强对比”的界面

直接把你能用的结论给你：

```
# LVGL 纯黑背景 UI 配色方案

## 背景与层级
- 页面背景：#000000
- 主卡片背景：#0B0B0F
- 浮层 / 弹窗 / 抬升层：#14141B
- 细描边 / 分割线：#2A2B35

## 主氛围色
- 主按钮紫：#6C5CFF
- 按下态深紫：#4B3CF0
- 信息蓝：#57C7FF
- 科技青：#2EF2FF

## 状态色
- 成功绿：#2BE66B
- 警告橙：#FF9A1F
- 强调洋红：#FF38D1
- 异常红：#FF4D67

## 文字层级
- 一级文字：#F5F7FF
- 二级文字：#B6BDD3
- 三级文字 / 禁用：#7D8398
- 亮色按钮上的深色字：#0A0A0D
```

我建议你在 LVGL 里这样落：

```
- 整页底色：#000000
- 普通卡片：#0B0B0F
- 当前选中卡片：#14141B
- 主操作按钮：#6C5CFF
- 次级信息图标：#57C7FF
- 小面积辉光：#2EF2FF
- 完成状态：#2BE66B
- 警告状态：#FF9A1F
- 心率/活跃/重点提示：#FF38D1
- 危险/告警：#FF4D67
- 标题数字：#F5F7FF
- 普通说明：#B6BDD3
- 次要说明/禁用：#7D8398
```

再补一个纯黑背景下的审美原则，比较重要：

1.  **不要大面积发光**  
    纯黑底一旦到处发光，会立刻显廉价。只给主按钮、关键图标、焦点环做弱辉光。
2.  **卡片别太亮**  
    纯黑底下卡片只要比背景亮一小档就够了，`#0B0B0F`、`#14141B` 这种就比较稳。
3.  **高饱和色只做点缀**  
    紫、蓝、橙、品红都应该是“小面积冲击”，不能当大片底色。
4.  **禁用态一定要灰蓝，不要纯灰**  
    纯灰会土，`#7D8398` 这种冷灰蓝会更科技。

如果你愿意，我下一步直接给你一份 **LVGL 主题色常量头文件**，比如 `ui_color.h` 那种可直接复制进工程里的版本。

## Prompt:
2026/4/9 15:30:04

给我一份lvgl 头文件，同时帮我思考如何导入figma

## Response:
2026/4/9 15:30:07

> 已思考 4s

下面给你一份可直接放进项目里的 **LVGL 主题头文件**，配色就是按你这套 **纯黑背景 + 科技霓虹点缀** 来的。

```
#ifndef UI_COLOR_H
#define UI_COLOR_H

#include "lvgl.h"

#ifdef __cplusplus
extern "C" {
#endif

/* =========================================================
 * LVGL Pure Black Neon Theme
 * 适用于：纯黑背景 / 科技感 / 嵌入式 UI
 * 建议：
 * 1. 页面背景尽量保持纯黑
 * 2. 发光色只小面积使用
 * 3. 大块面板不要太亮，否则会脏
 * ========================================================= */

/* =========================
 * Base / Surface
 * ========================= */
#define UI_COLOR_BG               lv_color_hex(0x000000)  /* 页面纯黑背景 */
#define UI_COLOR_SURFACE_1        lv_color_hex(0x0B0B0F)  /* 主卡片背景 */
#define UI_COLOR_SURFACE_2        lv_color_hex(0x14141B)  /* 浮层/高亮卡片 */
#define UI_COLOR_SURFACE_3        lv_color_hex(0x1C1D26)  /* 更强层级，可选 */
#define UI_COLOR_BORDER           lv_color_hex(0x2A2B35)  /* 细描边/分割线 */

/* =========================
 * Brand / Accent
 * ========================= */
#define UI_COLOR_PRIMARY          lv_color_hex(0x6C5CFF)  /* 主紫 */
#define UI_COLOR_PRIMARY_PRESS    lv_color_hex(0x4B3CF0)  /* 按下态深紫 */
#define UI_COLOR_BLUE             lv_color_hex(0x57C7FF)  /* 信息蓝 */
#define UI_COLOR_CYAN             lv_color_hex(0x2EF2FF)  /* 科技青 */
#define UI_COLOR_MAGENTA          lv_color_hex(0xFF38D1)  /* 洋红强调 */

/* =========================
 * Status
 * ========================= */
#define UI_COLOR_SUCCESS          lv_color_hex(0x2BE66B)  /* 成功 */
#define UI_COLOR_WARNING          lv_color_hex(0xFF9A1F)  /* 警告/热量 */
#define UI_COLOR_DANGER           lv_color_hex(0xFF4D67)  /* 错误/告警 */
#define UI_COLOR_GOLD             lv_color_hex(0xF6C445)  /* 目标/统计，可选 */

/* =========================
 * Text
 * ========================= */
#define UI_COLOR_TEXT_PRIMARY     lv_color_hex(0xF5F7FF)  /* 一级文字 */
#define UI_COLOR_TEXT_SECONDARY   lv_color_hex(0xB6BDD3)  /* 二级文字 */
#define UI_COLOR_TEXT_MUTED       lv_color_hex(0x7D8398)  /* 三级/禁用 */
#define UI_COLOR_TEXT_ON_BRIGHT   lv_color_hex(0x0A0A0D)  /* 亮底上的文字 */

/* =========================
 * Opacity helpers
 * ========================= */
#define UI_OPA_TRANSPARENT        LV_OPA_TRANSP
#define UI_OPA_10                 LV_OPA_10
#define UI_OPA_20                 LV_OPA_20
#define UI_OPA_30                 LV_OPA_30
#define UI_OPA_40                 LV_OPA_40
#define UI_OPA_50                 LV_OPA_50
#define UI_OPA_60                 LV_OPA_60
#define UI_OPA_70                 LV_OPA_70
#define UI_OPA_80                 LV_OPA_80
#define UI_OPA_90                 LV_OPA_90
#define UI_OPA_100                LV_OPA_100

/* =========================================================
 * 常用样式初始化函数
 * 你可以在 ui_theme.c 里实现，也可以直接放项目公共文件
 * ========================================================= */

static inline void ui_style_init_screen(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_BG);
    lv_style_set_text_color(style, UI_COLOR_TEXT_PRIMARY);
}

static inline void ui_style_init_card(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_radius(style, 20);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_SURFACE_1);
    lv_style_set_border_width(style, 1);
    lv_style_set_border_color(style, UI_COLOR_BORDER);
    lv_style_set_border_opa(style, LV_OPA_60);
    lv_style_set_pad_all(style, 16);
    lv_style_set_text_color(style, UI_COLOR_TEXT_PRIMARY);
}

static inline void ui_style_init_card_active(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_radius(style, 20);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_SURFACE_2);
    lv_style_set_border_width(style, 1);
    lv_style_set_border_color(style, UI_COLOR_PRIMARY);
    lv_style_set_border_opa(style, LV_OPA_70);
    lv_style_set_pad_all(style, 16);
    lv_style_set_text_color(style, UI_COLOR_TEXT_PRIMARY);
}

static inline void ui_style_init_btn_primary(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_radius(style, 18);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_PRIMARY);
    lv_style_set_border_width(style, 0);
    lv_style_set_text_color(style, UI_COLOR_TEXT_PRIMARY);
    lv_style_set_shadow_width(style, 18);
    lv_style_set_shadow_color(style, UI_COLOR_PRIMARY);
    lv_style_set_shadow_opa(style, LV_OPA_30);
    lv_style_set_shadow_spread(style, 1);
}

static inline void ui_style_init_btn_primary_pressed(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_radius(style, 18);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_PRIMARY_PRESS);
    lv_style_set_border_width(style, 0);
    lv_style_set_text_color(style, UI_COLOR_TEXT_PRIMARY);
    lv_style_set_shadow_width(style, 12);
    lv_style_set_shadow_color(style, UI_COLOR_PRIMARY_PRESS);
    lv_style_set_shadow_opa(style, LV_OPA_20);
}

static inline void ui_style_init_btn_secondary(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_radius(style, 18);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_SURFACE_2);
    lv_style_set_border_width(style, 1);
    lv_style_set_border_color(style, UI_COLOR_BORDER);
    lv_style_set_text_color(style, UI_COLOR_TEXT_SECONDARY);
}

static inline void ui_style_init_label_title(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_text_color(style, UI_COLOR_TEXT_PRIMARY);
}

static inline void ui_style_init_label_body(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_text_color(style, UI_COLOR_TEXT_SECONDARY);
}

static inline void ui_style_init_label_muted(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_text_color(style, UI_COLOR_TEXT_MUTED);
}

static inline void ui_style_init_line_divider(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_line_color(style, UI_COLOR_BORDER);
    lv_style_set_line_opa(style, LV_OPA_50);
    lv_style_set_line_width(style, 1);
}

static inline void ui_style_init_status_success(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_SUCCESS);
    lv_style_set_text_color(style, UI_COLOR_TEXT_ON_BRIGHT);
}

static inline void ui_style_init_status_warning(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_WARNING);
    lv_style_set_text_color(style, UI_COLOR_TEXT_ON_BRIGHT);
}

static inline void ui_style_init_status_danger(lv_style_t *style)
{
    lv_style_init(style);
    lv_style_set_bg_opa(style, LV_OPA_COVER);
    lv_style_set_bg_color(style, UI_COLOR_DANGER);
    lv_style_set_text_color(style, UI_COLOR_TEXT_PRIMARY);
}

#ifdef __cplusplus
} /* extern "C" */
#endif

#endif /* UI_COLOR_H */
```

你如果想更工程化一点，建议拆成两层：

第一层是 **颜色常量**  
第二层是 **业务语义别名**

也就是别在页面里直接写 `UI_COLOR_PRIMARY`，而是再封一层：

```
#define COLOR_PAGE_BG          UI_COLOR_BG
#define COLOR_CARD_BG          UI_COLOR_SURFACE_1
#define COLOR_CARD_ACTIVE      UI_COLOR_SURFACE_2
#define COLOR_BTN_MAIN         UI_COLOR_PRIMARY
#define COLOR_BTN_MAIN_PRESS   UI_COLOR_PRIMARY_PRESS
#define COLOR_TEXT_TITLE       UI_COLOR_TEXT_PRIMARY
#define COLOR_TEXT_BODY        UI_COLOR_TEXT_SECONDARY
#define COLOR_TEXT_DISABLED    UI_COLOR_TEXT_MUTED
#define COLOR_STATE_OK         UI_COLOR_SUCCESS
#define COLOR_STATE_WARN       UI_COLOR_WARNING
#define COLOR_STATE_ERR        UI_COLOR_DANGER
```

这样以后你整体换主题，只改映射层就够了。

* * *

关于怎么导入 Figma
------------

这里我先给你一个最实用的思路：  
**不是把 LVGL 头文件“导入”进 Figma，而是把这套颜色先整理成 Figma 的颜色样式，再拿 Figma 做视觉稿，最后再回填到 LVGL。**

更顺的流程是这个：

### 方案一：先定主题，再进 Figma

最推荐。

流程：

1.  先确定你的主题色表  
    就是我上面给你的这套颜色。
2.  在 Figma 里建立 Color Styles  
    命名建议直接和工程对齐：

```
bg/page
surface/card
surface/raised
border/default
text/primary
text/secondary
text/muted
accent/primary
accent/blue
accent/cyan
status/success
status/warning
status/danger
accent/magenta
```

3.  做组件时只用这些 Style，不手填颜色
4.  最终再导出一份对照表，回填到 `ui_color.h`

这套方式的优点是：

*   Figma 和代码命名统一
*   后续改主题很轻松
*   设计和开发不会跑偏

* * *

### 方案二：把头文件转成 Figma Tokens

如果你后面会经常做主题维护，这个更专业。

思路是把颜色整理成一份 token 文件，例如 JSON：

```
{
  "color": {
    "bg": {
      "page": { "value": "#000000" }
    },
    "surface": {
      "card": { "value": "#0B0B0F" },
      "raised": { "value": "#14141B" }
    },
    "text": {
      "primary": { "value": "#F5F7FF" },
      "secondary": { "value": "#B6BDD3" },
      "muted": { "value": "#7D8398" }
    },
    "accent": {
      "primary": { "value": "#6C5CFF" },
      "blue": { "value": "#57C7FF" },
      "cyan": { "value": "#2EF2FF" },
      "magenta": { "value": "#FF38D1" }
    },
    "status": {
      "success": { "value": "#2BE66B" },
      "warning": { "value": "#FF9A1F" },
      "danger": { "value": "#FF4D67" }
    }
  }
}
```

然后在 Figma 里通过 Tokens 类插件或变量体系导入。

这个适合你后面：

*   有多套主题
*   需要亮暗切换
*   想让设计稿和代码长期同步

* * *

### 方案三：最省事的土办法

适合你现在马上开工。

1.  我这边先给你颜色表
2.  你在 Figma 手动建立一组本地颜色样式
3.  组件里统一引用这些样式
4.  开发时用 `ui_color.h` 对齐

这个最快，不容易折腾偏。

* * *

我建议你的实际工作流
----------

你现在做的是 **LVGL 嵌入式 UI**，所以不要走纯视觉设计那套复杂链路，最稳的是：

### 第一步：先把设计约束定死

在 Figma 先定这几类基础 token：

*   背景
*   卡片
*   边框
*   主文字
*   次文字
*   禁用文字
*   主强调色
*   信息色
*   成功色
*   警告色
*   危险色

### 第二步：做最小组件集

在 Figma 先只做这些组件：

*   页面底板
*   普通卡片
*   选中卡片
*   主按钮
*   次按钮
*   标题文字
*   说明文字
*   状态标签
*   圆环进度
*   图标按钮

### 第三步：再回到 LVGL

不要一上来就按页面写，先写“样式函数 + 基础组件”。

也就是 LVGL 里优先做：

*   `ui_style_init_card`
*   `ui_style_init_btn_primary`
*   `ui_style_init_label_title`
*   `ui_style_init_status_success`

这样后面页面搭起来很快，而且不会越做越乱。

* * *

你这个项目里，Figma 需要注意的点
-------------------

因为你是 LVGL，不是手机 App，Figma 里要主动限制自己：

### 1\. 不要做太重的模糊

LVGL 很难低成本复刻大面积毛玻璃。

建议：

*   用深色面板
*   用轻微描边
*   用弱阴影
*   用局部辉光替代毛玻璃

### 2\. 不要做太多透明叠层

嵌入式设备里透明叠加多了容易吃性能，也难还原。

### 3\. 渐变少而精

建议只在下面这些地方用渐变：

*   主按钮
*   圆环进度
*   活跃卡片
*   小面积氛围光

### 4\. 阴影尽量“短”

不要 Figma 里那种很大的柔阴影，LVGL 里复刻会很重。

* * *

最后给你一个很实用的命名建议
--------------

Figma 和 LVGL 统一命名，不然后面一定乱。

建议统一成这套：

```
color.bg.page
color.surface.card
color.surface.raised
color.border.default
color.text.primary
color.text.secondary
color.text.muted
color.accent.primary
color.accent.blue
color.accent.cyan
color.accent.magenta
color.status.success
color.status.warning
color.status.danger
```

对应到 C 宏：

```
UI_COLOR_BG
UI_COLOR_SURFACE_1
UI_COLOR_SURFACE_2
UI_COLOR_BORDER
UI_COLOR_TEXT_PRIMARY
UI_COLOR_TEXT_SECONDARY
UI_COLOR_TEXT_MUTED
UI_COLOR_PRIMARY
UI_COLOR_BLUE
UI_COLOR_CYAN
UI_COLOR_MAGENTA
UI_COLOR_SUCCESS
UI_COLOR_WARNING
UI_COLOR_DANGER
```

这样设计稿和代码可以一眼对上。

下一步我可以直接继续给你一份 **Figma 颜色变量表 + 字体层级表**，你可以直接照着建。

## Prompt:
2026/4/9 15:51:05

推荐一个figma插件以方便我导入吧

## Response:
2026/4/9 15:51:07

> **寻找Figma相关的答案**
>
> 搜索结果不太理想。我在想也许可以尝试使用Figma的API工具？不过这个问题比较简单，可能只需要从Figma官网或者社区页面搜索一下，特别是查找插件页面，甚至通过直接的网络搜索就能找到答案。
>
> **寻找Tokens Studio官方文档**
>
> 我需要一个合理的答案，并且要有可靠的引用来源。既然我们使用了网络查询，所有的互联网信息都应有引用。我考虑过可能去搜索“Tokens Studio”相关的官方文档，尤其是尝试通过tokens.studio的官网，也许可以找到具体的资料。
>
> 已思考 21s

推荐你直接用 **Tokens Studio for Figma**。

它最适合你现在这个场景：你已经有一套 LVGL 颜色常量，下一步想把它们整理成可复用的设计变量，再在 Figma 里统一管理。Tokens Studio 本身就是给 Figma 做 design tokens 用的，支持把 token 以 JSON 方式管理，还能在设计里应用这些 token。官方文档明确说明它是一个 Figma 插件，用来把 token 集成进设计，并且 token 可以覆盖颜色、字体、圆角、间距等；它也支持 JSON 视图，把 Token Set 当成 JSON 来编辑。[Tokens Studio+2Tokens Studio+2](https://docs.tokens.studio/?utm_source=chatgpt.com)

你现在最实用的用法是这样：

先把我之前给你的颜色整理成一份 JSON，例如：

```
{
  "color": {
    "bg": {
      "page": { "value": "#000000", "type": "color" }
    },
    "surface": {
      "card": { "value": "#0B0B0F", "type": "color" },
      "raised": { "value": "#14141B", "type": "color" }
    },
    "text": {
      "primary": { "value": "#F5F7FF", "type": "color" },
      "secondary": { "value": "#B6BDD3", "type": "color" },
      "muted": { "value": "#7D8398", "type": "color" }
    },
    "accent": {
      "primary": { "value": "#6C5CFF", "type": "color" },
      "blue": { "value": "#57C7FF", "type": "color" },
      "cyan": { "value": "#2EF2FF", "type": "color" },
      "magenta": { "value": "#FF38D1", "type": "color" }
    },
    "status": {
      "success": { "value": "#2BE66B", "type": "color" },
      "warning": { "value": "#FF9A1F", "type": "color" },
      "danger": { "value": "#FF4D67", "type": "color" }
    }
  }
}
```

然后在 Figma 里装好 Tokens Studio，打开插件后直接用它的 token / JSON 工作流导入和管理。官方安装说明写明，这个插件可以直接从 Figma Community 保存到账号后在任意文件里打开使用。[Tokens Studio+1](https://docs.tokens.studio/get-started/install-figma-plugin?utm_source=chatgpt.com)

我不太建议你现在优先找那种“单纯导入颜色样式”的小插件。原因是你后面大概率不只是导颜色，还会继续扩展到字体层级、圆角、间距、组件状态色。Tokens Studio 在这方面更完整，而且还能把 Figma 变量和 token 体系互相衔接。官方文档里也提到它支持从 Figma Variables 和 Styles 导入到 Tokens Studio。[Tokens Studio+2GitHub+2](https://docs.tokens.studio/figma/import/variables?utm_source=chatgpt.com)

给你一个直接结论：

*   **现在就装：Tokens Studio for Figma**
*   **适合你原因：** 能导入/维护 JSON token，后面还能继续扩展字体、间距、圆角
*   **最适合你的工作流：** `LVGL 头文件颜色 -> JSON token -> Tokens Studio -> Figma Variables / Styles`

如果你要，我下一条直接给你一份 **可直接导入 Tokens Studio 的完整 JSON 文件**，把你这套纯黑 LVGL 主题一次性配好。

---

**Sources:**

- [Tokens Studio for Figma: Tokens Studio Plugin for Figma](https://docs.tokens.studio/?utm_source=chatgpt.com)

## Prompt:
2026/4/9 16:31:05

给我一个可直接导入 Tokens Studio 的完整 JSON 文件

## Response:
2026/4/9 16:31:09

> 已思考几秒

可以，下面这份就是 **可直接导入 Tokens Studio for Figma** 的完整 JSON。  
我按你现在的方向配成了：

*   **纯黑背景**
*   **适合 LVGL 的科技感暗色主题**
*   包含 **颜色 / 圆角 / 间距 / 描边 / 阴影 / 字体层级**
*   命名尽量和你后面写 `ui_color.h` 时能对上

你直接复制成一个 `lvgl-dark-neon.tokens.json` 就行。

```
{
  "global": {
    "color": {
      "bg": {
        "page": {
          "value": "#000000",
          "type": "color",
          "description": "整页纯黑背景"
        }
      },
      "surface": {
        "card": {
          "value": "#0B0B0F",
          "type": "color",
          "description": "普通卡片背景"
        },
        "raised": {
          "value": "#14141B",
          "type": "color",
          "description": "浮层/选中卡片背景"
        },
        "raisedStrong": {
          "value": "#1C1D26",
          "type": "color",
          "description": "更强层级背景"
        }
      },
      "border": {
        "default": {
          "value": "#2A2B35",
          "type": "color",
          "description": "默认边框/分割线"
        },
        "soft": {
          "value": "#343646",
          "type": "color",
          "description": "偏亮一点的弱描边"
        },
        "active": {
          "value": "#6C5CFF",
          "type": "color",
          "description": "激活描边"
        }
      },
      "text": {
        "primary": {
          "value": "#F5F7FF",
          "type": "color",
          "description": "一级文字"
        },
        "secondary": {
          "value": "#B6BDD3",
          "type": "color",
          "description": "二级文字"
        },
        "muted": {
          "value": "#7D8398",
          "type": "color",
          "description": "三级文字/禁用"
        },
        "disabled": {
          "value": "#5C6173",
          "type": "color",
          "description": "更弱禁用文字"
        },
        "onBright": {
          "value": "#0A0A0D",
          "type": "color",
          "description": "亮色按钮上的深色字"
        }
      },
      "accent": {
        "primary": {
          "value": "#6C5CFF",
          "type": "color",
          "description": "主强调紫"
        },
        "primaryPressed": {
          "value": "#4B3CF0",
          "type": "color",
          "description": "主按钮按下态"
        },
        "blue": {
          "value": "#57C7FF",
          "type": "color",
          "description": "信息蓝"
        },
        "cyan": {
          "value": "#2EF2FF",
          "type": "color",
          "description": "科技青"
        },
        "magenta": {
          "value": "#FF38D1",
          "type": "color",
          "description": "洋红强调"
        },
        "gold": {
          "value": "#F6C445",
          "type": "color",
          "description": "目标/统计金色"
        }
      },
      "status": {
        "success": {
          "value": "#2BE66B",
          "type": "color",
          "description": "成功"
        },
        "warning": {
          "value": "#FF9A1F",
          "type": "color",
          "description": "警告"
        },
        "danger": {
          "value": "#FF4D67",
          "type": "color",
          "description": "错误/危险"
        },
        "info": {
          "value": "#57C7FF",
          "type": "color",
          "description": "信息"
        }
      },
      "chart": {
        "ringStart": {
          "value": "#6C5CFF",
          "type": "color",
          "description": "环形图渐变起点"
        },
        "ringMid": {
          "value": "#57C7FF",
          "type": "color",
          "description": "环形图中段"
        },
        "ringEnd": {
          "value": "#2EF2FF",
          "type": "color",
          "description": "环形图渐变终点"
        },
        "track": {
          "value": "#242632",
          "type": "color",
          "description": "图表轨道底色"
        }
      },
      "overlay": {
        "scrim": {
          "value": "#000000CC",
          "type": "color",
          "description": "遮罩层"
        },
        "glowPurple": {
          "value": "#6C5CFF33",
          "type": "color",
          "description": "紫色弱辉光"
        },
        "glowBlue": {
          "value": "#57C7FF2E",
          "type": "color",
          "description": "蓝色弱辉光"
        },
        "glowCyan": {
          "value": "#2EF2FF26",
          "type": "color",
          "description": "青色弱辉光"
        }
      }
    },
    "radius": {
      "none": {
        "value": "0",
        "type": "borderRadius"
      },
      "xs": {
        "value": "6",
        "type": "borderRadius"
      },
      "sm": {
        "value": "10",
        "type": "borderRadius"
      },
      "md": {
        "value": "14",
        "type": "borderRadius"
      },
      "lg": {
        "value": "18",
        "type": "borderRadius"
      },
      "xl": {
        "value": "22",
        "type": "borderRadius"
      },
      "2xl": {
        "value": "28",
        "type": "borderRadius"
      },
      "pill": {
        "value": "999",
        "type": "borderRadius"
      }
    },
    "spacing": {
      "0": {
        "value": "0",
        "type": "spacing"
      },
      "1": {
        "value": "4",
        "type": "spacing"
      },
      "2": {
        "value": "8",
        "type": "spacing"
      },
      "3": {
        "value": "12",
        "type": "spacing"
      },
      "4": {
        "value": "16",
        "type": "spacing"
      },
      "5": {
        "value": "20",
        "type": "spacing"
      },
      "6": {
        "value": "24",
        "type": "spacing"
      },
      "7": {
        "value": "28",
        "type": "spacing"
      },
      "8": {
        "value": "32",
        "type": "spacing"
      },
      "10": {
        "value": "40",
        "type": "spacing"
      },
      "12": {
        "value": "48",
        "type": "spacing"
      }
    },
    "size": {
      "iconXs": {
        "value": "12",
        "type": "sizing"
      },
      "iconSm": {
        "value": "16",
        "type": "sizing"
      },
      "iconMd": {
        "value": "20",
        "type": "sizing"
      },
      "iconLg": {
        "value": "24",
        "type": "sizing"
      },
      "iconXl": {
        "value": "32",
        "type": "sizing"
      }
    },
    "stroke": {
      "none": {
        "value": "0",
        "type": "borderWidth"
      },
      "thin": {
        "value": "1",
        "type": "borderWidth"
      },
      "regular": {
        "value": "2",
        "type": "borderWidth"
      }
    },
    "opacity": {
      "0": {
        "value": "0",
        "type": "opacity"
      },
      "10": {
        "value": "0.1",
        "type": "opacity"
      },
      "20": {
        "value": "0.2",
        "type": "opacity"
      },
      "30": {
        "value": "0.3",
        "type": "opacity"
      },
      "40": {
        "value": "0.4",
        "type": "opacity"
      },
      "50": {
        "value": "0.5",
        "type": "opacity"
      },
      "60": {
        "value": "0.6",
        "type": "opacity"
      },
      "70": {
        "value": "0.7",
        "type": "opacity"
      },
      "80": {
        "value": "0.8",
        "type": "opacity"
      },
      "90": {
        "value": "0.9",
        "type": "opacity"
      },
      "100": {
        "value": "1",
        "type": "opacity"
      }
    },
    "shadow": {
      "card": {
        "value": {
          "x": "0",
          "y": "10",
          "blur": "30",
          "spread": "0",
          "color": "#00000066"
        },
        "type": "boxShadow",
        "description": "卡片阴影"
      },
      "purpleGlow": {
        "value": {
          "x": "0",
          "y": "0",
          "blur": "24",
          "spread": "0",
          "color": "#6C5CFF55"
        },
        "type": "boxShadow",
        "description": "紫色辉光"
      },
      "blueGlow": {
        "value": {
          "x": "0",
          "y": "0",
          "blur": "22",
          "spread": "0",
          "color": "#57C7FF44"
        },
        "type": "boxShadow",
        "description": "蓝色辉光"
      },
      "cyanGlow": {
        "value": {
          "x": "0",
          "y": "0",
          "blur": "20",
          "spread": "0",
          "color": "#2EF2FF33"
        },
        "type": "boxShadow",
        "description": "青色辉光"
      }
    },
    "fontFamily": {
      "primary": {
        "value": "Inter",
        "type": "fontFamilies",
        "description": "Figma 用；LVGL 里换成你的实际字体"
      },
      "mono": {
        "value": "JetBrains Mono",
        "type": "fontFamilies"
      }
    },
    "fontWeight": {
      "regular": {
        "value": "400",
        "type": "fontWeights"
      },
      "medium": {
        "value": "500",
        "type": "fontWeights"
      },
      "semibold": {
        "value": "600",
        "type": "fontWeights"
      },
      "bold": {
        "value": "700",
        "type": "fontWeights"
      }
    },
    "lineHeight": {
      "tight": {
        "value": "1.1",
        "type": "lineHeights"
      },
      "normal": {
        "value": "1.3",
        "type": "lineHeights"
      },
      "relaxed": {
        "value": "1.5",
        "type": "lineHeights"
      }
    },
    "fontSize": {
      "xs": {
        "value": "10",
        "type": "fontSizes"
      },
      "sm": {
        "value": "12",
        "type": "fontSizes"
      },
      "md": {
        "value": "14",
        "type": "fontSizes"
      },
      "lg": {
        "value": "16",
        "type": "fontSizes"
      },
      "xl": {
        "value": "20",
        "type": "fontSizes"
      },
      "2xl": {
        "value": "24",
        "type": "fontSizes"
      },
      "3xl": {
        "value": "32",
        "type": "fontSizes"
      },
      "4xl": {
        "value": "40",
        "type": "fontSizes"
      }
    },
    "typography": {
      "titleHero": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.bold}",
          "lineHeight": "{global.lineHeight.tight}",
          "fontSize": "{global.fontSize.4xl}"
        },
        "type": "typography"
      },
      "title1": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.bold}",
          "lineHeight": "{global.lineHeight.tight}",
          "fontSize": "{global.fontSize.3xl}"
        },
        "type": "typography"
      },
      "title2": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.semibold}",
          "lineHeight": "{global.lineHeight.normal}",
          "fontSize": "{global.fontSize.2xl}"
        },
        "type": "typography"
      },
      "title3": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.semibold}",
          "lineHeight": "{global.lineHeight.normal}",
          "fontSize": "{global.fontSize.xl}"
        },
        "type": "typography"
      },
      "body": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.regular}",
          "lineHeight": "{global.lineHeight.relaxed}",
          "fontSize": "{global.fontSize.lg}"
        },
        "type": "typography"
      },
      "bodySmall": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.regular}",
          "lineHeight": "{global.lineHeight.normal}",
          "fontSize": "{global.fontSize.md}"
        },
        "type": "typography"
      },
      "caption": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.medium}",
          "lineHeight": "{global.lineHeight.normal}",
          "fontSize": "{global.fontSize.sm}"
        },
        "type": "typography"
      },
      "metric": {
        "value": {
          "fontFamily": "{global.fontFamily.primary}",
          "fontWeight": "{global.fontWeight.bold}",
          "lineHeight": "{global.lineHeight.tight}",
          "fontSize": "{global.fontSize.3xl}"
        },
        "type": "typography"
      },
      "code": {
        "value": {
          "fontFamily": "{global.fontFamily.mono}",
          "fontWeight": "{global.fontWeight.medium}",
          "lineHeight": "{global.lineHeight.normal}",
          "fontSize": "{global.fontSize.md}"
        },
        "type": "typography"
      }
    }
  },
  "semantic": {
    "color": {
      "page": {
        "bg": {
          "value": "{global.color.bg.page}",
          "type": "color"
        }
      },
      "card": {
        "bg": {
          "value": "{global.color.surface.card}",
          "type": "color"
        },
        "bgActive": {
          "value": "{global.color.surface.raised}",
          "type": "color"
        },
        "border": {
          "value": "{global.color.border.default}",
          "type": "color"
        }
      },
      "button": {
        "primaryBg": {
          "value": "{global.color.accent.primary}",
          "type": "color"
        },
        "primaryBgPressed": {
          "value": "{global.color.accent.primaryPressed}",
          "type": "color"
        },
        "primaryText": {
          "value": "{global.color.text.primary}",
          "type": "color"
        },
        "secondaryBg": {
          "value": "{global.color.surface.raised}",
          "type": "color"
        },
        "secondaryBorder": {
          "value": "{global.color.border.default}",
          "type": "color"
        },
        "secondaryText": {
          "value": "{global.color.text.secondary}",
          "type": "color"
        }
      },
      "text": {
        "title": {
          "value": "{global.color.text.primary}",
          "type": "color"
        },
        "body": {
          "value": "{global.color.text.secondary}",
          "type": "color"
        },
        "muted": {
          "value": "{global.color.text.muted}",
          "type": "color"
        },
        "disabled": {
          "value": "{global.color.text.disabled}",
          "type": "color"
        }
      },
      "icon": {
        "default": {
          "value": "{global.color.text.secondary}",
          "type": "color"
        },
        "active": {
          "value": "{global.color.accent.blue}",
          "type": "color"
        },
        "accent": {
          "value": "{global.color.accent.cyan}",
          "type": "color"
        }
      },
      "status": {
        "success": {
          "value": "{global.color.status.success}",
          "type": "color"
        },
        "warning": {
          "value": "{global.color.status.warning}",
          "type": "color"
        },
        "danger": {
          "value": "{global.color.status.danger}",
          "type": "color"
        },
        "info": {
          "value": "{global.color.status.info}",
          "type": "color"
        }
      }
    },
    "radius": {
      "card": {
        "value": "{global.radius.xl}",
        "type": "borderRadius"
      },
      "button": {
        "value": "{global.radius.lg}",
        "type": "borderRadius"
      },
      "chip": {
        "value": "{global.radius.pill}",
        "type": "borderRadius"
      }
    },
    "spacing": {
      "pagePadding": {
        "value": "{global.spacing.6}",
        "type": "spacing"
      },
      "cardPadding": {
        "value": "{global.spacing.4}",
        "type": "spacing"
      },
      "buttonPaddingX": {
        "value": "{global.spacing.5}",
        "type": "spacing"
      },
      "buttonPaddingY": {
        "value": "{global.spacing.3}",
        "type": "spacing"
      },
      "sectionGap": {
        "value": "{global.spacing.6}",
        "type": "spacing"
      },
      "itemGap": {
        "value": "{global.spacing.3}",
        "type": "spacing"
      }
    },
    "shadow": {
      "card": {
        "value": "{global.shadow.card}",
        "type": "boxShadow"
      },
      "primaryGlow": {
        "value": "{global.shadow.purpleGlow}",
        "type": "boxShadow"
      },
      "infoGlow": {
        "value": "{global.shadow.blueGlow}",
        "type": "boxShadow"
      },
      "accentGlow": {
        "value": "{global.shadow.cyanGlow}",
        "type": "boxShadow"
      }
    },
    "typography": {
      "screenTitle": {
        "value": "{global.typography.title1}",
        "type": "typography"
      },
      "cardTitle": {
        "value": "{global.typography.title3}",
        "type": "typography"
      },
      "body": {
        "value": "{global.typography.body}",
        "type": "typography"
      },
      "bodySmall": {
        "value": "{global.typography.bodySmall}",
        "type": "typography"
      },
      "caption": {
        "value": "{global.typography.caption}",
        "type": "typography"
      },
      "metric": {
        "value": "{global.typography.metric}",
        "type": "typography"
      }
    }
  }
}
```

导入建议：

1.  打开 Tokens Studio
2.  新建一个 Token Set
3.  选择 Import JSON
4.  把上面整段粘进去
5.  先启用 `global`
6.  再启用 `semantic`

如果你后面想和 `ui_color.h` 完全对齐，我建议你最终只在业务里使用 `semantic` 这一层，因为它更像 UI 语义，不会把页面写死在具体颜色上。

再给你一份 **和这份 JSON 对应的 C 宏命名建议**，这样你 Figma 和 LVGL 更容易统一：

```
#define UI_COLOR_BG                0x000000
#define UI_COLOR_SURFACE_1         0x0B0B0F
#define UI_COLOR_SURFACE_2         0x14141B
#define UI_COLOR_BORDER            0x2A2B35

#define UI_COLOR_PRIMARY           0x6C5CFF
#define UI_COLOR_PRIMARY_PRESS     0x4B3CF0
#define UI_COLOR_BLUE              0x57C7FF
#define UI_COLOR_CYAN              0x2EF2FF
#define UI_COLOR_MAGENTA           0xFF38D1

#define UI_COLOR_SUCCESS           0x2BE66B
#define UI_COLOR_WARNING           0xFF9A1F
#define UI_COLOR_DANGER            0xFF4D67

#define UI_COLOR_TEXT_PRIMARY      0xF5F7FF
#define UI_COLOR_TEXT_SECONDARY    0xB6BDD3
#define UI_COLOR_TEXT_MUTED        0x7D8398
#define UI_COLOR_TEXT_DISABLED     0x5C6173
```

你要是想更省事，我下一条可以直接给你一份 **拆成 Tokens Studio 多个 Set 的版本**，比如：

*   `global/colors`
*   `global/spacing`
*   `global/typography`
*   `semantic/light-dark`

这种更适合你后期维护。



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)