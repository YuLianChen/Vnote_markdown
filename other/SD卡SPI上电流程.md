# SD卡SPI上电流程
[121712023](https://blog.csdn.net/LH_SMD/article/details/121712023)
说明：  
①测试的SD卡为高容量卡，支持SD卡2.0协议，容量为16G  
②采用[GPIO](https://so.csdn.net/so/search?q=GPIO&spm=1001.2101.3001.7020)模拟SPI时序的方式对SD卡进行驱动，很方便移植到没有硬件SPI或者SDIO的MCU，对于这类MCU，只需要将对应的延时函数和GPIO配置换成自己的就可以，其他的都无需变动。  
③对SPI有疑问或者的问题的，请移步之前写过的博文：  
[SD/TF卡驱动（一）--------SD卡相关简介](https://blog.csdn.net/LH_SMD/article/details/121605139)  
④如果内容有任何问题，恳请大家批评指正，谢谢。

### 一、 SD卡SPI初始化流程

### （1）大致流程分析

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42cd380a3685101884eac2b5502818bf.png)

流程图来自《SD卡2.0协议》P95页，SPI模式下的SD卡初始化流程，虚线框起来的部分不是必须要的流程。根据上面流程图，可以得到以下流程：

```
①上电后，将CS片选拉低，然后发送CMD0命令复位SD卡；
②主机发送CMD8命令，确认SD是否支持的协议版本；
③如果支持V1.X版本协议，则发送CMD58指令，确认电压是否兼容，兼容情况下发送ACMD41指令，确认是否已经准备好；
④如果支持V2.0协议，同样发送CMD58指令，确认电压是否兼容，兼容情况下发送ACMD41指令，确认是否已经准备好，待SD准备好之后，发送CMD58命令，确认SD卡是标准容量卡（2G或者2G以下）还是大容量卡（2G至32G（含））。
循环结束。
```

### （2）初始化流程完整指令分析

前文已经说过，SD卡的每一条控制命令都是6个byte，且每一次发送完命令后，SD卡都会有相应的回复，那么接下来，我们需要确定两点：

```
①流程中命令的参数和CRC是多少？
②每条命令执行成功回复是什么？
```

下面就来解决这两个问题。上篇博文提到：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9b7df18820318c570acb4d795e6125a.png)  
由此可以知道:

```
CMD0 			回复R1
ACMD41  		回复R3
CMD8/CMD58		回复R7
```

再来看看什么是R1、R3和R7：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f5f997603ab0e67ef8cb54fa4c41eb31.png)

如果看不清楚，请回到前面一篇博文，上面截图均来自上篇博文。

```
①R1比较简单，如果主机发送的时序和指令没有问题，不是返回0x00就是返回0x01

②R3 5个byte，第一个字节为R1，不是0x00，就是0x01，剩下的四个字节为OCR寄存器的值（OCR寄存器请看前文），事实上有关CMD8命令的回复，在协议文档中已经有说明，如下

③R7 也是5个byte，第一个字节为R1，不是0x00，就是0x01。第二个字节为命令版本、第三个字节保留、第四个字节为被接受的电压范围、第五个字节为模式检查（我也不晓得这是个啥东西，只知道它和CRC都是用来检查主机和卡之间通信的有效性）
```

主机发送CMD0命令，复位成功，SD卡应该回复0x01，我们将CMD0在协议文档中进行全局搜索，可以在P96页发现：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd72a117c52e40c7334a977875dcad6f.png)

所以，最终CMD0的完整命令为：

```
0x00|0x40,0x00,0x00,0x00,0x00,0x95----->0x01(复位成功)
```

还剩下CMD8/CMD58/和ACMD41有待进一步确定回复值

CMD8和CMD58都是回复R7数据的格式，在文档P108页中，有如下说明：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5fe7823c5f373a2badc3de425c957b92.png)

由文档可以知道，如果CMD8发送成功，SD卡回复的五个字节应该是

```
0x01		R1 
0x00 		command version
0x00 		Reserved bit
0x01 		voltage range（VCA）
(Echo Back)	check pattern
```

这个Echo Back是个什么鬼  
复制这个Echo Back进行搜索，在文档的P40页，可以找到下面说明：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/543e21b3c71798b57a6f9112d89e3afa.png)  
显而易见了，这个(Echo Back) check pattern就是0xAA了，事实上，从上面的文档说明里也可以得到CMD8的完整命令：0x80 | 0x40,0x00,0x00,0x01,0xAA,0x87(0x87为CRC，怎么算的自己找资料，实话实话我没找，哈哈哈)。  
于是就有了CMD8的完整命令及回复：

```
0x08 | 0x40,0x00,0x00,0x01,0xAA,0x87---->0x01,0x00,0x00,0x01,0xAA
```

还剩下两个：CMD58和ACMD41  
CMD58是READ_OCA，文档P106：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63249e0fa492f04bdc599ae0332cc3b2.png)

前面已经简单介绍过，这个命令用来确定电压范围和卡容量类型的，直接找到这个寄存器：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82df373510fcbe6d44ce97793bc947a3.png)

如果你的卡接接受电压范围为2.7-3.6V，发送CMD58命令，应该收到的回复是这样的：

```
0xC0
0xFF
0x80
0x00
```

所以，完整的CMD58命令及回复应该是：

```
0x3A|0x40,0x00,0x00,0x00,0x00,0x01--->0x00,0xC0,0xFF,0x80,0x00
```

