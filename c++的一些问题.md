# c++的一些问题
## 1.堆区与栈区

### 1.1 堆区和栈区定义

栈区往往比较常用于存放函数的参数值、局部变量，由编译器取决需要自动分配释放，相当于临时数据，用完会进行释放。遍历快。

堆区则通过new、malloc、realloc分配，编译器不负责释放，需要通过程序区进行释放，类似数据结构中的链表。遍历慢。

### 1.2 写法

栈区往往默认应用，没有固定的写法。

而堆区调用与释放一般通过以下写法：

```c++
int *p=new int(10);//开一个数值为10的空间并指向
delete p;//释放单个
int *arr=new int[10];//开一个长度10的数组并指向数组头
delete[] arr;//释放数组
```

在上述代码中，若要对数组进行赋值，还需要另外的命令，一般new int 指代的只是一个堆区地址。

### 1.3 思考：类中指针与深浅拷贝

由上述内容可以得知，堆区会释放，栈区会保留，但是定义和删除需要做到位，那么就有了下述思考：

```c++
int m_age;
int* m_weight;
```

假设示例的类含有上文两个内容。

1.在类中定义的指针该如何保留属性？使用一般的指针为什么不行？

在指针刚被定义的时候，和定义的整型、字符型内容不一样，他没有分配内存，此时称这个指针为浮空指针，于是，为了让类里面的浮空指针能够进行构造函数赋值的操作，需要调用堆区，给他开辟一块数值内存，此时指针便有了自己的意义。

```c++
person(int age,int height)
{
	cout << "有参函数调用" << endl;
	m_age = age;
	//*m_height = height;//没有指向的指针赋值无意义会报错
	m_height=new int(height);
    //如此在堆区开辟空间进行赋值可以保留指针数据内容
}
```

2.拷贝调用该怎么写，可以直接进行地址的传递操作吗？

若不定义对应的拷贝函数，则系统默认执行以下命令：

```c++
class person
{
    m_age = p.m_age;
	m_height = p.m_height;
}//也叫浅拷贝，指对每个类中数据进行机械复制
```

那么就有一个不可避免的问题，当进行拷贝时，会有多个m_height数据指向同一个地址，在这里看，这个问题好像不大。

若要解决这个问题，则需自写拷贝函数如下：

```c++
m_age = p.m_age;
m_height = new int(*p.m_height);
//同样的，开辟一个堆区赋值，以保留指针数据内容
//也叫深拷贝，不单单在乎数值，还在乎开辟的地址状况
```

3.为了避免内存泄露，析构函数该怎么写？

析构函数除了一般的结束之外，此时要将堆区的数据释放干净，也就是将类中指针指向地址进行内存释放：

```c++
if (m_height != NULL)
	{
		delete m_height;
		m_height = NULL;//堆区置空
	}
```

当加入此段代码，容易发现一个问题，若采用默认拷贝方式，析构的时候会报错，报错的理由也很简单，看如下代码：

```c++
person p1(18,160);
person p2(p1);
```

很明显构造了p1和p2并保留了其对应数值，但是在运行结束，析构的时候，问题来了。此处运行会报错。

因为析构函数中释放的内存释放了两次，栈区的类p2先被析构，释放了一次堆的内存，此后当p1被析构的时候，m_height空有地址而没有指向，因此产生了报错。

为了避免这一报错，其实还是要采用深拷贝的方式进行类内容的拷贝，这也是深拷贝的根本意义所在。

## 2.引用和指针

### 2.1 指针

```c++
int *p；//这行代码定义了一个指针p
```

**在指针p定义的同时，没有分配对应的空间，需要给指针定义明确的指向，他才有内存**，其次是借助指针可以修改对应指向，快速定位某个数据并完成修改功能；

其中有两种指针特殊：

```c++
const int *p1=&a;//定值指针，可以改地址不能改值

int* const p2=&a;//定位指针，可以改值不能改地址
```

### 2.2 引用

引用和指针的区别在于，不额外添加存储空间，而是调用已有的数据空间内容。

```c++
int &b=a;//b作为a的另一个代号，其实用的是a的空间

int &b;//写法有误，引用不能没有数据指向
```

其实真实情况是，在指针指向某个数据时，往往透过引用建立指针和数据之间的关联。

### 2.3 思考内容1

