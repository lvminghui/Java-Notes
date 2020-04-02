20  为后代设计接口　

  在 Java 8 之前，不可能在不破坏现有实现的情况下为接口添加方法。 如果向接口添加了一个新方法，现有的实现通常会缺少该方法，从而导致编译时错误。 在 Java 8 中，添加了默认方法（default method）构造[JLS 9.4]，目的是允许将方法添加到现有的接口。 但是增加新的方法到现有的接口是充满风险的。

　　默认方法的声明包含一个默认实现，该方法允许实现接口的类直接使用，而不必实现默认方法。 虽然在 Java 中添加默认方法可以将方法添加到现有接口，但不能保证这些方法可以在所有已有的实现中使用。 默认的方法被「注入（injected）」到现有的实现中，没有经过实现类的知道或同意。 在 Java 8 之前，这些实现是用默认的接口编写的，它们的接口永远不会获得任何新的方法。

　　许多新的默认方法被添加到 Java 8 的核心集合接口中，主要是为了方便使用 lambda 表达式（第 6 章）。 Java 类库的默认方法是高质量的通用实现，在大多数情况下，它们工作正常。 但是，编写一个默认方法并不总是可能的，它保留了每个可能的实现的所有不变量。

　　例如，考虑在 Java 8 中添加到 Collection 接口的 removeIf 方法。此方法删除给定布尔方法（或 Predicate 函数式接口）返回 true 的所有元素。默认实现被指定为使用迭代器遍历集合，调用每个元素的谓词，并使用迭代器的 remove 方法删除谓词返回 true 的元素。 据推测，这个声明看起来像这样：默认实现被指定为使用迭代器遍历集合，调用每个元素的 Predicate 函数式接口，并使用迭代器的 remove 方法删除 Predicate 函数式接口返回 true 的元素。 根据推测，这个声明看起来像这样：

// Default method added to the Collection interface in Java 8
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext(); ) {
        if (filter.test(it.next())) {
            it.remove();
            result = true;
        }
    }
    return result;
}
　　这是可能为 removeIf 方法编写的最好的通用实现，但遗憾的是，它在一些实际的 Collection 实现中失败了。 例如，考虑 org.apache.commons.collections4.collection.SynchronizedCollection 方法。 这个类出自 Apache Commons 类库中，与 java.util 包中的静态工厂 Collections.synchronizedCollection 方法返回的类相似。 Apache 版本还提供了使用客户端提供的对象进行锁定的能力，以代替集合。 换句话说，它是一个包装类（条目 18），它们的所有方法在委托给包装集合类之前在一个锁定对象上进行同步。

　　Apache 的 SynchronizedCollection 类仍然在积极维护，但在撰写本文时，并未重写 removeIf 方法。 如果这个类与 Java 8 一起使用，它将继承 removeIf 的默认实现，但实际上不能保持类的基本承诺：自动同步每个方法调用。 默认实现对同步一无所知，并且不能访问包含锁定对象的属性。 如果客户端在另一个线程同时修改集合的情况下调用 SynchronizedCollection 实例上的 removeIf 方法，则可能会导致 ConcurrentModificationException 异常或其他未指定的行为。

　　为了防止在类似的 Java 平台类库实现中发生这种情况，比如 Collections.synchronizedCollection 返回的包级私有的类，JDK 维护者必须重写默认的 removeIf 实现和其他类似的方法来在调用默认实现之前执行必要的同步。 原来不属于 Java 平台的集合实现没有机会与接口更改进行类似的改变，有些还没有这样做。

　　在默认方法的情况下，接口的现有实现类可以在没有错误或警告的情况下编译，但在运行时会失败。 虽然不是非常普遍，但这个问题也不是一个孤立的事件。 在 Java 8 中添加到集合接口的一些方法已知是易受影响的，并且已知一些现有的实现会受到影响。

　　应该避免使用默认方法向现有的接口添加新的方法，除非这个需要是关键的，在这种情况下，你应该仔细考虑，以确定现有的接口实现是否会被默认的方法实现所破坏。然而，默认方法对于在创建接口时提供标准的方法实现非常有用，以减轻实现接口的任务（详见第 20 条）。

　　还值得注意的是，默认方法不是被用来设计，来支持从接口中移除方法或者改变现有方法的签名的目的。在不破坏现有客户端的情况下，这些接口都不可能发生更改。

　　准则是清楚的。 尽管默认方法现在是 Java 平台的一部分，但是非常悉心地设计接口仍然是非常重要的。 虽然默认方法可以将方法添加到现有的接口，但这样做有很大的风险。 如果一个接口包含一个小缺陷，可能会永远惹怒用户。 如果一个接口严重缺陷，可能会破坏包含它的 API。

　　因此，在发布之前测试每个新接口是非常重要的。 多个程序员应该以不同的方式实现每个接口。 至少，你应该准备三种不同的实现。 编写多个使用每个新接口的实例来执行各种任务的客户端程序同样重要。 这将大大确保每个接口都能满足其所有的预期用途。 这些步骤将允许你在发布之前发现接口中的缺陷，但仍然可以轻松地修正它们。 虽然在接口被发布后可能会修正一些存在的缺陷，但不要太指望这一点。

22. 接口仅用来定义类型
　　当类实现接口时，该接口作为一种类型（type），可以用来引用类的实例。因此，一个类实现了一个接口，因此表明客户端可以如何处理类的实例。为其他目的定义接口是不合适的。

　　一种失败的接口就是所谓的常量接口（constant interface）。 这样的接口不包含任何方法; 它只包含静态 final 属性，每个输出一个常量。 使用这些常量的类实现接口，以避免需要用类名限定常量名。 这里是一个例子：

// Constant interface antipattern - do not use!
public interface PhysicalConstants {
    // Avogadro's number (1/mol)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // Boltzmann constant (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // Mass of the electron (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
　　常量接口模式是对接口的糟糕使用。 类在内部使用一些常量，完全属于实现细节。实现一个常量接口会导致这个实现细节泄漏到类的导出 API 中。对类的用户来说，类实现一个常量接口是没有意义的。事实上，它甚至可能使他们感到困惑。更糟糕的是，它代表了一个承诺：如果在将来的版本中修改了类，不再需要使用常量，那么它仍然必须实现接口，以确保二进制兼容性。如果一个非 final 类实现了常量接口，那么它的所有子类的命名空间都会被接口中的常量所污染。

　　Java 平台类库中有多个常量接口，如 java.io.ObjectStreamConstants。 这些接口应该被视为不规范的，不应该被效仿。

　　如果你想导出常量，有几个合理的选择方案。 如果常量与现有的类或接口紧密相关，则应将其添加到该类或接口中。 例如，所有数字基本类型的包装类，如 Integer 和 Double，都会导出 MIN_VALUE 和 MAX_VALUE 常量。 如果常量最好被看作枚举类型的成员，则应该使用枚举类型（详见第 34 条）导出它们。 否则，你应该用一个不可实例化的工具类来导出常量（详见第 4 条）。 下是前面所示的 PhysicalConstants 示例的工具类的版本：

// Constant utility class
package com.effectivejava.science;

public class PhysicalConstants {
  private PhysicalConstants() { }  // Prevents instantiation

  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
　　顺便提一下，请注意在数字文字中使用下划线字符（_）。 从 Java 7 开始，合法的下划线对数字字面量的值没有影响，但是如果使用得当的话可以使它们更容易阅读。 无论是固定的浮点数，如果他们包含五个或更多的连续数字，考虑将下划线添加到数字字面量中。 对于底数为 10 的数字，无论是整型还是浮点型的，都应该用下划线将数字分成三个数字组，表示一千的正负幂。

　　通常，实用工具类要求客户端使用类名来限定常量名，例如 PhysicalConstants.AVOGADROS_NUMBER。 如果大量使用实用工具类导出的常量，则通过使用静态导入来限定具有类名的常量：

// Use of static import to avoid qualifying constants
import static com.effectivejava.science.PhysicalConstants.*;

public class Test {
    double  atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
    ...
    // Many more uses of PhysicalConstants justify static import
}
　　总之，接口只能用于定义类型。 它们不应该仅用于导出常量。
  
  23. 类层次结构优于标签类
　　有时你可能会碰到一个类，它的实例有两个或更多的风格，并且包含一个标签字段（tag field），表示实例的风格。 例如，考虑这个类，它可以表示一个圆形或矩形：

// Tagged class - vastly inferior to a class hierarchy!
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // Tag field - the shape of this figure
    final Shape shape;

    // These fields are used only if shape is RECTANGLE
    double length;
    double width;

    // This field is used only if shape is CIRCLE
    double radius;

    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
          case RECTANGLE:
            return length * width;
          case CIRCLE:
            return Math.PI * (radius * radius);
          default:
            throw new AssertionError(shape);
        }
    }
}
　　这样的标签类具有许多缺点。 它们充斥着杂乱无章的样板代码，包括枚举声明，标签字段和 switch 语句。 可读性更差，因为多个实现在一个类中混杂在一起。 内存使用增加，因为实例负担属于其他风格不相关的领域。 字段不能成为 final，除非构造方法初始化不相关的字段，导致更多的样板代码。 构造方法在编译器的帮助下，必须设置标签字段并初始化正确的数据字段：如果初始化错误的字段，程序将在运行时失败。 除非可以修改其源文件，否则不能将其添加到标记的类中。 如果你添加一个风格，你必须记得给每个 switch 语句添加一个 case，否则这个类将在运行时失败。 最后，一个实例的数据类型没有提供任何关于风格的线索。 总之，标签类是冗长的，容易出错的，而且效率低下。

　　幸运的是，像 Java 这样的面向对象的语言为定义一个能够表示多种风格对象的单一数据类型提供了更好的选择：子类型化（subtyping）。标签类仅仅是一个类层次的简单的模仿。

　　要将标签类转换为类层次，首先定义一个包含抽象方法的抽象类，该标签类的行为取决于标签值。 在 Figure 类中，只有一个这样的方法，就是 area 方法。 这个抽象类是类层次的根。 如果有任何方法的行为不依赖于标签的值，把它们放在这个类中。 同样，如果有所有的方法使用的数据字段，把它们放在这个类。Figure 类中不存在这种与类型无关的方法或字段。

