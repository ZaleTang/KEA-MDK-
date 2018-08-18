# KEA-MDK-
# KEA128芯片电磁组用底层并包含了基本的函数
# 2018.7.29 by Zamir

	文件说明：

	一，drivers，存放的是芯片个功能的驱动文件，其中：
		1，adc adc的驱动，文件中注明了各个adc通道对应的引脚，对于其中各个函数的功能注释中均有详细说明。
		2，ftm 主要是pwm相关的驱动，其中也注明了各个ftm通道的引脚，其中函数说明注释之述备矣；
		*在FTM_PWM_Duty函数中，有一行ASSERT(duty <= FTM_PRECISON)，assert的功能是如果条件不满足，就会跳进assert_failed函数，会导致整个程序卡死在assert_failed函数中。这里采用该函数的主要原因是防止所给占空比过大使程序异常。若程序运行时异常卡死，可以找找看是不是所给占空比过大导致。
		3，gpio gpio驱动，其中函数说明注释之述备矣；
		4，pit 定时器驱动，其中函数说明注释之述备矣；
		5，systick 系统滴答计时器驱动，其中有两个1us延时程序。需要注意的是这里的延时函数优先级极高，在调用此处的延时函数时，整个系统是完全停止运行直道延时结束的，包括中断也不会被响应，慎用；
		6，uart 串口驱动，其中注明了串口的默认引脚与重映射引脚，其中函数说明注释之述备矣；

	二，Pherpherals，存放的是外设的驱动文件，其中：
		1，LQ12864 屏幕的驱动，其中主要函数有以下：
			1)，LCD_Init,用于初始化，其中端口初始化部分里初始化的端口为实际板子上连接的端口。若需要在程序中启用屏幕，则必须先调用该函数对其进行初始化。
			2），LCD_PutPixel，在屏幕上打印一个点，具体参数注释中有说明。
			3），LCD_P6x8Str，LCD_P8x16Str等在注释中说明了参数，分别为xy坐标以及显示的字符串，注意第三个参数是一个字符串，也就是一个字符数组。如需要显示特定参数，需要用sprintf函数整合字符串，sprintf函数与printf函数接近。举例说明：
			
				unsigned char a[30]={"0"};
				int y=2007;
				LCD_Init();
				void function()
				{
					...
					sprintf((char*)a,"Robot Asociation was found at %d04",y);
					LCD_P6x8Str(1,1,a);
				}

				则结果是会在屏幕上显示 
				Robot Asociation was found at 2007

			2，LQKEY，按键及拨码开关驱动，其中函数主要有
				1),KEY_Init,按键初始化，其中的引脚应对应实际板子引脚；
				2),KEY_Read,参数为KEY0-KEY4的枚举类型，用于读取按键引脚的电平状态，返回为0时为低电平，1为高电平；
				3),KEY1_Init,拨码开关初始化，其中引脚也应对应实际板子引脚，还需设置引脚上拉；
				4),KEY1_Read,参数为KEY0-KEY4的枚举类型，与KEY_Read一样，用于读取连接拨码开关引脚的电平状态；
				*在LQKEY.h中，我们可以看到KEYn_e和KEYs_e两个枚举类型，可以看出这两个枚举类型的定义只是把01234用了其他方式表示出来了而已；

			4，LQLED，板载LED灯驱动，其中函数主要有：
				1),LED_Init,LED引脚初始化，这在龙邱的核心板上是固定死的，其他核心板上可能不一样，要根据实际情况却定引脚；
				2),LED_Ctrl,LED控制，其中第一个参数为LED0-LED4以及LEDALL，同样是枚举类型，参照前面的KEY01234可以读懂，第二个是状态控制，分别为LEDON，LEDOF，LEDRVS，分别对应开，关，取反；
		
	三，common，CPU，CMSIS 其中common里的assert里面的函数在之前有介绍。其他为底层内核函数，包括开机底层初始化等，早期学习（大学阶段）不必去了解；

	四，RAbuild，存放的是智能车竞赛相关的代码，也是最顶层的代码，是调车阶段需要去添加或改动的地方，其中：
		1，System 存放的是系统初始化相关的函数，其中：
			1),SystermInit 全局驱动初始化，包括其他所有模块的初始化函数，通过调用本函数实现对所有模块的初始化；
			2),DATAinit 数据初始化，对所有外部变量进行初始化操作；
			3),FTM_init 对pwm相关外设进行初始化，主要有舵机，电机，编码器等；
	
		2，isr 存放的是中断执行的函数，其中：
			1),UART0(/1/2)_IRQHandle 存放的是串口中断执行的函数，在串口收到数据时触发并执行；
			2),PIT_CH0(/1)_IRQHandler 存放的是定时器中断执行的函数，本工程中启用了定时器0中断，将DirectionControl函数放在里面。这也就是说整个程序都是靠PIT0中断的不断响应完成的；
	
		3，getsig 存放的是获取传感器数据的函数，其中的getsig函数是主体，具体功能注释中有说明，要注意的是该函数的参数是一个数组的名字，也就是把数组的首地址作为参数传递进入，需要指针相关的知识来理解本函数。另外在getsig.h中，将各个adc通道读取程序通过宏定义订成了AD0-5，需要注意的是此处的AD0-5与实际的AD通道0-5并不对应；
		
		4，Motor 存放的是电机控制相关的函数，包括舵机，电机，注释之述备矣；

		5，deal 核心文件，存放的是DirectionControl函数，里面是控制车的核心代码，注释中对代码功能进行了少量说明；

		6，main 主函数所在位置，程序的入口位置
	
	由于本工程编写时间较早且为原始测试所用，与现在的实际情况可能略有出入，务必结合实际情况对相关部分进行修改。