最后一个ACMD41，这个命令用来启动初始化并检测初始化是否通过。必须要在发送第一个ACMD41命令之前发送CMD8命令。接收CMD8命令会拓展CMD58和ACMD41功能。  
ACMD41参数中的HCS（高容量支持）和CMD58响应中的CCS（卡容量状态），只有在卡接收了CMD8后才能被激活。  
ACMD41 R1响应中的“处于空闲状态”位通知主机ACMD41的初始化是否完成。将该位设置为“1”表示卡仍在初始化。将该位设置为“0”表示初始化完成。主机反复发出ACMD41，直到该位设置为“0”。

那么这个命令由SD卡初始化流程图可以知道，当SD卡为标准容量卡的时候，其参数为0，为高容量SD卡时，其HCS位为1，即此时其命令参数为：0x40,0x00,0x00,0x00,  
所以，最后ACMD41的完整命令参数及回复为：

```
标准容量卡：
		0x29|0x40,0x00,0x00,0x00,0x00,0x01---->0x00(初始化完成)
高容量卡：
		0x29|0x40,0x40,x000,0x00,0x00,0x01----> 0x00(初始化完成)
```

最后，总结一下初始化流程中用到的完整指令以及回复：

```
CMD0：
	0x00|0x40,0x00,0x00,0x00,0x00,0x95----->0x01(复位成功)
CMD8：
	0x08 | 0x40,0x00,0x00,0x01,0xAA,0x87---->0x01,0x00,0x00,0x01,0xAA
ACMD41：
	标准容量卡：
		0x29|0x40,0x00,0x00,0x00,0x00,0x01---->0x00(初始化完成)
	高容量卡：
		0x29|0x40,0x40,x000,0x00,0x00,0x01----> 0x00(初始化完成)
CMD58：
	0x3A|0x40,0x00,0x00,0x00,0x00,0x01--->0x00,0xC0,0xFF,0x80,0x00
```

### （3）SD卡SPI初始化代码思路梳理

这里需要说明一点，应该养成意识，在驱动一款设备时，一般设备上电后的一小段时间内都需要完成某些操作后才能接收指令，SD卡2.0协议中的P91页中，有如下说明：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3f2070d4fc5126f98f6ab4da10d8f50c.png)

现在整个初始化流程就出来了，用伪代码，表示如下：

```
Delay()   //等待SD卡上电稳定
Reply = 200
for(int I = 0;I < 20;i++) SPI_SendByte(0xff);  //内容随便发，但是必须>74个时钟周期。 
//等待SD卡复位成功，CMD0命令，没有参数，
//且根据文档其CRC为0x95（文档P96），复位正常应该返回0x01
CS = 0

while((ret = SendCmd(CMD0, 0, 0x95)) != 0x01 && Reply--);
if(ret == 0x01)  //复位成功
{
	Reply = 200
	While((ret = SendCmd(CMD8, 0x1AA, 0x87)) != 0x01 && Reply--);
	if(ret == 0x01)
	{
		for(int i = 0;i < 4;i++)  buf[i] = SPI_ReadByte();
		if(buf[2]==0X01&&buf[3]==0XAA)//卡是否支持2.7~3.6V
		{
			Reply = 20000
			do
			{
				ret = SendCmd(ACMD41, 0x40000000, 0x01);
			} While(ret != 0x00 && Reply--);  //等待SD初始化完毕
			ret = SendCmd(CMD58, 0x00, 0x01);
			if(ret == 0x00)
			{
				for(i=0;i<4;i++)buf[i]=SPI_ ReadByte ();//得到OCR值
			    if(buf[0]&0x40)SD_Type=SD_TYPE_V2HC;   //检查CCS 标准容量卡
			    else SD_Type=SD_TYPE_V2;         //高容量卡
			}
		}
	}
	else//SD V1.x/ MMC V3
	{
	    SD_SendCmd(CMD55,0,0X01);       //发送CMD55
	    r1=SD_SendCmd(ACMD41,0,0X01);    //发送CMD41
	    if(r1<=1)
	    {
	        SD_Type=SD_TYPE_V1;
	        retry=0XFFFE;
	        do //等待退出IDLE模式
	        {
	            SD_SendCmd(CMD55,0,0X01);   //发送CMD55
	            r1=SD_SendCmd(CMD41,0,0X01);//发送CMD41
	        }while(r1&&retry--);
	    }
	}
}
```

OK，到这里，基本上就剩下写代码了，有关于SPI，你可以用GPIO模拟，也可以用MCU硬件自带的SPI，毫无疑问，同一个芯片硬件的SPI肯定是要快过GPIO模拟的SPI的，如果你对自己使用芯片不太了解，而且时间又很赶，在不影响用户使用的情况下，用GPIO模拟的SPI也没关系。

事实上，如果你按照上面的思路完成代码，在保证SPI时序不出问题的情况下，有大概率是卡死在SendCmd(ACMD41, 0x40000000, 0x01)，细心的你可能会发现，别的地方是都CMD，为啥这里是ACMD，有啥区别？带着疑问，还是需要回到协议文档看看，复制ACMD41，搜索看看：  
在文档P107页，有以下说明：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e6ad4a9ae43acda9cc516a261e5a478f.png)

问题的答案就在这里，凡是以ACMD开头的都是特定应用程序命令，在发这些命令之前，需要发送CMD55，告知SD卡，接下来主机要发送ACMD命令，这里就不再分析CMD55命令了，直接在发ACMD41之前发送即可：SendCmd(CMD55, 0x00, 0x01)