　　接下来，为原始标签类的每种类型定义一个根类的具体子类。 在我们的例子中，有两个类型：圆形和矩形。 在每个子类中包含特定于改类型的数据字段。 在我们的例子中，半径字段是属于圆的，长度和宽度字段都是矩形的。 还要在每个子类中包含根类中每个抽象方法的适当实现。 这里是对应于 Figure 类的类层次：

// Class hierarchy replacement for a tagged class
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
　　这个类层次纠正了之前提到的标签类的每个缺点。 代码简单明了，不包含原文中的样板文件。 每种类型的实现都是由自己的类来分配的，而这些类都没有被无关的数据字段所占用。 所有的字段是 final 的。 编译器确保每个类的构造方法初始化其数据字段，并且每个类都有一个针对在根类中声明的每个抽象方法的实现。 这消除了由于缺少 switch-case 语句而导致的运行时失败的可能性。 多个程序员可以独立地继承类层次，并且可以相互操作，而无需访问根类的源代码。 每种类型都有一个独立的数据类型与之相关联，允许程序员指出变量的类型，并将变量和输入参数限制为特定的类型。

　　类层次的另一个优点是可以使它们反映类型之间的自然层次关系，从而提高了灵活性，并提高了编译时类型检查的效率。 假设原始示例中的标签类也允许使用正方形。 类层次可以用来反映一个正方形是一种特殊的矩形（假设它们是不可变的）：

class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
　　请注意，上述层次结构中的字段是直接访问的，而不是通过访问器方法访问的。 这里是为了简洁起见，如果类层次是公开的（详见第 16 条），这将是一个糟糕的设计。

　　总之，标签类很少有适用的情况。 如果你想写一个带有显式标签字段的类，请考虑标签字段是否可以被删除，并是否能被类层次结构替换。 当遇到一个带有标签字段的现有类时，可以考虑将其重构为一个类层次结构。

24. 支持使用静态成员类而不是非静态类
　　嵌套类（nested class）是在另一个类中定义的类。 嵌套类应该只存在于其宿主类（enclosing class）中。 如果一个嵌套类在其他一些情况下是有用的，那么它应该是一个顶级类。 有四种嵌套类：静态成员类，非静态成员类，匿名类和局部类。 除了第一种以外，剩下的三种都被称为内部类（inner class）。 这个条目告诉你什么时候使用哪种类型的嵌套类以及为什么使用。

　　静态成员类是最简单的嵌套类。 最好把它看作是一个普通的类，恰好在另一个类中声明，并且可以访问所有宿主类的成员，甚至是那些被声明为私有类的成员。 静态成员类是其宿主类的静态成员，并遵循与其他静态成员相同的可访问性规则。 如果它被声明为 private，则只能在宿主类中访问，等等。

　　静态成员类的一个常见用途是作为公共帮助类，仅在与其外部类一起使用时才有用。 例如，考虑一个描述计算器支持的操作的枚举类型（详见第 34 条）。 Operation 枚举应该是 Calculator 类的公共静态成员类。 Calculator 客户端可以使用 Calculator.Operation.PLUS 和 Calculator.Operation.MINUS 等名称来引用操作。

　　在语法上，静态成员类和非静态成员类之间的唯一区别是静态成员类在其声明中具有 static 修饰符。 尽管句法相似，但这两种嵌套类是非常不同的。 非静态成员类的每个实例都隐含地与其包含的类的宿主实例相关联。 在非静态成员类的实例方法中，可以调用宿主实例上的方法，或者使用限定的构造[JLS，15.8.4] 获得对宿主实例的引用。 如果嵌套类的实例可以与其宿主类的实例隔离存在，那么嵌套类必须是静态成员类：不可能在没有宿主实例的情况下创建非静态成员类的实例。

　　非静态成员类实例和其宿主实例之间的关联是在创建成员类实例时建立的，并且之后不能被修改。 通常情况下，通过在宿主类的实例方法中调用非静态成员类构造方法来自动建立关联。 尽管很少有可能使用表达式 enclosingInstance.new MemberClass(args) 手动建立关联。 正如你所预料的那样，该关联在非静态成员类实例中占用了空间，并为其构建添加了时间开销。

　　非静态成员类的一个常见用法是定义一个 Adapter [Gamma95]，它允许将外部类的实例视为某个不相关类的实例。 例如，Map 接口的实现通常使用非静态成员类来实现它们的集合视图，这些视图由 Map 的 keySet，entrySet 和 values 方法返回。 同样，集合接口（如 Set 和 List）的实现通常使用非静态成员类来实现它们的迭代器：

// Typical use of a nonstatic member class
public class MySet<E> extends AbstractSet<E> {
    ... // Bulk of the class omitted

    @Override 
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        ...
    }
}
　　如果你声明了一个不需要访问宿主实例的成员类，总是把 static 修饰符放在它的声明中，使它成为一个静态成员类，而不是非静态的成员类。 如果你忽略了这个修饰符，每个实例都会有一个隐藏的外部引用给它的宿主实例。 如前所述，存储这个引用需要占用时间和空间。 更严重的是，并且会导致即使宿主类在满足垃圾回收的条件时却仍然驻留在内存中（详见第 7 条）。 由此产生的内存泄漏可能是灾难性的。 由于引用是不可见的，所以通常难以检测到。

　　私有静态成员类的常见用法是表示由它们的宿主类表示的对象的组件。 例如，考虑将键与值相关联的 Map 实例。 许多 Map 实现对于映射中的每个键值对都有一个内部的 Entry 对象。 当每个 entry 都与 Map 关联时，entry 上的方法 (getKey，getValue 和 setValue) 不需要访问 Map。 因此，使用非静态成员类来表示 entry 将是浪费的：私有静态成员类是最好的。 如果意外地忽略了 entry 声明中的 static 修饰符，Map 仍然可以工作，但是每个 entry 都会包含对 Map 的引用，浪费空间和时间。

　　如果所讨论的类是导出类的公共或受保护成员，则在静态和非静态成员类之间正确选择是非常重要的。 在这种情况下，成员类是导出的 API 元素，如果不违反向后兼容性，就不能在后续版本中从非静态变为静态成员类。

　　正如你所期望的，一个匿名类没有名字。 它不是其宿主类的成员。 它不是与其他成员一起声明，而是在使用时同时声明和实例化。 在表达式合法的代码中，匿名类是允许的。 当且仅当它们出现在非静态上下文中时，匿名类才会封装实例。 但是，即使它们出现在静态上下文中，它们也不能有除常量型变量之外的任何静态成员，这些常量型变量包括 final 的基本类型，或者初始化常量表达式的字符串属性[JLS，4.12.4]。

　　匿名类的适用性有很多限制。 除了在声明的时候之外，不能实例化它们。 你不能执行 instanceof 方法测试或者做任何其他需要你命名的类。 不能声明一个匿名类来实现多个接口，或者继承一个类并同时实现一个接口。 匿名类的客户端不能调用除父类型继承的成员以外的任何成员。 因为匿名类在表达式中出现，所以它们必须保持简短 —— 约十行或更少 —— 否则可读性将受到影响。

　　在将 lambda 表达式添加到 Java（第 6 章）之前，匿名类是创建小函数对象和处理对象的首选方法，但 lambda 表达式现在是首选（详见第 42 条）。 匿名类的另一个常见用途是实现静态工厂方法（请参阅条目 20 中的 intArrayAsList）。

　　局部类是四种嵌套类中使用最少的。 一个局部类可以在任何可以声明局部变量的地方声明，并遵守相同的作用域规则。 局部类与其他类型的嵌套类具有共同的属性。 像成员类一样，他们有名字，可以重复使用。 就像匿名类一样，只有在非静态上下文中定义它们时，它们才会包含实例，并且它们不能包含静态成员。 像匿名类一样，应该保持简短，以免损害可读性。

　　回顾一下，有四种不同的嵌套类，每个都有它的用途。 如果一个嵌套的类需要在一个方法之外可见，或者太长而不能很好地适应一个方法，使用一个成员类。 如果一个成员类的每个实例都需要一个对其宿主实例的引用，使其成为非静态的; 否则，使其静态。 假设这个类属于一个方法内部，如果你只需要从一个地方创建实例，并且存在一个预置类型来说明这个类的特征，那么把它作为一个匿名类；否则，把它变成局部类。
  
  25. 将源文件限制为单个顶级类
　　虽然 Java 编译器允许在单个源文件中定义多个顶级类，但这样做没有任何好处，并且存在重大风险。 风险源于在源文件中定义多个顶级类使得为类提供多个定义成为可能。 使用哪个定义会受到源文件传递给编译器的顺序的影响。

　　为了具体说明，请考虑下面源文件，其中只包含一个引用其他两个顶级类（Utensil 和 Dessert 类）的成员的 Main 类：

public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + [Dessert.NAME](http://Dessert.NAME));
    }
}
复制ErrorOK!
　　现在假设在 Utensil.java 的源文件中同时定义了 Utensil 和 Dessert：

// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
复制ErrorOK!
　　当然，main 方法会打印 pancake。

　　现在假设你不小心创建了另一个名为 Dessert.java 的源文件，它定义了相同的两个类：

// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
复制ErrorOK!
　　如果你足够幸运，使用命令 javac Main.java Dessert.java 编译程序，编译将失败，编译器会告诉你，你已经多次定义了类 Utensil 和 Dessert。 这是因为编译器首先编译 Main.java，当它看到对 Utensil 的引用（它在 Dessert 的引用之前）时，它将在 Utensil.java 中查找这个类并找到 Utensil 和 Dessert。 当编译器在命令行上遇到 Dessert.java 时，它也将拉入该文件，导致它遇到 Utensil 和 Dessert 的定义。

　　如果使用命令 javac Main.java 或 javac Main.java Utensil.java 编译程序，它的行为与在编写 Dessert.java 文件（即打印 pancake）之前的行为相同。 但是，如果使用命令 javac Dessert.java Main.java 编译程序，它将打印 potpie。 程序的行为因此受到源文件传递给编译器的顺序的影响，这显然是不可接受的。

