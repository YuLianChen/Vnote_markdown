# lvgl组件旋转
lvgl旋转时，可以使用
```  C
   lv_obj_set_style_transform_pivot_x(list2, 240, LV_PART_MAIN | LV_STATE_DEFAULT);
   lv_obj_set_style_transform_pivot_y(list2, 200, LV_PART_MAIN | LV_STATE_DEFAULT);
```
    设置旋转的中心点
    再使用函数进行旋转就不会发生位置偏转
    lv_obj_set_style_transform_angle(list2, -900, LV_PART_MAIN | LV_STATE_DEFAULT); 