所以，伪代码应该是下面这样：

```
Delay()   //等待SD卡上电稳定
Reply = 200
//等待SD卡复位成功，CMD0命令，没有参数，
//且根据文档其CRC为0x95（文档P96），复位正常应该返回0x01
While((ret = SendCmd(CMD0, 0, 0x95)) != 0x01 && Reply--);
if(ret == 0x01)  //复位成功
{
	Reply = 200
	while((ret = SendCmd(CMD8, 0x1AA, 0x87)) != 0x01 && Reply--);
	if(ret == 0x01)
	{
		for(int i = 0;i < 4;i++)  buf[i] = SPI_ReadByte();
		if(buf[2]==0X01&&buf[3]==0XAA)//卡是否支持2.7~3.6V
		{
			Reply = 20000
			do
			{
								SendCmd(CMD55, 0x00, 0x01)
				ret = SendCmd(ACMD41, 0x40000000, 0x01);
			} While(ret != 0x00 && Reply--);  //等待SD初始化完毕
			ret = SendCmd(CMD58, 0x00, 0x01);
			if(ret == 0x00)
			{
				for(i=0;i<4;i++)buf[i]=SPI_ ReadByte ();//得到OCR值
			    if(buf[0]&0x40)SD_Type=SD_TYPE_V2HC;   //检查CCS 标准容量卡
			    else SD_Type=SD_TYPE_V2;         //高容量卡
			}
		}
	}
	else//SD V1.x
	{
	    SD_SendCmd(CMD55,0,0x01);       //发送CMD55
	    r1=SD_SendCmd(ACMD41,0,0x01);    //发送ACMD41
	    if(r1<=1)
	    {
	        SD_Type=SD_TYPE_V1;
	        retry=0XFFFE;
	        do //等待退出IDLE模式
	        {
	            SD_SendCmd(CMD55,0,0x01);   //发送CMD55
	            r1=SD_SendCmd(CMD41,0,0x01);//发送CMD41
	        }while(r1&&retry--);
	    }
	}
}
```

### （4）初始化代码分析

这里采用的GPIO模拟SPI时序来驱动SD卡的，再次说明，有关SPI时序分析和代码的编写，不清楚的可以我之前写过的东西：  
[SD/TF卡驱动（一）--------SD卡相关简介](https://blog.csdn.net/LH_SMD/article/details/121121133?spm=1001.2014.3001.5501)

```
uint8_t SD_Init(void)
{
    uint8_t r1;      // 存放SD卡的返回值
    uint16_t retry;  // 用来进行超时计数
    uint8_t buf[4];
    uint16_t i;

    SPI_GpioInit();
    for(i=0;i<20;i++)SPI_WriteReadByte(0XFF);//发送最少74个脉冲
    retry=20;
    do
    {
        r1=SD_SendCmd(CMD0,0,0x95);//进入IDLE状态
    }while((r1!=0X01) && retry--);
    SD_Type=0;//默认无卡
    if(r1==0X01)
    {
        printf("SD_SendCmd(CMD0,0,0x95) SUCCESS!\r\n");
        if(SD_SendCmd(CMD8,0x1AA,0x87)==1)//SD V2.0
{
//Get trailing return value of R7 resp
            for(i=0;i<4;i++)buf[i]=SPI_WriteReadByte(0XFF);  
            if(buf[2]==0X01&&buf[3]==0XAA)//卡是否支持2.7~3.6V
            {
                retry=0XFFFE;
                do
                {
                    SD_SendCmd(CMD55,0,0X01);   //发送CMD55
                    r1=SD_SendCmd(CMD41,0x40000000,0X01);//发送CMD41
                }while(r1&&retry--);
                if(retry&&SD_SendCmd(CMD58,0,0X01)==0)//鉴别SD2.0卡版本开始
                {
                    printf("SD_SendCmd(CMD41,0x40000000,0X01) SUCCESS!\r\n");
                    for(i=0;i<4;i++)buf[i]=SPI_WriteReadByte(0XFF);//得到OCR值
                    for(int i = 0;i < 4;i++)
                      {
                        printf("buf[%d] %02X\r\n",i, buf[i]);
                      }
                    if(buf[0]&0x40)SD_Type=SD_TYPE_V2HC;    //检查CCS
                    else SD_Type=SD_TYPE_V2;
                    printf("SD_Type %02X\r\n", SD_Type);
                }
            }
        }
        else//SD V1.x/ MMC V3
        {
            SD_SendCmd(CMD55,0,0X01);       //发送CMD55
            r1=SD_SendCmd(CMD41,0,0X01);    //发送CMD41
            if(r1<=1)
            {
                SD_Type=SD_TYPE_V1;
                retry=0XFFFE;
                do //等待退出IDLE模式
                {
                    SD_SendCmd(CMD55,0,0X01);   //发送CMD55
                    r1=SD_SendCmd(CMD41,0,0X01);//发送CMD41
                }while(r1&&retry--);
            }else//MMC卡不支持CMD55+CMD41识别
            {
                SD_Type=SD_TYPE_MMC;//MMC V3
                retry=0XFFFE;
                do //等待退出IDLE模式
                {
                    r1=SD_SendCmd(CMD1,0,0X01);//发送CMD1
                }while(r1&&retry--);
            }
            if(retry==0||SD_SendCmd(CMD16,512,0X01)!=0)SD_Type=SD_TYPE_ERR;//错误的卡
        }
    }
    SD_DisSelect();//取消片选
    if(SD_Type)return 0;
    else if(r1)return r1;
    return 0xFF;//其他错误
}
```

### 二、 SD卡读写流程分析

初始化SD卡过了，下面就是读写SD卡了，一样的，我们围绕文档，在文档P96页以及后面几页的内容里，详细介绍了如何读写SD卡，请仔细阅读文档。对命令的分析方式，和上面的初始化时候的思路一样，这里就不再详细叙述了，如果你初始化没问题，读写大概率是不会有问题的。

### （1）SD卡读写

1.SD卡读  
读SD卡有单块读，也有多块读，  
单块读的流程如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea5f4665a3bbed2a2ef3fb0c5a4661dc.png)