　　解决这个问题很简单，将顶层类（如我们的例子中的 Utensil 和 Dessert）分割成单独的源文件。 如果试图将多个顶级类放入单个源文件中，请考虑使用静态成员类（详见第 24 条）作为将类拆分为单独的源文件的替代方法。 如果这些类从属于另一个类，那么将它们变成静态成员类通常是更好的选择，因为它提高了可读性，并且可以通过声明它们为私有（详见第 15 条）来减少类的可访问性。下面是我们的例子看起来如何使用静态成员类：

// Static member classes instead of multiple top-level classes
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + [Dessert.NAME](http://Dessert.NAME));
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
复制ErrorOK!
　　这个教训很清楚：永远不要将多个顶级类或接口放在一个源文件中。 遵循这个规则保证在编译时不能有多个定义。 这又保证了编译生成的类文件以及生成的程序的行为与源文件传递给编译器的顺序无关。
  
  　自 Java 5 以来，泛型已经成为该语言的一部分。 在泛型之前，你必须转换从集合中读取的每个对象。 如果有人不小心插入了错误类型的对象，则在运行时可能会失败。 使用泛型，你告诉编译器在每个集合中允许哪些类型的对象。 编译器会自动插入强制转换，并在编译时告诉你是否尝试插入错误类型的对象。 这样做的结果是既安全又清晰的程序，但这些益处，不限于集合，是有代价的。 本章告诉你如何最大限度地提高益处，并将并发症降至最低。

26. 不要使用原始类型
　　首先，有几个术语。一个类或接口，它的声明有一个或多个类型参数（type parameters ），被称之为泛型类或泛型接口[JLS，8.1.2,9.1.2]。 例如，List 接口具有单个类型参数 E，表示其元素类型。 接口的全名是 List<E>（读作「E」的列表），但是人们经常称它为 List。 泛型类和接口统称为泛型类型（generic types）。

　　每个泛型定义了一组参数化类型（parameterized types），它们由类或接口名称组成，后跟一个与泛型类型的形式类型参数[JLS，4.4,4.5] 相对应的实际类型参数的尖括号「<>」列表。 例如，List<String>（读作「字符串列表」）是一个参数化类型，表示其元素类型为 String 的列表。 （String 是与形式类型参数 E 相对应的实际类型参数）。

　　最后，每个泛型定义了一个原始类型（raw type），它是没有任何类型参数的泛型类型的名称[JLS，4.8]。 例如，对应于 List<E> 的原始类型是 List。 原始类型的行为就像所有的泛型类型信息都从类型声明中被清除一样。 它们的存在主要是为了与没有泛型之前的代码相兼容。

　　在泛型被添加到 Java 之前，这是一个典型的集合声明。 从 Java 9 开始，它仍然是合法的，但并不是典型的声明方式了：

// Raw collection type - don't do this!

// My stamp collection. Contains only Stamp instances.
private final Collection stamps = ... ;
复制ErrorOK!
　　如果你今天使用这个声明，然后不小心把 coin 实例放入你的 stamp 集合中，错误的插入编译和运行没有错误（尽管编译器发出一个模糊的警告）：

// Erroneous insertion of coin into stamp collection
stamps.add(new Coin( ... )); // Emits "unchecked call" warning
复制ErrorOK!
　　直到您尝试从 stamp 集合中检索 coin 实例时才会发生错误：

// Raw iterator type - don't do this!
for (Iterator i = stamps.iterator(); i.hasNext(); )
    Stamp stamp = (Stamp) i.next(); // Throws ClassCastException
        stamp.cancel();
复制ErrorOK!
　　正如本书所提到的，在编译完成之后尽快发现错误是值得的，理想情况是在编译时。 在这种情况下，直到运行时才发现错误，在错误发生后的很长一段时间，以及可能远离包含错误的代码的代码中。 一旦看到 ClassCastException，就必须搜索代码类库，查找将 coin 实例放入 stamp 集合的方法调用。 编译器不能帮助你，因为它不能理解那个说「仅包含 stamp 实例」的注释。

　　对于泛型，类型声明包含的信息，而不是注释：

// Parameterized collection type - typesafe
private final Collection<Stamp> stamps = ... ;
复制ErrorOK!
　　从这个声明中，编译器知道 stamps 集合应该只包含 Stamp 实例，并保证它是 true，假设你的整个代码类库编译时不发出（或者抑制；参见条目 27）任何警告。 当使用参数化类型声明声明 stamps 时，错误的插入会生成一个编译时错误消息，告诉你到底发生了什么错误：

Test.java:9: error: incompatible types: Coin cannot be converted
to Stamp
    c.add(new Coin());
              ^
复制ErrorOK!
　　当从集合中检索元素时，编译器会为你插入不可见的强制转换，并保证它们不会失败（再假设你的所有代码都不会生成或禁止任何编译器警告）。 虽然意外地将 coin 实例插入 stamp 集合的预期可能看起来很牵强，但这个问题是真实的。 例如，很容易想象将 BigInteger 放入一个只包含 BigDecimal 实例的集合中。

　　如前所述，使用原始类型（没有类型参数的泛型）是合法的，但是你不应该这样做。 如果你使用原始类型，则会丧失泛型的所有安全性和表达上的优势。 鉴于你不应该使用它们，为什么语言设计者首先允许原始类型呢？ 答案是为了兼容性。 泛型被添加时，Java 即将进入第二个十年，并且有大量的代码没有使用泛型。 所有这些代码都是合法的，并且与使用泛型的新代码进行交互操作被认为是至关重要的。 将参数化类型的实例传递给为原始类型设计的方法必须是合法的，反之亦然。 这个需求，被称为迁移兼容性，驱使决策支持原始类型，并使用擦除来实现泛型（详见第 28 条）。

　　虽然不应使用诸如 List 之类的原始类型，但可以使用参数化类型来允许插入任意对象（如 List<Object>）。 原始类型 List 和参数化类型 List<Object> 之间有什么区别？ 松散地说，前者已经选择了泛型类型系统，而后者明确地告诉编译器，它能够保存任何类型的对象。 虽然可以将 List<String> 传递给 List 类型的参数，但不能将其传递给 List<Object> 类型的参数。 泛型有子类型的规则，List<String> 是原始类型 List 的子类型，但不是参数化类型 List<Object> 的子类型（条目 28）。 因此，如果使用诸如 List 之类的原始类型，则会丢失类型安全性，但是如果使用参数化类型（例如 List<Object>）则不会。

　　为了具体说明，请考虑以下程序：

// Fails at runtime - unsafeAdd method uses a raw type (List)!
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0); // Has compiler-generated cast
}

private static void unsafeAdd(List list, Object o) {
    list.add(o);
}
复制ErrorOK!
　　此程序可以编译，它使用原始类型列表，但会收到警告：

Test.java:10: warning: [unchecked] unchecked call to add(E) as a
member of the raw type List
    list.add(o);
            ^
复制ErrorOK!
　　实际上，如果运行该程序，则当程序尝试调用 strings.get(0) 的结果（一个 Integer）转换为一个 String 时，会得到 ClassCastException 异常。 这是一个编译器生成的强制转换，因此通常会保证成功，但在这种情况下，我们忽略了编译器警告并付出了代价。

　　如果用 unsafeAdd 声明中的参数化类型 List<Object> 替换原始类型 List，并尝试重新编译该程序，则会发现它不再编译，而是发出错误消息：

Test.java:5: error: incompatible types: List<String> cannot be
converted to List<Object>
    unsafeAdd(strings, Integer.valueOf(42));
复制ErrorOK!
　　你可能会试图使用原始类型来处理元素类型未知且无关紧要的集合。 例如，假设你想编写一个方法，它需要两个集合并返回它们共同拥有的元素的数量。 如果是泛型新手，那么您可以这样写：

