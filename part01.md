### windows常用API函数(一)

1. FindWindow 根据窗口类名或窗口标题名来获得窗口的句柄，该函数返回窗口的句柄
	- HWND WINAPI FindWindow(LPCSTR lpClassName,LPCSTR lpWindowName);
	第一个参数填窗口的类名，第二个填窗口的标题名，其实是不需要同时填两个参数的，也就是说，你只要知道窗口的类名或窗口的标题就可以了，没有的那个就用NULL代替。比如现在有一个窗口名为"无标题.txt - 记事本"的记事本程序。那么我就可以用上面的函数获得这个窗口的句柄，那获得了这个窗口的句柄我可以干什么呢？作用可大了，因为很多操作窗口的函数，都需要窗口句柄作参数，如移动、改变窗口大小的MoveWindow函数，在这里举个例子，大家就更能体会到这个FindWindow的用法、用处。
	- FindWindow例子：已知一个窗口名称，写一个程序关闭该窗口，假设当前电脑正有一个窗口名为"无标题.txt - 记事本"的记事本程序运行

		```c
		#include <windows.h>

		int main()
		{
			HWND wnd;//定义一个窗口句柄变量，用以存储找到的窗口句柄
			wnd = FindWindow(NULL,"无标题.txt - 记事本");//获得窗口名为"无标题.txt - 记事本"的窗口句柄
			SendMessage(wnd,WM_CLOSE,0,0);//调用SendMessage函数，发送一个WM_CLOSE（关闭）消息给wnd窗口句柄
			return 0;
		}
		```
2. SendMessage根据窗口句柄发送一个消息给窗口
	- 函数定义：LRESULT SendMessage（HWND hWnd，UINT Msg，WPARAM wParam，LPARAM IParam）；
	第一个参数是窗口句柄，第二参数个是消息类型，下面的消息表列举了所有消息，第三，四个参数是消息附带信息，解释依赖于消息类型，比如一个字符消息（WM_CHAR),那么第三个参数就储存有一个字符的ASCII码。消息机制大家都应该知道吧，Windows是基于消息的系统，鼠标移动键盘按键都会产生消息。
	- 接下来举一个例子，发送一个WM_CHAR消息给窗口，也就是模仿键盘按键，接收消息的窗口依旧以"无标题.txt - 记事本"为例： 

		```c
		//SendMessage例子：模仿键盘按键
		#include<windows.h>

		int main()
		{
			HWND wnd;
			wnd = FindWindow(NULL,"无标题.txt - 记事本");

			while(1)
			{
				SendMessage(wnd,WM_CHAR,WPARAM('a'),0);
				Sleep(300);
			}
			return 0;
		}
		```
	- 呵呵上面的例子是不是没用，这是为什么呢，哪里出错了吗？错倒是没有错，只是窗口句柄有问题，消息发送给了主窗口。接收消息的窗口不对。记事本窗口界面有些有什么东西呢？菜单，编辑框，状态栏等控件，控件也是窗口，既然是窗口，那当然它们也有窗口句柄，而在记事本里是在哪里打字的？编辑框控件里打字的嘛！所以消息应该发送编辑框控件，那如何获得记事本里编辑框控件的窗口句柄呢？用FindWindow吗？不知道编辑框窗口标题名，类名也不知道，当然也有其它方法获取编辑框窗口标题名和窗口类名，如Spy++。关于如何获得编辑框句柄，将在以后的函数中会有介绍，这里我们就用WindowFromPoint这个函数来获取，这个函数获取窗口句柄的方法比较笨，（相对于我这个例子来说），这个函数是根据什么来获取窗口句柄的呢？根据屏幕坐标点，如屏幕坐标点20，20，当前是哪个窗口占有，就返回哪个窗口的句柄。有了这个函数，我们还需要一个函数GetCursorPos获取鼠标当前位置（针对于屏幕）；
	- 可行的例子：模仿键盘按键：

		```c
		#include<windows.h>

		int main()
		{
			POINT curpos;//一个可储存坐标点的结构体变量，x横坐标，y,纵坐标，如curpos.x curpos.y

			while(1)
			{
				GetCursorPos(&amp;curpos);//获取当前鼠标的位置，位置将储存在curpos里。
				HWND hWnd = WindowFromPoint(curpos);//根据curpos所指的坐标点获取窗口句柄
				SendMessage(hWnd,WM_CHAR,WPARAM('g'),0);//发送一个字符（按键）消息g给当前鼠标所指向的窗口句柄
				Sleep(300);//睡眠三百毫秒，相当于等待三分之一秒
			}

		}
		```
	- 这个程序一运行后，只要把鼠标指向要输入字符的窗口句柄，那么就相当于键盘每三分之一秒按了一个g键，试试吧！
	如果这样觉得模仿键盘按键太麻烦的话，那么就用keybd_event这个函数，这个专门用于模仿键盘按键的，关于怎么用，自己百度一搜，就知道了。既然SendMessage能模仿键盘按键的话，那也能模仿鼠标左击，右击。而此时SendMessage函数第三，四个参数的解释就是储存有鼠标左击，右击时的位置。如模仿鼠标右击，想一想，一次鼠标右击有哪几步，分别是鼠标右键按下，鼠标右键松开，如果你按下鼠标右键不松开，那它是不是鼠标右击，不是的，直到你松开鼠标右键，才能算是一次完整的鼠标右击.鼠标右键按下的消息类型是“WM_RBUTTONDOWN”，右键松开的消息是“WM_RBUTTONUP”，那么一次完整的鼠标右击应该是：
	SendMessage(wnd,WM_RBUTTONDOWN,0,0);//鼠标右键按下,第三，四个参数说明了鼠标按下时的位置
	Sleep(100);//间隔100毫秒
	SendMessage(wnd,WM_RBUTTONUP,0,0);//鼠标右键松开
	同样，也有一个专门模仿鼠标动作的函数，mouse_event这个函数，可以模仿鼠标的移动，单击，双击等。以后会有专门介绍。