流程总结如下：

```
①发送 CMD17；
②接收卡响应 R1；
③接收数据；
④接收 2 个字节的 CRC，如果不使用 CRC，这两个字节在读取后可以丢掉。
```

多块读的流程如下：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/adb09876fb94dd3c50e9b291b02068c1.png)

总结起来流程如下：

```
①发送 CMD18；
②接收卡响应 R1；
③接收数据；
④接收 2 个字节的 CRC，如果不使用 CRC，这两个字节在读取后可以丢掉。
⑤数据接收完成后发送 CMD12，停止数据接收，等待回应，结束
```

2.SD卡写  
SD卡写和读类似，也有单块写，也有多块写，如下：  
单块写：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be43e2e18220506cade45969241cc2b1.png)

```
①发送 CMD24;
②接收卡响应 R1；
③发送写数据起始令牌 0XFE；
④发送数据；
⑤发送 2 字节的伪 CRC；
```

多块写：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/144f3eb158251fbdf453520c27ec5cbd.png)

```
①发送 CMD25;
②接收卡响应 R1；
③发送写数据起始令牌 0XFE；
④发送数据；
⑤发送 2 字节的伪 CRC；
⑥判断SD卡响应
⑦结束指令
```

### （2）SD卡读写代码分析

```
/*
 * 向sd卡写入一个数据包的内容 512字节
 * 参数：buf:数据缓存区    cmd:指令
 * 返回值：0,成功;其他,失败;
 * */
uint8_t SD_SendBlock(uint8_t*buf,uint8_t cmd)
{
    uint16_t t;
    if(SD_WaitReady())return 1;//等待准备失效
    SPI_WriteReadByte(cmd);
    if(cmd!=0XFD)//不是结束指令
    {
        for(t=0;t<512;t++)SPI_WriteReadByte(buf[t]);//提高速度,减少函数传参时间
        SPI_WriteReadByte(0xFF);//忽略crc
        SPI_WriteReadByte(0xFF);
        t=SPI_WriteReadByte(0xFF);//接收响应
        if((t&0x1F)!=0x05)return 2;//响应错误
    }
    return 0;//写入成功
}

/*
 * 读SD卡
 * 参数：buf:数据缓存区
 *      sector:扇区
 *      cnt:扇区数
 * 返回值：0,ok;其他,失败.
 * */
uint8_t SD_ReadDisk(uint8_t*buf,uint32_t sector,uint8_t cnt)
{
    uint8_t r1;
    if(SD_Type!=SD_TYPE_V2HC)sector <<= 9;//转换为字节地址
    if(cnt==1)
    {
        r1=SD_SendCmd(CMD17,sector,0X01);//读命令
        if(r1==0)//指令发送成功
        {
            r1=SD_RecvData(buf,512);//接收512个字节
        }
    }else
    {
        r1=SD_SendCmd(CMD18,sector,0X01);//连续读命令
        do
        {
            r1=SD_RecvData(buf,512);//接收512个字节
            buf+=512;
        }while(--cnt && r1==0);
        SD_SendCmd(CMD12,0,0X01);   //发送停止命令
    }
    SD_DisSelect();//取消片选
    return r1;//
}

/*
 * 写SD卡
 * 参数：buf:数据缓存区
 *      sector:扇区
 *      cnt:扇区数
 * 返回值：0,ok;其他,失败.
 * */
uint8_t SD_WriteDisk(uint8_t*buf,uint32_t sector,uint8_t cnt)
{
    uint8_t r1;
    uint8_t reply = 200;
    if(SD_Type!=SD_TYPE_V2HC)sector *= 512;//转换为字节地址
    if(cnt==1)
    {
        r1=SD_SendCmd(CMD24,sector,0X01);//读命令
        if(r1==0)//指令发送成功
        {
            r1=SD_SendBlock(buf,0xFE);//写512个字节
        }
    }
    else
    {
        if(SD_Type!=SD_TYPE_MMC)
        {
            do
            {
                SD_SendCmd(CMD55,0,0X01);
//            SD_SendCmd(CMD23,cnt,0X01);//发送指令
               r1 = SD_SendCmd(CMD22,0x00,0X01);
            }while(r1 != 0x00 && reply--);
            for(int i = 0;i < 4;i++)SPI_WriteReadByte(0xff);

        }
        r1=SD_SendCmd(CMD25,sector,0X01);//连续读命令
        if(r1==0)
        {
            do
            {
                r1=SD_SendBlock(buf,0xFC);//接收512个字节
                buf+=512;
            }while(--cnt && r1==0);
            r1=SD_SendBlock(0,0xFD);//接收512个字节
        }
    }
    SD_DisSelect();//取消片选
    return r1;//
}
```