// Use of raw type for unknown element type - don't do this!
static int numElementsInCommon(Set s1, Set s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
复制ErrorOK!
　　这种方法可以工作，但它使用原始类型，这是危险的。 安全替代方式是使用无限制通配符类型（unbounded wildcard types）。 如果要使用泛型类型，但不知道或关心实际类型参数是什么，则可以使用问号来代替。 例如，泛型类型 Set<E> 的无限制通配符类型是 Set<?>（读取「某种类型的集合」）。 它是最通用的参数化的 Set 类型，能够保持任何集合。 下面是 numElementsInCommon 方法使用无限制通配符类型声明的情况：

// Uses unbounded wildcard type - typesafe and flexible
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
复制ErrorOK!
　　无限制通配符 Set<?> 与原始类型 Set 之间有什么区别？ 问号真的给你放任何东西吗？ 这不是要点，但通配符类型是安全的，原始类型不是。 你可以将任何元素放入具有原始类型的集合中，轻易破坏集合的类型不变性（如第 119 页上的 unsafeAdd 方法所示）; 你不能把任何元素（除 null 之外）放入一个 Collection<?> 中。 试图这样做会产生一个像这样的编译时错误消息：

WildCard.java:13: error: incompatible types: String cannot be
converted to CAP#1
    c.add("verboten");
          ^
  where CAP#1 is a fresh type-variable:
    CAP#1 extends Object from capture of ?
复制ErrorOK!
　　不可否认的是，这个错误信息留下了一些需要的东西，但是编译器已经完成了它的工作，不管它的元素类型是什么，都不会破坏集合的类型不变性。 你不仅不能将任何元素（除 null 以外）放入一个 Collection<?> 中，并且根本无法猜测你会得到那种类型的对象。 如果这些限制是不可接受的，可以使用泛型方法（详见第 30 条）或有限制的通配符类型（详见第 31 条）。

　　对于不应该使用原始类型的规则，有一些小例外。 你必须在类字面值（class literals）中使用原始类型。 规范中不允许使用参数化类型（尽管它允许数组类型和基本类型）[JLS，15.8.2]。 换句话说，List.class，String[].class 和 int.class 都是合法的，但 List<String>.class 和 List<?>.class 都是不合法的。

　　规则的第二个例外与 instanceof 操作符有关。 因为泛型类型信息在运行时被擦除，所以在无限制通配符类型以外的参数化类型上使用 instanceof 运算符是非法的。 使用无限制通配符类型代替原始类型，不会对 instanceof 运算符的行为产生任何影响。 在这种情况下，尖括号（<>）和问号（?）就显得多余。 以下是使用泛型类型的 instanceof 运算符的首选方法：

// Legitimate use of raw type - instanceof operator
if (o instanceof Set) {       // Raw type
    Set<?> s = (Set<?>) o;    // Wildcard type
    ...
}
复制ErrorOK!
　　请注意，一旦确定 o 对象是一个 Set，则必须将其转换为通配符 Set<?>，而不是原始类型 Set。 这是一个受检查的（checked）转换，所以不会导致编译器警告。

　　总之，使用原始类型可能导致运行时异常，所以不要使用它们。 原始类型只是为了与引入泛型机制之前的遗留代码进行兼容和互用而提供的。 作为一个快速回顾，Set<Object> 是一个参数化类型，表示一个可以包含任何类型对象的集合，Set<?> 是一个通配符类型，表示一个只能包含某些未知类型对象的集合，Set 是一个原始类型，它不在泛型类型系统之列。 前两个类型是安全的，最后一个不是。

　　为了快速参考，下表中总结了本条目（以及本章稍后介绍的一些）中介绍的术语：

术语	中文含义	举例	所在条目
Parameterized type	参数化类型	List<String>	条目 26
Actual type parameter	实际类型参数	String	条目 26
Generic type	泛型类型	List<E>	条目 26 和 条目 29
Formal type parameter	形式类型参数	E	条目 26
Unbounded wildcard type	无限制通配符类型	List<?>	条目 26
Raw type	原始类型	List	条目 26
Bounded type parameter	限制类型参数	<E extends Number>	条目 29
Recursive type bound	递归类型限制	<T extends Comparable<T>>	条目 30
Bounded wildcard type	限制通配符类型	List<? extends Number>	条目 31
Generic method	泛型方法	static <E> List<E> asList(E[] a)	条目 30
Type token	类型令牌	String.class	条目 33



//
27. 消除非检查警告
　　使用泛型编程时，会看到许多编译器警告：未经检查的强制转换警告，未经检查的方法调用警告，未经检查的参数化可变长度类型警告以及未经检查的转换警告。 你使用泛型获得的经验越多，获得的警告越少，但不要期望新编写的代码能够干净地编译。

　　许多未经检查的警告很容易消除。 例如，假设你不小心写了以下声明：

Set<Lark> exaltation = new HashSet();
复制ErrorOK!
　　编译器会提醒你你做错了什么：

Venery.java:4: warning: [unchecked] unchecked conversion
        Set<Lark> exaltation = new HashSet();
                               ^
  required: Set<Lark>
  found:    HashSet。
复制ErrorOK!
　　然后可以进行指示修正，让警告消失。 请注意，实际上并不需要指定类型参数，只是为了表明它与 Java 7 中引入的钻石运算符（「<>」）一同出现。然后编译器会推断出正确的实际类型参数（在本例中为 Lark）：

Set<Lark> exaltation = new HashSet<>();
复制ErrorOK!
　　但一些警告更难以消除。 本章充满了这种警告的例子。 当你收到需要进一步思考的警告时，坚持不懈！ 尽可能地消除每一个未经检查的警告。 如果你消除所有的警告，你可以放心，你的代码是类型安全的，这是一件非常好的事情。 这意味着在运行时你将不会得到一个 ClassCastException 异常，并且增加了你的程序将按照你的意图行事的信心。

　　如果你不能消除警告，但你可以证明引发警告的代码是类型安全的，那么（并且只能这样）用 @SuppressWarnings("unchecked") 注解来抑制警告。 如果你在没有首先证明代码是类型安全的情况下压制警告，那么你给自己一个错误的安全感。 代码可能会在不发出任何警告的情况下进行编译，但是它仍然可以在运行时抛出 ClassCastException 异常。 但是，如果你忽略了你认为是安全的未经检查的警告（而不是抑制它们），那么当一个新的警告出现时，你将不会注意到这是一个真正的问题。 新出现的警告就会淹没在所有的错误警告当中。

　　SuppressWarnings 注解可用于任何声明，从单个局部变量声明到整个类。 始终在尽可能最小的范围内使用 SuppressWarnings 注解。 通常这是一个变量声明或一个非常短的方法或构造方法。 切勿在整个类上使用 SuppressWarnings 注解。 这样做可能会掩盖重要的警告。

　　如果你发现自己在长度超过一行的方法或构造方法上使用 SuppressWarnings 注解，则可以将其移到局部变量声明上。 你可能需要声明一个新的局部变量，但这是值得的。 例如，考虑这个来自 ArrayList 的 toArray 方法：

public <T> T[] toArray(T[] a) {
    if (a.length < size)
       return (T[]) Arrays.copyOf(elements, size, a.getClass());
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
       a[size] = null;
    return a;
}

如果编译 ArrayList 类，则该方法会生成此警告：
ArrayList.java:305: warning: [unchecked] unchecked cast
       return (T[]) Arrays.copyOf(elements, size, a.getClass());
                                 ^
  required: T[]
  found:    Object[]
复制ErrorOK!
　　在返回语句中设置 SuppressWarnings 注解是非法的，因为它不是一个声明[JLS，9.7]。 你可能会试图把注释放在整个方法上，但是不要这要做。 相反，声明一个局部变量来保存返回值并标注它的声明，如下所示：

// Adding local variable to reduce scope of @SuppressWarnings
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // This cast is correct because the array we're creating
        // is of the same type as the one passed in, which is T[].
        @SuppressWarnings("unchecked") T[] result =
            (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
复制ErrorOK!
　　所产生的方法干净地编译，并最小化未经检查的警告被抑制的范围。

　　每当使用 @SuppressWarnings(“unchecked”) 注解时，请添加注释，说明为什么是安全的。 这将有助于他人理解代码，更重要的是，这将减少有人修改代码的可能性，从而使计算不安全。 如果你觉得很难写这样的注释，请继续思考。 毕竟，你最终可能会发现未经检查的操作是不安全的。

　　总之，未经检查的警告是重要的。 不要忽视他们。 每个未经检查的警告代表在运行时出现 ClassCastException 异常的可能性。 尽你所能消除这些警告。 如果无法消除未经检查的警告，并且可以证明引发该警告的代码是安全类型的，则可以在尽可能小的范围内使用 @SuppressWarnings(“unchecked”) 注解来禁止警告。 记录你决定在注释中抑制此警告的理由。
  
  28. 列表优于数组
　　数组在两个重要方面与泛型不同。 首先，数组是协变的（covariant）。 这个吓人的单词意味着如果 Sub 是 Super 的子类型，则数组类型 Sub[] 是数组类型 Super[] 的子类型。 相比之下，泛型是不变的（invariant）：对于任何两种不同的类型 Type1 和 Type2，List<Type1> 既不是 List<Type2> 的子类型也不是父类型。[JLS，4.10; Naftalin07, 2.5]。 你可能认为这意味着泛型是不足的，但可以说是数组缺陷。 这段代码是合法的：

// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
　　但这个不是：

// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
　　无论哪种方式，你不能把一个 String 类型放到一个 Long 类型容器中，但是用一个数组，你会发现在运行时产生了一个错误；对于列表，可以在编译时就能发现错误。 当然，你宁愿在编译时找出错误。

　　数组和泛型之间的第二个主要区别是数组被具体化了（reified）[JLS，4.7]。 这意味着数组在运行时知道并强制执行它们的元素类型。 如前所述，如果尝试将一个 String 放入 Long 数组中，得到一个 ArrayStoreException 异常。 相反，泛型通过擦除（erasure）来实现[JLS，4.6]。 这意味着它们只在编译时执行类型约束，并在运行时丢弃（或擦除）它们的元素类型信息。 擦除是允许泛型类型与不使用泛型的遗留代码自由互操作（详见第 26 条），从而确保在 Java 5 中平滑过渡到泛型。

　　由于这些基本差异，数组和泛型不能很好地在一起混合使用。 例如，创建泛型类型的数组，参数化类型的数组，以及类型参数的数组都是非法的。 因此，这些数组创建表达式都不合法：new List<E>[]，new List<String>[]，new E[]。 所有将在编译时导致泛型数组创建错误。

　　为什么创建一个泛型数组是非法的？ 因为它不是类型安全的。 如果这是合法的，编译器生成的强制转换程序在运行时可能会因为 ClassCastException 异常而失败。 这将违反泛型类型系统提供的基本保证。

　　为了具体说明，请考虑下面的代码片段：

// Why generic array creation is illegal - won't compile!
List<String>[] stringLists = new List<String>[1];  // (1)
List<Integer> intList = List.of(42);               // (2)
Object[] objects = stringLists;                    // (3)
objects[0] = intList;                              // (4)
String s = stringLists[0].get(0);                  // (5)
　　让我们假设第 1 行创建一个泛型数组是合法的。第 2 行创建并初始化包含单个元素的 List<Integer>。第 3 行将 List<String> 数组存储到 Object 数组变量中，这是合法的，因为数组是协变的。第 4 行将 List<Integer> 存储在 Object 数组的唯一元素中，这是因为泛型是通过擦除来实现的：List<Integer> 实例的运行时类型仅仅是 List，而 List<String>[] 实例是 List[]，所以这个赋值不会产生 ArrayStoreException 异常。现在我们遇到了麻烦。将一个 List<Integer> 实例存储到一个声明为仅保存 List<String> 实例的数组中。在第 5 行中，我们从这个数组的唯一列表中检索唯一的元素。编译器自动将检索到的元素转换为 String，但它是一个 Integer，所以我们在运行时得到一个 ClassCastException 异常。为了防止发生这种情况，第 1 行（创建一个泛型数组）必须产生一个编译时错误。

　　类型 E，List<E> 和 List<String> 等在技术上被称为不可具体化的类型（nonreifiable types）[JLS，4.7]。 直观地说，不可具体化的类型是其运行时表示包含的信息少于其编译时表示的类型。 由于擦除，可唯一确定的参数化类型是无限定通配符类型，如 List<?> 和 Map<?, ?>（详见第 26 条）。 尽管很少有用，创建无限定通配符类型的数组是合法的。

　　禁止泛型数组的创建可能会很恼人的。 这意味着，例如，泛型集合通常不可能返回其元素类型的数组（但是参见条目 33 中的部分解决方案）。 这也意味着，当使用可变参数方法（详见第 53 条）和泛型时，会产生令人困惑的警告。 这是因为每次调用可变参数方法时，都会创建一个数组来保存可变参数。 如果此数组的元素类型不可确定，则会收到警告。 SafeVarargs 注解可以用来解决这个问题（详见第 32 条）。

　　当你在强制转换为数组类型时，得到泛型数组创建错误，或是未经检查的强制转换警告时，最佳解决方案通常是使用集合类型 List<E> 而不是数组类型 E[]。 这样可能会牺牲一些简洁性或性能，但作为交换，你会获得更好的类型安全性和互操作性。

　　例如，假设你想用带有集合的构造方法来编写一个 Chooser 类，并且有个方法返回随机选择的集合的一个元素。 根据传递给构造方法的集合，可以使用该类的实例对象作为游戏骰子，魔力 8 号球或蒙特卡罗模拟的数据源。 这是一个没有泛型的简单实现：

// Chooser - a class badly in need of generics!
public class Chooser {
    private final Object[] choiceArray;


    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }


    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
　　要使用这个类，每次调用方法时，都必须将 choose 方法的返回值从 Object 转换为所需的类型，如果类型错误，则转换在运行时失败。 我们先根据条目 29 的建议，试图修改 Chooser 类，使其成为泛型的。

// A first cut at making Chooser generic - won't compile
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();
    }

    // choose method unchanged
}
　　如果你尝试编译这个类，会得到这个错误信息：

