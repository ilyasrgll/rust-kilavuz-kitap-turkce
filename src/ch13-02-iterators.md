## Yineleyicilerle Bir Dizi Öğeyi İşlemek

Yineleyici deseni (iterator pattern), bir dizi öğe üzerinde sırayla bazı görevleri yerine getirmenizi sağlar. Bir yineleyici, her bir öğe üzerinde yineleme mantığından ve dizinin ne zaman bittiğini belirlemekten sorumludur. Yineleyicileri kullandığınızda, bu mantığı kendiniz yeniden uygulamanıza gerek kalmaz.

Rust'ta yineleyiciler tembeldir, yani yineleyiciyi kullanmak için onu tüketen metotları çağırmadığınız sürece hiçbir etkisi olmazlar. Örneğin, Listing 13-10'daki kod, Vec<T> üzerinde tanımlı iter metodunu çağırarak v1 vektöründeki öğeler üzerinde bir yineleyici oluşturur. Bu kodun kendisi tek başına yararlı bir iş yapmaz.

<Listing number="13-10" file-name="src/main.rs" caption="Creating an iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

Yineleyici v1_iter değişkeninde saklanır. Bir yineleyici oluşturduktan sonra, onu çeşitli şekillerde kullanabiliriz. Listing 3-5'te, bir dizi üzerinde for döngüsü kullanarak öğelerinden her birinde bazı kodları yürütmüştük. Arka planda, bu örtülü olarak bir yineleyici oluşturdu ve sonra onu tüketti, ancak bunun tam olarak nasıl çalıştığını şimdiye kadar detaylandırmamıştık.

Listing 13-11'deki örnekte, yineleyicinin oluşturulmasını, for döngüsünde yineleyicinin kullanılmasından ayırıyoruz. v1_iter içindeki yineleyici kullanılarak for döngüsü çağrıldığında, yineleyicideki her bir öğe döngünün bir tekrarında kullanılır ve her bir değeri yazdırır.

<Listing number="13-11" file-name="src/main.rs" caption="Using an iterator in a `for` loop">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Standart kütüphaneleri tarafından yineleyici sağlamayan dillerde, muhtemelen bu aynı işlevselliği, bir değişkeni 0 dizininde başlatarak, o değişkeni vektöre dizin olarak kullanarak bir değer elde etmek için ve değişkenin değerini döngüde vektördeki toplam öğe sayısına ulaşana kadar artırarak yazardınız.

Yineleyiciler, tüm bu mantığı sizin için halleder, böylece hata yapma potansiyeli olan tekrarlayan kodu azaltır. Yineleyiciler, aynı mantığı yalnızca vektörler gibi dizinleyebileceğiniz veri yapılarıyla değil, birçok farklı dizi türüyle kullanmak için size daha fazla esneklik sunar. Şimdi yineleyicilerin bunu nasıl yaptığını inceleyelim.

###  `Iterator` özelliği ve `next` Metodu

Tüm yineleyiciler, standart kütüphanede tanımlı `Iterator` adlı bir özellik (trait) uygular. Özelliğin tanımı şöyle görünür:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Notice that this definition uses some new syntax: `type Item` and `Self::Item`,
which are defining an _associated type_ with this trait. We’ll talk about
associated types in depth in Chapter 20. For now, all you need to know is that
this code says implementing the `Iterator` trait requires that you also define
an `Item` type, and this `Item` type is used in the return type of the `next`
method. In other words, the `Item` type will be the type returned from the
iterator.

The `Iterator` trait only requires implementors to define one method: the
`next` method, which returns one item of the iterator at a time, wrapped in
`Some`, and, when iteration is over, returns `None`.

We can call the `next` method on iterators directly; Listing 13-12 demonstrates
what values are returned from repeated calls to `next` on the iterator created
from the vector.

<Listing number="13-12" file-name="src/lib.rs" caption="Calling the `next` method on an iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Note that we needed to make `v1_iter` mutable: calling the `next` method on an
iterator changes internal state that the iterator uses to keep track of where
it is in the sequence. In other words, this code _consumes_, or uses up, the
iterator. Each call to `next` eats up an item from the iterator. We didn’t need
to make `v1_iter` mutable when we used a `for` loop because the loop took
ownership of `v1_iter` and made it mutable behind the scenes.

