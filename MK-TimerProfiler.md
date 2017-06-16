# MK-TimerProfiler

## 引言
> 性能调优对于每一个软件工程师来说，是一道必须面对的课题。项目的纵深推进，新功能的不断增减，不可避免的会出现卡顿等问题，如何快速定位到问题所在，进行优化，是我们每一位工程师必须掌握的技能。
	下面简要描述如何使用TimerProfiler对耗时性代码进行处理。
	
## 正文
> 我们这里创建一项测试工程，我们使用大量循环打印阻塞主线程，模拟卡屏现象,运行下面代码，发现要等3秒左右，才能看见按钮显示出来（3秒，简直不能容忍！）。
> 接下来我们将使用TimerProfiler定位到这段耗时代码，让后再对代码进行优化。代码如下：


	-(void)mkTest{
	    for (int i = 0; i <50000; i++) { NSLog(@"1");}
	    
	    UIButton * button = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 50, 100)];
    	button.backgroundColor = [UIColor redColor];
    	[self.view addSubview:button];
	}

> 如何启动TimerProfiler，就不赘述。正常操作后，应该是如下界面：

![](/Users/gw/Desktop/image-1.png)

* 1.为启动、关闭和暂停按钮。
* 2.启动后，请将Separate by Thread、Invert CallTree、Hide System Libraries三个选项勾上。

		解释：
		1）Separate By Thread:线程分离,只有这样才能在调用路径中能够清晰看到占用CPU最大的线程.
		2）Invert Call Tree:从上到下跟踪堆栈信息.这个选项可以快捷的看到方法调用路径最深方法占用CPU耗时,比如FuncA{FunB{FunC}},勾选后堆栈以C->B->A把调用层级最深的C显示最外面. 
		3）Hide Missing Symbols:如果dSYM无法找到你的APP或者调用系统框架的话，那么表中将看到调用方法名只能看到16进制的数值,勾选这个选项则可以隐藏这些符号，便于简化分析数据.
		4）Hide System Libraries:这个就更有用了,勾选后耗时调用路径只会显示app耗时的代码,性能分析普遍我们都比较关系自己代码的耗时而不是系统的.基本是必选项.注意有些代码耗时也会纳入系统层级，可以进行勾选前后前后对执行路径进行比对会非常有用.
	

> 接下来大家会看见如图3所标记的信息。信息显示，mkTest这个方法耗时占比达到72.7%（如果实在实际项目中，这真是卡出翔了），找到病因，才好对症下药。

> 耗时操作怎么处理？？？当然是子线程。改进后代码如下：

	- (void)viewDidLoad {
	    [super viewDidLoad];
	    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	        [self mkTest];
	    });
	    
	    UIButton * button = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 50, 100)];
	    button.backgroundColor = [UIColor redColor];
	    [self.view addSubview:button];
	}
	-(void)mkTest{
	    for (int i = 0; i <50000; i++) { NSLog(@"1");}
	}

>再启动timerProfiler检测下，结果如下图：

![](/Users/gw/Desktop/99.png)

* 工程跑起来能立刻看见按钮显示出来；
* 再结合上面截图，主线程耗时比为20.9%远低于之前的92.5%。

## 总结

> 上面我们模拟了从发现问题，定位问题再解决问题，整个性能调优过程。我们都明白，实际项目中会比这里情况更加麻烦，但问题总归是一个一个发现解决的，只有先掌握发现的技能，才能谈如何解决问题。
这只是性能调优方式之一，有机会再记录分享其他条有方式！
  