Chooser.java:9: error: incompatible types: Object[] cannot be
converted to T[]
        choiceArray = choices.toArray();
                                     ^
  where T is a type-variable:
    T extends Object declared in class Chooser
　　没什么大不了的，将 Object 数组转换为 T 数组：

choiceArray = (T[]) choices.toArray();
　　这没有了错误，而是得到一个警告：

Chooser.java:9: warning: [unchecked] unchecked cast
        choiceArray = (T[]) choices.toArray();
                                           ^
  required: T[], found: Object[]
  where T is a type-variable:
T extends Object declared in class Chooser
　　编译器告诉你在运行时不能保证强制转换的安全性，因为程序不会知道 T 代表什么类型——记住，元素类型信息在运行时会被泛型删除。 该程序可以正常工作吗？ 是的，但编译器不能证明这一点。 你可以证明这一点，在注释中提出证据，并用注解来抑制警告，但最好是消除警告的原因（详见第 27 条）。

　　要消除未经检查的强制转换警告，请使用列表而不是数组。 下面是另一个版本的 Chooser 类，编译时没有错误或警告：

// List-based Chooser - typesafe
public class Chooser<T> {
    private final List<T> choiceList;


    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }


    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
　　这个版本有些冗长，也许运行比较慢，但是值得一提的是，在运行时不会得到 ClassCastException 异常。

　　总之，数组和泛型具有非常不同的类型规则。 数组是协变和具体化的; 泛型是不变的，类型擦除的。 因此，数组提供运行时类型的安全性，但不提供编译时类型的安全性，对于泛型则是相反。 一般来说，数组和泛型不能很好地混合工作。 如果你发现把它们混合在一起，得到编译时错误或者警告，你的第一个冲动应该是用列表来替换数组。
  
  29. 优先考虑泛型
　　参数化声明并使用 JDK 提供的泛型类型和方法通常不会太困难。 但编写自己的泛型类型有点困难，但值得努力学习。

　　考虑条目 7 中的简单堆栈实现：

// Object-based collection - a prime candidate for generics
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
　　这个类应该已经被参数化了，但是由于事实并非如此，我们可以对它进行泛型化。 换句话说，我们可以参数化它，而不会损害原始非参数化版本的客户端。 就目前而言，客户端必须强制转换从堆栈中弹出的对象，而这些强制转换可能会在运行时失败。 泛型化类的第一步是在其声明中添加一个或多个类型参数。 在这种情况下，有一个类型参数，表示堆栈的元素类型，这个类型参数的常规名称是 E（详见第 68 条）。

　　下一步是用相应的类型参数替换所有使用的 Object 类型，然后尝试编译生成的程序：

// Initial attempt to generify Stack - won't compile!
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
    ... // no changes in isEmpty or ensureCapacity
}
　　你通常会得到至少一个错误或警告，这个类也不例外。 幸运的是，这个类只产生一个错误：

Stack.java:8: generic array creation
        elements = new E[DEFAULT_INITIAL_CAPACITY];
                   ^
　　如条目 28 所述，你不能创建一个不可具体化类型的数组，例如类型 E。每当编写一个由数组支持的泛型时，就会出现此问题。 有两种合理的方法来解决它。 第一种解决方案直接规避了对泛型数组创建的禁用：创建一个 Object 数组并将其转换为泛型数组类型。 现在没有了错误，编译器会发出警告。 这种用法是合法的，但不是（一般）类型安全的：

Stack.java:8: warning: [unchecked] unchecked cast
found: Object[], required: E[]
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
                       ^
　　编译器可能无法证明你的程序是类型安全的，但你可以。 你必须说服自己，不加限制的类型强制转换不会损害程序的类型安全。 有问题的数组（元素）保存在一个私有属性中，永远不会返回给客户端或传递给任何其他方法。 保存在数组中的唯一元素是那些传递给 push 方法的元素，它们是 E 类型的，所以未经检查的强制转换不会造成任何伤害。

　　一旦证明未经检查的强制转换是安全的，请尽可能缩小范围（条目 27）。 在这种情况下，构造方法只包含未经检查的数组创建，所以在整个构造方法中抑制警告是合适的。 通过添加一个注解来执行此操作，Stack 可以干净地编译，并且可以在没有显式强制转换或担心 ClassCastException 异常的情况下使用它：

// The elements array will contain only E instances from push(E).
// This is sufficient to ensure type safety, but the runtime
// type of the array won't be E[]; it will always be Object[]!
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
　　消除 Stack 中的泛型数组创建错误的第二种方法是将属性元素的类型从 E[] 更改为 Object[]。 如果这样做，会得到一个不同的错误：

Stack.java:19: incompatible types
found: Object, required: E
        E result = elements[--size];
                           ^
　　可以通过将从数组中检索到的元素转换为 E 来将此错误更改为警告：

Stack.java:19: warning: [unchecked] unchecked cast
found: Object, required: E
        E result = (E) elements[--size];
                               ^
　　因为 E 是不可具体化的类型，编译器无法在运行时检查强制转换。 再一次，你可以很容易地向自己证明，不加限制的转换是安全的，所以可以适当地抑制警告。 根据条目 27 的建议，我们只在包含未经检查的强制转换的分配上抑制警告，而不是在整个 pop 方法上：

// Appropriate suppression of unchecked warning
public E pop() {
    if (size == 0)
        throw new EmptyStackException();

    // push requires elements to be of type E, so cast is correct
    @SuppressWarnings("unchecked") E result =
        (E) elements[--size];

    elements[size] = null; // Eliminate obsolete reference
    return result;
}
　　两种消除泛型数组创建的技术都有其追随者。 第一个更可读：数组被声明为 E[] 类型，清楚地表明它只包含 E 实例。 它也更简洁：在一个典型的泛型类中，你从代码中的许多点读取数组；第一种技术只需要一次转换（创建数组的地方），而第二种技术每次读取数组元素都需要单独转换。 因此，第一种技术是优选的并且在实践中更常用。 但是，它确实会造成堆污染（heap pollution）（详见第 32 条）：数组的运行时类型与编译时类型不匹配（除非 E 碰巧是 Object）。 这使得一些程序员非常不安，他们选择了第二种技术，尽管在这种情况下堆的污染是无害的。

　　下面的程序演示了泛型 Stack 类的使用。 该程序以相反的顺序打印其命令行参数，并将其转换为大写。 对从堆栈弹出的元素调用 String 的 toUpperCase 方法不需要显式强制转换，而自动生成的强制转换将保证成功：

// Little program to exercise our generic Stack
public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for (String arg : args)
        stack.push(arg);
    while (!stack.isEmpty())
        System.out.println(stack.pop().toUpperCase());
}
　　上面的例子似乎与条目 28 相矛盾，条目 28 中鼓励使用列表优先于数组。 在泛型类型中使用列表并不总是可行或可取的。 Java 本身生来并不支持列表，所以一些泛型类型（如 ArrayList）必须在数组上实现。 其他的泛型类型，比如 HashMap，是为了提高性能而实现的。

　　绝大多数泛型类型就像我们的 Stack 示例一样，它们的类型参数没有限制：可以创建一个 Stack<Object>, Stack<int[]>，Stack<List<String>> 或者其他任何对象的 Stack 引用类型。 请注意，不能创建基本类型的堆栈：尝试创建 Stack<int> 或 Stack<double> 将导致编译时错误。 这是 Java 泛型类型系统的一个基本限制。 可以使用基本类型的包装类（详见第 61 条）来解决这个限制。

　　有一些泛型类型限制了它们类型参数的允许值。 例如，考虑 java.util.concurrent.DelayQueue，它的声明如下所示：

class DelayQueue<E extends Delayed> implements BlockingQueue<E>
　　类型参数列表（<E extends Delayed>）要求实际的类型参数 E 是 java.util.concurrent.Delayed 的子类型。 这使得 DelayQueue 实现及其客户端可以利用 DelayQueue 元素上的 Delayed 方法，而不需要显式的转换或 ClassCastException 异常的风险。 类型参数 E 被称为限定类型参数。 请注意，子类型关系被定义为每个类型都是自己的子类型[JLS，4.10]，因此创建 DelayQueue<Delayed> 是合法的。

　　总之，泛型类型比需要在客户端代码中强制转换的类型更安全，更易于使用。 当你设计新的类型时，确保它们可以在没有这种强制转换的情况下使用。 这通常意味着使类型泛型化。 如果你有任何现有的类型，应该是泛型的但实际上却不是，那么把它们泛型化。 这使这些类型的新用户的使用更容易，而不会破坏现有的客户端（条目 26）。
  
  31. 使用限定通配符来增加 API 的灵活性
