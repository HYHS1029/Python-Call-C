# Python API note
## python call C test
### 1. Python call C (動態連結庫)
Python呼叫C，不需經過封裝，直接打包成so/dll，再使用python的`ctypes`呼叫即可。
1. example:
```
#include <stdio.h>
#include <stdlib.h>
#if __linux__
#define DllExport
#else
#define DllExport __declspec(dllexport)
#endif

DllExport int c_add(int a, int b){
	printf("add in c, a = %d, b = %d\n", a, b);
	return a + b;
}
```
### 2. Python call C++ (類)動態連結庫
需使用 `extern "C"` 來輔助，也就是說還是能呼叫C函式，不能直接呼叫方法，但是能解析C++方法。不是用`extern "C"`，構建後的動態連結庫沒有這些函式的符號表。
1. example:
```
#include <iostream>  
using namespace std;    
class TestLib{  
     public:  
         void display();  
         void display(int a);  
};  
void TestLib::display(){  
    cout<<"First display"<<endl;  
}   
void TestLib::display(int a){  
    cout<<"Second display:"<<a<<endl;  
}  
extern "C" {  
    TestLib obj;  
    void display(){  
        obj.display();   
    }  
    void display_int(){  
        obj.display(2);   
    }  
}
```
### 3.  使用```ctypes``` 呼叫
#### 1. convert python array to C array:
**example:**

C code:
```
DllExport int c_arrSum(int arr[], int arr_size){
int sum = 0;
for (int i=0; i < arr_size; i++){	
    // printf("%d \n", arr[i]);
    sum += arr[i];
}
    return sum;
}
```
python code:
```
from ctypes import *

lib = cdll.LoadLibrary('./build/Debug/my_test.dll')
pyarray = [i+1 for i in range(3)] #output:[1,2,3]
carr = (c_int* len(pyarray))(*pyarray) 

print(lib.c_arrSum(carr, len(carr))) #output: 6
    
```
#### 2. convert C array to python array:
**example:**
C code:
```
DllExport int* makeArr(int len){
    int *Arr = malloc(len * sizeof(int));
	for(int i=0; i<len; i++){
		Arr[i] = i+1;
	}
    return Arr;
}

/* A function to free pointers */
DllExport void freeptr(void *ptr){
    free(ptr);
}
```
python code:
```
Arrlen = 4
lib.makeArr.restype = POINTER(c_int * Arrlen) # specified the return type to pointer.
# print(type(lib.makeArr(5)))
darrayptr = lib.makeArr(Arrlen) # retrieve the array pointer
pylist = [j for j in darrayptr.contents] # Convert the array pointer contents to Python list
print(pylist)
lib.freeptr.argtype = c_void_p
lib.freeptr.restype = None
lib.freeptr(darrayptr) # free the pointers
```
## 3. build DLL (leltek_ultrasound_source_x86_testap)
1. 缺少opencv -> 參考reference 3 設定路徑
2. Error:Make_unique' is not a member of 'std
-> 確認C++ 版本(C++17)、加入```#include <memory>```
3. 注意visual studio 版本(要用visual studio 2017來build)
4. SDK的版本不能太新(sdk 8.1)

## 4. python 無法順利用```ctypes``` 載入DLL
### 1. possible errors
1. **OSError: [WinError 193] %1 不是有效的 Win32 應用程式** : 可能原因為python是64 bit 而 dll 為32 bit.

solution:

可在VS的power shell中檢查dll是32 bit或64 bit
    (指令:```dumpbin /headers {dll name}.dll```)
    
2. **Could not find module "xxx.dll" Try using the full path with constructor syntax.**:

soulation:
1. 使用絕對路徑
2. 確認要載入的dll檔其dependencies有在所選的路徑上，可用VS power shell 查看該.dll dependencies
(指令```dumpbin /dependents {dll name}.dll```)    
    


**TO DO**
- [ ] 1. 讀寫register的function 
- [x] 2. python 無法load build 好的dll檔(已解決, 參考4.)
 
**reference:**
1. [https://www.796t.com/content/1540976311.html](https://www.796t.com/content/1540976311.html)
2. [ctypes](https://docs.python.org/zh-cn/3/library/ctypes.html)
3. [visual studio 中設定 opencv](https://tedliou.com/opencv-cpp-visual-studio-2022/)
4. [python ctypes](https://docs.python.org/3/library/ctypes.html)