### 三、 完整代码

Sd_card.h

```
/*
 * sd_card.h
 *
 *  Created on: 2021年11月26日
 *      Author: heng.liao
 */

#ifndef HARDWARE_SD_CARD_H_
#define HARDWARE_SD_CARD_H_

#include "sl_spidrv.h"
#include "em_gpio.h"
#include "sl_spidrv_spiflash_config.h"
#include "spi.h"

// SD卡类型定义
#define SD_TYPE_ERR     0X00
#define SD_TYPE_MMC     0X01
#define SD_TYPE_V1      0X02
#define SD_TYPE_V2      0X04
#define SD_TYPE_V2HC    0X06

// SD卡指令表
#define CMD0    0       //卡复位
#define CMD1    1
#define CMD8    8       //命令8 ，SEND_IF_COND
#define CMD9    9       //命令9 ，读CSD数据
#define CMD10   10      //命令10，读CID数据
#define CMD12   12      //命令12，停止数据传输
#define CMD16   16      //命令16，设置SectorSize 应返回0x00
#define CMD17   17      //命令17，读sector
#define CMD18   18      //命令18，读Multi sector
#define CMD22   22
#define CMD23   23      //命令23，设置多sector写入前预先擦除N个block
#define CMD24   24      //命令24，写sector
#define CMD25   25      //命令25，写Multi sector
#define CMD41   41      //命令41，应返回0x00
#define CMD55   55      //命令55，应返回0x01
#define CMD58   58      //命令58，读OCR信息
#define CMD59   59      //命令59，使能/禁止CRC，应返回0x00



//数据写入回应字意义
extern uint8_t  SD_Type;         //SD卡的类型

uint8_t SD_WaitReady(void);


uint8_t SD_Init(void);
uint32_t SD_GetSectorCount(void);
uint8_t SD_ReadDisk(uint8_t*buf,uint32_t sector,uint8_t cnt);
uint8_t SD_WriteDisk(uint8_t*buf,uint32_t sector,uint8_t cnt);

#endif /* HARDWARE_SD_CARD_H_ */
```

Sd_card.c