　　如条目 28 所述，参数化类型是不变的。换句话说，对于任何两个不同类型的 Type1 和 Type2，List<Type1> 既不是 List<Type2> 的子类型也不是其父类型。尽管 List<String> 不是 List<Object> 的子类型是违反直觉的，但它确实是有道理的。 可以将任何对象放入 List<Object> 中，但是只能将字符串放入 List<String> 中。 由于 List<String> 不能做 List<Object> 所能做的所有事情，所以它不是一个子类型（条目 10 中的里氏替代原则）。

　　相对于提供的不可变的类型，有时你需要比此更多的灵活性。 考虑条目 29 中的 Stack 类。下面是它的公共 API：

public class Stack<E> {

    public Stack();

    public void push(E e);

    public E pop();

    public boolean isEmpty();

}
　　假设我们想要添加一个方法来获取一系列元素，并将它们全部推送到栈上。 以下是第一种尝试：

// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
　　这种方法可以干净地编译，但不完全令人满意。 如果可遍历的 src 元素类型与栈的元素类型完全匹配，那么它工作正常。 但是，假设有一个 Stack<Number>，并调用 push(intVal)，其中 intVal 的类型是 Integer。 这是因为 Integer 是 Number 的子类型。 从逻辑上看，这似乎也应该起作用：

Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
　　但是，如果你尝试了，会得到这个错误消息，因为参数化类型是不变的：

StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
        numberStack.pushAll(integers);
                            ^
　　幸运的是，有对应的解决方法。 该语言提供了一种特殊的参数化类型来调用一个限定通配符类型来处理这种情况。 pushAll 的输入参数的类型不应该是「E 的 Iterable 接口」，而应该是「E 的某个子类型的 Iterable 接口」，并且有一个通配符类型，这意味着：Iterable<? extends E>。 （关键字 extends 的使用有点误导：回忆条目 29 中，子类型被定义为每个类型都是它自己的子类型，即使它本身没有继承。）让我们修改 pushAll 来使用这个类型：

// Wildcard type for a parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
　　有了这个改变，Stack 类不仅可以干净地编译，而且客户端代码也不会用原始的 pushAll 声明编译。 因为 Stack 和它的客户端干净地编译，你知道一切都是类型安全的。

　　现在假设你想写一个 popAll 方法，与 pushAll 方法相对应。 popAll 方法从栈中弹出每个元素并将元素添加到给定的集合中。 以下是第一次尝试编写 popAll 方法的过程：

// popAll method without wildcard type - deficient!
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
　　同样，如果目标集合的元素类型与栈的元素类型完全匹配，则干净编译并且工作正常。 但是，这又不完全令人满意。 假设你有一个 Stac<Number> 和 Object 类型的变量。 如果从栈中弹出一个元素并将其存储在该变量中，它将编译并运行而不会出错。 你不应该也这样做吗？

Stack<Number> numberStack = new Stack<Number>();

Collection<Object> objects = ... ;

numberStack.popAll(objects);
　　如果尝试将此客户端代码与之前显示的 popAll 版本进行编译，则会得到与我们的第一版 pushAll 非常类似的错误：Collection<Object> 不是 Collection<Number> 的子类型。 通配符类型再一次提供了一条出路。 popAll 的输入参数的类型不应该是「E 的集合」，而应该是「E 的某个父类型的集合」（其中父类型被定义为 E 是它自己的父类型[JLS，4.10]）。 再次，有一个通配符类型，正是这个意思：Collection<? super E>。 让我们修改 popAll 来使用它：

// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
　　通过这个改动，Stack 类和客户端代码都可以干净地编译。

　　这个结论很清楚。 为了获得最大的灵活性，对代表生产者或消费者的输入参数使用通配符类型。 如果一个输入参数既是一个生产者又是一个消费者，那么通配符类型对你没有好处：你需要一个精确的类型匹配，这就是没有任何通配符的情况。

　　这里有一个助记符来帮助你记住使用哪种通配符类型： PECS 代表： producer-extends，consumer-super。

　　换句话说，如果一个参数化类型代表一个 T 生产者，使用 <? extends T>；如果它代表 T 消费者，则使用 <? super T>。 在我们的 Stack 示例中，pushAll 方法的 src 参数生成栈使用的 E 实例，因此 src 的合适类型为 Iterable<? extends E>；popAll 方法的 dst 参数消费 Stack 中的 E 实例，因此 dst 的合适类型是 Collection <? super E>。 PECS 助记符抓住了使用通配符类型的基本原则。 Naftalin 和 Wadler 称之为获取和放置原则（Get and Put Principle）[Naftalin07,2.4]。

　　记住这个助记符之后，让我们来看看本章中以前项目的一些方法和构造方法声明。 条目 28 中的 Chooser 类构造方法有这样的声明：

public Chooser(Collection<T> choices)
　　这个构造方法只使用集合选择来生产类型 T 的值（并将它们存储起来以备后用），所以它的声明应该使用一个 extends T 的通配符类型。下面是得到的构造方法声明：

// Wildcard type for parameter that serves as an T producer

public Chooser(Collection<? extends T> choices)
　　这种改变在实践中会有什么不同吗？ 是的，会有不同。 假你有一个 List<Integer>，并且想把它传递给 Chooser<Number> 的构造方法。 这不会与原始声明一起编译，但是它只会将限定通配符类型添加到声明中。

　　现在看看条目 30 中的 union 方法。下是声明：

public static <E> Set<E> union(Set<E> s1, Set<E> s2)
　　两个参数 s1 和 s2 都是 E 的生产者，所以 PECS 助记符告诉我们该声明应该如下：

public static <E> Set<E> union(Set<? extends E> s1,  Set<? extends E> s2)
　　请注意，返回类型仍然是 Set<E>。 不要使用限定通配符类型作为返回类型。除了会为用户提供额外的灵活性，还强制他们在客户端代码中使用通配符类型。 通过修改后的声明，此代码将清晰地编译：

Set<Integer>  integers =  Set.of(1, 3, 5);

Set<Double>   doubles  =  Set.of(2.0, 4.0, 6.0);

Set<Number>   numbers  =  union(integers, doubles);
　　如果使用得当，类的用户几乎不会看到通配符类型。 他们使方法接受他们应该接受的参数，拒绝他们应该拒绝的参数。 如果一个类的用户必须考虑通配符类型，那么它的 API 可能有问题。

　　在 Java 8 之前，类型推断规则不够聪明，无法处理先前的代码片段，这要求编译器使用上下文指定的返回类型（或目标类型）来推断 E 的类型。union 方法调用的目标类型如前所示是 Set<Number>。 如果尝试在早期版本的 Java 中编译片段（以及适合的 Set.of 工厂替代版本），将会看到如此长的错综复杂的错误消息：

Union.java:14: error: incompatible types
        Set<Number> numbers = union(integers, doubles);
                                   ^
  required: Set<Number>
  found:    Set<INT#1>
  where INT#1,INT#2 are intersection types:
    INT#1 extends Number,Comparable<? extends INT#2>
    INT#2 extends Number,Comparable<?>
　　幸运的是有办法来处理这种错误。 如果编译器不能推断出正确的类型，你可以随时告诉它使用什么类型的显式类型参数[JLS，15.12]。 甚至在 Java 8 中引入目标类型之前，这不是你必须经常做的事情，这很好，因为显式类型参数不是很漂亮。 通过添加显式类型参数，如下所示，代码片段在 Java 8 之前的版本中进行了干净编译：

// Explicit type parameter - required prior to Java 8
Set<Number> numbers = Union.<Number>union(integers, doubles);
　　接下来让我们把注意力转向条目 30 中的 max 方法。这里是原始声明：

public static <T extends Comparable<T>> T max(List<T> list)
　　为了从原来到修改后的声明，我们两次应用了 PECS。首先直接的应用是参数列表。 它生成 T 实例，所以将类型从 List<T> 更改为 List<? extends T>。 棘手的应用是类型参数 T。这是我们第一次看到通配符应用于类型参数。 最初，T 被指定为继承 Comparable<T>，但 Comparable 的 T 消费 T 实例（并生成指示顺序关系的整数）。 因此，参数化类型 Comparable<T> 被替换为限定通配符类型 Comparable<? super T>。 Comparable 实例总是消费者，所以通常应该使用 Comparable<? super T> 优于 Comparable<T>。 Comparator 也是如此。因此，通常应该使用 Comparator<? super T> 优于 Comparator<T>。

　　修改后的 max 声明可能是本书中最复杂的方法声明。 增加的复杂性是否真的起作用了吗？ 同样，它的确如此。 这是一个列表的简单例子，它被原始声明排除，但在被修改后的版本里是允许的：

List<ScheduledFuture<?>> scheduledFutures = ... ;
　　无法将原始方法声明应用于此列表的原因是 ScheduledFuture 不实现 Comparable<ScheduledFuture>。 相反，它是 Delayed 的子接口，它继承了 Comparable<Delayed>。 换句话说，一个 ScheduledFuture 实例不仅仅和其他的 ScheduledFuture 实例相比较： 它可以与任何 Delayed 实例比较，并且足以导致原始的声明拒绝它。 更普遍地说，通配符要求来支持没有直接实现 Comparable（或 Comparator）的类型，但继承了一个类型。

　　还有一个关于通配符相关的话题。 类型参数和通配符之间具有双重性，许多方法可以用一个或另一个声明。 例如，下面是两个可能的声明，用于交换列表中两个索引项目的静态方法。 第一个使用无限制类型参数（条目 30），第二个使用无限制通配符：

// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
　　这两个声明中的哪一个更可取，为什么？ 在公共 API 中，第二个更好，因为它更简单。 你传入一个列表（任何列表），该方法交换索引的元素。 没有类型参数需要担心。 通常， 如果类型参数在方法声明中只出现一次，请将其替换为通配符。 如果它是一个无限制的类型参数，请将其替换为无限制的通配符; 如果它是一个限定类型参数，则用限定通配符替换它。

　　第二个 swap 方法声明有一个问题。 这个简单的实现不会编译：

public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
　　试图编译它会产生这个不太有用的错误信息：

Swap.java:5: error: incompatible types: Object cannot be
converted to CAP#1
        list.set(i, list.set(j, list.get(i)));
                                        ^
  where CAP#1 is a fresh type-variable:
    CAP#1 extends Object from capture of ?
