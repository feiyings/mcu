## ESP8266WIFI模块的学习

注：学习的例程在svn文档[ESP8266WIFI]()

### 一、基础掌握和通信

#### 1、ESP8266

​		ESP8266 是一个完整且自成体系的 WiFi 网络解决方案，能够独立运行， 也可以作为 slave 搭载于其他 Host 运行。ESP8266 在搭载应用并作为设备中唯一的

应用处理器时，能够直接从外接闪存中启动。内置的高速缓冲存储器有利于提高系统性能，并减少内存需求。

​		ESP8266 支持 softAP 模式，station 模式，softAP + station 共存模式三种。利用 ESP8266 可以实现十分灵活的组网方式和网络拓扑。（SoftAP：即无线接

入点，是一个无线网络的中心节点。通常使用的无线路由器就是一个无线接入点。Station：即无线终端，是一个无线网络的终端端。）

#### 2、通信方式

​		ESP8266是一款高性能的WIFI串口模块，内部集成MCU，能实现单片机之间串口通信，是目前使用最广泛的一种WIFI模块之一。可以简单理解为一个WIFI转

串口的设备，不用知道太多WIFI相关知识，只需要知道串口怎么使用就可以。

具体引脚连接：

这是ESP8266的引脚

​		![image-20230429151150587](images/image-20230429151150587.png)![4444](images/4444.png)

| 引脚名称 | 含义                                              |
| -------- | ------------------------------------------------- |
| VCC      | 电源正极                                          |
| RST      | 复位                                              |
| CH_PD    | 使能                                              |
| UTXD     | 写                                                |
| URXD     | 读                                                |
| GPIO0    | 低电平代表进入系统升级状态，高电平代表从FLASH启动 |
| GPIO2    | 保持高电平                                        |
| GND      | 接地                                              |

VCC、GPIO0、GPIO2接3.3V，GND接地。UTXD接串口的RX（读），URXD接串口的TX（写）。

![image-20230429155432038](images/image-20230429155432038.png)

#### 3、开发方式

​		ESP8266系列一般具有两种开发方式：AT指令开发和SDK开发。、

​		AT指令：厂家出厂时预先在ESP8266芯片烧入好固件，封装好WiFi的协议栈，内部已经实现透传，而用户只需要使用一个USB转TTL的模块或者单片机的串口

就能实现与WiFi模块的通信，发送AT指令来对WiFi模块进行控制。（和蓝牙透传模块类似）

​		SDK开发：由于ESP8266本身即是可编程的芯片，可以把它视为一个带有无线通信的单片机，而用户需要在专门的IDE中编写对应的程序，然后通过烧写固件

的方式将程序写入到芯片中，因此，想要实现WiFi通信，需要自定义WiFi协议栈，对用户掌握的相关知识要求更高。

我们这儿使用AT指令的方式

#### 4、AT指令

常用AT指令

| 指令名                            | 响应                       | 含义                          |
| --------------------------------- | -------------------------- | ----------------------------- |
| AT                                | OK                         | 测试指令                      |
| AT+RST                            | OK                         | 重启                          |
| AT+RESTORE                        | OK                         | 恢复出厂设置                  |
| AT+CWMODE=<mode>                  | OK                         | 设置应用模式（STA/AP/AP_STA） |
| AT+CWMODE?                        | +CWMODE:<mode>             | 查询当前模式                  |
| AT+CWLAP                          | +CWLAP:<ecn>,<ssid>,<rssi> | 列出当前可用AP                |
| AT+CWJAP=<ssid>,<pwd>             | OK                         | 加入某一AP                    |
| AT+CWJAP?                         | +CWJAP:<ssid>              | 返回当前加入的AP              |
| AT+CWQAP                          | OK                         | 退出当前连接的AP              |
| AT+CIPSTART=<type>,<addr>,<port>  | OK                         | 建立TCP/UDP连接               |
| AT+CWSAP=<ssid>,<pwd>,<chl>,<ecn> | OK                         | 配置AP参数                    |
| AT+CIPMUX=<mode>                  | OK                         | 是否启用多连接                |
| AT+CIPSEND=<length>               | OK                         | 发送数据                      |
| AT+CIPSERVER=<mode>,<param>       | OK                         | 建立TCP服务器                 |
| AT+CIPMODE=<mode>                 | OK                         | 设置传输模式                  |

详细指令可参考文档: [AT指令集_用户手册]()

#### 5、串口测试

使用USB转TTL将ESP8266接到PC上，打开串口。默认使用的是115200的波特率。

![image-20230429155715534](images/image-20230429155715534.png)



### 二、透传模式

#### 1、ESP8266作为TCP Client 单连接透传

***这种模式是使用了ESP8266的STA模式。作为TCP Client和TCP Server连入同一个外界热点进行透传***

指令顺序

```
AT+CWMODE=3                             // 设置ESP8266的模式是AP+STA模式
AT+CWJAP="MERCURY50","xhk123456"        // 连接热点（这个热点需要和TCP Server的网络是同一个热点）
AT+CIFSR                                // 查看连接上热点之后的ESP8266的ip地址
AT+CIPMUX=0                             // 设置单连接
AT+CIPSTART="TCP","192.168.0.109",9898  // 连接TCP server
AT+CIPMODE=1                            // 开启透传
AT+CIPSEND                              // 发送数据
输入+++结束发送
```

##### (1)配置 WiFi 模式

```
AT+CWMODE=3    // softAP+station 模式
```

响应：

![image-20230427182320858](images/image-20230427182320858.png)

##### (2)连接路由器

```
AT+CWJAP="SSID","password"  // SSID：路由器名字 password：密码
```

响应：

![image-20230427182447417](images/image-20230427182447417.png)

##### (3)查询 ESP8266 设备的 IP 地址

```
AT+CIFSR                                // 查看连接上热点之后的ESP8266的ip地址
```

响应：

![image-20230427182541463](images/image-20230427182541463.png)



##### (4)PC与ESP8266连接同一个路由器，在PC端使用网络调试工具，建立一个TCP服务器。

假设创建的TCP服务器IP地址是192.168.0.109。端口是9898。

![image-20230427182802021](images/image-20230427182802021.png)

##### (5)设置连接方式为单连接

```
AT+CIPMUX=0                             // 设置单连接
```

响应：

![image-20230427182954658](images/image-20230427182954658.png)





##### (6) ESP8266 设备作为 TCP client 连接到上述服务器

```
AT+CIPSTART="TCP","192.168.0.109",9898  // 连接TCP server
```

响应：

![image-20230427183025385](images/image-20230427183025385.png)

##### (7)开启透传

```
AT+CIPMODE=1                            // 开启透传
```

相应：

![image-20230427183115169](images/image-20230427183115169.png)

##### (8)发送数据

```
AT+CIPSEND                              // 发送数据
```

响应：

![image-20230427183200297](images/image-20230427183200297.png)



##### (9)结束发送

```
+++
```

![image-20230427183315654](images/image-20230427183315654.png)



#### 2、ESP8266发起作为AP，单连接穿透

***这种模式是使用ESP8266的AP模式。发出热点，PC或者其他设备连上ESP8266的热点。然后ESP8266作为TCP Client，其他设备作为TCP Server进行单连接透传***

指令顺序

```
AT+CWMODE=3                           // 设置模块工作在AP+STA模式
AT+CWSAP="ESP8266","12345678",1,0     // 发出热点
AT+CIPMODE=1                          // 开启透传
AT+CIPMUX=0                           // 单连接模式
AT+CIPSTART="TCP","192.168.4.2",9898  // 模块作为tcp client客户端连接手机tcp server服务器端。这儿的IP和port是TCP Server的
AT+CIPSEND                            // 进入透传
```



##### (1)设置模块工作方式

```
AT+CWMODE=3                           // 设置模块工作在AP+STA模式
```

响应：

![image-20230427192927409](images/image-20230427192927409.png)



##### (2)发出热点

```
AT+CWSAP="ESP8266","12345678",1,0     // 发出热点
```

响应：

![image-20230427193023970](images/image-20230427193023970.png)

这个时候我们通过PC就可以搜索到ESP8266这个wifi。连接即可。

![image-20230427193120072](images/image-20230427193120072.png)

##### (3)开启透传

```
AT+CIPMODE=1                          // 开启透传
```

响应：

![image-20230427193152161](images/image-20230427193152161.png)

##### (4)单连接模式

```
AT+CIPMUX=0                           // 单连接模式
```

响应：

![image-20230427193315980](images/image-20230427193315980.png)

##### (5)连接TCP Server

首先在PC上用网络调试助手创建一个TCP Server

![image-20230427193713958](images/image-20230427193713958.png)

这儿的ip和port为192.168.4.2:9999

```
AT+CIPSTART="TCP","192.168.4.2",9999  // 模块作为tcp client客户端连接手机tcp server服务器端。这儿的IP和port是TCP Server的
```

响应：

![image-20230427193649104](images/image-20230427193649104.png)



##### (6)进入透传

```
AT+CIPSEND                            // 进入透传
```

响应：

![image-20230427193740073](images/image-20230427193740073.png)