Also note that the values we get from the calls to `next` are immutable
references to the values in the vector. The `iter` method produces an iterator
over immutable references. If we want to create an iterator that takes
ownership of `v1` and returns owned values, we can call `into_iter` instead of
`iter`. Similarly, if we want to iterate over mutable references, we can call
`iter_mut` instead of `iter`.

### Methods That Consume the Iterator

The `Iterator` trait has a number of different methods with default
implementations provided by the standard library; you can find out about these
methods by looking in the standard library API documentation for the `Iterator`
trait. Some of these methods call the `next` method in their definition, which
is why you’re required to implement the `next` method when implementing the
`Iterator` trait.

Methods that call `next` are called _consuming adapters_, because calling them
uses up the iterator. One example is the `sum` method, which takes ownership of
the iterator and iterates through the items by repeatedly calling `next`, thus
consuming the iterator. As it iterates through, it adds each item to a running
total and returns the total when iteration is complete. Listing 13-13 has a
test illustrating a use of the `sum` method.

<Listing number="13-13" file-name="src/lib.rs" caption="Calling the `sum` method to get the total of all items in the iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

We aren’t allowed to use `v1_iter` after the call to `sum` because `sum` takes
ownership of the iterator we call it on.

### Methods That Produce Other Iterators

_Iterator adapters_ are methods defined on the `Iterator` trait that don’t
consume the iterator. Instead, they produce different iterators by changing
some aspect of the original iterator.

Listing 13-14 shows an example of calling the iterator adapter method `map`,
which takes a closure to call on each item as the items are iterated through.
The `map` method returns a new iterator that produces the modified items. The
closure here creates a new iterator in which each item from the vector will be
incremented by 1.

<Listing number="13-14" file-name="src/main.rs" caption="Calling the iterator adapter `map` to create a new iterator">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

However, this code produces a warning:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

The code in Listing 13-14 doesn’t do anything; the closure we’ve specified
never gets called. The warning reminds us why: iterator adapters are lazy, and
we need to consume the iterator here.

To fix this warning and consume the iterator, we’ll use the `collect` method,
which we used with `env::args` in Listing 12-1. This method consumes the
iterator and collects the resultant values into a collection data type.

In Listing 13-15, we collect the results of iterating over the iterator that’s
returned from the call to `map` into a vector. This vector will end up
containing each item from the original vector, incremented by 1.

<Listing number="13-15" file-name="src/main.rs" caption="Calling the `map` method to create a new iterator, and then calling the `collect` method to consume the new iterator and create a vector">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Because `map` takes a closure, we can specify any operation we want to perform
on each item. This is a great example of how closures let you customize some
behavior while reusing the iteration behavior that the `Iterator` trait
provides.

You can chain multiple calls to iterator adapters to perform complex actions in
a readable way. But because all iterators are lazy, you have to call one of the
consuming adapter methods to get results from calls to iterator adapters.

### Using Closures That Capture Their Environment

Many iterator adapters take closures as arguments, and commonly the closures
we’ll specify as arguments to iterator adapters will be closures that capture
their environment.

For this example, we’ll use the `filter` method that takes a closure. The
closure gets an item from the iterator and returns a `bool`. If the closure
returns `true`, the value will be included in the iteration produced by
`filter`. If the closure returns `false`, the value won’t be included.

In Listing 13-16, we use `filter` with a closure that captures the `shoe_size`
variable from its environment to iterate over a collection of `Shoe` struct
instances. It will return only shoes that are the specified size.

<Listing number="13-16" file-name="src/lib.rs" caption="Using the `filter` method with a closure that captures `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

The `shoes_in_size` function takes ownership of a vector of shoes and a shoe
size as parameters. It returns a vector containing only shoes of the specified
size.

In the body of `shoes_in_size`, we call `into_iter` to create an iterator
that takes ownership of the vector. Then we call `filter` to adapt that
iterator into a new iterator that only contains elements for which the closure
returns `true`.

The closure captures the `shoe_size` parameter from the environment and
compares the value with each shoe’s size, keeping only shoes of the size
specified. Finally, calling `collect` gathers the values returned by the
adapted iterator into a vector that’s returned by the function.

The test shows that when we call `shoes_in_size`, we get back only shoes
that have the same size as the value we specified.
