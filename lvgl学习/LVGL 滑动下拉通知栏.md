## 一 、首先是处理这种动画的核心逻辑

- 在顶部的一部分区域设置一块透明的触控区域
- 在触控区域被点击，然后持续按压的时候，在触摸响应事件里计算手指滑动的距离
- 在触摸响应事件里将两个页面分别平移手指移动的距离
- 最后在手指松开的时候，在触摸响应事件里计算总的移动距离是否达到了设置的阈值，将页面放置到合适的位置，或者设置动画完成剩下的距离
- 返回上一层界面的操作，只需要监视滑动手势，然后渐隐消失

## 二 、代码实现部分

顶部的手势触发区域设置
```c
    //顶部下滑页面

    main_obj->top_gesture_obj = lv_obj_create(lv_layer_sys());

    lv_obj_set_align(main_obj->top_gesture_obj, LV_ALIGN_TOP_MID);

    lv_obj_set_size(main_obj->top_gesture_obj, lv_pct(100), 10);

    lv_obj_set_style_opa(main_obj->top_gesture_obj, LV_OPA_TRANSP, 0);

    lv_obj_add_event_cb(main_obj->top_gesture_obj, top_drag_event_handler, LV_EVENT_PRESSING, NULL);

    lv_obj_add_event_cb(main_obj->top_gesture_obj, top_drag_event_handler, LV_EVENT_RELEASED, NULL);
```

触发响应部分
```c
static void top_drag_event_handler(lv_event_t * e)

{

    lv_obj_t * obj = lv_event_get_target(e);

    lv_event_code_t code = lv_event_get_code(e);

  //ui_screen_page_drop.ui_main 为下滑通知栏的页面句柄

    if(code == LV_EVENT_PRESSING) //持续按压则持续计算移动距离

    {

        if(!lv_obj_is_valid(ui_screen_page_drop.ui_main))

        {

            ui_screen_page_drop.onLoad(NULL);

        }

        lv_indev_t * indev = lv_indev_get_act();

        if(indev == NULL)  return;

  

        lv_point_t vect;

        lv_indev_get_vect(indev, &vect);

  

        //计算当前页面下滑多少

        int32_t y = lv_obj_get_y_aligned(obj) + vect.y;

        lv_obj_set_y(obj, y);

        printf("Y-axis movement range:%d\n", y);

  

        //计算下拉菜单滑动多少

        lv_obj_set_y(ui_screen_page_drop.ui_main, y-502);

    }

    else if(code == LV_EVENT_RELEASED) //根据结束的时候移动距离做出是否成功判断

    {

        int32_t y = lv_obj_get_y_aligned(obj);

        if(y >= 160)

        {

            lv_obj_set_y(ui_screen_page_drop.ui_main, 0);

            lv_obj_set_y(obj, 0);

            return;

        }

        lv_obj_set_y(obj, 0);

        lv_obj_set_y(ui_screen_page_drop.ui_main, -502);

        if(lv_obj_is_valid(ui_screen_page_drop.ui_main))

        {

            ui_screen_page_drop.unLoad;

        }

    }

}
```

实现下滑界面的布局
```c
//实现ui_screen_page_drop
```