透传测试：

![image-20230427193922545](images/image-20230427193922545.png)

串口调试助手通过USB转TTL跟ESP8266通信

### 三、代码实现

#### 1、ESP8266驱动文件

包含头文件`ESP8266_driver.h`和源文件`ESP8266_driver.c`

##### 1.1 头文件

`ESP8266_driver.h`

```c
#include "gd32f10x.h"
#include "ESP8266_public.h"
#include <stdio.h>
#include <stdbool.h>
#include "systick.h"
#include "data_usart.h"

#if defined ( __CC_ARM   )
#pragma anon_unions
#endif

//ESP8266模式定义
typedef enum
{
	STA,
  	AP,
  	STA_AP
}ENUM_Net_ModeTypeDef;


// ESP8266连接ID定义
typedef enum{
	Multiple_ID_0 = 0,
	Multiple_ID_1 = 1,
	Multiple_ID_2 = 2,
	Multiple_ID_3 = 3,
	Multiple_ID_4 = 4,
	Single_ID_0 = 5,
} ENUM_ID_NO_TypeDef;

// AP的ecn加密方式
typedef enum
{
  OPEN = 0,
  WEP = 1,
  WPA_PSK = 2,
  WPA2_PSK = 3,
  WPA_WPA2_PSK = 4,
} ENUM_ID_NO_AP_ECN;

// 配置服务器时的server模式 0:关闭server模式  1:开启server模式
typedef enum
{
   SERVER_CLOSED = 0,             
   SERVER_OPEN = !SERVER_CLOSED,  
} ENUM_MODE_AP_SERVER;


#define ESP8266_RST_Pin_SetH     gpio_bit_set(GPIOB,GPIO_PIN_0)    // ESP8266硬件复位
#define ESP8266_RST_Pin_SetL     gpio_bit_reset(GPIOB,GPIO_PIN_0)      // ESP8266硬件置位


#define ESP8266_CH_PD_Pin_SetH     gpio_bit_set(GPIOA,GPIO_PIN_5)    // ESP8266透传控制CH_PD置位
#define ESP8266_CH_PD_Pin_SetL     gpio_bit_reset(GPIOA,GPIO_PIN_5)      // ESP8266透传控制CH_PD复位

#define ESP8266_USART(fmt, ...)	 USART_printf (USART1, fmt, ##__VA_ARGS__)    // 定义ESP8266串口传输


#define RX_BUF_MAX_LEN 1024		                        //最大接收缓存字节数
extern struct STRUCT_USART_Fram	                                //定义一个全局串口数据帧的处理结构体
{
	char Data_RX_BUF[RX_BUF_MAX_LEN];
	union
	{
    	volatile uint16_t InfAll;
    	struct
		{
		  	volatile uint16_t FramLength       :15;                               // 14:0
		  	volatile uint16_t FramFinishFlag   :1;                                // 15
	  	}InfBit;
  	};
}ESP8266_Fram_Record_Struct;


//函数的声明 
void ESP8266_Init(uint32_t bound);                                                                    // 初始化ESP8266
bool ESP8266_Send_AT_Cmd(char *cmd,char *ack1,char *ack2,uint32_t time);                              // 对ESP8266模块发送AT指令
void ESP8266_Rst(void);                                                                               // 重启ESP8266模块
uint8_t ESP8266_AT_Test(void);                                                                        // 对ESP8266模块进行AT测试启动
bool ESP8266_Net_Mode_Choose(ENUM_Net_ModeTypeDef enumMode);                                          // 选择ESP8266模块的工作模式
bool ESP8266_JoinAP(char * pSSID, char * pPassWord);                                                  // ESP8266模块连接外部WiFi
bool ESP8266_Enable_MultipleId (ControlStatus controlStatus);                                         // ESP8266模块启动多连接
bool ESP8266_Soft_Rst(void);                                                                          // ESP8266软件复位  AT+RST
bool ESP8266_Create_AP( char * pSSID, char * pPassWord, uint8_t channel, ENUM_ID_NO_AP_ECN enumEcn);  // ESP8266模块设置AP模式下的参数
bool ESP8266_Server_Port_Set(ENUM_MODE_AP_SERVER mode, char * port);                                  // ESP8266模块配置为服务器
bool ESP8266_Get_IPAdress(void);                                                                      // ESP8266获取本地IP地址
bool ESP8266_OPEN_UnvarnishSend(void);                                                                // 开启透传
bool ESP8266_Connect_Tcp_Server(char * ip, char * port);                                              // 连接TCP Server
void ESP8266_Exit_UnvarnishSend (void);                                                               // ESP8266模块退出透传模式
uint8_t ESP8266_Get_LinkStatus (void);                                                                // 获取ESP8266 的连接状态  
bool ESP8266_SendString(ControlStatus unvarnish_status, char * pStr);                                 // ESP8266模块发送字符串
void ESP8266_SendArray(unsigned char send_array[],unsigned char num);                                 // ESP8266模块发送数组
bool ESP8266_RESTORE(void);                                                                           // 恢复出厂设置，恢复出厂设置会导致机器重启
```

##### 1.2 源文件

`ESP8266_driver.c`

