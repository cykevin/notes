# chap 9 
## 1 可选参数和命名参数
可以在声明方法时指定默认值

    public class Calc
	{
		public int Add(int a1=0,int a2=0)
		{
			return a1 + a2;
		}
	}

然后在调用方法时可以不传入参数让程序使用默认值，也可以指定为哪个参数传递值。

    Add(1);
	Add(a2:19);

注意：

* 定义了默认值的参数必须在无默认值参数之后
* 默认值必须是在编译时就能被确定的常量值
* 尽量不要修改参数名，否则以指定参数名称调用的代码会出错
* 参数有默认值时也不能出现Add(1,,1)这样的调用。

编译器的工作：
为添加了默认值的参数添加特性：System.Runtime.InteropServices.OptionalAttribute和System.Runtime.InteropServices.DefaultParameterValueAttribute，这些特性会生成到无数据文件中，并向DefaultParamterValueAttribute传递你指定的常量值。

## 2 隐式类型的局部变量

	public void Show()
	{
		var number=10;
		var data=new Dictionary<int,Object>();
	}

如上，用var声明变量时，编译器会根据=右侧的表达式推断出number和data的类型。

注意：

* 这种语句只能出现在方法内部，即声明局部变量的地方
* 不能出现在类型中声明成员变量的地方
* 不能出现在定义方法的参数的地方

## 3 以传引用的方式向方法传递参数
clr默认方法参数的传递方式都是值传递，我们可以通过ref和out关键字来指定通过引用传递。

编译器会将这两个关键字区别对待：

使用了out关键字的方法：

* 方法不希望调用者初始化该参数
* 方法内部会初始化该参数
* 方法在返回前会向该参数写入值

使用了ref关键字的方法：

* 调用者必须在调用方法之前初始化该参数
* 方法内部会读取或写入该参数

对值类型使用ref/out参数，方法返回后值将会被修改

对引用类型使用ref/out参数，不仅可以修改引用类型指向的堆中的对象，还可以让引用类型指向堆中的其他对象。

    public static void StringBuilderNoRef(StringBuilder s)
    {
        // 修改堆中对象
        s.Append(" World");

        // 无意义，因为修改的只是传入参数的一个副本（在当前方法的调用线桢上）
        s = new StringBuilder("hi");
    }

    public static void StringBuilderByRef(ref StringBuilder s)
    {
        // 修改堆中对象
        s.Append(" World");

        // 让传入的引用指向新的对象
        s = new StringBuilder("hi");
    }
    
可变数量的参数

可以在定义方法时在参数前加上params关键字来表示该参数是一个可变数量的参数

编译器检测到需要调用这样的方法时会自动生成数组，装载数据。

这当然为我们编程带来了方便，但是因为生成数组需要在堆上分配空间，对性能会有一些影响。建议尽量不要这样定义方法，或者在定义这样的方法的同时定义带几个参数的重载方法。

参数和返回类型的设计规范

* 定义方法时参数类型最好用弱数据类型

用IEnumberable<T>，不要用List<T>，不要用ICollection<T>，不要用IList<T>(除非方法需要列表对象)

* 定义方法时将返回值声明为最强的类型

因为方法的调用者可以将返回的强类型视为本身或是弱一些的类型，而如果返回了弱类型就只能视为弱类型。


常量性

clr不允许将方法或参数声明为常量字段