　　看起来我们不能把一个元素放回到我们刚刚拿出来的列表中。 问题是列表的类型是 List<?>，并且不能将除 null 外的任何值放入 List<?> 中。 幸运的是，有一种方法可以在不使用不安全的转换或原始类型的情况下实现此方法。 这个想法是写一个私有辅助方法来捕捉通配符类型。 辅助方法必须是泛型方法才能捕获类型。 以下是它的定义：

public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
　　swapHelper 方法知道该列表是一个 List<E>。 因此，它知道从这个列表中获得的任何值都是 E 类型，并且可以安全地将任何类型的 E 值放入列表中。 这个稍微复杂的 swap 的实现可以干净地编译。 它允许我们导出基于通配符的漂亮声明，同时利用内部更复杂的泛型方法。 swap 方法的客户端不需要面对更复杂的 swapHelper 声明，但他们从中受益。 辅助方法具有我们认为对公共方法来说过于复杂的签名。

　　总之，在你的 API 中使用通配符类型，虽然棘手，但使得 API 更加灵活。 如果编写一个将被广泛使用的类库，正确使用通配符类型应该被认为是强制性的。 记住基本规则： producer-extends, consumer-super（PECS）。 还要记住，所有 Comparable 和 Comparator 都是消费者。
  
  32. 合理地结合泛型和可变参数
　　在 Java 5 中，可变参数方法（详见第 53 条）和泛型都被添加到平台中，所以你可能希望它们能够正常交互; 可悲的是，他们并没有。 可变参数的目的是允许客户端将一个可变数量的参数传递给一个方法，但这是一个脆弱的抽象（leaky abstraction）：当你调用一个可变参数方法时，会创建一个数组来保存可变参数；那个应该是实现细节的数组是可见的。 因此，当可变参数具有泛型或参数化类型时，会导致编译器警告混淆。

　　回顾条目 28，非具体化（non-reifiable）的类型是其运行时表示比其编译时表示具有更少信息的类型，并且几乎所有泛型和参数化类型都是不可具体化的。 如果某个方法声明其可变参数为非具体化的类型，则编译器将在该声明上生成警告。 如果在推断类型不可确定的可变参数参数上调用该方法，那么编译器也会在调用中生成警告。 警告看起来像这样：

warning: [unchecked] Possible heap pollution from
    parameterized vararg type List<String>
复制ErrorOK!复制ErrorOK!
　　当参数化类型的变量引用不属于该类型的对象时会发生堆污染（Heap pollution）[JLS，4.12.2]。 它会导致编译器的自动生成的强制转换失败，违反了泛型类型系统的基本保证。

　　例如，请考虑以下方法，该方法是第 127 页上的代码片段的一个不太明显的变体：

// Mixing generics and varargs can violate type safety!
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList;             // Heap pollution
    String s = stringLists[0].get(0); // ClassCastException
}
复制ErrorOK!复制ErrorOK!
　　此方法没有可见的强制转换，但在调用一个或多个参数时抛出 ClassCastException 异常。 它的最后一行有一个由编译器生成的隐形转换。 这种转换失败，表明类型安全性已经被破坏，并且将值保存在泛型可变参数数组参数中是不安全的。

　　这个例子引发了一个有趣的问题：为什么声明一个带有泛型可变参数的方法是合法的，当明确创建一个泛型数组是非法的时候呢？ 换句话说，为什么前面显示的方法只生成一个警告，而 127 页上的代码片段会生成一个错误？ 答案是，具有泛型或参数化类型的可变参数参数的方法在实践中可能非常有用，因此语言设计人员选择忍受这种不一致。 事实上，Java 类库导出了几个这样的方法，包括 Arrays.asList(T... a)，Collections.addAll(Collection<? super T> c, T... elements)，EnumSet.of(E first, E... rest)。 与前面显示的危险方法不同，这些类库方法是类型安全的。

　　在 Java 7 中，@SafeVarargs 注解已添加到平台，以允许具有泛型可变参数的方法的作者自动禁止客户端警告。 实质上，@SafeVarargs 注解构成了作者对类型安全的方法的承诺。 为了交换这个承诺，编译器同意不要警告用户调用可能不安全的方法。

　　除非它实际上是安全的，否则注意不要使用 @SafeVarargs 注解标注一个方法。 那么需要做些什么来确保这一点呢？ 回想一下，调用方法时会创建一个泛型数组，以容纳可变参数。 如果方法没有在数组中存储任何东西（它会覆盖参数）并且不允许对数组的引用进行转义（这会使不受信任的代码访问数组），那么它是安全的。 换句话说，如果可变参数数组仅用于从调用者向方法传递可变数量的参数——毕竟这是可变参数的目的——那么该方法是安全的。

　　值得注意的是，你可以违反类型安全性，即使不会在可变参数数组中存储任何内容。 考虑下面的泛型可变参数方法，它返回一个包含参数的数组。 乍一看，它可能看起来像一个方便的小工具：

// UNSAFE - Exposes a reference to its generic parameter array!
static <T> T[] toArray(T... args) {
    return args;
}
复制ErrorOK!复制ErrorOK!
　　这个方法只是返回它的可变参数数组。 该方法可能看起来并不危险，但它是！ 该数组的类型由传递给方法的参数的编译时类型决定，编译器可能没有足够的信息来做出正确的判断。 由于此方法返回其可变参数数组，它可以将堆污染传播到调用栈上。

　　为了具体说明，请考虑下面的泛型方法，它接受三个类型 T 的参数，并返回一个包含两个参数的数组，随机选择：

static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
      case 0: return toArray(a, b);
      case 1: return toArray(a, c);
      case 2: return toArray(b, c);
    }
    throw new AssertionError(); // Can't get here
}
复制ErrorOK!复制ErrorOK!
　　这个方法本身不是危险的，除了调用具有泛型可变参数的 toArray 方法之外，不会产生警告。

　　编译此方法时，编译器会生成代码以创建一个将两个 T 实例传递给 toArray 的可变参数数组。 这段代码分配了一个 Object[] 类型的数组，它是保证保存这些实例的最具体的类型，而不管在调用位置传递给 pickTwo 的对象是什么类型。 toArray 方法只是简单地将这个数组返回给 pickTwo，然后 pickTwo 将它返回给调用者，所以 pickTwo 总是返回一个 Object[] 类型的数组。

public static void main(String[] args) {
    String[] attributes = pickTwo("Good", "Fast", "Cheap");
}
复制ErrorOK!复制ErrorOK!
　　这种方法没有任何问题，因此它编译时不会产生任何警告。 但是当运行它时，抛出一个 ClassCastException 异常，尽管不包含可见的转换。 你没有看到的是，编译器已经生成了一个隐藏的强制转换为由 pickTwo 返回的值的 String[] 类型，以便它可以存储在属性中。 转换失败，因为 Object[] 不是 String[] 的子类型。 这种故障相当令人不安，因为它从实际导致堆污染（toArray）的方法中移除了两个级别，并且在实际参数存储在其中之后，可变参数数组未被修改。

　　这个例子是为了让人们认识到给另一个方法访问一个泛型的可变参数数组是不安全的，除了两个例外：将数组传递给另一个可变参数方法是安全的，这个方法是用 @SafeVarargs 正确标注的， 将数组传递给一个非可变参数的方法是安全的，该方法仅计算数组内容的一些方法。

　　这里是安全使用泛型可变参数的典型示例。 此方法将任意数量的列表作为参数，并按顺序返回包含所有输入列表元素的单个列表。 由于该方法使用 @SafeVarargs 进行标注，因此在声明或其调用站位置上不会生成任何警告：

// Safe method with a generic varargs parameter
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
复制ErrorOK!复制ErrorOK!
　　决定何时使用 @SafeVarargs 注解的规则很简单：在每种方法上使用 @SafeVarargs，并使用泛型或参数化类型的可变参数，这样用户就不会因不必要的和令人困惑的编译器警告而担忧。 这意味着你不应该写危险或者 toArray 等不安全的可变参数方法。 每次编译器警告你可能会受到来自你控制的方法中泛型可变参数的堆污染时，请检查该方法是否安全。 提醒一下，在下列情况下，泛型可变参数方法是安全的：

它不会在可变参数数组中存储任何东西
它不会使数组（或克隆）对不可信代码可见。 如果违反这些禁令中的任何一项，请修复。
　　请注意，SafeVarargs 注解只对不能被重写的方法是合法的，因为不可能保证每个可能的重写方法都是安全的。 在 Java 8 中，注解仅在静态方法和 final 实例方法上合法; 在 Java 9 中，它在私有实例方法中也变为合法。

　　使用 SafeVarargs 注解的替代方法是采用条目 28 的建议，并用 List 参数替换可变参数（这是一个变相的数组）。 下面是应用于我们的 flatten 方法时，这种方法的样子。 请注意，只有参数声明被更改了：

// List as a typesafe alternative to a generic varargs parameter
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
复制ErrorOK!复制ErrorOK!
　　然后可以将此方法与静态工厂方法 List.of 结合使用，以允许可变数量的参数。 请注意，这种方法依赖于 List.of 声明使用 @SafeVarargs 注解：

audience = flatten(List.of(friends, romans, countrymen));
复制ErrorOK!复制ErrorOK!
　　这种方法的优点是编译器可以证明这种方法是类型安全的。 不必使用 @SafeVarargs 注解来证明其安全性，也不用担心在确定安全性时可能会犯错。 主要缺点是客户端代码有点冗长，运行可能会慢一些。

　　这个技巧也可以用在不可能写一个安全的可变参数方法的情况下，就像第 147 页的 toArray 方法那样。它的列表模拟是 List.of 方法，所以我们甚至不必编写它；Java 类库作者已经为我们完成了这项工作。 pickTwo 方法然后变成这样：

static <T> List<T> pickTwo(T a, T b, T c) {
    switch(rnd.nextInt(3)) {
      case 0: return List.of(a, b);
      case 1: return List.of(a, c);
      case 2: return List.of(b, c);
    }
    throw new AssertionError();
}
复制ErrorOK!复制ErrorOK!
　　main 方变成这样：

public static void main(String[] args) {
    List<String> attributes = pickTwo("Good", "Fast", "Cheap");
}
复制ErrorOK!复制ErrorOK!
　　生成的代码是类型安全的，因为它只使用泛型，不是数组。