```c
#include "ESP8266_driver.h"
#include <stdio.h>
#include <string.h>
#include <stdbool.h>

struct STRUCT_USART_Fram ESP8266_Fram_Record_Struct = { 0 };   // 初始化串口数据接收

/**
* 初始化ESP8266
*/
void ESP8266_Init(uint32_t bound)
{
  ESP8266_RST_Pin_SetH;	
}

/**
* 对ESP8266模块发送AT指令
* cmd：待发送的指令
* ack1，ack2：期待的响应，为NULL表不需响应，两者为或逻辑关系
* time：等待响应的时间
* 返回 1：发送成功 0：失败
*/
bool ESP8266_Send_AT_Cmd(char *cmd,char *ack1,char *ack2,uint32_t time)
{
  ESP8266_Fram_Record_Struct.InfBit.FramLength = 0;	//从新开始接收新的数据包
  ESP8266_USART("%s\r\n", cmd);
  if(ack1==0&&ack2==0)	 //不需要接收数据
  {
    return true;
  }
  delay_1ms(time);	  //延时time时间
  
  ESP8266_Fram_Record_Struct.Data_RX_BUF[ESP8266_Fram_Record_Struct.InfBit.FramLength ] = '\0';
  
  if(ack1!=0 && ack2!=0)
  {
    return ( ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack1 ) ||
            ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack2 ) );
  }
  else if( ack1 != 0 )
    return ( ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack1 ) );
  
  else
    return ( ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack2 ) );
}


/**
* 恢复出厂设置，恢复出厂设置会导致机器重启
* 返回 1：恢复出厂设置成功，0：恢复失败
*/
bool ESP8266_RESTORE(void)
{
  return ESP8266_Send_AT_Cmd("AT+ RESTORE","OK",NULL,2500);
}


/**
* 重启ESP8266模块
*/
void ESP8266_Rst(void)
{
  ESP8266_RST_Pin_SetL;
  delay_1ms(500);
  ESP8266_RST_Pin_SetH;
}


/**
* 对ESP8266模块进行AT测试启动
* 返回 1：测试启动成功，0：超时失败
*/
uint8_t ESP8266_AT_Test(void)
{
  char count=0;	
  ESP8266_CH_PD_Pin_SetH;
  while(count < 10)
  {
    if(ESP8266_Send_AT_Cmd("AT","OK",NULL,2500))
      return 1;
    ++ count;
  }
  return 0;
}

/**
* 选择ESP8266模块的工作模式
* enumMode：工作模式
* 返回1：选择成功 0：选择失败
*/
bool ESP8266_Net_Mode_Choose(ENUM_Net_ModeTypeDef enumMode)
{
  switch ( enumMode )
  {
  case STA:
    return ESP8266_Send_AT_Cmd ( "AT+CWMODE=1", "OK", "no change", 2500 );
    
  case AP:
    return ESP8266_Send_AT_Cmd ( "AT+CWMODE=2", "OK", "no change", 2500 );
    
  case STA_AP:
    return ESP8266_Send_AT_Cmd ( "AT+CWMODE=3", "OK", "no change", 2500 );
  default:
    return false;
  }		
}


/**
* ESP8266模块连接外部WiFi
* pSSID：WiFi名称字符串
* pPassWord：WiFi密码字符串
* 返回 1：连接成功 0：连接失败
*/
bool ESP8266_JoinAP( char * pSSID, char * pPassWord )
{
  char cCmd [120];
  sprintf (cCmd, "AT+CWJAP=\"%s\",\"%s\"", pSSID, pPassWord );
  return ESP8266_Send_AT_Cmd( cCmd, "OK", NULL, 5000 );
}


/**
* ESP8266模块启动多连接
* controlStatus：配置是否多连接 1:配置，0：不配置多连接
* 返回1：配置成功 0：配置失败
*/
bool ESP8266_Enable_MultipleId (ControlStatus controlStatus )
{
  char cStr [20];
  
  sprintf ( cStr, "AT+CIPMUX=%d", ( controlStatus ? 1 : 0 ) );
  
  return ESP8266_Send_AT_Cmd ( cStr, "OK", 0, 500 );
  
}

/**
* ESP8266软件复位  AT+RST
* 返回1：重启成功 0：重启失败
*/
bool ESP8266_Soft_Rst(void)
{
  return ESP8266_Send_AT_Cmd("AT+RST","OK",NULL,2500);
}

/**
* ESP8266模块设置AP模式下的参数
* pSSID：WiFi名称字符串
* pPassWord：WiFi密码字符串
* channel：通道号
* enumEcn：ecn加密方式
* 返回 1：设置成功 0：设置失败
*/
bool ESP8266_Create_AP( char * pSSID, char * pPassWord, uint8_t channel, ENUM_ID_NO_AP_ECN enumEcn)
{
  char cCmd [120];
  sprintf (cCmd, "AT+CWSAP=\"%s\",\"%s\",%d,%d", pSSID, pPassWord,channel,enumEcn);
  return ESP8266_Send_AT_Cmd( cCmd, "OK", NULL, 5000 );
}

/**
* ESP8266模块配置为服务器
* mode:  0：关闭server模式，1：开启server模式
* port: 端口号，缺省为333
* 返回 1：设置成功 0：设置失败
*/
bool ESP8266_Server_Port_Set(ENUM_MODE_AP_SERVER mode, char * port)
{
  char cCmd [60];
  bool flag = false;
  sprintf (cCmd, "AT+CIPSERVER=%d,%s", mode, port);
  
  if(ESP8266_Send_AT_Cmd( cCmd, "OK", NULL, 5000))
  {
    flag = true;
  }
  return flag;
}

/**
* ESP8266获取本地IP地址
* 返回 1：获取成功 0：获取失败
*/
bool ESP8266_Get_IPAdress(void)
{
  return ESP8266_Send_AT_Cmd("AT+CIFSR","OK",NULL,2500);
}


/**
* 开启透传
* 返回 1：设置成功 0：设置失败
*/
bool ESP8266_OPEN_UnvarnishSend(void)
{
  return ESP8266_Send_AT_Cmd("AT+CIPMODE=1","OK",NULL,500);
}

/**
* 连接TCP Server
* ip：tcp服务器的ip地址
* port：tcp服务器的端口
* 返回 1：连接成功 0：连接失败
*/
bool ESP8266_Connect_Tcp_Server(char * ip, char * port)
{
  char cCmd [60];
  sprintf (cCmd, "AT+CIPSTART=\"%s\",\"%s\",%s","TCP", ip, port);
  return ESP8266_Send_AT_Cmd( cCmd, "OK", NULL, 5000);
}


/**
* ESP8266模块退出透传模式
*/
void ESP8266_Exit_UnvarnishSend ( void )
{
  delay_1ms(1000);
  ESP8266_USART( "+++" );
  delay_1ms(500);
}

/**
* 获取ESP8266 的连接状态
* 返回   0：获取状态失败  2：获得ip  3：建立连接  4：失去连接
*/
uint8_t ESP8266_Get_LinkStatus ( void )
{
  if (ESP8266_Send_AT_Cmd( "AT+CIPSTATUS", "OK", 0, 500 ) )
  {
    if ( strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, "STATUS:2\r\n" ) )
      return 2;
    else if ( strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, "STATUS:3\r\n" ) )
      return 3;
    else if ( strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, "STATUS:4\r\n" ) )
      return 4;		
  }
  return 0;	
}

/**
* ESP8266模块发送字符串
* unvarnish_status：是否开启透传，pStr：要发送的字符串
* 返回1：发送成功 0：发送失败
*/
bool ESP8266_SendString(ControlStatus unvarnish_status, char * pStr)
{
  if (unvarnish_status)
  {
    ESP8266_USART ( "%s", pStr );
    return true;
  }
  return false;
}


/**
* ESP8266模块发送数组
* send_array：数组(的数据) num：数组长度1-255
*/
void ESP8266_SendArray(unsigned char send_array[],unsigned char num) 
{
  unsigned char i=0; 
  while(i)
  {
    usart_data_transmit(USART1,send_array[i]);                // 通过库函数  发送数据
    while( usart_flag_get(USART1, USART_FLAG_TC) == RESET );  // 等待发送完成。检测USART_FLAG_TC是否置1；
    i++;  //值 加一
  }
}
```

#### 2、功能性函数文件

##### 2.1 发送AT指令函数文件

头文件`ESP8266_public.h`和源文件`ESP8266_public.c`

头文件`ESP8266_public.h`

```c
#include "gd32f10x_usart.h"
#include <stdarg.h>

void USART_printf (uint32_t usart_periph, char * Data, ... );
static char *itoa( int value, char *string, int radix );
```

源文件`ESP8266_public.c`

```c
#include "ESP8266_public.h"


void USART_printf (uint32_t usart_periph, char * Data, ... )
{
  const char *s;
  int d;
  char buf[16];
  
  va_list ap;
  va_start(ap, Data);
  
  while ( * Data != 0 )     // 判断是否到达字符串结束符
  {				
    if ( * Data == 0x5c )  //'\'
    {									
      switch ( *++Data )
      {
      case 'r':							          //回车符
        usart_data_transmit(usart_periph, 0x0d);
        Data ++;
        break;
        
      case 'n':							          //换行符
        usart_data_transmit(usart_periph, 0x0a);	
        Data ++;
        break;
        
      default:
        Data ++;
        break;
      }			
    }
    
    else if ( * Data == '%')
    {								
      switch ( *++Data )
      {				
      case 's':								  //字符串
        s = va_arg(ap, const char *);
        
        for ( ; *s; s++)
        {
          usart_data_transmit(usart_periph,*s);
          while( usart_flag_get(usart_periph, USART_FLAG_TBE) == RESET );
        }
        
        Data++;
        
        break;
        
      case 'd':			
        //十进制
        d = va_arg(ap, int);
        itoa(d, buf, 10);
        
        for (s = buf; *s; s++)
        {
          usart_data_transmit(usart_periph,*s);
          while( usart_flag_get(usart_periph, USART_FLAG_TBE) == RESET );
        }
        
        Data++;
        
        break;
        
      default:
        Data++;
        
        break;
        
      }		
    }
    else usart_data_transmit(usart_periph, *Data++);
    
    while ( usart_flag_get(usart_periph, USART_FLAG_TBE) == RESET );
  }
}

/**
* 整型转字符串
*/
static char *itoa( int value, char *string, int radix )
{
  int     i, d;
  int     flag = 0;
  char    *ptr = string;
  /* This implementation only works for decimal numbers. */
  if (radix != 10)
  {
    *ptr = 0;
    return string;
  }
  if (!value)
  {
    *ptr++ = 0x30;
    *ptr = 0;
    return string;
  }
  /* if this is a negative value insert the minus sign. */
  if (value < 0)
  {
    *ptr++ = '-';
    /* Make the value positive. */
    value *= -1;
  }
  for (i = 10000; i > 0; i /= 10)
  {
    d = value / i;
    if (d || flag)
    {
      *ptr++ = (char)(d + 0x30);
      value -= (d * i);
      flag = 1;
    }
  }
  /* Null terminate the string. */
  *ptr = 0;
  return string;
} 
```

##### 2.2 接收程序文件

中断处理文件`gd32f10x_it.h`、`gd32f10x_it.c`

`gd32f10x_it.c`