1.在以下三个交换a，b的子函数中，哪一个可以成功将主函数中的内容交换？

```
void swap1(int a,int b);
void swap2(int* a,int *b);
void swap3(int &a,int &b);
```

1是肯定不行的，因为交换发生在子函数的形参，当子函数运行完毕后，a,b的交换值会发生回归；

2是可以的，很明显在下文的操作调用中，对指针指向的对应地址数据进行了值的更改；

3也是可以的，其实引用也是一种数据指向方式，和指针的不同就在于没有独立空间罢了。

2.在下面引用做返回值的案例中，哪一个可以作为返回值运用？

```
int& test1()
{
	int a=10；
	return a;
}
int& test2()
{
	static int a=10;
	return a;
}
```

很明显，引用仅仅是作为某个数据地址，他没有对应的内存，若是像1一样作为形参的引用，在正文执行的时候，栈区地址已经清空了；

如果像2一样给形参分配了静态，作为全局变量，地址保留，是可以作为返回值引用的；

```
test2（）=1000；
```

在主函数甚至可以通过这个代码实现对子函数调用的静态变量a的值的更改；

### 2.4 思考内容2

在类中的this指针，若要调用该指针对类中对象进行值的改动和值的传递、叠加，首先假设进行操作的类如下：

```c++
class person
{
public:
	person(int age)
	{
		//this指向被调用的成员函数所属的对象
		this->age = age;
	}
	int age;//假如名称一样，输出不了,需要this指针
};
```

在person类中，若要使两者的年龄实现叠加，定义personaddage函数，如下所示：

```c++
void personaddage(person &p)
	{
		this->age += p.age;
	}
```

刚开始，我们写的是这样的一个函数，用引用的方式将要叠加的人的年龄属性引入，随后被叠加的人调用this指针，指向这个对象，并实现加法，结果也达到了预期，在调用完之后，可以看到属性的增减。比如：

```c++
person p1(10);
person p2(10);
p2.personaddage(p1);
cout<<p2.age<<endl;
```

这一子段函数执行后，输出p2的年龄是20。

假如，我们需要加多个p1的age属性，具体该怎么实现？一般的想法是：

```c++
p2.personaddage(p1);
p2.personaddage(p1);
p2.personaddage(p1);
......
```

那可不可以写成这样呢？

```c++
p2.personaddage(p1).personaddage(p1).personaddage(p1);
```

其实是不行的，因为从左到右调用一次之后，p2的指针指向已经返回他自身了，但是没有作为person返回值返回，函数最多只能执行一次，那么要实现链式反应，我们应该怎么做？尝试如下改写：

```c++
person personaddage(person &p)
	{
		this->age += p.age;
    	return *this;
	}
```

貌似这个操作可行，可是当验证的时候，你会发现p2的值仍然为20。

原因是调用p2第一次之后，返回这个对象的时候，其实默认使用了拷贝，也就是说，返回的对象已经不再是p2了。

那怎么办？这时候就需要引用来帮忙了。

```c++
person& personaddage(person &p)
	{
		this->age += p.age;
    	return *this;
	}
```

这个时候，返回的这个对象person相当于纳入了p2的别名访问中，也就是通过引用的方式指向p2，这时候，链式传递能够成功实现了。

## 3.类-初始化

### 3.1 隐式转换法

什么叫隐式转换法？

借用标准里的话来说，就是当你只有一个类型T1，但是当前表达式需要类型为T2的值，如果这时候T1自动转换为了T2那么这就是隐式类型转换。

借用栗子如下：

```c++
int a=0;
long b=a+1;
if(a==b){}//int转long

std::shared_ptr<int>ptr=func();
if(ptr){}//自定义类型转标量后转bool值
```

可以看到，在代码中间，当执行一些语句的时候，语句本身更加重视比较过程，因此类型被自动转化。

在类的初始化中，有一个person的类

```c++
class person
{
public:
	person(int a)
	{
		age=a;
	}
	int age;
};
```

在这个类中，存在一个有参构造函数，对类中对象赋初值，当存在这个有参构造函数时，我们调用有两种方法：

```c++
//显式构造法
person p1(10);//创建一个p1且调用有参构造
//隐式构造法
person p1=10;//这个方法和上述效果一样
```

