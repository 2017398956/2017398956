# java 枚举类型的实现原理

转载自 http://blog.csdn.net/mhmyqn/article/details/48087247

Java从JDK1.5开始支持枚举，也就是说，Java一开始是不支持枚举的，就像泛型一样，都是JDK1.5才加入的新特性。通常一个特性如果在一开始没有提供，在语言发展后期才添加，会遇到一个问题，就是向后兼容性的问题。像Java在1.5中引入的很多特性，为了向后兼容，编译器会帮我们写的源代码做很多事情，比如泛型为什么会擦除类型，为什么会生成桥接方法，foreach迭代，自动装箱/拆箱等，这有个术语叫“语法糖”，而编译器的特殊处理叫“解语法糖”。那么像枚举也是在JDK1.5中才引入的，又是怎么实现的呢？

Java在1.5中添加了java.lang.Enum抽象类，它是所有枚举类型基类。提供了一些基础属性和基础方法。同时，对把枚举用作Set和Map也提供了支持，即java.util.EnumSet和java.util.EnumMap。

如何定义枚举类型

比如表示加减乘除操作，我们可以定义如下枚举：
 
	public enum Operator {  
	  
	    ADD

	}  

上面的枚举定义了枚举常量，同时，在枚举中还可以定义普通方法、抽象方法，如下所示：

	public enum Operator {  
	  
	    ADD {  
	        @Override  
	        public int calculate(int a, int b) {  
	            return a + b;  
	        }  
	    } ;  
	  
	    public abstract int calculate(int a, int b);  
	  
	}  

从上面可以看到，我们基本可以像定义类一样来定义枚举。我们还可以定义属性、构造方法等：

	public enum Operator {  
	  
	    ADD ("+") {  
	        @Override  
	        public int calculate(int a, int b) {  
	            return a + b;  
	        }  
	    } ;  
	  
	    Operator (String operator) {  
	        this.operator = operator;  
	    }  
	  
	    private String operator;  
	  
	    public abstract int calculate(int a, int b);  
	  
	    public String getOperator() {  
	        return operator;  
	    }  
	  
	}

## 实现原理分析  

既然可以像使用普通的类一样使用枚举，编译器究竟为我们做了些什么事呢？要想知道这其中的秘密，最有效的途径就是查看生成的字节码。下面就来看看上面定义的枚举生成的字节码是怎么样的。
首先使用命令行工具 javap Operator.class 来看看反编译的基本信息：

	localhost:mikan mikan$ javap Operator.class  
	Compiled from "Operator.java"  
	public abstract class com.mikan.Operator extends java.lang.Enum<com.mikan.Operator> {  
	  public static final com.mikan.Operator ADD;  
	  public static com.mikan.Operator[] values();  
	  public static com.mikan.Operator valueOf(java.lang.String);  
	  public abstract int calculate(int, int);  
	  public java.lang.String getOperator();  
	  com.mikan.Operator(java.lang.String, int, java.lang.String, com.mikan.Operator$1);  
	  static {};  
	}  