```c
#include "gd32f10x_it.h"

extern usart_state_t usart2_state_machine;
extern usart_state_t usart1_state_machine;
uint16_t usart2_rec_time=0;                     // 中断2的接收时间
uint16_t usart1_rec_time=0;                     // 中断1的接收时间

uint16_t Loop_CNT = 0;                           // 循环计数器。每隔25s循环一次
extern uint8_t Timer_OK_Flag;                    // 循环计数标志
extern uint8_t UnvarnishSend_Status_Flag;        // 全局的透传标志，用来标识是否可以透传了。

/**
* USART1中断处理函数
*/
void USART1_IRQHandler( void )
{
  // 这儿主要是为了区分开当前是指令AT的配置阶段，还是透传阶段，AT指令设置阶段，发送的命令和响应不可变。不能加帧头、检验位等。
  // 但是透传阶段，是用户自己的数据，可以做校验和处理。
  if(UnvarnishSend_Status_Flag == UnvarnishSend_Closed)  // 透传未开启
  {
    AT_IRQHandler();
  } else {                                               // 开启透传
    NO_AT_IRQHandler();
  }
}

/**
* AT指令配置时中断处理进入这里。
*/
void AT_IRQHandler(void)
{
   uint8_t ucCh;
  
  if(usart_flag_get( USART1, USART_FLAG_RBNE ) != RESET )
  {
    ucCh  = usart_data_receive( USART1 );
    if(ESP8266_Fram_Record_Struct.InfBit.FramLength < ( RX_BUF_MAX_LEN - 1 ) )
    {
      //预留1个字节写结束符
      ESP8266_Fram_Record_Struct.Data_RX_BUF[ ESP8266_Fram_Record_Struct.InfBit.FramLength++ ]  = ucCh;	
    }
  }
  
  if( usart_flag_get( USART1, USART_FLAG_IDLEF ) == SET ) //数据帧接收完毕
  {
    ESP8266_Fram_Record_Struct.InfBit.FramFinishFlag = 1;
    ucCh = usart_data_receive( USART1 );         //由软件序列清除中断标志位(先读USART_SR，然后读USART_DR)
    TcpClosedFlag = strstr(ESP8266_Fram_Record_Struct.Data_RX_BUF, "CLOSED\r\n") ? 1 : 0;
  }
}

/**
* AT指令配置完毕，进入透传时，中断处理进入这里
*/
void NO_AT_IRQHandler(void)
{	
  uint8_t Res;
  uint32_t ptr;
  if(usart_flag_get(USART1, USART_FLAG_RBNE)==SET)                    //接收到数据
  {
    Res =usart_data_receive(USART1) & 0xff;
    if(usart1_state_machine.status == USART1_IDLE)                     // 空闲，代表新的一帧开始接收
    {
      if(Res == USART_FRAME_HEAD1)
      {
        usart1_rec_time = 0;                                             // 新的一帧开始时，将时间置为0
        usart1_state_machine.usart_rec_ptr = 0;                          // 当前计数指针指向0地址
        ptr = usart1_state_machine.usart_rec_ptr;                        // 将数据读取的地址指向0
        usart1_state_machine.USART_RX_BUF[ptr] = Res;                    // 开始读取数据
        usart1_state_machine.status = USART1_RECEIVE;                    // 状态置为接收中
        usart1_state_machine.usart_rec_ptr++;                            // 计数指针+1
      }
    } 
    else if (usart1_state_machine.status == USART1_RECEIVE)                 // 传输中，代表当前帧未传输完成
    {
      ptr = usart1_state_machine.usart_rec_ptr;                        // 将当前数据的地址赋值给读取数据的地址
      usart1_state_machine.USART_RX_BUF[ptr] = Res;                    // 读取数据
      ptr = usart1_state_machine.usart_rec_ptr++;                      // 计数指针+1。并且赋值给当前的读取数据地址的指针
      if((usart1_state_machine.USART_RX_BUF[0]==USART_FRAME_HEAD1) 
         && (usart1_state_machine.USART_RX_BUF[1]==USART_FRAME_HEAD2)) // 判断帧头正确
      {
        if(ptr == (usart1_state_machine.USART_RX_BUF[3]))              // 如果帧的长度和当前计数指针相同，代表当前帧已经传输完成
        {
          usart1_state_machine.status = USART1_RECEIVE_OK;             // 将状态机置为传输完成
        }
      } 
      else 
      {                                                         // 帧头错误，清空结构体，状态机变为空闲状态
        usart1_state_machine.status = USART1_IDLE; 
        usart1_state_machine.usart_rec_ptr = 0;
        memset(usart1_state_machine.USART_RX_BUF,0,usart1_state_machine.usart_rec_ptr);     // 数据清空
      }
    }
  }
}


/**
* USART2中断处理函数
*/
void USART2_IRQHandler(void)
{	
  uint8_t Res;
  uint32_t ptr;
  if(usart_flag_get(USART2, USART_FLAG_RBNE)==SET)                    //接收到数据
  {
    Res =usart_data_receive(USART2) & 0xff;
    if(usart2_state_machine.status == USART2_IDLE)                     // 空闲，代表新的一帧开始接收
    {
      usart2_rec_time = 0;                                             // 新的一帧开始时，将时间置为0
      usart2_state_machine.usart_rec_ptr = 0;                          // 当前计数指针指向0地址
      ptr = usart2_state_machine.usart_rec_ptr;                        // 将数据读取的地址指向0
      usart2_state_machine.USART_RX_BUF[ptr] = Res;                    // 开始读取数据
      usart2_state_machine.status = USART2_RECEIVE;                    // 状态置为接收中
      usart2_state_machine.usart_rec_ptr++;                            // 计数指针+1
    } 
    else if (usart2_state_machine.status == USART2_RECEIVE)            // 传输中，代表当前帧未传输完成          
    {
      ptr = usart2_state_machine.usart_rec_ptr;                        // 将当前数据的地址赋值给读取数据的地址
      usart2_state_machine.USART_RX_BUF[ptr] = Res;                    // 读取数据
      ptr = usart2_state_machine.usart_rec_ptr++;                      // 计数指针+1。并且赋值给当前的读取数据地址的指针
      if((usart2_state_machine.USART_RX_BUF[0]==USART_FRAME_HEAD1) 
         && (usart2_state_machine.USART_RX_BUF[1]==USART_FRAME_HEAD2)) // 判断帧头正确
      {
        if(ptr == (usart2_state_machine.USART_RX_BUF[3]))              // 如果帧的长度和当前计数指针相同，代表当前帧已经传输完成
        {
          usart2_state_machine.status = USART2_RECEIVE_OK;             // 将状态机置为传输完成
        }
      }
    }
  }
}

/**
* 系统时钟中断
*/
void SysTick_Handler(void)
{
  usart2_rec_time++;  // usart接收数据的时间
  usart1_rec_time++;
  delay_decrement();
  if (Loop_CNT > 24)           // 25ms
  {
    Loop_CNT = 0;
    Timer_OK_Flag = 1;          // 隔25s将Timer_OK_Flag标志位置为1
  }
}
```

##### 2.3 透传发送文件

`data_usart.h`和`data_usart.c`

头文件`data_usart.h`

```c
#ifndef DATA_USART_H
#define DATA_USART_H

#include "util.h"
#include "string.h"
#include <stdbool.h>

#define RX_BUF_MAX_LEN_USART    200		                        //最大接收数据字节数
#define USART_REC_LEN     200		                        //最大接收缓存字节数

/**
* USART接收状态定义
*/
#define USART2_IDLE  			0      // 空闲状态
#define USART2_SEND  			1      // 发送
#define USART2_RECEIVE  	        2      // 接收中
#define USART2_RECEIVE_OK  	        3      // 接收完成

#define USART1_IDLE  			0      // 空闲状态
#define USART1_SEND  			1      // 发送
#define USART1_RECEIVE  	        2      // 接收中
#define USART1_RECEIVE_OK  	        3      // 接收完成

#define USART_REC_TIMEOUT  	        8000   // 4S 每一帧的传输最大时间
#define SEND_TIME_INTV                  2000   // 1s 两个帧之间的最大间隔时间


#define USART_FRAME_HEAD1   0xA5                                 // 帧头开始字段1
#define USART_FRAME_HEAD2   0x5A                                 // 帧头开始字段2
#define USART_FRAME_END    0x7D                                  // 帧尾结束字段

#pragma pack(1)

// USART接收时的状态结构体
typedef struct  USART_STATE
{
  uint8_t   status;                          // 状态
  uint32_t  usart_rec_ptr;                   // 接收对应帧地址
  uint8_t USART_RX_BUF[USART_REC_LEN];       // 接收缓冲,最大USART_REC_LEN个字节.
} usart_state_t, *usart_state_tp;

// USART接收时的数据结构体
typedef struct DATA_FRAME
{
  uint8_t start[2];                       // 帧头
  uint8_t cmd;                            // 命令字
  uint8_t length;                         // 数据长度
  uint8_t data[RX_BUF_MAX_LEN_USART];     // 数据区
  uint8_t check[2];                       // 数据校验
  uint8_t end;                            // 帧尾
} usart_data_frame; 
#pragma pack()

// USART发送的命令
typedef enum
{
  CMD_TEST_OK = 0x55,           
  CMD_TEST_TRANS = 0xAA,            
} ENUM_CMD_USART;


/**
* 函数定义
*/
bool Data_Handler(unsigned char * buf, uint32_t length);
void Send(uint8_t *data, uint8_t len);
void Build_Fram_Data(uint8_t cmd,char *datas,uint8_t len);
void usart1_receive_task(void);
void usart2_receive_task(void);

#endif
```



源文件`data_usart.c`

