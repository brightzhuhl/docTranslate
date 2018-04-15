# 接口 Map<K,V>


 **Map** 接口是一个将 **key** 映射到 **value** 的对象。 **map** 不能包含重复的 **key** ；每个 **key** 可以映射到至多一个 **value** 。

本接口替代了 **Dictioary** 类， **Dictionary** 类是一个抽象类而不是一个接口。

 **Map** 接口提供了三个集合视图，这三个视图可以让 **map** 的类容展示为：所有 **key** 的 **set** 集合，所有 **value** 的 **collection** 集合或者 **key** - **value** 对的 **set** 集合。 **map** 的顺序定义为 **map** 的集合视图的迭代器返回元素的顺序。一些 **map** 的实现，比如 **TreeMap** 类，它对元素顺序做出了具体的保证；而其他的实现类，比如 **HashMap** ，没有相应保证。

注意：如果易变的对象作为 **map** 的 **key** ，则应该十分小心。当一个对象作为 **map** 的 **key** 时，如果以影响 **equals** 比较的方式改变了对象的值， **map** 的行为是没有规定的。这个禁令的一个特例是不允许 **map** 将自身作为 **key** ，但是允许 **map** 包含自身作为 **key** 的值，十分小心的建议： **equals** 、 **hashCode** 这两个方法不再能在这样的 **map** 上有很好的定义。

所有常规用途的 **map** 实现应该提供两个标准的构造方法：一个创建一个空集合的 **void** （无参）构造方法；一个单个 **Map** 类型参数的构造方法，这个构造方法创建一个有和传入的 **map** 一样的键值对映射的新 **map** 。实际上，后一个构造方法允许用户复制 **map** ，生成一个和期望的类实例相等的 **map** 。无法强制要求这个建议（因为接口不能包含构造方法），但是 **JDK** 中所有的常规用途的 **Map** 实现类都遵循了这个建议。

这个接口包含的那些“有害的”方法，即：那些改变了它们操作的 **map** 的方法，如果 **map** 不支持它们的操作，则规定抛出一个 **UnsupportedOperationException**  。如果是以下这种情况——调用方法不会有对 **map** 影响，那么该方法可以但不是必须抛出一个 **UnsupportedOperationException**  。例如，在一个不可修改的 **map** 上执行 **putAll** ( **Map** )方法，而要叠加的映射为空时(译者注：传入的 **map** 为空)，可以但不是必须抛出异常。


一些 **map** 实现类对它们能包含的 **key** 和 **value** 有限制。例如：一些实现类禁止 **key** 或 **value** 为 **null** ，而一些实现类对 **key** 的类型有限制。试图插入不符合规定的 **key** 或 **value** 会抛出检查型异常，主要是 **NullPointerException**  或 **ClassCastException** 。试图查询一个不符合规定的 **key** 或 **value** 的存在性会抛出异常，或者简单地返回 **false** ；一些实现类会表现前一个行为，而一些表现后一个行为。更通常的，尝试在不合规定的 **key** 或者 **value** 上进行操作而操作完成不会导致不正确的元素的插入，这可能抛出一个异常，也可能成功进行，两种情况可供实现类选择。这样的异常在这个接口规范中被标记为可选( **optional** )异常。

集合框架接口中的很多方法都是用 **equals** 方法定义的。比如， **containsKey** ( **Object**   **key** )方法的规范中提到：“返回 **true** 当且仅当 **map** 中包含一个 **key**   **k** ，( **key** == **null**  ?  **k** == **null**  :  **key** . **equqls** ( **k** ))”， This specification should not be construed to imply that invoking Map。非空 **key** 参数的 **containsKey** 方法调用将在所有 **key**   **k** 上执行 **key** . **equals** ( **k** )。实现类可以自由地实现优化，从而避免调用 **equals** 方法，比如，先比较两个 **key** 的 **hashcode** ( **Object** . **hashCode** ()规范保证两个有不同 **hashcode** 的对象一定不相等)。更普遍的，各个集合框架接口的实现可以自由地利用 **Object** 底层方法的特定行为，无论实现者认为这是恰当的。

在 **map** 中直接或间接包含自身的自引用实例中进行递归遍历操作时可能会失败并抛出异常。这包括 **clone** (), **equals** (), **hashCode** ()和 **toString** ()方法。实现类可以有选择地处理自引用，然而目前大多数实现类都没有这样做。

本接口是 **Java** 集合框架成员之一。

### 泛型参数：

	K - map持有的key的类型

	V - key对应的值的类型

### 所有已知子接口：

	Bindings, ConcurrentMap<K,V>, ConcurrentNavigableMap<K,V>, LogicalMessageContext, MessageContext, NavigableMap<K,V>, SOAPMessageContext, SortedMap<K,V>

### 所有已知实现类：

	AbstractMap, Attributes, AuthProvider, ConcurrentHashMap, ConcurrentSkipListMap, EnumMap, HashMap, Hashtable, IdentityHashMap, LinkedHashMap, PrinterStateReasons, Properties, Provider, RenderingHints, SimpleBindings, TabularDataSupport, TreeMap, UIDefaults, WeakHashMap