我们很纳闷，int和person不同类，怎么编译器还能识别？

其实编译器是识别出你的有参构造函数，随后类型一对比，就知道你想要构造一个类的对象了，这种方法快而简洁。

### 3.2 类的初始化列表

一般用于类对象有多个需要赋初值的情况时，比如对下面这个类：

```c++
class phone
{
public:
	string m_pname;
};
class person
{
public:
	string m_name;
	phone m_phone;
};
```

对于person这个类，我需要赋初值，怎么写有参构造函数？

```c++
person(string a,string b)
{
	m_name=a;
	m_phone.m_pname=b;
}
```

一般写法如此，但是引入初始化列表的概念后，可这么写

```c++
person(string a,string b):m_name(a),m_phone(b)
{}
```

这里首先是用到了初始化列表的写法，有点像对函数跨文件调用的写法。这里也复习一下跨文件调用咋写的：

```c++
class point
{
public:
	void inputp();
	double getx();
	double gety();
private:
	double x;
	double y;
};
```

比如说，在头文件中定义的一个类如下，可以看到有几个类函数，这时候我们在另一个文件对他进行声明，因为头文件一般只负责定义：

```c++
#include "point.h"
void point::inputp()
{
	cout << "输入点位信息：" << endl;
	cout << "x坐标：";
	cin >> x;
	cout << "y坐标：";
	cin >> y;
}
double point::getx()
{
	return x;
}
double point::gety()
{
	return y;
}
```

可以看到在这个文件，只是具体定义了函数信息，而其它信息包含在头文件中，这个时候为了识别这个函数名是包含在什么类里面的（不同类，同一名函数区分），需要加上point::的说明。

与初始化列表不同的是，:的作用一个是声明函数作用域，一个是表示后面引入初始化列表赋值。

## 4.类的包含

### 4.1 问题的引入

在VS2022环境下的c++中，涉及多个类的互相包含时，往往会爆出莫名的错误。因为前面定义的类假如使用了后面的内容，类之间是无法完成识别的，编译器只能够前向识别，因此，往往需要附加定义信息，但是定义信息附加的选取往往很多余，一般，没有必要对一个类的存在反复声明的，但是错误就是会莫名的出现。

比如下面一段代码，需要借用友元对一个类中的对象函数作全局访问处理，他的类是这样的：

```c++
//class build;
class goodgay
{
public:
	goodgay();
	~goodgay();
	void visit();//可访问building私有
	void visit2();//不可访问
	build *building;
};
class build
{
	friend void goodgay::visit();
public:
	build();
	string m_sittingroom;
private:
	string m_bedroom;
};
build::build()
{
	m_sittingroom = "客厅";
	m_bedroom = "卧室";
}
goodgay::goodgay()
{
	building = new build;
	cout << "构建完成" << endl;
}
goodgay::~goodgay()
{
	if (building != NULL)
	{
		delete building;
		building = NULL;
	}
	cout << "析构完成" << endl;
}
void goodgay::visit()
{
	cout << "visit正在访问" << building->m_sittingroom << endl;
	cout << "visit正在访问" << building->m_bedroom << endl;
}
void goodgay::visit2()
{
	cout << "visit2正在访问" << building->m_sittingroom<<endl;
	
}
```

先定义了goodgay类，引用了build的对象；随后在build中引入友元，对visit（）函数赋予友元......在这种情况下，其实不难理解加声明的操作，这是必要的，因此要加上第一行代码。

在不同的编译器中，其实运行效果不尽相同，但是在VS2022中，因为互相引用就会引入未定义问题。

### 4.2 问题的解决方法

这个问题的解决方法其实就是避免错误C2027:使用了未定义类型的产生。一种方法上文也说过了，就是两个类互相引用的时候，多加一个前向声明，还有一个方法是，把两个类存到不同的头文件区分开来，之后预编译即可，在标准设计下，类包含较多时，往往采用第二个方法。

后面学到了模板，会发现更多bug，这些问题出现没法修，只能作为写代码的人尽量不去写出这种语法的bug。因此知道了问题下次避免重返就好。

## 5.多态的深层原理

在学习多态时，看到下面一段代码，你认为输出结果是什么样的？

