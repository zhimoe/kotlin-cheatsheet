Comprehensive Kotlin Cheatsheet
===============================

Contents
--------

 [`Main`](#Main)
 [`Nullable`](#Nullable)

Main
---

```kotlin
val name = "Kotlin" // val for value (readonly after initial), var for variable

fun main() {
    println("Hello $name") // string templates $name or ${name.toUpperCase}
}

fun maxOf(a: Int, b: Int) = if (a > b) a else b // if is expression

val items = listOf("apple", "banana", "kiwifruit")
for (item in items) { // or: for (item in items.indices){}
    println(item)
}

// while and when
fun describe(obj: Any): String =
    when (obj) {
        1 -> "One"
        "Hello" -> "Greeting"
        is Long -> "Long"
        !is String -> "Not a string"
        else -> "Unknown"
    }

// Range
for (i in 1..10 step 2) {
    println("$i")
}

// multi-line string
"""
multi-line
strings
""" //trimIndent/trimMargin
```

Nullable
---

```kotlin
//Nullable type names have ? at the end.

// null check
val b: String? = "Kotlin"
if (b != null && b.length > 0) {
    print("String of length ${b.length}")
} else {
    print("Empty string")
}

// safe call
val b: String? = null
println(b?.length) // no NPE here cuz the b?.length return null
// what if we want call print only when b is not null? use ?.let{}
b?.let { print(b.length) }
// safe call is useful in chains
bob?.department?.head?.name
// either `person` or `person.department` is null, the function is not called
person?.department?.head = managersPool.getManager()

// elvis operator
val l = b?.length ?: -1

// !!
val l = b!!.length // throw NPE when b is null

// safe casts
val maybe: Int? = a as? Int // null is cast failed

// collection of nullable type
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()


// from java to kotlin null check
// val order = findOrder()

// Direct conversion
// if (order != null){
//     processCustomer(order.customer)
// }
// in kotlin:
findOrder()?.customer?.let(::processCustomer)


```

Delegates
---

```kotlin
// calculates value before first usage
val i by lazy { print("init "); 10 }
print(i) // Prints: init 10
print(i) // Prints: 10

// observable/vetoable - calls function every time
//        value changes. In vetoable function also decides
//        if new value should be set.
var name by observable("Unset") { p, old, new ->
    println("${p.name} changed $old -> $new")
}
name = "Marcin"
// Prints: name changed Unset -> Marcin

val map = mapOf("a" to 10)
val a by map
print(a) // Prints: 10
```

Function
---

```kotlin
// it : implicit name of a single parameter

fun sum(a: Int, b: Int): Int { // return type, Unit can be omitted
    return a + b
}

// expression as body
fun sum(a: Int, b: Int) = a + b // return only can be used in {}
// lambda
{ a, b -> a + b }

//infix operator function: called using infix notation
operator fun Counter.unaryMinus() = Counter(-this.value)

// 如果最后一个参数是function 且该实参arg是lambda时，可以将arg放在()外面。该语法糖称为trailing lambda
val product = items.fold(1) { acc, e -> acc * e }
// infix operator function 和 trailing lambda 是kotlin DSL的基础语法糖，使用广泛, 注意下面{}后面可以 chains call
strings.filter { it.length == 5 }.sortedBy { it }.map { it.uppercase() }

// `Nothing` type can be used as a return type for a function that always throws an exception

// extension methods


// inline function
// Kotlin里面会有函数类型（(Int)->Unit）实际在JVM就是FunctionN等类的匿名对象。
// 如果在一个函数中有函数类型参数，如果这个函数被反复调用，就会导致创建大量FunctionN的匿名对象实例。
// 为了避免优化这种函数，可以在 fun 前面增加inline，将函数类型的参数的实参在调用处展开。
// 但是有一种情况，函数如果将这个函数类型参数作为一个对象返回：
inlne fun process(name: String, action: (Int, Int) -> Unit): () -> Unit {
    print("hello $name")
    action() // call action arg
    return action // return action arg function
}
// 上面的返回值是一个函数类型的实参action，这里必须当作一个对象处理，不能展开了。
// 原则：当你函数参数有函数类型参数时，建议加上inline，如果编译报错，再查看时那个函数类型参数需要加上noinline
```

Class
---

```kotlin
this::class // is KClass
this::class.java // Class of Java

// data class: auto have "hashCode" "equals" "toString" "copy" methods, and can be destructured by fields
data class Person(val name: String, val age: Int)

// enum has ordinal and name property
enum class WeekDay {
    Monday, TuesDay
}

// object
object Singleton {
}


//<!--this.class:: java -->
//<!--internal keyword -->

// auto cast
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` is automatically cast to `String` in this branch
    return obj.length
}
// is and !is for runtime type check
if (obj !is String) { // same as !(obj is String)
    print("Not a String")
} else {
    print(obj.length)
}
// unsafe cast
val x: String = y as String
val x: String? = y as? String // return null if cast failed

// generics type check
// no intList is List<Int> or list is T
if (something is List<*>) {
    something.forEach { println(it) } // The items are typed as `Any?`
}

```

Collection
---

```kotlin
// collection literals
// arrayOf listOf mutableListOf setOf mutableSetOf mapOf mutableMapOf sequenceOf
1 to "A" // Pair<Int, String>
List(4) { it * 2 } // List<Int>
generateSequence(4) { it + 2 } // Sequence<Int>

// type of collections item
data class Customer(val name: String, val city: City, val orders: List)
data class Order(val products: List, val isDelivered: Boolean)
data class Product(val name: String, val price: Double)
data class City(val name: String)

customers.sortedByDescending { it.orders.size }
// other functions: map filter all any count find take average flatmap fold reduce onEach(return List)/forEach(return Unit) 
// min minBy max maxBy first 

// from list to map
customers.associateBy(Customer::name) // name -> customer
customers.associateWith(Customer::city) // customer -> city
customers.associate { it.name to it.city } // name -> city

customers.groupBy { it.city } //

customers.filter {
    val (delivered, undelivered) = it.orders.partition { it.isDelivered }
    undelivered.size > delivered.size
}.toSet()

//flatMap accept fun that return list
val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
println(numberSets.flatten())

customers.maxByOrNull { it.orders.size }

customer.orders
    .flatMap(Order::products)
    .maxByOrNull(Product::price)

customer.orders.flatMap { it.products }.sumOf { it.price }

customers.map(Customer::getOrderedProducts).reduce { orderedByAll, customer ->
    orderedByAll.intersect(customer)
}

orders.flatMap(Order::products).toSet()

return customer.orders
    .filter(Order::isDelivered)
    .flatMap(Order::products)
    .maxByOrNull(Product::price)

return customers
    .flatMap(Customer::getOrderedProducts)
    .count { it == product }

return customer.orders
    .asSequence()
    .filter(Order::isDelivered)
    .flatMap(Order::products)
    .maxByOrNull(Product::price)


// map destructuring in lambdas
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```

From Java
---

```kotlin
// get logger
private val log = LoggerFactory.getLogger(this::class.java)

// no setter, use = 

// mybatis kotlin config
/* usage:
1 add kotlin-jpa plugin
2 add plugin and pluginOption no-arg:annotation=moe.zhi.NoArg
3 add @NoArg annotation to dao model data class
*/
annotation class NoArg

// use more scope function: with/apply/let/run/also
// this vs it: as a lambda receiver (this-apply/run/with) or as a lambda argument (it-let/also). 
// this can omit, it can rename

// diff in apply|run|with:
// apply return this, think as a config context;
// run return anything, it is an extension function, so it can chain, but also can be shadowed;
// with return anything, it is not extension function.

// diff in let/also
// both are extension function, but let return anything, also return this.


```

Coroutine
---

```kotlin


```