```c
#include "data_usart.h"

usart_state_t usart2_state_machine;                               // 定义一个全局的状态结构体来控制usart2串口传输的状态机变化 
usart_state_t usart1_state_machine;                               // 定义一个全局的状态结构体来控制usart1串口传输的状态机变化
extern uint16_t usart2_rec_time;                                  // 一个帧的数据接收时间
extern uint16_t usart1_rec_time;                                  // 一个帧的数据接收时间

/**
* usart2接收数据的任务
*/
void usart2_receive_task(void)
{
  uint32_t length = 0;
  if(usart2_state_machine.status == USART2_RECEIVE_OK)             // 判断当前状态机是传输完成。做数据处理
  {
    length = usart2_state_machine.usart_rec_ptr;                   // 数据长度等于计数指针，或者USART_RX_BUF[3]
    
    while(Data_Handler(usart2_state_machine.USART_RX_BUF,length))
    {
        usart2_state_machine.status = USART2_IDLE;                 // 状态机置为空闲状态              
        usart2_rec_time = 0;                                       // 清空接收时间
        usart2_state_machine.usart_rec_ptr = 0;                    // 指针清空
        memset(usart2_state_machine.USART_RX_BUF,0,USART_REC_LEN); // 数据清空
    }
  }
  if(usart2_rec_time >= USART_REC_TIMEOUT)                         // 当接收时间大于等于超时时间时代表当前传输的帧长度已超过规定时间    
  {
    usart2_rec_time = 0;                                            // 清空接收时间
    usart2_state_machine.status = USART2_IDLE;                      // 态机置为空闲状态 
    usart2_state_machine.usart_rec_ptr = 0;                         // 指针清空
  } 
}

/**
* usart1接收数据的任务
*/
void usart1_receive_task(void)
{
  uint32_t length = 0;
  if(usart1_state_machine.status == USART1_RECEIVE_OK)             // 判断当前状态机是传输完成。做数据处理
  {
    length = usart1_state_machine.usart_rec_ptr;                   // 数据长度等于计数指针，或者USART_RX_BUF[3]
    
    while(Data_Handler(usart1_state_machine.USART_RX_BUF,length))
    {
        usart1_state_machine.status = USART1_IDLE;                 // 状态机置为空闲状态              
        usart1_rec_time = 0;                                       // 清空接收时间
        usart1_state_machine.usart_rec_ptr = 0;                    // 指针清空
        memset(usart1_state_machine.USART_RX_BUF,0,USART_REC_LEN); // 数据清空
    }
  }
  if(usart1_rec_time >= USART_REC_TIMEOUT)                         // 当接收时间大于等于超时时间时代表当前传输的帧长度已超过规定时间    
  {
    usart1_rec_time = 0;                                            // 清空接收时间
    usart1_state_machine.status = USART1_IDLE;                      // 态机置为空闲状态 
    usart1_state_machine.usart_rec_ptr = 0;                         // 指针清空
  } 
}

/**
* 对数据进行处理
* buf：接收的数据，总数据长度
* 返回 1：表示处理成功， 0：表示处理失败
*/
bool Data_Handler(unsigned char * buf, uint32_t length)
{
  // todo 数据解析和处理
  return true;
}


/**
* 一个字节一个字节发送数据
*/
void Send(uint8_t *data, uint8_t len)
{
  uint8_t i;
  for(i = 0; i < len; i++)
  {
    usart_data_transmit(USART1,data[i]);  // 发送一个字节
  }
}

/**
* 按照一定的格式组装发送数据 
* cmd: 命令，data:数据，len：数据的长度
*/
void Build_Fram_Data(uint8_t cmd,char *datas,uint8_t len)
{
    uint8_t buf[100],i,cnt=0;
    uint16_t crc16;
    memset(buf,0,100);                // 清空上一次的buf的数据
    buf[cnt++] = USART_FRAME_HEAD1;   // 帧头第一个字节
    buf[cnt++] = USART_FRAME_HEAD2;   // 帧头第二个字节
    buf[cnt++] = cmd;                 // 命令
    buf[cnt++] = len;                 // 数据长度
    for(i=0;i<len;i++)
    {
        buf[cnt++] = datas[i];        // 数据
    }
    crc16 = CRC16(buf,len);         
    buf[cnt++] = (crc16 & 0xFF)>>8;   // CRC16校验码的高位字节
    buf[cnt++] = crc16 & 0xFF;        // CRC16校验码的低位字节
    Send(buf,cnt);
}
```

##### 2.4 主函数测试文件

`main.c`

```c
#include "conf.h"

#define User_ESP8266_SSID	  "liupeng"	      //要连接的热点的名称
#define User_ESP8266_PWD	  "12345678"	      //要连接的热点的密码

#define AP_ESP8266_SSID	  "zhangsan"	              //配置的AP的SSID
#define AP_ESP8266_PWD	  "12345678"	              //配置的AP的密码

#define AP_ESP8266_SERVER_PORT       "1888"           // 配置的AP的服务端口

#define TCP_Server_Ip                "192.168.4.2"      // TCP服务器的IP
#define TCP_Server_Port              "9898"             // TCP服务器的端口


void systick_config(void);
void ESP8266_STA_Test(void);
void ESP8266_AP_Test(void);
void ESP8266_STA_AP_TEST(void);

uint8_t Timer_OK_Flag;
uint8_t TxBuf[2] = {0xD5,0x25};
uint8_t UnvarnishSend_Status_Flag;                   // 全局的透传标志，用来标识是否可以透传了。
/**
* 实现GD32F103和ESP8826之间的WIFI通信
*/
int main(void)
{
  
  systick_config();                  // 系统时钟配置
  usart1_config();                   // USART1初始化配置
  
  ESP8266_CH_PD_Pin_SetH;            // CH_PD拉高，使能
  ESP8266_Rst();                     // 重启
  delay_1ms(2000);                   // 延时2s
//  ESP8266_Exit_UnvarnishSend();
  ESP8266_RESTORE();
  delay_1ms(4000);                   // 延时2s
  ESP8266_STA_AP_TEST();             // AP+STA模式透传设置
  // 当透传通信标识打开时，做透传工作。
//  while(UnvarnishSend_Status_Flag == UnvarnishSend_Open)
//  {
//    // todo 透传 此时ESP8266是在透传设置下，此时传输的数据就是我们自己的数据，跟AT指令没关系
//    Build_Fram_Data(CMD_TEST_TRANS,(char *)TxBuf,2);
//    delay_1ms(1000);
//    UnvarnishSend_Status_Flag = UnvarnishSend_Closed;
//    ESP8266_Exit_UnvarnishSend();
//  }
  
  //  ESP8266_STA_Test();
  //  ESP8266_STA_AP_TEST();
  //  ESP8266_AP_Test();
  //  while(1)
  //  {
  //    usart1_receive_task();
  //    if(ESP8266_AT_Test())
  //    {
  //      break;
  //    }
  //  }
} 

/**
* AP+STA模式透传
* 这种模式是使用ESP8266的AP模式。发出热点，PC或者其他设备连上ESP8266的热点。然后ESP8266作为TCP Client，其他设备作为TCP Server进行单连接透传
* 1：测试连接AT
* 2：设置AP+STA模式 AT+CWMODE=3 
* 3：配置AP AT+CWSAP="zhangsan","12345678",2,4
* 4：开启透传 AT+CIPMODE=1
* 5：单连接模式 AT+CIPMUX=0
* 6：连接tcp server服务器端 AT+CIPSTART="TCP","192.168.4.2",9898
* 7：进入透传 AT+CIPSEND
* 8：关闭透传  发送+++
*/
void ESP8266_STA_AP_TEST(void)
{
  UnvarnishSend_Status_Flag = UnvarnishSend_Closed;   // 透传标识刚开始置为0
  if(ESP8266_AT_Test())  // 测试AT连接ESP8266
  {
    if(ESP8266_Net_Mode_Choose(STA_AP) == true) // 设置模式为STA+AP
    {
      if(ESP8266_Create_AP(AP_ESP8266_SSID,AP_ESP8266_PWD,2, WPA_WPA2_PSK) == true) // 设置AP模式的参数
      {
        if(ESP8266_OPEN_UnvarnishSend() == true)  // 开启透传
        {
          if(ESP8266_Enable_MultipleId (DISABLE) == true) // 单连接
          {
            if(ESP8266_Connect_Tcp_Server(TCP_Server_Ip, TCP_Server_Port) == true) // 连接tcp 服务器
            {
              if(ESP8266_OPEN_UnvarnishSend() == true)      //开启透传
              {
                UnvarnishSend_Status_Flag = UnvarnishSend_Open;   // 配置好之后将状态标志置为1。代表透传完全配置好。
              }
            }
          }
        }
      }
    } 
  }
}


/**
* 测试AP模式
* 1：测试连接 AT 2:设置AP模式 AT+CWMODE=2  3:配置AP AT+CWSAP="zhangsan","12345678",1,3
* 4: 开启多路连接 AT+CIPMUX=1 5：设置端口号 AT+CIPSERVER=1,1888 
*/
void ESP8266_AP_Test(void)
{
  while(ESP8266_AT_Test())  // 测试AT连接ESP8266
  {
    while(ESP8266_Net_Mode_Choose(AP)) // 设置模式
    {
      while(ESP8266_Create_AP(AP_ESP8266_SSID,AP_ESP8266_PWD,2, WPA2_PSK)) // 设置AP模式的参数
      {
        while(ESP8266_Enable_MultipleId (ENABLE)) // 开启多路连接
        {
           return; 
        }
      }
    }
  }
}

/**
* 测试热点连接
* 1:测试AT连接AT    2：设置STA模式 AT+CWMODE=1     3：连接热点：AT+CWJAP="ssid","password"  4: 关闭多连接 AT+CIPMUX=0
*/
void ESP8266_STA_Test(void)
{
  while(ESP8266_AT_Test())  // 测试AT连接ESP8266
  {
    while(ESP8266_Net_Mode_Choose(STA)) // 设置模式
    {
      while(ESP8266_JoinAP(User_ESP8266_SSID, User_ESP8266_PWD)); // 连接热点
      while(ESP8266_Enable_MultipleId (DISABLE)); // 多连接关闭
    } 
  }
}

```