```
/*
 * sd_card.c
 *
 *  Created on: 2021年11月26日
 *      Author: heng.liao
 */

#include "sd_card.h"
#include <stdio.h>
#include "spidrv.h"
#include "sl_udelay.h"

uint8_t  SD_Type;

/*
 * 取消片选，释放SPI总线
 *
 * 参数：void
 * 返回值：void
 * */
void SD_DisSelect(void)
{
    SD_CS_SET_GPIO_OUTPUT_STATUS(1);//SD_CS=1;
    SPI_WriteReadByte(0xff);//提供额外的8个时钟
}

/*
 * 等待SD卡准备好
 * 参数：void
 * 返回值：准备好了返回0  否则返回其他
 * */
uint8_t SD_WaitReady(void)
{
    uint32_t t=0;
    do
    {
        if(SPI_WriteReadByte(0XFF)==0XFF)return 0;//OK
        t++;
    }while(t<0XFF);//等待
    return 1;
}

/*
 * 选择SD卡，并且等待SD卡准备好
 * 参数：void
 * 返回值：0,成功;1,失败
 * */
uint8_t SD_Select(void)
{
    SD_CS_SET_GPIO_OUTPUT_STATUS(0);//SD_CS=0;
    if(SD_WaitReady()==0)return 0;//等待成功
    SD_DisSelect();
    return 1;//等待失败
}

/*
 * 向SD卡发送一个命令
 * 参数: u8 cmd   命令
 *      u32 arg  命令参数
 *      u8 crc   crc校验值
 * 返回值：SD卡返回的响应
 * */
uint8_t SD_SendCmd(uint8_t cmd, uint32_t arg, uint8_t crc)
{
    uint8_t r1;
    uint8_t Retry=0;
    SD_DisSelect();//取消上次片选
    if(SD_Select())//return 0XFF;//片选失效
        SD_CS_SET_GPIO_OUTPUT_STATUS(0);
    //发送
    SPI_WriteReadByte(cmd | 0x40);//分别写入命令
    SPI_WriteReadByte(arg >> 24);
    SPI_WriteReadByte(arg >> 16);
    SPI_WriteReadByte(arg >> 8);
    SPI_WriteReadByte(arg);
    SPI_WriteReadByte(crc);
    if(cmd==CMD12)SPI_WriteReadByte(0xff);//Skip a stuff byte when stop reading
    //等待响应，或超时退出
    Retry=0X1F;
    do
    {
        r1=SPI_WriteReadByte(0xFF);
    }while((r1&0X80) && Retry--);
    //返回状态值
    return r1;
}

/*
 * 等待SD卡回应
 * 参数：Response:要得到的回应值
 * 返回值：0,成功得到了该回应值
 *      其他,得到回应值失败
 * */
uint8_t SD_GetResponse(uint8_t Response)
{
    uint16_t Count=0xFFFF;//等待次数
    while ((SPI_WriteReadByte(0XFF)!=Response)&&Count)Count--;//等待得到准确的回应
    if (Count==0)return 1;//得到回应失败
    else return 0;//正确回应
}

/*
 * 从sd卡读取一个数据包的内容
 * 参数：buf:数据缓存区
 *      len:要读取的数据长度.
 * 返回值:0,成功;其他,失败;
 * */
uint8_t SD_RecvData(uint8_t*buf,uint16_t len)
{
    if(SD_GetResponse(0xFE))return 1;//等待SD卡发回数据起始令牌0xFE
    while(len--)//开始接收数据
    {
        *buf=SPI_WriteReadByte(0xFF);
        buf++;
    }
    //下面是2个伪CRC（dummy CRC）
    SPI_WriteReadByte(0xFF);
    SPI_WriteReadByte(0xFF);
    return 0;//读取成功
}

/*
 * 获取SD卡的CID信息，包括制造商信息
 * 参数：u8 *cid_data(存放CID的内存，至少16Byte）
 * 返回值：0：NO_ERR   1：错误
 * */
uint8_t SD_GetCID(uint8_t *cid_data)
{
    uint8_t r1;
    //发CMD10命令，读CID
    r1=SD_SendCmd(CMD10,0,0x01);
    if(r1==0x00)
    {
        r1=SD_RecvData(cid_data,16);//接收16个字节的数据
    }
    SD_DisSelect();//取消片选
    if(r1)return 1;
    else return 0;
}

/*
 * 获取SD卡的CSD信息，包括容量和速度信息
 * 参数:u8 *cid_data(存放CID的内存，至少16Byte）
 *
 * 返回值:0：NO_ERR
 *      1：错误
 * */
uint8_t SD_GetCSD(uint8_t *csd_data)
{
    uint8_t r1;
    r1=SD_SendCmd(CMD9,0,0x01);//发CMD9命令，读CSD
    if(r1==0)
    {
        r1=SD_RecvData(csd_data, 16);//接收16个字节的数据
    }
    SD_DisSelect();//取消片选
    if(r1)return 1;
    else return 0;
}

/*
 * 获取SD卡的总扇区数（扇区数）
 *
 * 参数：void
 *
 * 返回值:0： 取容量出错
 *          其他:SD卡的容量(扇区数/512字节)
 * 每扇区的字节数必为512，因为如果不是512，则初始化不能通过.
 */
uint32_t SD_GetSectorCount(void)
{
    uint8_t csd[16];
    uint32_t Capacity;
    uint8_t n;
    uint16_t csize;
    //取CSD信息，如果期间出错，返回0
    if(SD_GetCSD(csd)!=0) return 0;
    //如果为SDHC卡，按照下面方式计算
    if((csd[0]&0xC0)==0x40)  //V2.00的卡
    {
        csize = csd[9] + ((uint16_t)csd[8] << 8) + 1;
        Capacity = (uint32_t)csize << 10;//得到扇区数
    }else//V1.XX的卡
    {
        n = (csd[5] & 15) + ((csd[10] & 128) >> 7) + ((csd[9] & 3) << 1) + 2;
        csize = (csd[8] >> 6) + ((uint16_t)csd[7] << 2) + ((uint16_t)(csd[6] & 3) << 10) + 1;
        Capacity= (uint32_t)csize << (n - 9);//得到扇区数
    }
    return Capacity;
}


/*
 * SD卡初始化
 *
 * 参数：void
 *
 * 返回值：初始化成功返回0
 *      失败返回其他
 * */
uint8_t SD_Init(void)
{
    uint8_t r1;      // 存放SD卡的返回值
    uint16_t retry;  // 用来进行超时计数
    uint8_t buf[4];
    uint16_t i;

    SPI_GpioInit();
    for(i=0;i<20;i++)SPI_WriteReadByte(0XFF);//发送最少74个脉冲
    retry=20;
    do
    {
        r1=SD_SendCmd(CMD0,0,0x95);//进入IDLE状态
    }while((r1!=0X01) && retry--);
    SD_Type=0;//默认无卡
    if(r1==0X01)
    {
        printf("SD_SendCmd(CMD0,0,0x95) SUCCESS!\r\n");
        if(SD_SendCmd(CMD8,0x1AA,0x87)==1)//SD V2.0
        {
            for(i=0;i<4;i++)buf[i]=SPI_WriteReadByte(0XFF);  //Get trailing return value of R7 resp
            if(buf[2]==0X01&&buf[3]==0XAA)//卡是否支持2.7~3.6V
            {
                retry=0XFFFE;
                do
                {
                    SD_SendCmd(CMD55,0,0X01);   //发送CMD55
                    r1=SD_SendCmd(CMD41,0x40000000,0X01);//发送CMD41
                }while(r1&&retry--);
                if(retry&&SD_SendCmd(CMD58,0,0X01)==0)//鉴别SD2.0卡版本开始
                {
                    printf("SD_SendCmd(CMD41,0x40000000,0X01) SUCCESS!\r\n");
                    for(i=0;i<4;i++)buf[i]=SPI_WriteReadByte(0XFF);//得到OCR值
                    for(int i = 0;i < 4;i++)
                      {
                        printf("buf[%d] %02X\r\n",i, buf[i]);
                      }
                    if(buf[0]&0x40)SD_Type=SD_TYPE_V2HC;    //检查CCS
                    else SD_Type=SD_TYPE_V2;
                    printf("SD_Type %02X\r\n", SD_Type);
                }
            }
        }
        else//SD V1.x/ MMC V3
        {
            SD_SendCmd(CMD55,0,0X01);       //发送CMD55
            r1=SD_SendCmd(CMD41,0,0X01);    //发送CMD41
            if(r1<=1)
            {
                SD_Type=SD_TYPE_V1;
                retry=0XFFFE;
                do //等待退出IDLE模式
                {
                    SD_SendCmd(CMD55,0,0X01);   //发送CMD55
                    r1=SD_SendCmd(CMD41,0,0X01);//发送CMD41
                }while(r1&&retry--);
            }else//MMC卡不支持CMD55+CMD41识别
            {
                SD_Type=SD_TYPE_MMC;//MMC V3
                retry=0XFFFE;
                do //等待退出IDLE模式
                {
                    r1=SD_SendCmd(CMD1,0,0X01);//发送CMD1
                }while(r1&&retry--);
            }
            if(retry==0||SD_SendCmd(CMD16,512,0X01)!=0)SD_Type=SD_TYPE_ERR;//错误的卡
        }
    }
    SD_DisSelect();//取消片选
    if(SD_Type)return 0;
    else if(r1)return r1;
    return 0xaa;//其他错误
}

/*
 * 向sd卡写入一个数据包的内容 512字节
 * 参数：buf:数据缓存区    cmd:指令
 * 返回值：0,成功;其他,失败;
 * */
uint8_t SD_SendBlock(uint8_t*buf,uint8_t cmd)
{
    uint16_t t;
    if(SD_WaitReady())return 1;//等待准备失效
    SPI_WriteReadByte(cmd);
    if(cmd!=0XFD)//不是结束指令
    {
        for(t=0;t<512;t++)SPI_WriteReadByte(buf[t]);//提高速度,减少函数传参时间
        SPI_WriteReadByte(0xFF);//忽略crc
        SPI_WriteReadByte(0xFF);
        t=SPI_WriteReadByte(0xFF);//接收响应
        if((t&0x1F)!=0x05)return 2;//响应错误
    }
    return 0;//写入成功
}

/*
 * 读SD卡
 * 参数：buf:数据缓存区
 *      sector:扇区
 *      cnt:扇区数
 * 返回值：0,ok;其他,失败.
 * */
uint8_t SD_ReadDisk(uint8_t*buf,uint32_t sector,uint8_t cnt)
{
    uint8_t r1;
    if(SD_Type!=SD_TYPE_V2HC)sector <<= 9;//转换为字节地址
    if(cnt==1)
    {
        r1=SD_SendCmd(CMD17,sector,0X01);//读命令
        if(r1==0)//指令发送成功
        {
            r1=SD_RecvData(buf,512);//接收512个字节
        }
    }else
    {
        r1=SD_SendCmd(CMD18,sector,0X01);//连续读命令
        do
        {
            r1=SD_RecvData(buf,512);//接收512个字节
            buf+=512;
        }while(--cnt && r1==0);
        SD_SendCmd(CMD12,0,0X01);   //发送停止命令
    }
    SD_DisSelect();//取消片选
    return r1;//
}

/*
 * 写SD卡
 * 参数：buf:数据缓存区
 *      sector:扇区
 *      cnt:扇区数
 * 返回值：0,ok;其他,失败.
 * */
uint8_t SD_WriteDisk(uint8_t*buf,uint32_t sector,uint8_t cnt)
{
    uint8_t r1;
    uint8_t reply = 200;
    if(SD_Type!=SD_TYPE_V2HC)sector *= 512;//转换为字节地址
    if(cnt==1)
    {
        r1=SD_SendCmd(CMD24,sector,0X01);//读命令
        if(r1==0)//指令发送成功
        {
            r1=SD_SendBlock(buf,0xFE);//写512个字节
        }
    }
    else
    {
        if(SD_Type!=SD_TYPE_MMC)
        {
            do
            {
                SD_SendCmd(CMD55,0,0X01);
//            SD_SendCmd(CMD23,cnt,0X01);//发送指令
               r1 = SD_SendCmd(CMD22,0x00,0X01);
            }while(r1 != 0x00 && reply--);
            for(int i = 0;i < 4;i++)SPI_WriteReadByte(0xff);

        }
        r1=SD_SendCmd(CMD25,sector,0X01);//连续读命令
        if(r1==0)
        {
            do
            {
                r1=SD_SendBlock(buf,0xFC);//接收512个字节
                buf+=512;
            }while(--cnt && r1==0);
            r1=SD_SendBlock(0,0xFD);//接收512个字节
        }
    }
    SD_DisSelect();//取消片选
    return r1;//
}
```