可以看到，一个枚举在经过编译器编译过后，变成了一个抽象类，它继承了 java.lang.Enum ；而枚举中定义的枚举常量，变成了相应的 public static final 属性，而且其类型就抽象类的类型，名字就是枚举常量的名字，同时我们可以在 Operator.class 的相同路径下看到一些内部类的 .class 文件 com/mikan/Operator$xxx.class ，也就是说这些命名字段分别使用了内部类来实现的；同时添加了两个方法 values() 和 valueOf(String) ；我们定义的构造方法本来只有一个参数，但却变成了三个参数；同时还生成了一个静态代码块。这些具体的内容接下来仔细看看。看下面详细的反编译信息：

	localhost:mikan mikan$ javap -c -v Operator.class  
	Classfile xxxxxxxxxxxxxxxxx/Operator.class
	  Last modified 2018-3-13; size 1436 bytes
	  MD5 checksum 1b92b16eb6557686951c020a464f9a51
	  Compiled from "Operator.java"
	public abstract class com.mikan.Operator extends java.lang.Enum<com.mikan.Operator>
	  minor version: 0
	  major version: 52
	  flags: ACC_PUBLIC, ACC_SUPER, ACC_ABSTRACT, ACC_ENUM
	Constant pool:
	   #1 = Methodref          #5.#51         // com/mikan/Operator."<init>":(Ljava/lang/String;ILjava/lang/String;)V/mikan/Operator;
	   #3 = Methodref          #53.#54        // "[Lcom/mikan/Operator;".clone:()Ljava/lang/Object;
	   #4 = Class              #21            // "[Lcom/mikan/Operator;"
	   #5 = Class              #55            // com/mikan/Operator
	   #6 = Methodref          #14.#56        // java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
	   #7 = Methodref          #14.#57        // java/lang/Enum."<init>":(Ljava/lang/String;I)V
	   #8 = Fieldref           #5.#58         // com/mikan/Operator.operator:Ljava/lang/String;
	   #9 = Class              #59            // com/mikan/Operator$1
	  #10 = String             #16            // ADD
	  #11 = String             #60            // +
	  #12 = Methodref          #9.#51         // com/mikan/Operator$1."<init>":(Ljava/lang/String;ILjava/lang/String;)V
	  #13 = Fieldref           #5.#61         // com/mikan/Operator.ADD:Lcom/mikan/Operator;
	  #14 = Class              #62            // java/lang/Enum
	  #15 = Utf8               InnerClasses
	  #16 = Utf8               ADD
	  #17 = Utf8               Lcom/mikan/Operator;
	  #18 = Utf8               operator
	  #19 = Utf8               Ljava/lang/String;
	  #20 = Utf8               $VALUES
	  #21 = Utf8               [Lcom/mikan/Operator;
	  #22 = Utf8               values
	  #23 = Utf8               ()[Lcom/mikan/Operator;
	  #24 = Utf8               Code
	  #25 = Utf8               LineNumberTable
	  #26 = Utf8               valueOf
	  #27 = Utf8               (Ljava/lang/String;)Lcom/mikan/Operator;
	  #28 = Utf8               LocalVariableTable
	  #29 = Utf8               name
	  #30 = Utf8               <init>
	  #31 = Utf8               (Ljava/lang/String;ILjava/lang/String;)V
	  #32 = Utf8               this
	  #33 = Utf8               Signature
	  #34 = Utf8               (Ljava/lang/String;)V
	  #35 = Utf8               calculate
	  #36 = Utf8               (II)I
	  #37 = Utf8               getOperator
	  #38 = Utf8               ()Ljava/lang/String;
	  #39 = Utf8               (Ljava/lang/String;ILjava/lang/String;Lcom/mikan/Operator$1;)V
	  #40 = Utf8               x0
	  #41 = Utf8               x1
	  #42 = Utf8               I
	  #43 = Utf8               x2
	  #44 = Utf8               x3
	  #45 = Utf8               Lcom/mikan/Operator$1;
	  #46 = Utf8               <clinit>
	  #47 = Utf8               ()V
	  #48 = Utf8               Ljava/lang/Enum<Lcom/mikan/Operator;>;
	  #49 = Utf8               SourceFile
	  #50 = Utf8               Operator.java
	  #51 = NameAndType        #30:#31        // "<init>":(Ljava/lang/String;ILjava/lang/String;)V
	  #52 = NameAndType        #20:#21        // $VALUES:[Lcom/mikan/Operator;
	  #53 = Class              #21            // "[Lcom/mikan/Operator;"
	  #54 = NameAndType        #63:#64        // clone:()Ljava/lang/Object;
	  #55 = Utf8               com/mikan/Operator
	  #56 = NameAndType        #26:#65        // valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
	  #57 = NameAndType        #30:#66        // "<init>":(Ljava/lang/String;I)V
	  #58 = NameAndType        #18:#19        // operator:Ljava/lang/String;
	  #59 = Utf8               com/mikan/Operator$1
	  #60 = Utf8               +
	  #61 = NameAndType        #16:#17        // ADD:Lcom/mikan/Operator;
	  #62 = Utf8               java/lang/Enum
	  #63 = Utf8               clone
	  #64 = Utf8               ()Ljava/lang/Object;
	  #65 = Utf8               (Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
	  #66 = Utf8               (Ljava/lang/String;I)V
	{
	  public static final com.mikan.Operator ADD;
	    descriptor: Lcom/mikan/Operator;
	    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL, ACC_ENUM
	
	  public static com.mikan.Operator[] values();
	    descriptor: ()[Lcom/mikan/Operator;
	    flags: ACC_PUBLIC, ACC_STATIC
	    Code:
	      stack=1, locals=0, args_size=0
	         0: getstatic     #2                  // Field $VALUES:[Lcom/mikan/Operator;
	         3: invokevirtual #3                  // Method "[Lcom/mikan/Operator;".clone:()Ljava/lang/Object;
	         6: checkcast     #4                  // class "[Lcom/mikan/Operator;"
	         9: areturn
	      LineNumberTable:
	        line 6: 0
	
	  public static com.mikan.Operator valueOf(java.lang.String);
	    descriptor: (Ljava/lang/String;)Lcom/mikan/Operator;
	    flags: ACC_PUBLIC, ACC_STATIC
	    Code:
	      stack=2, locals=1, args_size=1
	         0: ldc           #5                  // class com/mikan/Operator
	         2: aload_0
	         3: invokestatic  #6                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
	         6: checkcast     #5                  // class com/mikan/Operator
	         9: areturn
	      LineNumberTable:
	        line 6: 0
	      LocalVariableTable:
	        Start  Length  Slot  Name   Signature
	            0      10     0  name   Ljava/lang/String;
	
	  public abstract int calculate(int, int);
	    descriptor: (II)I
	    flags: ACC_PUBLIC, ACC_ABSTRACT
	
	  public java.lang.String getOperator();
	    descriptor: ()Ljava/lang/String;
	    flags: ACC_PUBLIC
	    Code:
	      stack=1, locals=1, args_size=1
	         0: aload_0
	         1: getfield      #8                  // Field operator:Ljava/lang/String;
	         4: areturn
	      LineNumberTable:
	        line 23: 0
	      LocalVariableTable:
	        Start  Length  Slot  Name   Signature
	            0       5     0  this   Lcom/mikan/Operator;
	
	  com.mikan.Operator(java.lang.String, int, java.lang.String, com.mikan.Operator$1);
	    descriptor: (Ljava/lang/String;ILjava/lang/String;Lcom/mikan/Operator$1;)V
	    flags: ACC_SYNTHETIC
	    Code:
	      stack=4, locals=5, args_size=5
	         0: aload_0
	         1: aload_1
	         2: iload_2
	         3: aload_3
	         4: invokespecial #1                  // Method "<init>":(Ljava/lang/String;ILjava/lang/String;)V
	         7: return
	      LineNumberTable:
	        line 6: 0
	      LocalVariableTable:
	        Start  Length  Slot  Name   Signature
	            0       8     0  this   Lcom/mikan/Operator;
	            0       8     1    x0   Ljava/lang/String;
	            0       8     2    x1   I
	            0       8     3    x2   Ljava/lang/String;
	            0       8     4    x3   Lcom/mikan/Operator$1;
	
	  static {};
	    descriptor: ()V
	    flags: ACC_STATIC
	    Code:
	      stack=5, locals=0, args_size=0
	         0: new           #9                  // class com/mikan/Operator$1
	         3: dup
	         4: ldc           #10                 // String ADD
	         6: iconst_0
	         7: ldc           #11                 // String +
	         9: invokespecial #12                 // Method com/mikan/Operator$1."<init>":(Ljava/lang/String;ILjava/lang/String;)V
	        12: putstatic     #13                 // Field ADD:Lcom/mikan/Operator;
	        15: iconst_1
	        16: anewarray     #5                  // class com/mikan/Operator
	        19: dup
	        20: iconst_0
	        21: getstatic     #13                 // Field ADD:Lcom/mikan/Operator;
	        24: aastore
	        25: putstatic     #2                  // Field $VALUES:[Lcom/mikan/Operator;
	        28: return
	      LineNumberTable:
	        line 7: 0
	        line 6: 15
	}
	Signature: #48                          // Ljava/lang/Enum<Lcom/mikan/Operator;>;
	SourceFile: "Operator.java"
	InnerClasses:
	     static #9; //class com/mikan/Operator$1

下面分析一下字节码中的各部分，其中：

	InnerClasses:
	     static #9; //class com/mikan/Operator$1

是 Operator 的内部类，这个内部类的详细信息后面会分析。

**静态代码块：**

	  static {};
	    descriptor: ()V
	    flags: ACC_STATIC
	    Code:
	      stack=5, locals=0, args_size=0
			 // 创建一个 Operator$1 内部类对象  
	         0: new           #9                  // class com/mikan/Operator$1
	         3: dup
			 // 接下来的三条指令分别是把三个参数推送到栈顶，然后调用 Operator$1 的编译器生成的 <init> 方法  
	         4: ldc           #10                 // String ADD
	         6: iconst_0
	         7: ldc           #11                 // String +
			 // 调用<init>方法 
	         9: invokespecial #12                 // Method com/mikan/Operator$1."<init>":(Ljava/lang/String;ILjava/lang/String;)V
	        // 设置 ADD 属性的值为新创建的对象  
			12: putstatic     #13                 // Field ADD:Lcom/mikan/Operator;
			// 下面是 new 了一个长度为 1 的 Operator 类型的数组，并分别设置数组中各元素的值为枚举类 Operator 的属性值
	        15: iconst_1
	        16: anewarray     #5                  // class com/mikan/Operator
	        19: dup
	        20: iconst_0
	        21: getstatic     #13                 // Field ADD:Lcom/mikan/Operator;
	        24: aastore
			// 下面是设置属性 $VALUES 的值为刚创建的数组  
	        25: putstatic     #2                  // Field $VALUES:[Lcom/mikan/Operator;
	        28: return
	      LineNumberTable:
	        line 7: 0
	        line 6: 15
	}

