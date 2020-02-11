## IIC总线传输协议

*时钟线scl    数据线sda*
*当SCL为高电平期间，数据线上的数据必须保持稳定；SCL为低电平期间SDA允许变化*

### IIC的起始与终止信号

​	IIC的初始化要求SDA与SCL均为高点平；之后SCL为高电平期间，SDA线由高电平向低电平的变化表示起始信号；SCL为高电平期间，SDA线由低电平向高电平的变化表示终止信号。
![image-20200204212048765](D:%5C%E5%B0%8F%E5%87%AF%E6%96%87%5CDocuments%5CIIC%E9%80%9A%E4%BF%A1EEPROM.assets%5Cimage-20200204212048765.png)

```cpp
/*I2C初始化*/
void I2C_init()	
{
	SDA = 1;
	_nop_();
	SCL = 1;
	_nop_();
}

/*I2C起始信号*/
void I2C_Start()  
{
	SCL = 1;
	_nop_();
	SDA = 1;
	delay_5us();
	SDA = 0;
	delay_5us();
}

/*I2C终止信号*/
void I2C_Stop()
{
	SDA = 0;
	_nop_();
	SCL = 1;
	delay_5us();
	SDA = 1;
	delay_5us();
}
```



 ### IIC字节的传送与应答

​	每一个字节必须保证长度是8位。数据传送时，先传送最高位（MSB），每一个被传送的字节后面都必须跟随一位应答位（即一帧共有9位）

![image-20200204212759542](D:%5C%E5%B0%8F%E5%87%AF%E6%96%87%5CDocuments%5CIIC%E9%80%9A%E4%BF%A1EEPROM.assets%5Cimage-20200204212759542.png)

```cpp
/*主机发送应答*/
void Master_ACK(bit i)		
{
	SCL = 0; // 拉低时钟总线允许SDA数据总线上的数据变化
	_nop_(); // 让总线稳定
	if (i)	 //如果i = 1 那么拉低数据总线 表示主机应答
	{
		SDA = 0;
	}
	else	 
	{
		SDA = 1;	 //发送非应答
	}
	_nop_();//让总线稳定
	SCL = 1;//拉高时钟总线 让从机从SDA线上读走 主机的应答信号
	delay_5us();
	SCL = 0;//拉低时钟总线， 占用总线继续通信
	_nop_();
	SDA = 1;//释放SDA数据总线。
	_nop_();
}

/*检测从机应答*/
bit Test_ACK()
{
	SCL = 1;
	delay_5us();
	if (SDA)
	{
		SCL = 0;
		_nop_();
		I2C_Stop();//发送终止信号
		return(0);
	}
	else
	{
		SCL = 0;
		_nop_();
		return(1);
	}
}
```



 ###  IIC写数据流程

首先发送起始信号，在起始信号后必须传送一个从机的地址（7位）第八位是数据的传送方向位（R/T）用0表示主机发送数据（T），1表示主机接收数据（R），然后指定从哪一个位置开始写。

### IIC读数据流程

​	在读取数据时也要先发送器件地址，读写方向为写（**即第8位为0**），从机发送应答之后主机指定从哪一个数据开始读，从机再次应答主机再次发送起始信号再次发送从机地址，这次读写位为读（**即第八位为1**），从机应答后就可以从总线上读取数据，从机每发送一帧数据主机需要应答，主机希望停止读取时先发送非应答，再发送停止信号。

### 软件模拟IIC通信时序

51单片机不具有IIC接口，所以与这种接口的器件通讯时需要软件模拟IIC，IIC总线的数据传送有严格的时序要求，如图

![image-20200205212338955](D:%5C%E5%B0%8F%E5%87%AF%E6%96%87%5CDocuments%5CIIC%E9%80%9A%E4%BF%A1EEPROM.assets%5Cimage-20200205212338955.png)

```cpp
/*发送一个字节*/
void I2C_send_byte(uchar byte)
{
	uchar i;
	for(i = 0 ; i < 8 ; i++)
	{
		SCL = 0;
		_nop_();
		if (byte & 0x80)//取最高位字节
		{				
			SDA = 1;	
			_nop_();				   
		}				
		else
		{
			SDA = 0;
			_nop_();
		}
		SCL = 1;
		_nop_();
		byte <<= 1;	// 0101 0100B 
	}
	SCL = 0;
	_nop_();
	SDA = 1;
	_nop_();
}


/*I2C 读一字节*/
uchar I2C_read_byte()
{
	uchar dat,i;
	SCL = 0;
	_nop_();
	SDA = 1;
	_nop_();
	for(i = 0 ; i < 8 ; i++)
	{
		SCL = 1;
		_nop_();
		if (SDA)			    
		{
			 dat |= 0x01; //
		}
		else
		{
			dat &=  0xfe;	//1111 1110
		}
		_nop_();
		SCL = 0 ;
		_nop_();
		if(i < 7)
		{
			dat = dat << 1;	
		}
	}
	return(dat);
}

```



### 常见的串行总线协议

IIC总线、SPI总线、SCI总线

![image-20200204203008531](D:%5C%E5%B0%8F%E5%87%AF%E6%96%87%5CDocuments%5CIIC%E9%80%9A%E4%BF%A1EEPROM.assets%5Cimage-20200204203008531.png)