3. GetCursorPos获取鼠标当前位置（屏幕）
	- 这个函数在SendMessage函数有介绍，这里仅举一个例子，在界面里不停的输出鼠标当前位置。

		```c
		#include<windows.h>
		#include<stdio.h>

		int main()
		{
			POINT curpos;
			while(1)
			{
				GetCursorPos(&amp;curpos);
				printf("x:%d,y:%d",curpos.x,curpos.y);
				Sleep(300);
				printf("\n");
			}
		}
		```
4. WindowFromPoint根据坐标点获得对应的窗口句柄
	- 在SendMessage有解释，这里仅举一个例子，鼠标指向哪个窗口，就关闭哪个窗口。

		```c
		#include<windows.h>

		int main()
		{
			Sleep(2500);//等待一会儿，用于把鼠标移到其它窗口上去，避免指向本身进程的窗口，关掉自己的窗口。
			POINT curpos;
			while(1)
			{
				GetCursorPos(&curpos);
				HWND wnd = WindowFromPoint(curpos);
				SendMessage(wnd,WM_CLOSE,0,0);
				Sleep(300);
			}
		}
		```
5. MoveWindow根据窗口句柄移动窗口，改变窗口大小
	- 函数定义：BOOL MoveWindow( HWND hWnd, int X, int Y, intnWidth, int nHeight, BOOL bRepaint );
	hWnd是要改变大小的窗口的句柄，x,y相对于屏幕的坐标，窗口左上角的位置与之相对应，nWidth和nHeight是窗口新的宽高，bRepaint指定窗口是否重画。
	这里依旧以"无标题.txt - 记事本"为例子，改变这个窗口大小，并把窗口移到左上角去。

		```c
		#include<windows.h>

		int main()
		{
			HWND wnd;
			wnd=FindWindow(NULL,"无标题.txt - 记事本");
			MoveWindow(wnd,0,0,220,120,NULL);
			return 0;
		}
		```
6. ShowWindow设置窗口显示状态，如隐藏，最大化，最小化
	- 函数定义BOOL ShowWinow(HWND hWnd,int nCmdShow); 第一个参数hWnd指明了窗口句柄，第二个参数指明了窗口的状态，第二个参数常用取值范围：
		- SW_HIDE：隐藏窗口并激活其他窗口。
		- SW_MAXIMIZE：最大化指定的窗口。
		- SW_MINIMIZE：最小化指定的窗口并且激活在Z序中的下一个顶层窗口。
		- SW_RESTORE：激活并显示窗口。如果窗口最小化或最大化，则系统将窗口恢复到原来的尺寸和位置。在恢复最小化窗口时，应用程序应该指定这个标志。
		- SW_SHOW：在窗口原来的位置以原来的尺寸激活和显示窗口。
	- ShowWindow例子：程序运行后，在桌面上隐藏一个指定的窗口，并在4秒后再将其显示

		```c
		#include<windows.h>

		int main()
		{
			HWND wnd;
			wnd = FindWindow(NULL,"无标题.txt - 记事本");
			ShowWindow(wnd,SW_HIDE);
			Sleep(5000);
			ShowWindow(wnd,SW_SHOW);
			return 0;
		}
		```
7. SetCursorPos 设置鼠标的位置、把鼠标移动到指定的位置
	- 函数定义：BOOL SetCursorPos(int x,int y); 
	- 这个函数的两个参数我想大家应该知道是什么意思吧，屏幕的坐标点。直接看例子：

		```c
		#include<windows.h>

		int main()
		{
			int sec=0;
			while(sec<200)
			{
				SetCursorPos(rand()%1024,rand()%768);//随机设置鼠标的位置
				Sleep(20);
				sec++;
			}
			return 0;
		}
		```
8. CopyFile 复制一个文件
	- 比如，我要把E盘的abb.txt的文本文件复制到d盘的zhengyong.txt,则调用语句

		```CopyFile("e:\\abb.txt","d:\\zhengyong.txt",FALSE);```

	- 第三个参数：如果设为TRUE（非零），一旦目标文件已经存在，函数调用会失败。否则目标文件会被覆盖掉。
9. DeleteFile 删除一个文件
	- 如何删除一个文件?

		```DeleteFile("e\\abb.txt");```

	- 如果目标为隐藏或只读，则无用。
10. CreateDirectory 创建一个文件夹（目录）
	- 假如E盘下什么文件也没有,下面的代码是错的，不能同时建两个文件，除非E盘下已经有了个aaa文件夹了

		```CreateDirectory("e:\\aaa\\bbb",NULL);```

	- 这样才是对的

		```CreateDirectory("e:\\aaa",NULL);```

