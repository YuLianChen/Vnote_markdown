# kconfig
1. 如果想用`select TSC_USING_CST820 if BSP_USING_TOUCHD`类似的选中的语句，那么必须要有选项先被设置，否则无法选中
```kconfig
config TSC_USING_CST820
    bool
    default n
```