/*************************************************************************/

### 附：SPI源码（SPI模式0）

Spi.h

```
/*
 * spi.h
 *
 *  Created on: 2021年11月27日
 *      Author: lhsmd
 */

#ifndef HARDWARE_SPI_H_
#define HARDWARE_SPI_H_

#include "em_gpio.h"
#include "em_cmu.h"


// USART0 TX on PC00
#define SPI_MOSI_PORT               gpioPortC
#define SPI_MOSI_PIN                1

// USART0 RX on PC01
#define SPI_MISO_PORT               gpioPortC
#define SPI_MISO_PIN               3

// USART0 CLK on PC02
#define SPI_CLK_PORT              gpioPortC
#define SPI_CLK_PIN               0

// USART0 CS on PC03
#define SPI_CS_PORT               gpioPortC
#define SPI_CS_PIN                2

#define SD_CS_SET_GPIO_OUTPUT_STATUS(status)        {if(status == 1)   GPIO_PinOutSet(SPI_CS_PORT, SPI_CS_PIN);\
                                                    else if(status == 0)  GPIO_PinOutClear(SPI_CS_PORT, SPI_CS_PIN);}


#define SD_CLK_SET_GPIO_OUTPUT_STATUS(status)        {if(status == 1)   GPIO_PinOutSet(SPI_CLK_PORT, SPI_CLK_PIN);\
                                                    else if(status == 0)  GPIO_PinOutClear(SPI_CLK_PORT, SPI_CLK_PIN);}


#define SD_MOSI_SET_GPIO_OUTPUT_STATUS(status)        {if(status == 1)   GPIO_PinOutSet(SPI_MOSI_PORT, SPI_MOSI_PIN);\
                                                    else if(status == 0)  GPIO_PinOutClear(SPI_MOSI_PORT, SPI_MOSI_PIN);}

void SPI_GpioInit(void);
void SPI_WriteByte(uint8_t data);
uint8_t SPI_ReadByte(void);
uint8_t SPI_WriteReadByte(uint8_t data);

#endif /* HARDWARE_SPI_H_ */
```