```c++
#include<iostream>
using namespace std;
class animal
{
public:
	void speak()
	{
		cout << "动物在说话" << endl;
	}
	/*virtual void speak()//记录虚函数的入口地址
	{
		cout << "动物在说话" << endl;
	}*/
};
class cat :public animal
{
public:
	/*virtual*/ void speak()//同样子类中虚函数表替换成子类的入口地址
	{
		cout << "小猫在说话" << endl;
	}
};
void dospeak(animal& a)
{
	a.speak();
}
void test01()
{
	cat c;
	dospeak(c);
}
void test02()
{
	cout << "sizeof animal=" << sizeof(animal) << endl;//空类=1
	//virtual=8，代表指针，x64指针是8，x86是4
}
int main()
{
	test01();
	test02();
	system("pause");
	return 0;
}
```

此时test01和test02的输出结果是什么样的？运行之后发现，test01会输出动物在说话，test02会输出1，这时候逻辑还是比较明确的；

加入virtual在animal的speak重名函数上会怎么样？test01可以输出小猫在说话了，同时test02会输出8（x64），4（x86）。

很容易理解嘛，加上virtual之后，相当于把函数赋予了指针意义，在要求父类，但定义子类给函数的时候，对象会调用子类的同名函数，相当于指向子类，这时候这个东西叫做vfptr；他随类而继承，但是随定义的重名函数会变更指针调用函数的地址。

## 6.语句中的变量定义

今天注意到一个问题：

```c++
for (int i = 0; i < 10; i++)
	{
		v1.push_back(i);
	}
	for (i = 0; i < v1.size(); i++)//报错
	{
		cout << v1[i] << " ";
	}
```

可是我上面很明显已经定义了i，为什么会发生未知错误？

是由于编译器对于for循环/if中的语句，不知道是否能够执行导致的，这时候，在编译器中，他们属于未定义需要定义的变量，且使用完会自动销毁。

那在for(){//这里定义}会怎么样？

同样的，执行完for之后会自动清楚不保留，因此对于for的初始化i，未定义的每次都定义，或者在函数for之前率先定义，就不会导致错误。

## 7.为什么参数传递多半使用引用进行+for_each的用法

这个问题虽然上面已经强调了引用和非引用在函数中的区别，不过今天遇到了一个新情况：

起因是调用for_each中的函数对vector数组进行全体输出，写的function部分是这样的：

```c++
class person
{
public:
    person(string name)
    {
        m_name = name;
    }
    string m_name;
    double m_score;
    deque<int> ms;
};

vector<person>xuanshou;

void printvector(person p)
{
    cout << "选手" << p.m_name << "的最终得分为:" << p.m_score << endl;
}
```

在调用printvector时，出现了错误，原因是person未定义啥的，起初我没看懂，后来我了解了，原来是没有对应的构造函数。

那我们应该怎么样修改printvector呢？

```c++
void printvector(const person &p)
{
    cout << "选手" << p.m_name << "的最终得分为:" << p.m_score << endl;
}
for_each(xuanshou.begin(),xuanshou.end(),printvector);
```

可以看到，修改中引入了const person &p进行修改，这时候不需要借助对象构造来对p的数据进行读取，而是利用其本身的属性。

这里的printvector的完整形式为[](const person&p){printvector(p);})，代表使用lambda匿名函数对已有数据p进行捕获，此后再调用printvector，因为此前调用函数是读取的，可以直接读取到对应参数的值。for_each可以缺省表达。

## 8.在使用仿函数定义容器建立排序规则报错C3848

仿函数需要创建一个重载()的类，并且作用到对应的容器排序中，如map，set等。

```c++
class mycom
{
public:
	bool operator()(int v1, int v2)
	{
		return v1 > v2;
    }
};
void printmap(map<int,int,mycom> &m)
{
	for (map<int, int,mycom>::iterator it = m.begin(); it != m.end(); it++)
		cout << "key=" << (*it).first << " value=" << it->second << endl;
	cout << endl;
}
```

可是对于这样一个类来说，使用printmap会报错，显示C3848：具有类型“const mycom”的表达式会丢失一些 const-volatile 限定符以调用“bool mycom::operator ()(int,int)”

其实意思是，你定义的规则体现在构造中的时候，他应该具有不变性，即const属性，因此解决方法为：
	`bool operator()(int v1, int v2)const`
对规则进行了const约束，这样能够正常运行。
