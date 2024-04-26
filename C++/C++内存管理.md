# C++内存管理从基础到复杂
> **锱铢必较 俱往矣 且看今朝**          
> --侯杰

## Prime内存组件
### c++中使用memory的途径
![alt text](image.png)
### C++ memory primitives
![alt text](image-1.png)
**常见用法**
```c++
    void* p1 = malloc(512);	//512 bytes
    free(p1);

    complex<int>* p2 = new complex<int>; //one object
    delete p2;             

    void* p3 = ::operator new(512); //512 bytes
    ::operator delete(p3);

//以下使用 C++ 標準庫提供的 allocators。
//其接口雖有標準規格，但實現廠商並未完全遵守；下面三者形式略異。
#ifdef _MSC_VER
    //以下兩函數都是 non-static，定要通過 object 調用。以下分配 3 個 ints.
    int* p4 = allocator<int>().allocate(3, (int*)0); //参数2无用
    allocator<int>().deallocate(p4,3);           
#endif
#ifdef __BORLANDC__
    //以下兩函數都是 non-static，定要通過 object 調用。以下分配 5 個 ints.
    int* p4 = allocator<int>().allocate(5);  
    allocator<int>().deallocate(p4,5);       
#endif
#ifdef __GNUC__
    //以下兩函數都是 static，可通過全名調用之。以下分配 512 bytes.
    //void* p4 = alloc::allocate(512); 
    //alloc::deallocate(p4,512);   
    
    //以下兩函數都是 non-static，定要通過 object 調用。以下分配 7 個 ints.    
	void* p4 = allocator<int>().allocate(7); 
    allocator<int>().deallocate((int*)p4,7);     
	
    //以下兩函數都是 non-static，定要通過 object 調用。以下分配 9 個 ints.	
	void* p5 = __gnu_cxx::__pool_alloc<int>().allocate(9); 
    __gnu_cxx::__pool_alloc<int>().deallocate((int*)p5,9);	
#endif

```
### new 表达式
![alt text](image-2.png)
可以看到new在编译器转换后的形式，
大致为：使用全局new然后再new中调用malloc如果内存不足使用callnewh，然后返回创建的内存指针，之后调用构造函数。
### delete 表达式
![alt text](image-3.png)
大致为：调用析构，然后使用delete函数触发free释放内存

### array new ，array delete
![alt text](image-4.png)
可以发现就算使用array new 缺使用 一般delete，如果类没有ptr成员也可能不会产生内存泄漏。
![alt text](image-6.png)
采用栈的结构创建和析构。
### array size，in memory
![alt text](image-5.png)
> **图中pad是为了内存对齐的填充，黄色部分是debug模式下额外内存**

在这对于int调用delete 而不是delete[]是无所谓的，因为int没有析构。
![alt text](image-7.png)
必须使用delete[]（要求demo 析构有意义）

### placement new 
![alt text](image-8.png)
placementnew 不调用maloc 直接返回已经分配内存
总的来说 placement new 就是直接再给定的位置上调用构造函数

### C++程序内存分配途径
![alt text](image-9.png)

### C++ 容器内存分配
![alt text](image-10.png)

### 重载 全局new delete
![alt text](image-11.png)
### 重载类中 new delete
![alt text](image-12.png)
![alt text](image-13.png)
其中new 和delete 最好使用static
### 示例
![alt text](image-14.png)
![alt text](image-15.png)
![alt text](image-16.png)
使用全局new delete的结果

### 重载new（）/ delete（）
![alt text](image-17.png)

**placement new 必须要有一个默认输入  size_t**
![alt text](image-18.png)
![alt text](image-19.png)

![alt text](image-20.png)
***表示在重载placement new 和placement delete 的时候只有在 new调用构造函数失败才会调用这些重载delete***

![alt text](image-21.png)

### 分配器1（per-class1）
![alt text](image-22.png)
![alt text](image-23.png)
目的减少maloc次数以用于减少cooki产生，而且让new内存在一起
美中不足的是多了一个next指针

### 分配器2（per-class2）
![alt text](image-24.png)
![alt text](image-25.png)
使用embedded pointer 嵌入式指针 也就是吧next指针当作空间使用，写入数据会覆盖他。
缺点是不free空间
### 分配器3 （static allocator）
![alt text](image-26.png)
![alt text](image-27.png)
![alt text](image-28.png)
一小块一小块的几个连续逻辑地址存储思路和上一个差不多，不用把内存分配各个版本都写一次
### 分配器4 （macro for static allocator）
![alt text](image-29.png)

### 分配器5 （global allocator）
![alt text](image-30.png)

### new handler
当operate new 无法分配出申请的内存就调用这个函数然后看情况抛出异常
newhandler 函数 一般用来申请更多内存或者（跳出循环，终止程序）
![alt text](image-31.png)
![alt text](image-32.png)

### ==default，=delete
![alt text](image-33.png)
![alt text](image-34.png)

## std::allocator