#### 3、代码解读

##### 3.1 AT指令的发送和接收

首先需要明确发送和接收

对于接收，我们定义了一个结构体来接收ESP8266返回的数据

```c
extern struct STRUCT_USART_Fram	                                //定义一个全局串口数据帧的处理结构体
{
	char Data_RX_BUF[RX_BUF_MAX_LEN];
	union
	{
    	volatile uint16_t InfAll;
    	struct
		{
		  	volatile uint16_t FramLength       :15;                               // 14:0
		  	volatile uint16_t FramFinishFlag   :1;                                // 15
	  	}InfBit;
  	};
}ESP8266_Fram_Record_Struct;
```

这个结构体的Data_RX_BUF用来存储返回的数据。

处理返回是在中断处理函数中实现的。见`gd32f10x_it.c`

```c
/**
* USART1中断处理函数
*/
void USART1_IRQHandler( void )
{
  // 这儿主要是为了区分开当前是指令AT的配置阶段，还是透传阶段，AT指令设置阶段，发送的命令和响应不可变。不能加帧头、检验位等。
  // 但是透传阶段，是用户自己的数据，可以做校验和处理。
  if(UnvarnishSend_Status_Flag == UnvarnishSend_Closed)  // 透传未开启
  {
    AT_IRQHandler();
  } else {                                               // 开启透传
    NO_AT_IRQHandler();
  }
}

/**
* AT指令配置时中断处理进入这里。
*/
void AT_IRQHandler(void)
{
   uint8_t ucCh;
  
  if(usart_flag_get( USART1, USART_FLAG_RBNE ) != RESET )
  {
    ucCh  = usart_data_receive( USART1 );
    if(ESP8266_Fram_Record_Struct.InfBit.FramLength < ( RX_BUF_MAX_LEN - 1 ) )
    {
      //预留1个字节写结束符
      ESP8266_Fram_Record_Struct.Data_RX_BUF[ ESP8266_Fram_Record_Struct.InfBit.FramLength++ ]  = ucCh;	
    }
  }
  
  if( usart_flag_get( USART1, USART_FLAG_IDLEF ) == SET ) //数据帧接收完毕
  {
    ESP8266_Fram_Record_Struct.InfBit.FramFinishFlag = 1;
    ucCh = usart_data_receive( USART1 );         //由软件序列清除中断标志位(先读USART_SR，然后读USART_DR)
    TcpClosedFlag = strstr(ESP8266_Fram_Record_Struct.Data_RX_BUF, "CLOSED\r\n") ? 1 : 0;
  }
}

/**
* AT指令配置完毕，进入透传时，中断处理进入这里
*/
void NO_AT_IRQHandler(void)
{	
  uint8_t Res;
  uint32_t ptr;
  if(usart_flag_get(USART1, USART_FLAG_RBNE)==SET)                    //接收到数据
  {
    Res =usart_data_receive(USART1) & 0xff;
    if(usart1_state_machine.status == USART1_IDLE)                     // 空闲，代表新的一帧开始接收
    {
      if(Res == USART_FRAME_HEAD1)
      {
        usart1_rec_time = 0;                                             // 新的一帧开始时，将时间置为0
        usart1_state_machine.usart_rec_ptr = 0;                          // 当前计数指针指向0地址
        ptr = usart1_state_machine.usart_rec_ptr;                        // 将数据读取的地址指向0
        usart1_state_machine.USART_RX_BUF[ptr] = Res;                    // 开始读取数据
        usart1_state_machine.status = USART1_RECEIVE;                    // 状态置为接收中
        usart1_state_machine.usart_rec_ptr++;                            // 计数指针+1
      }
    } 
    else if (usart1_state_machine.status == USART1_RECEIVE)                 // 传输中，代表当前帧未传输完成
    {
      ptr = usart1_state_machine.usart_rec_ptr;                        // 将当前数据的地址赋值给读取数据的地址
      usart1_state_machine.USART_RX_BUF[ptr] = Res;                    // 读取数据
      ptr = usart1_state_machine.usart_rec_ptr++;                      // 计数指针+1。并且赋值给当前的读取数据地址的指针
      if((usart1_state_machine.USART_RX_BUF[0]==USART_FRAME_HEAD1) 
         && (usart1_state_machine.USART_RX_BUF[1]==USART_FRAME_HEAD2)) // 判断帧头正确
      {
        if(ptr == (usart1_state_machine.USART_RX_BUF[3]))              // 如果帧的长度和当前计数指针相同，代表当前帧已经传输完成
        {
          usart1_state_machine.status = USART1_RECEIVE_OK;             // 将状态机置为传输完成
        }
      } 
      else 
      {                                                         // 帧头错误，清空结构体，状态机变为空闲状态
        usart1_state_machine.status = USART1_IDLE; 
        usart1_state_machine.usart_rec_ptr = 0;
        memset(usart1_state_machine.USART_RX_BUF,0,usart1_state_machine.usart_rec_ptr);     // 数据清空
      }
    }
  }
}
```

每次发送之后返回都会进入中断，在中断函数中将返回值写入全局的ESP8266_Fram_Record_Struct结构体中。



发送的结构就是"AT+XXX"的字符串指令。

所以可以通过构造发送函数，见`ESP8266_driver.c`如下：

```c
struct STRUCT_USART_Fram ESP8266_Fram_Record_Struct = { 0 };   // 初始化串口数据接收

/**
* 对ESP8266模块发送AT指令
* cmd：待发送的指令
* ack1，ack2：期待的响应，为NULL表不需响应，两者为或逻辑关系
* time：等待响应的时间
* 返回 1：发送成功 0：失败
*/
bool ESP8266_Send_AT_Cmd(char *cmd,char *ack1,char *ack2,uint32_t time)
{
  ESP8266_Fram_Record_Struct.InfBit.FramLength = 0;	//从新开始接收新的数据包
  ESP8266_USART("%s\r\n", cmd);
  if(ack1==0&&ack2==0)	 //不需要接收数据
  {
    return true;
  }
  delay_1ms(time);	  //延时time时间
  
  ESP8266_Fram_Record_Struct.Data_RX_BUF[ESP8266_Fram_Record_Struct.InfBit.FramLength ] = '\0';
  
  if(ack1!=0 && ack2!=0)
  {
    return ( ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack1 ) ||
            ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack2 ) );
  }
  else if( ack1 != 0 )
    return ( ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack1 ) );
  
  else
    return ( ( bool ) strstr ( ESP8266_Fram_Record_Struct.Data_RX_BUF, ack2 ) );
}
```

`ESP8266_USART("%s\r\n", cmd)`发送文件在`ESP8266_public.c`中

`#define ESP8266_USART(fmt, ...)	 USART_printf (USART1, fmt, ##__VA_ARGS__)    // 定义ESP8266串口传输`这句是在`ESP8266_public.h`中的宏定义

```c
void USART_printf (uint32_t usart_periph, char * Data, ... )
{
  const char *s;
  int d;
  char buf[16];
  
  va_list ap;
  va_start(ap, Data);
  
  while ( * Data != 0 )     // 判断是否到达字符串结束符
  {				
    if ( * Data == 0x5c )  //'\'
    {									
      switch ( *++Data )
      {
      case 'r':							          //回车符
        usart_data_transmit(usart_periph, 0x0d);
        Data ++;
        break;
        
      case 'n':							          //换行符
        usart_data_transmit(usart_periph, 0x0a);	
        Data ++;
        break;
        
      default:
        Data ++;
        break;
      }			
    }
    
    else if ( * Data == '%')
    {								
      switch ( *++Data )
      {				
      case 's':								  //字符串
        s = va_arg(ap, const char *);
        
        for ( ; *s; s++)
        {
          usart_data_transmit(usart_periph,*s);
          while( usart_flag_get(usart_periph, USART_FLAG_TBE) == RESET );
        }
        
        Data++;
        
        break;
        
      case 'd':			
        //十进制
        d = va_arg(ap, int);
        itoa(d, buf, 10);
        
        for (s = buf; *s; s++)
        {
          usart_data_transmit(usart_periph,*s);
          while( usart_flag_get(usart_periph, USART_FLAG_TBE) == RESET );
        }
        
        Data++;
        
        break;
        
      default:
        Data++;
        
        break;
        
      }		
    }
    else usart_data_transmit(usart_periph, *Data++);
    
    while ( usart_flag_get(usart_periph, USART_FLAG_TBE) == RESET );
  }
}
```

发送和接收都做了理解，那就可以拿一个示例来看。比如

```c
/**
* 对ESP8266模块进行AT测试启动
* 返回 1：测试启动成功，0：超时失败
*/
uint8_t ESP8266_AT_Test(void)
{
  char count=0;	
  ESP8266_CH_PD_Pin_SetH;
  while(count < 10)
  {
    if(ESP8266_Send_AT_Cmd("AT","OK",NULL,2500))
      return 1;
    ++ count;
  }
  return 0;
}
```

这个函数的调用就是实现发送"AT"，然后ESP8266恢复"AT OK"指令的测试函数。

##### 3.2 添加热点

