#接口 Map<K,V>

Map接口是一个将key映射到value的对象。map不能包含重复的key；每个key可以映射到至多一个value。

本接口替代了Dictioary类，Dictionary类是一个抽象类而不是一个接口。

Map接口提供了三个集合视图，这三个视图可以让map的类容展示为：所有key的set集合，所有value的collection集合或者key-value对的set集合。map的顺序定义为map的集合视图的迭代器返回元素的顺序。一些map的实现，比如TreeMap类，它对元素顺序做出了具体的保证；而其他的实现类，比如HashMap，没有相应保证。

注意：如果易变的对象作为map的key，则应该十分小心。当一个对象作为map的key时，如果以影响equals比较的方式改变了对象的值，map的行为是没有规定的。这个禁令的一个特例是不允许map将自身作为key，但是允许map包含自身作为key的值，十分小心的建议：equals、hashCode这两个方法不再能在这样的map上有很好的定义。

所有常规用途的map实现应该提供两个标准的构造方法：一个创建一个空集合的void（无参）构造方法；一个单个Map类型参数的构造方法，这个构造方法创建一个有和传入的map一样的键值对映射的新map。实际上，后一个构造方法允许用户复制map，生成一个和期望的类实例相等的map。无法强制要求这个建议（因为接口不能包含构造方法），但是JDK中所有的常规用途的Map实现类都遵循了这个建议。

这个接口包含的那些“有害的”方法，即：那些改变了它们操作的map的方法，如果map不支持它们的操作，则规定抛出一个UnsupportedOperationException 。如果是以下这种情况——调用方法不会有对map影响，那么该方法可以但不是必须抛出一个UnsupportedOperationException 。例如，在一个不可修改的map上执行putAll(Map)方法，而要叠加的映射为空时(译者注：传入的map为空)，可以但不是必须抛出异常。


一些map实现类对它们能包含的key和value有限制。例如：一些实现类禁止key或value为null，而一些实现类对key的类型有限制。试图插入不符合规定的key或value会抛出检查型异常，主要是NullPointerException 或ClassCastException。试图查询一个不符合规定的key或value的存在性会抛出异常，或者简单地返回false；一些实现类会表现前一个行为，而一些表现后一个行为。更通常的，
###泛型参数：

K - map持有的key的类型

V - key对应的值的类型

###所有已知子接口：

Bindings, ConcurrentMap<K,V>, ConcurrentNavigableMap<K,V>, LogicalMessageContext, MessageContext, NavigableMap<K,V>, SOAPMessageContext, SortedMap<K,V>

###所有已知实现类：
AbstractMap, Attributes, AuthProvider, ConcurrentHashMap, ConcurrentSkipListMap, EnumMap, HashMap, Hashtable, IdentityHashMap, LinkedHashMap, PrinterStateReasons, Properties, Provider, RenderingHints, SimpleBindings, TabularDataSupport, TreeMap, UIDefaults, WeakHashMap