Spi.c

```
/*
 * spi.c
 *
 *  Created on: 2021年11月27日
 *      Author: lhsmd
 */

#include "spi.h"
#include "sl_udelay.h"


static void SPI_Delay(uint32_t us)
{
    sl_udelay_wait(us);
}


void SPI_GpioInit(void)
{
    CMU_ClockEnable(cmuClock_GPIO, true);  /* 使能GPIO时钟 */
    GPIO_PinModeSet(SPI_CS_PORT, SPI_CS_PIN, gpioModePushPull, 0);
    GPIO_PinModeSet(SPI_MOSI_PORT, SPI_MOSI_PIN, gpioModePushPull, 0);
    GPIO_PinModeSet(SPI_CLK_PORT, SPI_CLK_PIN, gpioModePushPull, 0);

    GPIO_PinModeSet(SPI_MISO_PORT, SPI_MISO_PIN, gpioModeInputPull, 0);

    SD_CS_SET_GPIO_OUTPUT_STATUS(1);
    SD_CLK_SET_GPIO_OUTPUT_STATUS(0);
}

/*
* 函数名：void SPI_WriteByte(uint8_t data)
* 输入参数：data -> 要写的数据
* 输出参数：无
* 返回值：无
* 函数作用：模拟 SPI 写一个字节
*/
void SPI_WriteByte(uint8_t data)
{
    uint8_t i = 0;
    uint8_t temp = 0;
    for(i=0; i<8; i++)
    {
        temp = ((data&0x80)==0x80)? 1:0;
        data = data<<1;
        SD_CLK_SET_GPIO_OUTPUT_STATUS(0); //SPI_CLK(0); //CPOL=0
        SD_MOSI_SET_GPIO_OUTPUT_STATUS(temp);//SPI_MOSI(temp);
        SPI_Delay(1);
        SD_CLK_SET_GPIO_OUTPUT_STATUS(1);//SPI_CLK(1); //CPHA=0
        SPI_Delay(1);
    }
    SD_CLK_SET_GPIO_OUTPUT_STATUS(0);//SPI_CLK(0);
}
/*
* 函数名：uint8_t SPI_ReadByte(void)
* 输入参数：
* 输出参数：无
* 返回值：读到的数据
* 函数作用：模拟 SPI 读一个字节
*/
uint8_t SPI_ReadByte(void)
{
    uint8_t i = 0;
    uint8_t read_data = 0xFF;
    for(i=0; i<8; i++)
    {
        read_data = read_data << 1;
        SD_CLK_SET_GPIO_OUTPUT_STATUS(0);//SPI_CLK(0);
        SPI_Delay(1);
        SD_CLK_SET_GPIO_OUTPUT_STATUS(1);//SPI_CLK(1);
        SPI_Delay(1);
        if(GPIO_PinInGet(SPI_MISO_PORT, SPI_MISO_PIN)==1)
        {
            read_data = read_data + 1;
        }
    }
    SD_CLK_SET_GPIO_OUTPUT_STATUS(0);//SPI_CLK(0);
    return read_data;
}




/*
* 函数名：uint8_t SPI_WriteReadByte(uint8_t data)
* 输入参数：data -> 要写的一个字节数据
* 输出参数：无
* 返回值：读到的数据
* 函数作用：模拟 SPI 读写一个字节
*/
uint8_t SPI_WriteReadByte(uint8_t data)
{
    uint8_t i = 0;
    uint8_t temp = 0;
    uint8_t read_data = 0xFF;
    for(i=0;i<8;i++)
    {
        temp = ((data&0x80)==0x80)? 1:0;
        data = data<<1;
        read_data = read_data<<1;
        SD_CLK_SET_GPIO_OUTPUT_STATUS(0);//SPI_CLK(0);
        SD_MOSI_SET_GPIO_OUTPUT_STATUS(temp);//SPI_MOSI(temp);
        SPI_Delay(1);
        SD_CLK_SET_GPIO_OUTPUT_STATUS(1);//SPI_CLK(1);
        SPI_Delay(1);
        if(GPIO_PinInGet(SPI_MISO_PORT, SPI_MISO_PIN)==1)
        {
            read_data = read_data + 1;
        }
    }
    SD_CLK_SET_GPIO_OUTPUT_STATUS(0); //SPI_CLK(0);
    return read_data;
}
```