```c
/**
* 测试热点连接
* 1:测试AT连接AT    2：设置STA模式 AT+CWMODE=1     3：连接热点：AT+CWJAP="ssid","password"  4: 关闭多连接 AT+CIPMUX=0
*/
void ESP8266_STA_Test(void)
{
  while(ESP8266_AT_Test())  // 测试AT连接ESP8266
  {
    while(ESP8266_Net_Mode_Choose(STA)) // 设置模式
    {
      while(ESP8266_JoinAP(User_ESP8266_SSID, User_ESP8266_PWD)); // 连接热点
      while(ESP8266_Enable_MultipleId (DISABLE)); // 多连接关闭
    } 
  }
}
```

这个简单，步骤就是

​		测试AT -->设置STA模式   -->连接热点 --> 关闭多连接

连接的热点可以选择手机发出的热点或者办公室的路由器等等。

##### 3.3 作为AP发出热点

作为AP发出热点，即意味着ESP8266将作为路由器。

```c
/**
* 测试AP模式
* 1：测试连接 AT 2:设置AP模式 AT+CWMODE=2  3:配置AP AT+CWSAP="zhangsan","12345678",1,3
* 4: 开启多路连接 AT+CIPMUX=1 5：设置端口号 AT+CIPSERVER=1,1888 
*/
void ESP8266_AP_Test(void)
{
  while(ESP8266_AT_Test())  // 测试AT连接ESP8266
  {
    while(ESP8266_Net_Mode_Choose(AP)) // 设置模式
    {
      while(ESP8266_Create_AP(AP_ESP8266_SSID,AP_ESP8266_PWD,2, WPA2_PSK)) // 设置AP模式的参数
      {
        while(ESP8266_Enable_MultipleId (ENABLE)) // 开启多路连接
        {
           return; 
        }
      }
    }
  }
}
```

步骤：

​		测试连接 AT  --> 设置AP模式 --> 配置AP --> 开启多路连接 

这四步做完即可完成热点的发出。用手机或者PC都可以连上ESP8266的热点

##### 3.4 作为AP发出热点并实现透传

###### 3.4.1 传输数据格式定义

透传是我们的重点。

​		首先需要定义一个全局变量UnvarnishSend_Status_Flag来表示是否打开了透传。透传和非透传我们对数据的处理方式不一样。

因为透传数据我们主要保证数据的准确性，所以需要对数据做一些操作。定义透传串口数据的格式

```
                帧头（2个字节）+命令字（1个字节）+数据长度（1个字节）+数据区（可变，最大100个字节）+校验位（2个字节）
```

```c
// USART接收时的状态结构体
typedef struct  USART_STATE
{
  uint8_t   status;                          // 状态
  uint32_t  usart_rec_ptr;                   // 接收对应帧地址
  uint8_t USART_RX_BUF[USART_REC_LEN];       // 接收缓冲,最大USART_REC_LEN个字节.
} usart_state_t, *usart_state_tp;

// USART接收时的数据结构体
typedef struct DATA_FRAME
{
  uint8_t start[2];                       // 帧头
  uint8_t cmd;                            // 命令字
  uint8_t length;                         // 数据长度
  uint8_t data[RX_BUF_MAX_LEN_USART];     // 数据区
  uint8_t check[2];                       // 数据校验
} usart_data_frame; 
```

这个结构体的status表示状态机，USART_RX_BUF表示接收到的数据，usart_rec_ptr表示接收时的指针指向USART_RX_BUF数据的第几个。

状态总共分为三种

```c
#define USART2_IDLE  			    0      // 空闲状态
#define USART2_RECEIVE  	        2      // 接收中
#define USART2_RECEIVE_OK  	        3      // 接收完成
```

空闲状态：可接收数据，并且第一个字节数据是我们的帧头。

接收中：可接收数据，如果当前usart_state_t的指针值大于usart_data_frame的数据长度，那状态机会立刻变为接收完成。

接收完成：主函数的轮询中会进入处理这一帧数据。处理完成之后状态变为空闲状态。

###### 3.4.2 接收

这个的表现主要体现在我们的中断函数中

```c

/**
* AT指令配置完毕，进入透传时，中断处理进入这里
*/
void NO_AT_IRQHandler(void)
{	
  uint8_t Res;
  uint32_t ptr;
  if(usart_flag_get(USART1, USART_FLAG_RBNE)==SET)                    //接收到数据
  {
    Res =usart_data_receive(USART1) & 0xff;
    if(usart1_state_machine.status == USART1_IDLE)                     // 空闲，代表新的一帧开始接收
    {
      if(Res == USART_FRAME_HEAD1)
      {
        usart1_rec_time = 0;                                             // 新的一帧开始时，将时间置为0
        usart1_state_machine.usart_rec_ptr = 0;                          // 当前计数指针指向0地址
        ptr = usart1_state_machine.usart_rec_ptr;                        // 将数据读取的地址指向0
        usart1_state_machine.USART_RX_BUF[ptr] = Res;                    // 开始读取数据
        usart1_state_machine.status = USART1_RECEIVE;                    // 状态置为接收中
        usart1_state_machine.usart_rec_ptr++;                            // 计数指针+1
      }
    } 
    else if (usart1_state_machine.status == USART1_RECEIVE)                 // 传输中，代表当前帧未传输完成
    {
      ptr = usart1_state_machine.usart_rec_ptr;                        // 将当前数据的地址赋值给读取数据的地址
      usart1_state_machine.USART_RX_BUF[ptr] = Res;                    // 读取数据
      ptr = usart1_state_machine.usart_rec_ptr++;                      // 计数指针+1。并且赋值给当前的读取数据地址的指针
      if((usart1_state_machine.USART_RX_BUF[0]==USART_FRAME_HEAD1) 
         && (usart1_state_machine.USART_RX_BUF[1]==USART_FRAME_HEAD2)) // 判断帧头正确
      {
        if(ptr == (usart1_state_machine.USART_RX_BUF[3]))              // 如果帧的长度和当前计数指针相同，代表当前帧已经传输完成
        {
          usart1_state_machine.status = USART1_RECEIVE_OK;             // 将状态机置为传输完成
        }
      } 
      else 
      {                                                         // 帧头错误，清空结构体，状态机变为空闲状态
        usart1_state_machine.status = USART1_IDLE; 
        usart1_state_machine.usart_rec_ptr = 0;
        memset(usart1_state_machine.USART_RX_BUF,0,usart1_state_machine.usart_rec_ptr);     // 数据清空
      }
    }
  }
}
```

我们数据的发送过程中帧头，命令字都可以提前宏定义出来。

```c
#define USART_FRAME_HEAD1   0xA5                                 // 帧头开始字段1
#define USART_FRAME_HEAD2   0x5A                                 // 帧头开始字段2

// USART发送的命令
typedef enum
{
  CMD_TEST_OK = 0x55,           
  CMD_TEST_TRANS = 0xAA,            
} ENUM_CMD_USART;
```

###### 3.4.3 校验

校验使用的是CRC16校验。具体在这儿不讲CRC校验了。它的实现在`util.c`文件中

