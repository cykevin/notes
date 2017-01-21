# Chap4 类型基础
1. 所有的类型都从System.Object中派生

	clr要求所有的类型最终都从System.Object中派生，因此会直接继承四个方法：
	+ Equals
	+ ToString
	+ GetType
	+ GetHashCode
	
	clr要求所有的对象必须从new操作符创建，使用new操作符时，clr会做一系列的工作：
	1. 检查对象所需要的字节数，包含对象及其继承过来的所有实例属性的字节数，以及额外的**类型对象指针**和**同步索引块**需要的字节数
	1. 在托管堆上分配这些字节数并且都置为0，
	1. 初始化类型对象指针和同步索引块
	1. 调用在new操作符中传入参数的构造函数，返回这个托管堆的首地址引用
1. 编译器会在编译时检查类型转换
	
	在程序运行的时候，clr总是知道一个对象的类型是什么，可以通过调用GetType方法得到。c#语言提供了类型转型的语法：
	+ 显式转型
	+ 隐式转型：
	
	将一个类型转换成他的任意的基类型时不需要任何特殊语法，直接赋值，因为clr认为这是类型安全的。相反将一个类型转换成他的派生类型的时候需要进行显式转换，这时候可能会抛出运行时错误。
	
	c#还提供了**is**与**as**操作符来帮助我们进行类型转换。is 操作符判断左边对象的类型是否兼容于右边的类型（是否是右边的类型或是派生类）。进行判断时，clr会首先检查对象是否是右侧的类型，如果不是，再遍历对象的继承结构，依次用父类型去匹配右侧类型。as 操作符也会进行类型检查，如果通过会进行强制类型转换，不通过则返回null。两者都不会抛出异常。
1. 命名空间与程序集并没有直接的关系

	使用命名空间来对类型进行逻辑分组，建议创建类型的时候使用公司全称
	
	通过/reference开关，告诉编译器在检查类型定义时在哪些程序集中检查。编译器会扫描这些程序集，在其中查找，一旦找到就将程序集的信息写入到生成的托管模块的元数据中。
	
	using 指令
	编译器进行语法检查时，需要确认引用的每个类型都确实存在，调用了确实存在的方法，给方法传递了正确的实参，正确使用了方法的返回值等。如果在源文件或者引用的程序集中找不到指定类型，编译器会尝试在类型前添加using名称，然后在引用的程序集中查找。如果找到了多个符合的类型，则会告知开发者存在“不明确的引用”
	
	可以使用using a=b;的方式来给某个命名空间下的类型起别名，从而防止该命名空间下的其他类型来污染当前的命名空间
	
	对于编译器来说，命名空间的作用就是为类型附加更长的名称，使得名称唯一。
	在运行时clr会在指定的程序集中加载指定的类型（全名），clr并不知道有命名空间的概念。
	
	
1. 类型、对象、托管堆 、线程栈在运行时的相互关系
	每个线程在被创建时会被分配1MB大小的栈空间，用来存储局部变量以及方法参数。

	每个方法在线程中执行时，都会在栈中压入一个栈桢，这个栈桢反映了这个方法的调用情况。在方法执行完毕后，该栈桢被弹出。 方法的参数先入栈，接着是方法的返回地址入栈，接着才是方法的栈桢，后续的局部变量以及内部的方法实参在运行时进入栈桢，方法执行完毕后，线桢被弹出，CPU的指令指针会被指向栈中方法的返回地址
	
	clr在jit编译时会为每个类型创建一个**类型对象**，这个类型对象包含该类型所有的静态字段，还包含了一个方法表，里面是该类型**所有的静态方法以及实例方法**
	
	调用静态方法时，clr会定位与调用静态方法的类型对应的类型对象，然后查找方法表，编译执行
	
	调用实例方法时，clr定位**发起调用的实例对象对应的类型对象**，然后查找方法表，找不到时回溯查找
	
	调用虚方法时，JIT编译器要在方法中生成一些额外的代码，这些代码会检查发出调用的变量类型，并找到变量在堆上的对象，根据对象的类型对象指针找到对应的类型对象。最后查找方法表，编译执行。