### windows常用API函数(三)

1. CreateWindow 创建一个窗口

	- 之前API函数的例子，都是针对DOS编程的，严格来说是在windows下的仿DOS（cmd)进行编程，编写控制台应用程序大家都知道，主函数是main，那针对windows编程的主函数也是main吗？不是的，windows下的主函数（入口函数）是WinMain。在定义main主函数的时候，可以给它带两个参数，也可以不带。而WinMain函数就不能这样了，它有固定的格式，它必须带四个参数。

	- 现给出WinMain函数的固定格式：

		> int WINAPI WinMain( HINSTANCE hInstance, HINSTANCEhPrevInstance, LPSTR lpCmdLine, int nCmdShow)

	- 简单的例子如下：

		```c
		#include "stdafx.h"

		int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
		{
			while(1)
			Sleep(100);
			return 0;
		}
		```

	- 怎么样够简单吧，是不是觉得奇怪，怎么没有窗口，因为窗口要自己创建，不像控制台程序，只要一运行便会有窗口。虽然没有窗口，但你创建了一个进程，打开任务管理器，可以找到你所创建的那个进程，其实也没什么奇怪的，像WINDOWS本身的一些系统服务，也是只有进程，没有窗口的像spoolsv.exe,svchost.exe。

	- 那要如何创建一个窗口呢？要创建一个窗口,就必须要向系统提供窗口的信息，如你要创建的窗口名字叫什么，窗口图标是什么，窗口大小，窗口背景色等，不然，系统怎么给你创建窗口呢？所以为了方便，VC就定义了一个结构，专门用于存储窗口信息。

	- 现给出这个结构的定义。
		```c
		typedef struct _WNDCLASS {
			UINT style; //描述类风格
			WNDPROC lpfnWndProc; //窗口处理函数
			int cbClsExtra; //表示窗口类结构之后分配的额外的字节数。系统将该值初始化为0
			int cbWndExtra; //表示窗口实例之后分配的额外的字节数。系统将该值初始化为0
			HINSTANCE hInstance;// 应用程序实例句柄由WinMain函数传进来
			HICON hIcon; //窗口图标句柄
			HCURSOR hCursor; //窗口光标句柄
			HBRUSH hbrBackground; //画刷句柄
			LPCTSTR lpszMenuName; //窗口菜单名
			LPCTSTR lpszClassName; //窗口类名
		} WNDCLASS, *PWNDCLASS;
		```
	- 好了，如果我们已经把窗口信息填好了，那我们要怎样把这个信息告诉系统呢，也就是把要创建窗口的信息传给系统。这里我们调用RegisterClass函数就能实现这个功能。注册完窗口，我们就要创建窗口,用CreateWindow函数就能实现，不要问为什么注册窗口后直接显示不就行了，还要搞什么创建窗口。这我也不知道，反正你只要记住这格式就行了，硬式规定的，你想创建一个窗口，就必须按这些步骤来。

	- 好了，窗口创建了，我们就要调用ShowWindow函数显示窗口，然后用UpdateWindow函数刷新一下，确保窗口能立即显示。

	- 以下详细实现代码：

		```c
		#include "stdafx.h"
		#include<windows.h>

		int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
		{
			WNDCLASS wndcls; //定义一个存储窗口信息WNDCLASS变量
			wndcls.cbClsExtra=0; //默认为0
			wndcls.cbWndExtra=0; //默认为0
			wndcls.hbrBackground=(HBRUSH)GetStockObject(GRAY_BRUSH); //背景画刷
			wndcls.hCursor=LoadCursor(NULL,IDC_CROSS); //十字光标
			wndcls.hIcon=LoadIcon(NULL,IDI_ERROR); //窗口图标
			wndcls.hInstance=hInstance; //应用程序实例句柄由WinMain函数传进来
			wndcls.lpfnWndProc=NULL; //窗口消息处理函数
			wndcls.lpszClassName="windowclass"; //窗口类名
			wndcls.lpszMenuName=NULL; //窗口菜单名，没有菜单，为NULL
			wndcls.style=CS_HREDRAW | CS_VREDRAW;//窗口类型，CS_HREDRAW和CS_VERDRAW 表明
			//当窗口水平方向垂直方向的宽度变化时重绘整个窗口
			RegisterClass(&wndcls); //把窗口信息提交给系统，注册窗口类
			HWND hwnd; //用以存储CreateWindow函数所创建的窗口句柄
			hwnd=CreateWindow("windowclass","first windows",
			WS_OVERLAPPEDWINDOW,0,0,600,400,NULL,NULL,hInstance,NULL);//创建窗口
			ShowWindow(hwnd,SW_SHOWNORMAL);//窗口创建完了，显示它
			UpdateWindow(hwnd); //更新窗口，让窗口毫无延迟的显示
			return 0;
		}
		```

	- 是不是出错了，内存不能读取，为什么了呢，因为你创建的窗口没有消息处理函数，windows系统当然不允许这样一个窗口存在，对按键，鼠标都没有反应，这样的窗口是没有实际意义的。 wndcls.lpfnWndProc=NULL; //窗口消息处理函数，就是前面这句，必须要填
	窗口过程（消息）处理函数，那这个函数是怎样定义的呢，像WinMain一样，它也有固定的格式。

	- 窗口过程处理函数的格式：
		> LRESULT CALLBACK WinSunProc(HWND wnd,UINTuMsg,WPARAM wParam,LPARAM lParam)

	- 下面的这个是一个窗口创建的完整例子：

		```c
		#include "stdafx.h"
		#include<windows.h>

		LRESULT CALLBACK WinSunProc(HWND hwnd,UINT uMsg,WPARAM wParam,LPARAM lParam)
		{
			if(uMsg==WM_LBUTTONDOWN)MessageBox(NULL,"kdjfkdf","Kjdfkdfj",MB_OK);//处理鼠标按下消息，弹出消息框
			return DefWindowProc(hwnd,uMsg,wParam,lParam); //未处理的消息通过DefWindowProc函数交给系统处理
		}

		int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
		{

			WNDCLASS wndcls; //定义一个存储窗口信息WNDCLASS变量
			wndcls.cbClsExtra=0; //默认为0
			wndcls.cbWndExtra=0; //默认为0
			wndcls.hbrBackground=(HBRUSH)GetStockObject(GRAY_BRUSH); //背景画刷
			wndcls.hCursor=LoadCursor(NULL,IDC_ARROW); //光标
			wndcls.hIcon=LoadIcon(NULL,IDI_ERROR); //窗口图标
			wndcls.hInstance=hInstance; //应用程序实例句柄由WinMain函数传进来
			wndcls.lpfnWndProc=WinSunProc; //窗口消息处理函数
			wndcls.lpszClassName="windowclass"; //窗口类名
			wndcls.lpszMenuName=NULL; //窗口菜单名，没有菜单，为NULL
			wndcls.style=CS_HREDRAW | CS_VREDRAW;//窗口类型，CS_HREDRAW和CS_VERDRAW 表明
			//当窗口水平方向垂直方向的宽度变化时重绘整个窗口
			RegisterClass(&wndcls); //把窗口信息提交给系统，注册窗口类
			HWND hwnd; //用以存储CreateWindow函数所创建的窗口句柄
			hwnd=CreateWindow("windowclass","first windows",
			WS_OVERLAPPEDWINDOW,0,0,600,400,NULL,NULL,hInstance,NULL);//创建窗口
			ShowWindow(hwnd,SW_SHOWNORMAL);//窗口创建完了，显示它
			UpdateWindow(hwnd); //更新窗口，让窗口毫无延迟的显示
			MSG msg;//消息结构类型
			while(GetMessage(&msg,NULL,0,0))//获取消息
			{
				TranslateMessage(&msg); //此函数用于把键盘消息(WM_KEYDOWN,WM_KEYUP)转换成字符消息WM_CHAR
				DispatchMessage(&msg); //这个函数调用窗口过程处理函数，并把MSG里的信息处理后传给过程函数的四个参数
			}
			return 0;
		}
		```

	- WinSunProc函数的四个参数，分别对应着SendMessage函数四个参数，详情参见SendMessage函数参数解释。

	- MSG类型解释：

		```c
		typedef struct tagMSG
		{
			HWND hwnd;//hwnd表示消息将要发送给的窗口句柄
			UINT message;//消息类型，如WM_WMCLOSE,WM_CHAR,WM_LBUTTONDOWN,参见消息表
			WPARAM wParam;//消息附带信息，取值的意思具体依据消息类型而定
			LPARAM lParam;//消息附带信息，取值的意思具体依据消息类型而定
			DWORD time;//消息的发送时间，不常用
			POINT pt;//消息发送时，鼠标所在的位置，不常用
		}MSG;
		```

	- 大家试着把上面的例子运行一遍，然后关掉窗口，再运行一遍，是不是出错了，因为前一个程序虽然窗口关闭了，但进程还在运行，还记得那个循环语句吗？```while(GetMessage(&msg,NULL,0,0))```就是这个。只要条件成立，进程就会一直运行下去。如何让这个循环结束呢？用 ```PostQuitMessage(0);``` 这个语句就行了，参数0表示给自身窗口发送一个退出消息，当GetMessage函数接到PostQuitMessage函数发出的消息后，就会返回0值。

	- 如在窗口过程函数中处理窗口关闭WM_CLOSE消息:if(uMsg==WM_CLOSE）PostQuitMessage(0); 这样只要一关闭窗口，它的进程也会结束。

	- 接下来解释一下CreateWindow函数参数的意思

		```c
		LPCTSTR lpClassName,//窗口类名，应与WNDCLASS结构里的成员lpszClassName一致
		LPCTSTR lpWindowName,//窗口标题名
		DWORD dwStyle，//窗口的风格，取值参见表Style
		int x,
		int y,//x,y表示所创建窗口左上角位置
		int nWidth,
		int nHeight,//nWidth,nHeight表示窗口的宽高
		HWND hWndParent,//父窗口句柄，如果不是子窗口，这里取值为NULL
		HMENU hMenu,//菜单句柄，没菜单的话，取NULL值
		HANDLE hlnstance,//对应着WinMain函数的第一个参数
		LPVOID lpParam);//NULL
		```

		- 表Style：（参考：百度）
			```
			WS_BORDER：创建一个单边框的窗口。 　　
			WS_CAPTION：创建一个有标题框的窗口（包括WS_BODER风格）。
			WS_CHILD：创建一个子窗口。这个风格不能与WS_POPUP风格合用。
			WS_CHLDWINDOW：与WS_CHILD相同。
			WS_CLIPCHILDREN:当在父窗口内绘图时，排除子窗口区域。在创建父窗口时使用这个风格。 　　
			WS_CLlPBLINGS；排除子窗口之间的相对区域，也就是，当一个特定的窗口接收到WM_PAINT消息时，WS_CLIPSIBLINGS 风格将所有层叠窗口排除在绘图之外，只重绘指定的子窗口。如果未指定WS_CLIPSIBLINGS风格，并且子窗口是层叠的，则在重绘子窗口的客户区时，就会重绘邻近的子窗口。
			WS_DISABLED:创建一个初始状态为禁止的子窗口。一个禁止状态的窗口不能接受来自用户的输入信息.
			WS_DLGFRAME:创建一个带对话框边框风格的窗口。这种风格的窗口不能带标题条。
			WS_GROUP:指定一组控制的第一个控制。这个控制组由第一个控制和随后定义的控制组成，自第二个控制开始每个控制，具有WS_GROUP风格，每个组的第一个控制带有WS_TABSTOP风格，从而使用户可以在组间移动。用户随后可以使用光标在组内的控制间改变键盘焦点。　　
			WS_HSCROLL：创建一个有水平滚动条的窗口。 　　
			WS_ICONIC：创建一个初始状态为最小化状态的窗口。
			与WS_MINIMIZE风格相同。 　　
			WS_MAXIMIZE：创建一个初始状态为最大化状态的窗口。 　　
			WS_MAXIMIZEBOX：创建一个具有最大化按钮的窗口。该风格不能与WS_EX_CONTEXTHELP风格同时出现，同时必须指定WS_SYSMENU风格。 　　
			WS_OVERLAPPED:产生一个层叠的窗口。一个层叠的窗口有一个标题条和一个边框。与WS_TILED风格相同。　　WS_OVERLAPPEDWINDOW：创建一个具有WS_OVERLAPPED，WS_CAPTION，WS_SYSMENU WS_THICKFRAME，WS_MINIMIZEBOX，WS_MAXIMIZEBOX风格的层叠窗口，与WS_TILEDWINDOW风格相同。 WS_POPUP；创建一个弹出式窗口。该风格不能与WS_CHLD风格同时使用。 　　
			WS_POPUWINDOW：创建一个具有WS_BORDER，WS_POPUP,WS_SYSMENU风格的窗口，WS_CAPTION和WS_POPUPWINDOW必须同时设定才能使窗口某单可见。　　
			WS_SIZEBOX：创建一个可调边框的窗口，与WS_THICKFRAME风格相同。 　　
			WS_SYSMENU：创建一个在标题条上带有窗口菜单的窗口，必须同时设定WS_CAPTION风格。　　
			WS_TABSTOP：创建一个控制，这个控制在用户按下Tab键时可以获得键盘焦点。按下Tab键后使键盘焦点转移到下一具有WS_TABSTOP风格的控制。 　　
			WS_THICKFRAME：创建一个具有可调边框的窗口，与WS_SIZEBOX风格相同。 　　
			WS_TILED：产生一个层叠的窗口。一个层叠的窗口有一个标题和一个边框。
			与WS_OVERLAPPED风格相同。 　　
			WS_TILEDWINDOW:创建一个具有WS_OVERLAPPED，WS_CAPTION，WS_SYSMENU， WS_THICKFRAME，WS_MINIMIZEBOX，WS_MAXMIZEBOX风格的层叠窗口。与WS_OVERLAPPEDWINDOW风格相同。　　
			WS_VISIBLE创建一个初始状态为可见的窗口。 　　
			WS_VSCROLL：创建一个有垂直滚动条的窗口。
			```
2. GetMessage 获取窗口消息， 参照 CreateWindow  
3. RegisterClass 注册窗口类，参照 CreateWindow
4. UpdateWindow 参照 CreateWindow
5. DispatchMessage 参照 CreateWindow
6. LoadCursorFromFile 从磁盘加载一个光标文件，函数返回该光标句柄
	```c
	//假设e盘下有一个名为a.cur的光标文件。
	HCURSOR cursor; //定义一个光标句柄，用于存放LoadCursorFromFile函数返回的光标句柄
	cursor=LoadCursorFromFile("e:\\a.cur");
	```
	获得了光标句柄有什么用呢？看一下窗口类WNDCLASS里的hCursor成员，这个成员也是一个光标句柄，明白了吧！
7. CreateSolidBrush 创建一个画刷，函数返回画刷句柄
	- ```HBRUSH hbr = CreateSolidBrush(RGB(12,172,59));```
	- 三个数字分别表明RGB的颜色值，RGB根据三种颜色值返回一个COLORREF类型的值		
8. LoadImage 装载位图、图标、光标函数
	- 函数定义：
		- HANDLE LoadImage(HINSTANCE hinst,LPCTSTR lpszName,UINTuType,int cxDesired,int CyDesired,UINT fuLoad)
	- 这里我们只要这个函数的几个简单功能：从磁盘加载位图，从磁盘加载图标，从磁盘加载光标。
		- 所以第一个参数hinst我们不用管它，直接填NULL就行
		- 第二个参数lpszName是图片文件所在路径名
		- 第三个参数uType指明要加载的是什么类型的图片，是位图（填IMAGE_BITMAP），还是光标（填IMAGE_CURSOR），还是图标（填IMAGE_ICON）
		- 第四个cxDesired和第五个参数CyDesired,指定要加载的图片的宽高（可以放大光标，或者缩小），如果加载的是位图的话，则两个参数必须为0
		- 第六个参数fuLoad表示以何种方式加载文件，这里我们是从磁盘加载文件，所以填LR_LOADFROMFILE
	- 好了，假设e盘下有一个c.cur和i.ico文件。例子：设置窗口图标和光标，还有背景色

		```c
		#include "stdafx.h"//这个头文件是编译器自动生成的，不是空工程，都会有，如果是直接建C++源文件，包含这个头文件，会出错
		#include <windows.h>
		#include <stdio.h>

		LRESULT CALLBACK WinSunProc(
			HWND hwnd, // handle to window
			UINT uMsg, // message identifier
			WPARAM wParam, // first message parameter
			LPARAM lParam // second message parameter
		); //窗口过程函数声明

		int WINAPI WinMain(
			HINSTANCE hInstance, 
			HINSTANCE hPrevInstance, // handle to previous instance
			LPSTR lpCmdLine, // command line
			int nCmdShow // show state
		)
		{
			//设计一个窗口类
			WNDCLASS wndcls;
			wndcls.cbClsExtra=0;
			wndcls.cbWndExtra=0;
			wndcls.hbrBackground=CreateSolidBrush(RGB(12,172,59));//画刷
			wndcls.hCursor=(HCURSOR)LoadImage(NULL,"e:\\c.cur",IMAGE_CURSOR,24,24,LR_LOADFROMFILE);//加载光标
			wndcls.hIcon=(HICON)LoadImage(NULL,"e:\\i.ico",IMAGE_ICON,48,48,LR_LOADFROMFILE);//加载图标
			wndcls.hInstance=hInstance; //应用程序实例句柄由WinMain函数传进来
			wndcls.lpfnWndProc=WinSunProc; //定义窗口处理函数
			wndcls.lpszClassName="windowclass";
			wndcls.lpszMenuName=NULL;
			wndcls.style=CS_HREDRAW | CS_VREDRAW;
			RegisterClass(&wndcls);

			//创建窗口，定义一个变量用来保存成功创建窗口后返回的句柄
			HWND hwnd;
			hwnd=CreateWindow("windowclass","first window",
			WS_OVERLAPPEDWINDOW,0,0,600,400,NULL,NULL,hInstance,NULL);
			//显示及刷新窗口
			ShowWindow(hwnd,SW_SHOWNORMAL);
			UpdateWindow(hwnd);
			//定义消息结构体，开始消息循环
			MSG msg;
			while(GetMessage(&msg,NULL,0,0))
			{
				TranslateMessage(&msg);
				DispatchMessage(&msg);
			}
			return msg.wParam;
		}

		//编写窗口过程函数
		LRESULT CALLBACK WinSunProc(
			HWND hwnd, // handle to window
			UINT uMsg, // message identifier
			WPARAM wParam, // first message parameter
			LPARAM lParam // second message parameter
		)
		{
			switch(uMsg)
			{
				case WM_CHAR: //字符消息
					char szChar[20];
					sprintf(szChar,"char code is %c",wParam);
					MessageBox(hwnd,szChar,"char",0);
				break;
				case WM_LBUTTONDOWN: //鼠标左键按下消息
					MessageBox(hwnd,"mouse clicked","message",0);
				break;
				case WM_CLOSE:
					if(IDYES==MessageBox(hwnd,"是否真的结束？","message",MB_YESNO))
					{
						DestroyWindow(hwnd); //销毁窗口，并发送WM_DESTROY消息给自身窗口
					}
				break;
				case WM_DESTROY:
					PostQuitMessage(0);
				break;
				default:
					return DefWindowProc(hwnd,uMsg,wParam,lParam);
			}
			return 0;
		}
		```
9. GetDC 根据窗口句柄获取设备上下文（DC）返回DC句柄
	- 得到了一个窗口的设备上下文，就可以进行画图操作了，像画圆，画正方形，显示图片等函数都是要设备上下文(DC）句柄做参数的。

		```c
		HDC dc//定义一个DC句柄
		HWND wnd = FindWindow(NULL,"无标题.txt- 记事本");//获取窗口句柄
		dc = GetDC(wnd)//获取这个窗口的设备上下文
		```
10. Rectnagle 在窗口中画一个矩形
	- 以"无标题.txt - 记事本"窗口为例，在这个窗口简单的画一个矩形

		```c
		#include<windows.h>

		int main()
		{
			HDC dc;
			HWND wnd = FindWindow(NULL,"无标题.txt - 记事本");
			dc = GetDC(wnd);//获取窗口设备上下文（DC）
			while(1)//用循环语句重复画，是为了确保不会被窗口刷新给刷掉
			{
				Rectangle(dc,50,50,200,200);//画一个矩形
				Sleep(200);
			}
			return 0;
		}
		```
