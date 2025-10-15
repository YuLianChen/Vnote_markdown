# lvgl文件系统踩坑
1. 无法解析图片，必须查看是否是文件路径错误没有打开，使用lvgl内建的文件管理函数打开文件尝试，如果无法打开应该尝试使用debug分析路径信息
```c
    //文件系统测试
    {
        lv_fs_file_t f;
        lv_fs_res_t res;
        res = lv_fs_open(&f, speed.image_background_path, LV_FS_MODE_RD);
        if(res != LV_FS_RES_OK) printf("Failed to open file: %s\n", speed.image_background_path);
        else
        {
            printf("File opened successfully: %s\n", speed.image_background_path);
            lv_fs_close(&f);
        } 
        
    }
```


2. 扫描检查目录下的文件，对比是否是正确路径
```c
    {
        lv_fs_dir_t dir;
        lv_fs_res_t res;
        res = lv_fs_dir_open(&dir, "S:images/speed_time/"); // "S" 是你注册的驱动器字母

        if (res != LV_FS_RES_OK)
        {
            // 错误处理
        }

        char fn[256];
        while (1)
        {
            res = lv_fs_dir_read(&dir, fn);
            if (res != LV_FS_RES_OK || strlen(fn) == 0)
                break;

            printf("Found file: %s\n", fn); // 输出文件名
        }

        lv_fs_dir_close(&dir);
    }
```