　　总而言之，可变参数和泛型不能很好地交互，因为可变参数机制是在数组上面构建的脆弱的抽象，并且数组具有与泛型不同的类型规则。 虽然泛型可变参数不是类型安全的，但它们是合法的。 如果选择使用泛型（或参数化）可变参数编写方法，请首先确保该方法是类型安全的，然后使用 @SafeVarargs 注解对其进行标注，以免造成使用不愉快。
  
  33. 优先考虑类型安全的异构容器
　　泛型的常见用法包括集合，如 Set<E> 和 Map<K，V> 和单个元素容器，如 ThreadLocal<T> 和 AtomicReference<T>。 在所有这些用途中，它都是参数化的容器。 这限制了每个容器只能有固定数量的类型参数。 通常这正是你想要的。 一个 Set 有单一的类型参数，表示它的元素类型; 一个 Map 有两个，代表它的键和值的类型；等等。

　　然而有时候，你需要更多的灵活性。 例如，数据库一行记录可以具有任意多列，并且能够以类型安全的方式访问它们是很好的。 幸运的是，有一个简单的方法可以达到这个效果。 这个想法是参数化键（key）而不是容器。 然后将参数化的键提交给容器以插入或检索值。 泛型类型系统用于保证值的类型与其键一致。

　　作为这种方法的一个简单示例，请考虑一个 Favorites 类，它允许其客户端保存和检索任意多种类型的 favorite 实例。 该类型的 Class 对象将扮演参数化键的一部分。其原因是这 Class 类是泛型的。 类的类型从字面上来说不是简单的 Class，而是 Class<T>。 例如，String.class 的类型为 Class<String>，Integer.class 的类型为 Class<Integer>。 当在方法中传递字面类传递编译时和运行时类型信息时，它被称为类型令牌（type token）[Bracha04]。

　　Favorites 类的 API 很简单。 它看起来就像一个简单 Map 类，除了该键是参数化的以外。 客户端在设置和获取 favorites 实例时呈现一个 Class 对象。 如下是 API：

// Typesafe heterogeneous container pattern - API
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> T getFavorite(Class<T> type);
}
复制ErrorOK!
　　下面是一个演示 Favorites 类，保存，检索和打印喜欢的 String，Integer 和 Class 实例：

// Typesafe heterogeneous container pattern - client
public static void main(String[] args) {
    Favorites f = new Favorites();
    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorites.class);

    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);
    System.out.printf("%s %x %s%n", favoriteString,
        favoriteInteger, favoriteClass.getName());
}
复制ErrorOK!
　　正如你所期望的，这个程序打印 Java cafebabe Favorites。 请注意，顺便说一下，Java 的 printf 方法与 C 语言的不同之处在于，你应该在 C 中使用 \n 的地方改用 %n。%n 用于生成适用于特定平台的行分隔符，在大多数平台上面的值为 \n，但并不是所有平台的分隔符都为 \n。

　　Favorites 实例是类型安全的：当你请求一个字符串时它永远不会返回一个整数。 它也是异构的：与普通 Map 不同，所有的键都是不同的类型。 因此，我们将 Favorites 称为类型安全异构容器（typesafe heterogeneous container）。

　　Favorites 的实现非常小巧。 这是完整的代码：

// Typesafe heterogeneous container pattern - implementation
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public<T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public<T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
复制ErrorOK!
　　这里有一些微妙的事情发生。 每个 Favorites 实例都由一个名为 favorites 私有的 Map<Class<?>, Object> 来支持。 你可能认为无法将任何内容放入此 Map 中，因为这是无限定的通配符类型，但事实恰恰相反。 需要注意的是通配符类型是嵌套的：它不是通配符类型的 Map 类型，而是键的类型。 这意味着每个键都可以有不同的参数化类型：一个可以是 Class<String>，下一个 Class<Integer> 等等。 这就是异构的由来。

　　接下来要注意的是，favorites 的 Map 的值类型只是 Object。 换句话说，Map 不保证键和值之间的类型关系，即每个值都是由其键表示的类型。 事实上，Java 的类型系统并不足以表达这一点。 但是我们知道这是真的，并在检索一个 favorite 时利用了这点。

　　putFavorite 实现很简单：只需将给定的 Class 对象映射到给定的 favorites 的实例即可。 如上所述，这丢弃了键和值之间的“类型联系（type linkage）”；无法知道这个值是不是键的一个实例。 但没关系，因为 getFavorites 方法可以并且确实重新建立这种关联。

　　getFavorite 的实现比 putFavorite 更复杂。 首先，它从 favorites Map 中获取与给定 Class 对象相对应的值。 这是返回的正确对象引用，但它具有错误的编译时类型：它是 Object（favorites map 的值类型），我们需要返回类型 T。因此，getFavorite 实现动态地将对象引用转换为 Class 对象表示的类型，使用 Class 的 cast 方法。

　　cast 方法是 Java 的 cast 操作符的动态模拟。它只是检查它的参数是否由 Class 对象表示的类型的实例。如果是，它返回参数；否则会抛出 ClassCastException 异常。我们知道，假设客户端代码能够干净地编译，getFavorite 中的强制转换不会抛出 ClassCastException 异常。 也就是说，favorites map 中的值始终与其键的类型相匹配。

　　那么这个 cast 方法为我们做了什么，因为它只是返回它的参数？ cast 的签名充分利用了 Class 类是泛型的事实。 它的返回类型是 Class 对象的类型参数：

public class Class<T> {
    T cast(Object obj);
}
复制ErrorOK!
　　这正是 getFavorite 方法所需要的。 这正是确保 Favorites 类型安全，而不用求助一个未经检查的强制转换的 T 类型。

　　Favorites 类有两个限制值得注意。 首先，恶意客户可以通过使用原始形式的 Class 对象，轻松破坏 Favorites 实例的类型安全。 但生成的客户端代码在编译时会生成未经检查的警告。 这与正常的集合实现（如 HashSet 和 HashMap）没有什么不同。 通过使用原始类型 HashSet（条目 26），可以轻松地将字符串放入 HashSet<Integer> 中。 也就是说，如果你愿意为此付出一点代价，就可以拥有运行时类型安全性。 确保 Favorites 永远不违反类型不变的方法是，使 putFavorite 方法检查该实例是否由 type 表示类型的实例，并且我们已经知道如何执行此操作。只需使用动态转换：

// Achieving runtime type safety with a dynamic cast
public<T> void putFavorite(Class<T> type, T instance) {
    favorites.put(type, type.cast(instance));
}
复制ErrorOK!
　　java.util.Collections 中有一些集合包装类，可以发挥相同的诀窍。 它们被称为 checkedSet，checkedList，checkedMap 等等。 他们的静态工厂除了一个集合（或 Map）之外还有一个 Class 对象（或两个）。 静态工厂是泛型方法，确保 Class 对象和集合的编译时类型匹配。 包装类为它们包装的集合添加了具体化。 例如，如果有人试图将 Coin 放入你的 Collection<Stamp> 中，则包装类在运行时会抛出 ClassCastException。 这些包装类对于追踪在混合了泛型和原始类型的应用程序中添加不正确类型的元素到集合的客户端代码很有用。

　　Favorites 类的第二个限制是它不能用于不可具体化的（non-reifiable）类型（详见第 28 条）。 换句话说，你可以保存你最喜欢的 String 或 String[]，但不能保存 List<String>。 如果你尝试保存你最喜欢的 List<String>，程序将不能编译。 原因是无法获取 List<String> 的 Class 对象。 List<String>.class 是语法错误，也是一件好事。 List<String> 和 List<Integer> 共享一个 Class 对象，即 List.class。 如果“字面类型（type literals）”List<String> .class 和 List<Integer>.class 合法并返回相同的对象引用，那么它会对 Favorites 对象的内部造成严重破坏。 对于这种限制，没有完全令人满意的解决方法。

　　Favorites 使用的类型令牌 type tokens) 是无限制的：getFavorite 和 putFavorite 接受任何 Class 对象。 有时你可能需要限制可传递给方法的类型。 这可以通过一个有限定的类型令牌来实现，该令牌只是一个类型令牌，它使用限定的类型参数（详见第 30 条）或限定的通配符（详见第 31 条）来放置可以表示的类型的边界。

　　注解 API（详见第 39 条）广泛使用限定类型的令牌。 例如，以下是在运行时读取注解的方法。 此方法来自 AnnotatedElement 接口，该接口由表示类，方法，属性和其他程序元素的反射类型实现：

public <T extends Annotation>
    T getAnnotation(Class<T> annotationType);
复制ErrorOK!
　　参数 annotationType 是表示注解类型的限定类型令牌。 该方法返回该类型的元素的注解（如果它有一个）；如果没有，则返回 null。 本质上，注解元素是一个类型安全的异构容器，其键是注解类型。

　　假设有一个 Class<?> 类型的对象，并且想要将它传递给需要限定类型令牌（如 getAnnotation）的方法。 可以将对象转换为 Class<? extends Annotation>，但是这个转换没有被检查，所以它会产生一个编译时警告（详见第 52 条）。 幸运的是，Class 类提供了一种安全（动态）执行这种类型转换的实例方法。 该方法被称为 asSubclass，并且它转换所调用的 Class 对象来表示由其参数表示的类的子类。 如果转换成功，该方法返回它的参数；如果失败，则抛出 ClassCastException 异常。

　　以下是如何使用 asSubclass 方法在编译时读取类型未知的注解。 此方法编译时没有错误或警告：

// Use of asSubclass to safely cast to a bounded type token
static Annotation getAnnotation(AnnotatedElement element,
                                String annotationTypeName) {
    Class<?> annotationType = null; // Unbounded type token
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }
    return element.getAnnotation(
        annotationType.asSubclass(Annotation.class));
}
复制ErrorOK!
　　总之，泛型 API 的通常用法（以集合 API 为例）限制了每个容器的固定数量的类型参数。 你可以通过将类型参数放在键上而不是容器上来解决此限制。 可以使用 Class 对象作为此类型安全异构容器的键。 以这种方式使用的 Class 对象称为类型令牌。 也可以使用自定义键类型。 例如，可以有一个表示数据库行（容器）的 DatabaseRow 类型和一个泛型类型 Column<T> 作为其键。
