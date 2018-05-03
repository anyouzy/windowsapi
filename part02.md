### windows常用API函数(二)

1. GetClientRect 获得窗口大小(客户区）
	- 直接上例子:

		```c
		#include<windows.h>
		#include<stdio.h>

		int main(int argc, char* argv[])
		{
			HWND wnd;
			while(1)
			{
				wnd=FindWindow(NULL,"无标题.txt - 记事本");
				RECT rect;//专门用来存储窗口大小
				GetClientRect(wnd,&rect);//获取窗口大小
				printf("%d,%d,%d,%d\n",rect.left,rect.top,rect.right,rect.bottom);//输出窗口大小，试着用鼠标改变窗口大小
				Sleep(300);
			}
			return 0;
		}
		```
2. GetWindowRect 获得窗口大小（相对屏幕）
	- 直接上例子:

		```c
		#include<windows.h>
		#include<stdio.h>

		int main(int argc, char* argv[])
		{
			HWND wnd;
			while(1)
			{
				wnd=FindWindow(NULL,"无标题.txt - 记事本");
				RECT rect;//专门用来存储窗口大小
				GetWindowRect(wnd,&rect);//获取窗口大小
				printf("%d,%d,%d,%d\n",rect.left,rect.top,rect.right,rect.bottom);//输出窗口大小，试着用鼠标改变窗口大小
				Sleep(300);
			}
			return 0;
		}
		```
3. FindFirstFile 寻找文件以及获得文件的信息
	- 这里举一个例子吧，列举E盘第一目录下的所有文件，包括文件夹，结合FindNextFile

		```c
		#include<windows.h>
		#include<stdio.h>
		int main()
		{
			BOOL done=TRUE;
			WIN32_FIND_DATA fd;
			HANDLE hFind = FindFirstFile("e:\\*.*", &fd);//第一个参数是路径名，可以使用通配符，懂DOS的人应该知道吧！fd存储有文件的信息
			while (done)
			{
				printf("%s\n",fd.cFileName);
				done=FindNextFile(hFind, &fd); //返回的值如果为0则没有文件要寻了
			}
			return 0;
		}
		```
	- 当然也可以直接找一个文件，不使用通配符，但这样有什么意义呢？，如FindFirstFile("e:\\aaa.txt",&fd);其实这个可以获取一个文件的信息，如文件是不是隐藏的，或者有没有只读属性等。
	- 当然通过控制通配符，也可以寻找特定类型的文件，比如我只要找文本文件，那么就是这个语句FindFirstFile("e:\\*.txt",&fd);就行了，关键看你自己灵活运用。
	- 前面说过fd里存储有文件的信息，那怎么根据fd里面的成员判断这个文件的属性，文件是否隐藏，是不是文件夹。
	- fd里的dwFileAttributes存储有文件的信息，如判断是否为文件夹，只要把这个变量和FILE_ATTRIBUTE_DIRECTORY进行按位与运算，如果不为0的话，表明为文夹件，如if(fd.dwFileAttributes&FILE_ATTRIBUTE_DIRECTORY) printf("%s是文件夹\n",fd.cFileName);
	- 其它判断也是一样，现在给出文件的属性(常用几个）
		- FILE_ATTRIBUTE_HIDDEN（隐藏）
		- FILE_ATTRIBUTE_READONLY（只读）
		- FILE_ATTRIBUTE_SYSTEM（系统）
4. FindNextFile 寻找文件
	- 参照FindFirstFile函数的例子
5. MoveFile 移动文件
	- 如把一个盘里的文本移到另一个盘里去:
		```MoveFile("e:\\a.txt","d:\\abc.txt");``` 意思把e盘下的a.txt移到d盘下去，并改名为abc.txt.
6. GetClassName 根据窗口句柄获得窗口类名
	- 函数定义：int GetClassName(HWND hWnd, LPTSTR IpClassName, intnMaxCount)；
		> 这种函数不需要再解释了吧，前面有太多类似的例子
7. SetFileAttributes 设置文件属性
	- 函数定义：BOOL SetFileAttributes( LPCTSTRlpFileName,DWORDdwFileAttributes);
	- 这个函数的第二个参数dwFileAttributes和前面讲过的WIN32_FIND_DATA结构里的dwFileAttributes成员相对应。假设E盘第一目录下有一个文本文件a.txt的正常文件，我要把它设为只读和隐藏那要如何做呢？在前面介绍过WIN32_FIND_DATA结构里dwFileAttributes成员的几个常用属性，根据这个我们知道隐藏是FILE_ATTRIBUTE_HIDDEN，只读是FILE_ATTRIBUTE_READONLY。
	- 那么把E盘下文本文件的属性设为隐藏和只读的语句就是：
		```SetFileAttributes("e:\\a.txt",FILE_ATTRIBUTE_HIDDEN|FILE_ATTRIBUTE_READONLY);```
	- (说明：这个函数同样也能设置文件夹属性）
	- 虽然这个语句可以达到要求，但不建议用，因为会覆盖掉文件的原来属性，也就是说如果这个文件之前有系统属性（系统文件）的话，那么这个语句一旦执行后，文件就只有隐藏和只读属性了。
	- 比如一个文件原先就有隐藏属性，依旧以a.txt为例子，那么我把它设为只读，是不是这个语句就可以呢？
		```SetFileAttributes("e:\\a.txt",FILE_ATTRIBUTE_READONLY）;```
	- 这样的话，虽然文件有只读属性了，但隐藏属性却没有了。那要如何在不覆盖掉原来的属性下，把文件设为只读呢，其实说了这么多的废话，总结起来就一句话：如何增加一个文件的属性！
	- 前提是要获得这个文件的原有属性：获得文件的属性，在FindFirstFile函数讲过。好吧！直接看例子：
	- 假设e盘的a.txt文件属性为隐藏，给它增加只读属性：

		```c
		#include<windows.h>
		int main()

		{
			WIN32_FIND_DATA fd;
			FindFirstFile("e:\\a.txt",&fd);
			fd.dwFileAttributes|=FILE_ATTRIBUTE_READONLY;//在原来的属性下增加只读属性
			SetFileAttributes("e:\\a.txt",fd.dwFileAttributes);//设置文件的属性
			return 0;
		}
		```

	- 第二个例子：如何去掉一个文件的属性
	- 我想懂这里的按位或、按位与或者按位异或运算的人应该知道该如何去掉一个文件的属性。其实一个文件信息都是以二进制码说明的。
	- 比如一个八位二进制码：10000010，这里的每一位是不是只有0和1取值，不是0，就是1，正好可以表示一个文件属性的有无，如这个文件是隐藏的吗？只有是和不是，这样我们规定把这八位二进制码的第一位用于确定文件是否具有隐藏属性，如果为1那便是隐藏，无则没有，以此类推第二位就代表文件的只读，第三位系统。。。但要如何判断呢，或者把某一位的值改变呢，用按位运算就可以，00000010，我要把第2位的值设为0，其它位上的值保持不变，用按位异或运算即可，与00000010进行按位异或运算，但这里并不是与它本身进行运算，不管任何八位二进制数的值是多少只要与00000010进行按位异或运算，那第二位都会变成0，而其它的位保持不变。这样为了方便，我们就把00000010进行宏定义，方便记忆，这个二进制数的十进制为2。宏定义#define FILE_ATTRIBUTE_READONLY 2, 明白了这个我们就来清除一个文件的一种属性吧！
	- 清除一个文件的隐藏属性，假设a.txt为隐藏文件：

		```c
		#include<windows.h>
		int main()

		{
			WIN32_FIND_DATA fd;
			FindFirstFile("e:\\a.txt",&fd);//获取文件信息
			fd.dwFileAttributes^=FILE_ATTRIBUTE_HIDDEN;//在原来的属性下删除隐藏属性
			SetFileAttributes("e:\\a.txt",fd.dwFileAttributes);//设置文件的属性
			return 0;
		}
		```
	- 如果单单只针对文件的属性进行操作的话，可以用GetFileAttributes函数获取文件的属性，该函数只一个参数，那就是文件的路径，函数返回一个DWORD值，包含文件属性信息。
8. ShellExecute 运行一个程序
	- 函数定义:ShellExecute(HWND hwnd, LPCSTR lpOperation, LPCSTRlpFile, LPCSTR lpParameters, LPCSTR lpDirectory, INT nShowCmd);
		- 第一个参数hwnd是父窗口的句柄,可以为NULL
		- 第二个参数lpOperation表示行为
		- 第三个参数lpFile是程序的路径名
		- 第四个参数lpParameters是给所打开程序的参数,可以为NULL
		- 第五个参数lpDirectory可以为NULL
		- 第六个参数nShowCmd跟ShowWindow函数的第二个参数一样,作用也一样,如果打开的程序有窗口的话,这个参数就指明了窗口如何显示.
	- 例如打开一个记事本:
		```ShellExecute(NULL,"open","NOTEPAD.EXE",NULL,NULL,SW_SHOWNORMAL);```
	- 而且这个函数还可以指定程序打开一个属于程序本身类型的文件,假如e盘有一个a.txt文件;我调用函数运行记事本程序并打开这个文本文件.
		```ShellExecute(NULL,"open","NOTEPAD.EXE","e:\\a.txt",NULL,SW_SHOWNORMAL);```
		> 这里由于记事本程序属于系统本身自带的程序,所以没有绝对路径.
	- 这个函数还可以打开一个网站:
		```c
			ShellExecute(NULL,"open","http://www.baidu.com",NULL,NULL,SW_SHOWNORMAL);
			ShellExecute(NULL,"open","C:",NULL,NULL,SW_SHOWNORMAL);
		```
	- 还可以根据文件后缀名选择相应的程序打开一个文件：
		```ShellExecute(NULL,"open","e:\\a.bmp",NULL,NULL,SW_SHOWNORMAL);```
	- 类似的函数还有WinExec，
		- 只有两个参数,它的最后一个参数跟ShellExecute函数的最后一个参数一样.
		- 而第一个参数则是程序路径名.
		- 举个例子:```WinExec("NOTEPAD.EXE",SW_SHOWNORMAL);```
		- 这个函数也可以给程序传递一个文件名供要运行的程序打开,那要如何加进去呢,这里又没有第三个参数,方法就是把路径名加在NOTPEPAD.EXE的后面,要以空格来分开如:```WinExec("NOTEPAD.EXE e:\\a.txt",SW_SHOWNORMAL);```
9. PlaySound 播放一个WAV文件
	- 函数定义：BOOL PlaySound(LPCSTR pszSound, HMODULE hmod,DWORDfdwSound);
		- 第一个参数是WAV文件的路径名
		- 第二个参数如果不是播放MFC里以资源ID命名的文件，则可以为空，
		- 第三个参数，指明了以何种方式播放文件
		- 注意这个函数只能播放100K以下的WAV文件。
	- 假如E盘有个a.wav文件，下面这个例子播放这个文件：

		```c
		#include<windows.h>
		#include<mmsystem.h>//PlaySound函数的头文件

		#pragma comment(lib, "winmm.lib")//链接库，PlaySound函数必须使用

		int main()
		{
			PlaySound("e:\\19.wav",NULL,SND_SYNC);
			return 0;
		}
		```
10. GetModuleFileName 根据模块导入表获取程序的完整路径
	- 函数定义：DWORD GetModuleFileName( HMODULE hModule, LPTSTRlpFilename, DWORD nSize );
		- 关于第一个参数，将在以后的动态链接库里会有介绍，这里我们只要获得程序本身的路径，那么第一个参数可以为空。
		- 第二个参数用以存储路径，nSize指明字符数组大小。
	- 例子: 把自身程序移动到e盘下，并改名为a.exe;

		```c
		#include<windows.h>


		int main()
		{
			char szAppName[128]={0};
			GetModuleFileName(NULL,szAppName,128);
			MoveFile(szAppName,"e:\\a.exe");
			return 0;
		}
		```
