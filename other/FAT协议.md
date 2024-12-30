# FAT协议
FAT结构体
``` C
typedef struct {
    BYTE    fs_type;        /* 文件系统类型 (0: 未挂载) */
    BYTE    pdrv;           /* 物理驱动器编号，表示该卷所在的驱动器 */
    BYTE    ldrv;           /* 逻辑驱动器编号（仅在 FF_FS_REENTRANT 配置启用时使用） */
    BYTE    n_fats;         /* FAT 表数量（通常为 1 或 2） */
    BYTE    wflag;          /* 当前 win[] 缓冲区的状态标志 (b0: 脏标志，表示需要写回) */
    BYTE    fsi_flag;       /* FSINFO 的状态标志 (b7: 禁用标志, b0: 脏标志) */
    WORD    id;             /* 卷挂载 ID，用于唯一标识卷 */
    WORD    n_rootdir;      /* 根目录条目数 (仅 FAT12/16 使用) */
    WORD    csize;          /* 每簇的扇区数 */
#if FF_MAX_SS != FF_MIN_SS
    WORD    ssize;          /* 每扇区的字节数（512、1024、2048 或 4096） */
#endif
#if FF_USE_LFN
    WCHAR*  lfnbuf;         /* 长文件名 (LFN) 工作缓冲区 */
#endif
#if FF_FS_EXFAT
    BYTE*   dirbuf;         /* 用于 exFAT 的目录条目块缓冲区 */
#endif
#if !FF_FS_READONLY
    DWORD   last_clst;      /* 上次分配的簇号 */
    DWORD   free_clst;      /* 可用簇的数量 */
#endif
#if FF_FS_RPATH
    DWORD   cdir;           /* 当前目录的起始簇号 (0: 根目录) */
#if FF_FS_EXFAT
    DWORD   cdc_scl;        /* 包含当前目录的起始簇号（cdir 为 0 时无效） */
    DWORD   cdc_size;       /* b31-b8: 包含目录的大小, b7-b0: 链接状态 */
    DWORD   cdc_ofs;        /* 当前目录在包含目录中的偏移量（cdir 为 0 时无效） */
#endif
#endif
    DWORD   n_fatent;       /* FAT 表项数量（簇数量 + 2） */
    DWORD   fsize;          /* 每个 FAT 表占用的扇区数 */
    LBA_t   volbase;        /* 卷的基地址扇区 */
    LBA_t   fatbase;        /* FAT 表的起始扇区地址 */
    LBA_t   dirbase;        /* 根目录的基地址扇区（FAT12/16）或起始簇（FAT32/exFAT） */
    LBA_t   database;       /* 数据区域的起始扇区地址 */
#if FF_FS_EXFAT
    LBA_t   bitbase;        /* 分配位图的基地址扇区 */
#endif
    LBA_t   winsect;        /* 当前出现在 win[] 中的扇区地址 */
    BYTE    win[FF_MAX_SS]; /* 用于目录、FAT 或文件数据访问的缓冲区 */
} FATFS;

```
 - 1