其实编译器生成的这个静态代码块做了如下工作：设置生成的一个公共静态常量字段的值，同时编译器还生成了一个静态字段 $VALUES ，保存的是枚举类型定义的所有枚举常量。相当于下面的代码：

	Operator ADD = new Operator1();  
	Operator[] $VALUES = new Operator[4];  
	$VALUES[0] = ADD; 

**编译器添加的 values 方法：**

	public static com.mikan.Operator[] values();
	    descriptor: ()[Lcom/mikan/Operator;
	    flags: ACC_PUBLIC, ACC_STATIC
	    Code:
	      stack=1, locals=0, args_size=0
	         0: getstatic     #2                  // Field $VALUES:[Lcom/mikan/Operator;
	         3: invokevirtual #3                  // Method "[Lcom/mikan/Operator;".clone:()Ljava/lang/Object;
	         6: checkcast     #4                  // class "[Lcom/mikan/Operator;"
	         9: areturn

这个方法是一个公共的静态方法，所以我们可以直接调用该方法（Operator.values()）,返回这个枚举值的数组，另外，这个方法的实现是，克隆在静态代码块中初始化的$VALUES字段的值，并把类型强转成 Operator[] 类型返回。它相当于下面的代码：

	public static com.mikan.Operator[] values() {  
		return (Operator[])$VALUES.clone();  
	} 

**编译器添加的 valueOf 方法：**

	public static com.mikan.Operator valueOf(java.lang.String);
	    descriptor: (Ljava/lang/String;)Lcom/mikan/Operator;
	    flags: ACC_PUBLIC, ACC_STATIC
	    Code:
	      stack=2, locals=1, args_size=1
	         0: ldc           #5                  // class com/mikan/Operator
	         2: aload_0
	         3: invokestatic  #6                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
	         6: checkcast     #5                  // class com/mikan/Operator
	         9: areturn

这个方法是一个公共的静态方法，所以我们可以直接调用该方法（Operator.valueOf()）,返回参数字符串表示的枚举常量，另外，这个方法的实现是，调用父类 Enum 的 valueOf 方法，并把类型强转成 Operator。它相当于如下的代码：

	public static com.mikan.Operator valueOf(String name) {  
		return (Operator)Enum.valueOf(Operator.class, name);  
	}  

Enum.valueOf(Operator.class, name) 的代码如下：

    public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null）			// 如果 name 是 null 则直接抛出空指向异常
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException( // 如果找不到枚举类中对应的枚举常量，则抛出非法参数异常
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }

从上面的代码，我们可以看出获取枚举值的时候是通过 enumConstantDirectory() 方法获得的，而 enumConstantDirectory() 的返回值是一个 Map 集合，其 key 为枚举常量，如本例中的 ADD ； 而 value 是一个泛型，在本例中为 Operator 。有代码为证：

	Map<String, T> enumConstantDirectory() {
        if (enumConstantDirectory == null) {
            T[] universe = getEnumConstantsShared();
            if (universe == null)
                throw new IllegalArgumentException(
                    getName() + " is not an enum type");
            Map<String, T> m = new HashMap<>(2 * universe.length);
            for (T constant : universe)
                m.put(((Enum<?>)constant).name(), constant);
            enumConstantDirectory = m;
        }
        return enumConstantDirectory;
    }

**生成的内部类**

下面看看生成的内部类Operator$1：

	>javap Operator$1.class
	Compiled from "Operator.java"
	final class com.mikan.Operator$1 extends com.mikan.Operator {
	  com.mikan.Operator$1(java.lang.String, int, java.lang.String);
	  public int calculate(int, int);
	}

可以看到，实现内部类是继承自Operator，即

	ADD {  
	    @Override  
	    public int calculate(int a, int b) {  
	        return a + b;  
	    }  
	}

这就是说，我们定义的每个枚举常量，最终都生成了一个像上面这样的内部类。

**构造方法为什么增加了两个参数？**

有一个问题，构造方法我们明明只定义了一个参数，为什么生成的构造方法是三个参数呢？

从Enum类中我们可以看到，为每个枚举都定义了两个属性，name 和 ordinal，name 表示我们定义的枚举常量的名称，如 ADD 等，而 ordinal 是一个顺序号，根据定义的顺序分别赋予一个整形值，从 **0** 开始。在枚举常量初始化时，会自动为初始化这两个字段，设置相应的值，所以才在构造方法中添加了两个参数。即：

	com.mikan.Operator$1(String name, int ordinal, String operator);  

另外,举常量生成的内部类基本上差不多，读者可以自行添加一个枚举常量如 SUBTRACT 试试，这里就不重复说明了。我们可以从 Enum 类的代码中看到，定义的 name 和 ordinal 属性都是 final 的，而且大部分方法也都是 final 的，特别是 clone、readObject、writeObject 这三个方法，这三个方法和枚举通过静态代码块来进行初始化一起，它保证了枚举类型的不可变性，不能通过克隆，不能通过序列化和反序列化来复制枚举，这能保证一个枚举常量只是一个实例，即是单例的，所以在 effective java 中推荐使用枚举来实现单例。

## 总结
 
枚举本质上是通过普通的类来实现的，只是编译器为我们进行了处理。每个枚举类型都继承自 java.lang.Enum，并自动添加了 values 和 valueOf 方法。而每个枚举常量是一个静态常量字段，使用内部类实现，该内部类继承了枚举类。所有枚举常量都通过静态代码块来进行初始化，即在类加载期间就初始化。另外通过把 clone、readObject、writeObject 这三个方法定义为 final 的，同时实现是抛出相应的异常。这样保证了每个枚举类型及枚举常量都是不可变的。可以利用枚举的这两个特性来实现线程安全的单例。