```c
#include "util.h"

// CRC校验的高位表
const unsigned char auchCRCHi[] = { 
  0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81,
  0x40, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0,
  0x80, 0x41, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81, 0x40, 0x01,
  0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x01, 0xC0, 0x80, 0x41,
  0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81,
  0x40, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x01, 0xC0,
  0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x01,
  0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40,
  0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81,
  0x40, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0,
  0x80, 0x41, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81, 0x40, 0x01,
  0xC0, 0x80, 0x41, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41,
  0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81,
  0x40, 0x01, 0xC0, 0x80, 0x41, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0,
  0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x01,
  0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81, 0x40, 0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41,
  0x00, 0xC1, 0x81, 0x40, 0x01, 0xC0, 0x80, 0x41, 0x01, 0xC0, 0x80, 0x41, 0x00, 0xC1, 0x81,
  0x40
};

// CRC校验的低位表
const unsigned char auchCRCLo[] = { 
  0x00, 0xC0, 0xC1, 0x01, 0xC3, 0x03, 0x02, 0xC2, 0xC6, 0x06, 0x07, 0xC7, 0x05, 0xC5, 0xC4,
  0x04, 0xCC, 0x0C, 0x0D, 0xCD, 0x0F, 0xCF, 0xCE, 0x0E, 0x0A, 0xCA, 0xCB, 0x0B, 0xC9, 0x09,
  0x08, 0xC8, 0xD8, 0x18, 0x19, 0xD9, 0x1B, 0xDB, 0xDA, 0x1A, 0x1E, 0xDE, 0xDF, 0x1F, 0xDD,
  0x1D, 0x1C, 0xDC, 0x14, 0xD4, 0xD5, 0x15, 0xD7, 0x17, 0x16, 0xD6, 0xD2, 0x12, 0x13, 0xD3,
  0x11, 0xD1, 0xD0, 0x10, 0xF0, 0x30, 0x31, 0xF1, 0x33, 0xF3, 0xF2, 0x32, 0x36, 0xF6, 0xF7,
  0x37, 0xF5, 0x35, 0x34, 0xF4, 0x3C, 0xFC, 0xFD, 0x3D, 0xFF, 0x3F, 0x3E, 0xFE, 0xFA, 0x3A,
  0x3B, 0xFB, 0x39, 0xF9, 0xF8, 0x38, 0x28, 0xE8, 0xE9, 0x29, 0xEB, 0x2B, 0x2A, 0xEA, 0xEE,
  0x2E, 0x2F, 0xEF, 0x2D, 0xED, 0xEC, 0x2C, 0xE4, 0x24, 0x25, 0xE5, 0x27, 0xE7, 0xE6, 0x26,
  0x22, 0xE2, 0xE3, 0x23, 0xE1, 0x21, 0x20, 0xE0, 0xA0, 0x60, 0x61, 0xA1, 0x63, 0xA3, 0xA2,
  0x62, 0x66, 0xA6, 0xA7, 0x67, 0xA5, 0x65, 0x64, 0xA4, 0x6C, 0xAC, 0xAD, 0x6D, 0xAF, 0x6F,
  0x6E, 0xAE, 0xAA, 0x6A, 0x6B, 0xAB, 0x69, 0xA9, 0xA8, 0x68, 0x78, 0xB8, 0xB9, 0x79, 0xBB,
  0x7B, 0x7A, 0xBA, 0xBE, 0x7E, 0x7F, 0xBF, 0x7D, 0xBD, 0xBC, 0x7C, 0xB4, 0x74, 0x75, 0xB5,
  0x77, 0xB7, 0xB6, 0x76, 0x72, 0xB2, 0xB3, 0x73, 0xB1, 0x71, 0x70, 0xB0, 0x50, 0x90, 0x91,
  0x51, 0x93, 0x53, 0x52, 0x92, 0x96, 0x56, 0x57, 0x97, 0x55, 0x95, 0x94, 0x54, 0x9C, 0x5C,
  0x5D, 0x9D, 0x5F, 0x9F, 0x9E, 0x5E, 0x5A, 0x9A, 0x9B, 0x5B, 0x99, 0x59, 0x58, 0x98, 0x88,
  0x48, 0x49, 0x89, 0x4B, 0x8B, 0x8A, 0x4A, 0x4E, 0x8E, 0x8F, 0x4F, 0x8D, 0x4D, 0x4C, 0x8C,
  0x44, 0x84, 0x85, 0x45, 0x87, 0x47, 0x46, 0x86, 0x82, 0x42, 0x43, 0x83, 0x41, 0x81, 0x80,
  0x40
};
/*------------------------------------------------------------------------------
* 函数名称： unsigned short CRC16( unsigned char *puchMsg, unsigned short usDataLen )
* 函数功能： 带多项式的crc16校验
* 参数：     puchMsg  要进行CRC校验的消息, usDataLen 消息中字节数
* 返回值：   crc16校验
*-------------------------------------------------------------------------------*/
unsigned short CRC16( unsigned char *puchMsg, unsigned short usDataLen ) 
{ 
  unsigned char uchCRCHi = 0xFF ;  /* high byte of CRC initialized   */
  unsigned char uchCRCLo = 0xFF ;  /* low byte of CRC initialized   */ 
  unsigned uIndex ;  /* will index into CRC lookup table   */  /* CRC循环中的索引*/
 	while (usDataLen--)  /* pass through message buffer   */
  { 
     uIndex = uchCRCLo ^ *puchMsg++ ;  /* calculate the CRC   */ 
     uchCRCLo = uchCRCHi ^ auchCRCHi[uIndex] ; 
     uchCRCHi = auchCRCLo[uIndex] ; 
  } 
  return (unsigned short)(uchCRCHi << 8 | uchCRCLo) ; 
}


/**
* 发送方：获取和校验的校验值
* buffer：发送的数组
* len：数组长度
* 返回：校验值
*/
uint8_t TX_CHECK_Sum(uint8_t * buffer,uint8_t len)
{
  uint8_t i,value = 0;
  for(i = 0; i < len; i++)
  {
    value += *(buffer++); 
  }
  value = ~value;
  return value;
}

/**
* 接收方：校验收到的数据是否符合和检验
* buffer：收到的数组
* len：数组长度
*/
uint8_t RX_CHECK_Sum(uint8_t * buffer,uint8_t len)
{
  uint8_t i,value = 0;
  for(i = 0; i < len; i++)
  {
    value += *(buffer++); 
  }
  return value+1;
}
```

###### 3.4.4 发送

发送需要先构造数据发送格式，再通过串口发送,可见`data_usart.c`

```c

/**
* 一个字节一个字节发送数据
*/
void Send(uint8_t *data, uint8_t len)
{
  uint8_t i;
  for(i = 0; i < len; i++)
  {
    usart_data_transmit(USART1,data[i]);  // 发送一个字节
  }
}

/**
* 按照一定的格式组装发送数据 
* cmd: 命令，data:数据，len：数据的长度
*/
void Build_Fram_Data(uint8_t cmd,char *datas,uint8_t len)
{
    uint8_t buf[100],i,cnt=0;
    uint16_t crc16;
    memset(buf,0,100);                // 清空上一次的buf的数据
    buf[cnt++] = USART_FRAME_HEAD1;   // 帧头第一个字节
    buf[cnt++] = USART_FRAME_HEAD2;   // 帧头第二个字节
    buf[cnt++] = cmd;                 // 命令
    buf[cnt++] = len;                 // 数据长度
    for(i=0;i<len;i++)
    {
        buf[cnt++] = datas[i];        // 数据
    }
    crc16 = CRC16(buf,len);         
    buf[cnt++] = (crc16 & 0xFF)>>8;   // CRC16校验码的高位字节
    buf[cnt++] = crc16 & 0xFF;        // CRC16校验码的低位字节
    Send(buf,cnt);
}
```

###### 3.4.5 透传的测试文件

​		主函数中我们通过调用`ESP8266_STA_AP_TEST()`透传设置，将全局标量UnvarnishSend_Status_Flag置为1。代表透传打开。

然后调用Build_Fram_Data函数将测试数据发送。发送完成之后关闭透传模式

```c
int main(void)
{
  systick_config();                  // 系统时钟配置
  usart1_config();                   // USART1初始化配置
  
  ESP8266_CH_PD_Pin_SetH;            // CH_PD拉高，使能
  ESP8266_Rst();                     // 重启
  delay_1ms(1000);                   // 延时1s
  ESP8266_RESTORE();
  delay_1ms(1000);                   // 延时1s
  ESP8266_STA_AP_TEST();             // AP+STA模式透传设置
  // 当透传通信标识打开时，做透传工作。
  while(UnvarnishSend_Status_Flag == UnvarnishSend_Open)
  {
    // todo 透传 此时ESP8266是在透传设置下，此时传输的数据就是我们自己的数据，跟AT指令没关系
    Build_Fram_Data(CMD_TEST_TRANS,(char *)TxBuf,2);
    delay_1ms(1000);
    UnvarnishSend_Status_Flag = UnvarnishSend_Closed;
    ESP8266_Exit_UnvarnishSend();
  }
} 
/**
* AP+STA模式透传
* 这种模式是使用ESP8266的AP模式。发出热点，PC或者其他设备连上ESP8266的热点。然后ESP8266作为TCP Client，其他设备作为TCP Server进行单连接透传
* 1：测试连接AT
* 2：设置AP+STA模式 AT+CWMODE=3 
* 3：配置AP AT+CWSAP="zhangsan","12345678",2,4
* 4：开启透传 AT+CIPMODE=1
* 5：单连接模式 AT+CIPMUX=0
* 6：连接tcp server服务器端 AT+CIPSTART="TCP","192.168.4.2",9898
* 7：进入透传 AT+CIPSEND
* 8：关闭透传  发送+++
*/
void ESP8266_STA_AP_TEST(void)
{
  UnvarnishSend_Status_Flag = UnvarnishSend_Closed;   // 透传标识刚开始置为0
  if(ESP8266_AT_Test())  // 测试AT连接ESP8266
  {
    if(ESP8266_Net_Mode_Choose(STA_AP) == true) // 设置模式为STA+AP
    {
      if(ESP8266_Create_AP(AP_ESP8266_SSID,AP_ESP8266_PWD,2, WPA_WPA2_PSK) == true) // 设置AP模式的参数
      {
        if(ESP8266_OPEN_UnvarnishSend() == true)  // 开启透传
        {
          if(ESP8266_Enable_MultipleId (DISABLE) == true) // 单连接
          {
            if(ESP8266_Connect_Tcp_Server(TCP_Server_Ip, TCP_Server_Port) == true) // 连接tcp 服务器
            {
              if(ESP8266_OPEN_UnvarnishSend() == true)      //开启透传
              {
                UnvarnishSend_Status_Flag = UnvarnishSend_Open;   // 配置好之后将状态标志置为1。代表透传完全配置好。
              }
            }
          }
        }
      }
    } 
  }
}
```

透传的设置步骤：

```
       测试连接 --> 设置AP+STA模式 --> 配置AP --> 开启透传 --> 单连接模式 --> 连接tcp server服务器端 --> 进入透传
```