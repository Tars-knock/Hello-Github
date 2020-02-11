```cpp
int roll()

{
	while(1)
	{
		int roll1,roll2=0x40;
		int time;
		for(roll1=0x01;roll1!=0x40;roll1=_crol_(roll1,1))
		{						
				if(roll2!=0x01)	roll2=_cror_(roll2,1);
				else roll2=0x01;

				for(time=50;time>0;time--)
				{
					wei1=1;wei2=1;wei3=0;
					P0=roll1;
					delay(1);P0=0x00;
					wei1=0;wei2=0;wei3=1;
					//这个地方如果不加这个判断会使得P0传入0x40而不传入0x01
					if(_crol_(roll2,1)==0x40)	P0=0x01;
					else						P0=_crol_(roll2,1);
					delay(1);P0=0x00;
				}
			
		}
		
	}
}
```



日后可以改进使得传入1-7的参数，以选择roll哪一位数码管

可以使用定时器