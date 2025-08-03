<!-- Old heading. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## Kapanışlar: Ortamlarını Yakalayabilen Anonim Fonksiyonlar

Rust’taki kapanışlar, bir değişkene kaydedebileceğiniz veya başka fonksiyonlara argüman olarak verebileceğiniz anonim fonksiyonlardır. Kapanışı bir yerde tanımlayıp, daha sonra farklı bir bağlamda değerlendirmek üzere başka bir yerde çağırabilirsiniz. Fonksiyonlardan farklı olarak, kapanışlar tanımlandıkları kapsamdan değerleri yakalayabilirler. Bu bölümde kapanışların bu özelliklerinin kod tekrarını nasıl azaltabileceğini ve davranışları nasıl özelleştirebileceğini göstereceğiz.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### Kapanışlarla Ortamın Yakalanması

Öncelikle kapanışları, tanımlandıkları ortamdan değerleri yakalayarak daha sonra kullanmak için nasıl kullanabileceğimizi inceleyeceğiz. Senaryo şöyle: Tişört şirketimiz belirli aralıklarla, posta listemizdeki bir kişiye promosyon amacıyla özel üretim, sınırlı sayıda bir tişört hediye ediyor. Posta listesinde yer alan kişiler, isteğe bağlı olarak profillerine favori renklerini ekleyebiliyor. Eğer seçilen kişi favori rengini belirtmişse, o renkte tişört alıyor. Eğer bir renk belirtilmemişse, şirketin o an en çok stokta bulunan rengi gönderiliyor.


Bunu uygulamanın birçok yolu vardır. Bu örnekte, `ShirtColor` adında bir `enum` kullanacağız; bu `enum` yalnızca `Red` ve `Blue` varyantlarına sahiptir (basitlik olması açısından renk sayısını sınırlıyoruz). Şirketin envanterini, içinde `shirts` adında bir alan bulunan `Inventory` adlı bir `struct` ile temsil edeceğiz. Bu alan, şu anda stokta bulunan tişört renklerini içeren bir `Vec<ShirtColor>` olacaktır. `Inventory` yapısı üzerinde tanımlı olan `giveaway` metodu, ücretsiz tişört kazanacak kişinin isteğe bağlı tişört rengi tercih bilgilerini alır ve hangi rengin verileceğini döndürür. Bu kurulum Liste 13-1’de gösterilmektedir.

<Listing number="13-1" file-name="src/main.rs" caption="Tişört şirketinin promosyon dağıtım durumu">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`main` fonksiyonu içerisinde tanımlanan `store`, bu özel üretim promosyon için iki mavi ve bir kırmızı tişörte sahiptir. `giveaway` metodunu, biri kırmızı tişört tercih eden ve diğeri herhangi bir tercih belirtmeyen iki kullanıcı için çağırıyoruz.

Bu kod elbette başka şekillerde de uygulanabilirdi, fakat burada odağımız kapanışlar (closures) olduğu için, halihazırda öğrendiğiniz kavramlara sadık kalıyoruz; yalnızca `giveaway` metodunun gövdesi farklıdır çünkü burada bir kapanış kullanılmıştır. `giveaway` metodunda, kullanıcı tercihini `Option<ShirtColor>` türünde bir parametre olarak alıyoruz ve `user_preference` üzerinde `unwrap_or_else` metodunu çağırıyoruz. [`Option<T>` üzerindeki `unwrap_or_else` metodu][unwrap-or-else]<!-- ignore --> standart kütüphane tarafından tanımlanmıştır. Bu metot, bağımsız değişken olarak argümansız bir kapanış alır ve bu kapanış `T` türünde (bu örnekte `ShirtColor`) bir değer döndürür. Eğer `Option<T>` değeri `Some` ise, `unwrap_or_else` içerisindeki değeri döndürür. Eğer değer `None` ise, kapanışı çalıştırır ve onun döndürdüğü değeri verir.

`unwrap_or_else` metoduna argüman olarak `|| self.most_stocked()` kapanış ifadesini veriyoruz. Bu kapanış, herhangi bir parametre almaz (eğer alsaydı, parametreler iki dikey çizgi arasına yazılırdı). Kapanış gövdesi `self.most_stocked()` fonksiyonunu çağırır. Kapanışı burada tanımlıyoruz, ama `unwrap_or_else` metodunun implementasyonu kapanışı yalnızca gerekli olduğunda çalıştıracaktır.


Bu kodu çalıştırdığınızda aşağıdaki çıktıyı verir:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Burada ilginç olan şeylerden biri, mevcut `Inventory` örneği üzerinde `self.most_stocked()` metodunu çağıran bir kapanışı `unwrap_or_else` metoduna aktarmış olmamızdır. Standart kütüphane, tanımladığımız `Inventory` veya `ShirtColor` türleri hakkında hiçbir şey bilmek zorunda değildir; bu senaryoda kullanmak istediğimiz mantığı da bilmez. Kapanış, `self` adlı `Inventory` örneğine değiştirilemez (immutable) bir referans yakalar ve bu referansla birlikte tanımladığımız kodu `unwrap_or_else` metoduna aktarır. Öte yandan, normal fonksiyonlar bu şekilde ortamlarını yakalayamazlar.

### Kapanışlarda Tür Çıkarımı ve Açık Tür Belirtimi

Fonksiyonlar ile kapanışlar arasında başka farklar da vardır. Kapanışlar genellikle parametrelerin veya döndürdükleri değerin türünü belirtmenizi gerektirmez; oysa `fn` anahtar sözcüğüyle tanımlanan fonksiyonlarda bu tür açıklamaları yapmak zorunludur. Fonksiyonlarda tür belirtimi zorunludur çünkü bu türler kullanıcılarınıza açık bir arayüz olarak sunulur. Bu arayüzü net biçimde tanımlamak, fonksiyonun ne tür değerler alıp ne tür değerler döndüreceği konusunda herkesin hemfikir olmasını sağlar. Kapanışlar ise böyle dışa açık arayüzlerde kullanılmazlar; genellikle değişkenlerde saklanırlar ve kütüphanemizi kullanan dış kodlara ad verilerek sunulmazlar.


Kapanışlar genellikle kısa olur ve geniş çaplı durumlar yerine dar kapsamlı özel senaryolarda kullanılır. Bu sınırlı bağlamlar içerisinde, derleyici (compiler) parametrelerin ve dönüş değerinin türlerini tıpkı değişkenlerde olduğu gibi kendiliğinden çıkarabilir. (Nadir durumlarda kapanışlar için açık tür tanımlamaları da gerekebilir.)

Tıpkı değişkenlerde olduğu gibi, daha açık ve net olması adına tür açıklamaları da ekleyebiliriz; fakat bu, gerekli olmayan fazladan sözdizimsel ayrıntı anlamına gelir. Bir kapanış için tür açıklamaları eklemek, Liste 13-2’de gösterildiği gibi görünür. Bu örnekte, kapanışı doğrudan argüman olarak geçirmek yerine bir değişkene atayarak tanımlıyoruz (Liste 13-1’deki gibi doğrudan geçmiyoruz).

<Listing number="13-2" file-name="src/main.rs" caption="Kapanışta parametre ve dönüş türlerinin isteğe bağlı olarak belirtilmesi">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Tür açıklamaları eklendiğinde, kapanışların sözdizimi fonksiyonlarınkine daha çok benzemeye başlar. Burada, bir parametreye 1 ekleyen bir fonksiyon ve aynı davranışı gösteren bir kapanış tanımlıyoruz. İlgili bölümleri hizalamak için bazı boşluklar ekledik. Bu örnek, kapanış sözdiziminin fonksiyon sözdizimine ne kadar benzediğini; farkların yalnızca boru çizgileri (`| |`) ve bazı sözdizim öğelerinin opsiyonel olmasında olduğunu gösteriyor:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

İlk satır, klasik bir fonksiyon tanımıdır. İkinci satırda tam tür belirtilmiş bir kapanış tanımı yer alır. Üçüncü satırda tür açıklamaları kaldırılmıştır. Dördüncü satırda ise süslü parantezler (`{}`) kaldırılmıştır — bu, gövde yalnızca bir ifade içerdiğinde opsiyoneldir. Tüm bu tanımlar geçerlidir ve çağrıldıklarında aynı davranışı gösterirler. `add_one_v3` ve `add_one_v4` satırlarında, türlerin kullanım bağlamından çıkarılabilmesi için kapanışların gerçekten değerlendirilmesi gerekir. Bu durum, Rust’un `let v = Vec::new();` ifadesinde de geçerlidir — derleyici `Vec`’in türünü çıkarabilmek için ya tür açıklaması ister ya da içine eklenecek öğelere bakar.

Kapanış tanımlarında, derleyici her parametre ve dönüş değeri için tek bir kesin tür çıkarımı yapar. Örneğin, Liste 13-3’te yalnızca aldığı değeri aynen geri döndüren kısa bir kapanış tanımı yer alır. Bu kapanış, yalnızca örnek amacıyla gösterilmiştir ve gerçek kullanım açısından çok anlamlı değildir. Dikkat ederseniz, tanımda hiçbir tür açıklaması yapılmamıştır. Bu nedenle kapanışı ilk kez `String` türüyle çağırdığımızda bu tür kapanışa yerleşir. Daha sonra aynı kapanışı bir tamsayı (integer) ile çağırmaya çalışırsak, hata alırız.


<Listing number="13-3" file-name="src/main.rs" caption="Türleri çıkarımla belirlenmiş bir kapanışı iki farklı türle çağırma girişimi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Derleyici bize şu hatayı verir:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

`example_closure` adlı kapanışı ilk kez bir `String` değeriyle çağırdığımızda, derleyici `x` değişkeninin türünü ve kapanışın dönüş türünü `String` olarak çıkarımlar. Bu türler artık `example_closure` kapanışına sabitlenmiş olur ve aynı kapanışı daha sonra farklı bir türle (örneğin bir tamsayıyla) çağırmaya çalıştığımızda tür uyumsuzluğu hatası alırız.

### Referansları Yakalama veya Sahipliği Taşıma

Kapanışlar, tanımlandıkları ortamdan değerleri üç farklı yolla yakalayabilirler. Bu yakalama yöntemleri, bir fonksiyonun parametre alırken kullandığı üç temel yönteme karşılık gelir: değiştirilemez (immutable) ödünç alma, değiştirilebilir (mutable) ödünç alma ve sahiplenme (ownership).

Liste 13-4’te, yalnızca değeri yazdırmak için değiştirilemez referansa ihtiyaç duyan bir kapanış tanımlıyoruz. Bu kapanış, `list` adlı vektöre değiştirilemez bir referans yakalar.

<Listing number="13-4" file-name="src/main.rs" caption="Değiştirilemez referans yakalayan bir kapanış tanımlama ve çağırma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Bu örnek aynı zamanda bir değişkenin bir kapanışı tutabileceğini ve kapanışı daha sonra, tıpkı bir fonksiyon gibi, değişken ismini ve parantezleri kullanarak çağırabileceğimizi gösterir.

Aynı anda birden fazla değiştirilemez referans bulundurabileceğimiz için `list` vektörü; kapanıştan önce, kapanış tanımlandıktan sonra ama çağırılmadan önce, ve kapanış çağrıldıktan sonra da hâlâ erişilebilir durumdadır. Bu kod derlenir, çalıştırılır ve şu çıktıyı verir:


```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Sıradaki örnek olan Liste 13-5’te, kapanışın gövdesini değiştiriyoruz ve `list` vektörüne bir eleman eklemesini sağlıyoruz. Bu durumda kapanış artık `list`'e değiştirilebilir (mutable) bir referans yakalıyor.

<Listing number="13-5" file-name="src/main.rs" caption="Değiştirilebilir referans yakalayan bir kapanış tanımlama ve çağırma">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Bu kod derlenir, çalışır ve şu çıktıyı verir:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Dikkat edin, `borrows_mutably` kapanışının tanımı ile çağrılması arasında artık bir `println!` satırı yoktur. Bunun nedeni, `borrows_mutably` kapanışı tanımlandığında `list`'e değiştirilebilir bir referans yakalamasıdır. Kapanış çağrıldıktan sonra tekrar kullanılmadığı için değiştirilebilir ödünç alma (borrow) sona ermiş olur. Ancak kapanış tanımı ile çağrılması arasına bir `println!` eklemeye çalışırsanız, bu mümkün olmayacaktır çünkü Rust, bir değişken değiştirilebilir olarak ödünç alınmışken başka hiçbir ödünç alıma izin vermez. Bunu kendiniz deneyerek hata mesajını görebilirsiniz!

Eğer kapanışın gövdesi sahipliği (ownership) gerçekten gerektirmese bile, çevresindeki ortamdan yakaladığı değerlere sahip olmasını istiyorsanız, kapanış parametre listesinin önüne `move` anahtar kelimesini koyabilirsiniz.

Bu teknik genellikle bir kapanışı yeni bir iş parçacığına (thread) aktarırken kullanılır. Böylece yakalanan veriler, yeni iş parçacığına ait olur. İş parçacıklarını ve neden ihtiyaç duyduğumuzu 16. Bölüm’de ayrıntılı olarak ele alacağız. Ancak şimdilik, `move` anahtar kelimesi gerektiren bir kapanış kullanarak yeni bir iş parçacığı başlatmaya kısaca bakalım. Liste 13-6, Liste 13-4’ün bir iş parçacığında çalışacak şekilde değiştirilmiş hâlidir; vektörü ana iş parçacığı yerine yeni iş parçacığında yazdırır.


<Listing number="13-6" file-name="src/main.rs" caption="Bir iş parçacığına verilen kapanışın `list`'in sahipliğini alması için `move` kullanılması">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Yeni bir iş parçacığı (thread) başlatıyoruz ve bu iş parçacığına çalıştırması için bir kapanış veriyoruz. Kapanışın gövdesi, `list` vektörünü yazdırır. Liste 13-4’teki örnekte, kapanış sadece `list`'e değiştirilemez (immutable) bir referans yakalıyordu çünkü yazdırmak için bu kadarlık erişim yeterliydi. Bu örnekte, kapanış gövdesi hâlâ yalnızca değiştirilemez referansa ihtiyaç duymasına rağmen, `list`'in sahipliğini kapanışa aktarmamız gerekir. Bunu sağlamak için kapanış tanımının başına `move` anahtar kelimesi ekleriz.

Eğer ana iş parçacığı, `join` çağrılmadan önce başka işlemler yapıyor olsaydı, yeni iş parçacığı kalan işlemlerden önce tamamlanabilir veya tam tersi olabilir. Ana iş parçacığı `list`’in sahipliğini koruyup, yeni iş parçacığı tamamlanmadan önce sonlanır ve `list`’i düşürürse (drop), iş parçacığında kalan referans geçersiz hâle gelir. Bu nedenle derleyici, `list`’in sahipliğinin iş parçacığına verilen kapanışa taşınmasını zorunlu kılar. `move` anahtar kelimesini kaldırmayı veya kapanış tanımından sonra `list`'i ana iş parçacığında kullanmayı deneyerek derleyici hatalarını kendiniz görebilirsiniz!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### Yakalanan Değerleri Kapanışın Dışına Taşımak ve `Fn` Trait'leri

Bir kapanış, tanımlandığı ortamdan bir değere referans yakaladığında ya da onun sahipliğini aldığında (_closure içine_ ne taşınacağı etkilenmiş olur), kapanışın gövdesindeki kod kapanış daha sonra çalıştırıldığında (_closure dışına_ neyin taşınacağına) karar verir.

Bir kapanışın gövdesi şunları yapabilir: yakalanan değeri dışarı taşımak (move), yakalanan değeri değiştirmek (mutate), hiçbir şey taşımamak ya da değiştirmemek, veya hiçbir şey yakalamamak.

Bir kapanışın ortamdan değerleri nasıl yakaladığı ve bunlarla ne yaptığı, hangi trait’leri uyguladığını (implement ettiği) belirler. Trait’ler, fonksiyonların ve yapıların hangi türde kapanışları kullanabileceğini belirtmesini sağlar. Kapanışlar, gövdelerinin değerleri nasıl işlediğine bağlı olarak şu üç trait’ten birini, ikisini veya hepsini otomatik olarak uygularlar:

- `FnOnce`: En az bir kez çağrılabilen tüm kapanışlar bu trait’i uygular. Eğer kapanış, yakaladığı bir değeri gövdesinden dışarı taşıyorsa, yalnızca `FnOnce` trait’ini uygular ve diğer `Fn` trait’lerini uygulayamaz, çünkü yalnızca bir kez çağrılabilir.
- `FnMut`: Eğer kapanış değerleri dışarı taşımıyorsa ama değiştirebiliyorsa, `FnMut` uygulanır. Bu tür kapanışlar birden fazla kez çağrılabilir.
- `Fn`: Eğer kapanış hem değerleri dışarı taşımıyor hem de değiştirmiyorsa veya hiçbir şey yakalamıyorsa, `Fn` trait’ini uygular. Bu kapanışlar, ortamı değiştirmeden birçok kez çağrılabilir. Bu, özellikle kapanışın eşzamanlı (concurrent) olarak birden fazla kez çağrılması gereken durumlarda önemlidir.

Şimdi, Liste 13-1’de kullandığımız `Option<T>` tipi üzerindeki `unwrap_or_else` metodunun tanımına bakalım:


```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Burada `T`, `Option` türünün `Some` varyantında tutulan değerin türünü temsil eden jenerik (generic) türdür. Aynı `T` türü, `unwrap_or_else` fonksiyonunun dönüş türüdür. Örneğin, `Option<String>` üzerinde `unwrap_or_else` çağırırsak, geri dönüş değeri `String` olacaktır.

Ayrıca `unwrap_or_else` fonksiyonunun `F` adında ikinci bir jenerik tür parametresine sahip olduğuna dikkat edin. Bu `F` türü, fonksiyona `f` parametresiyle verilen kapanışın türünü temsil eder.

`F` türüne uygulanan trait sınırı `FnOnce() -> T` şeklindedir. Bu, `F` türünden bir değer (yani kapanış) en az bir kez çağrılabilir olmalı, argüman almamalı ve bir `T` değeri döndürmelidir. Trait sınırında `FnOnce` kullanılması, `unwrap_or_else` fonksiyonunun `f`'yi en fazla bir kez çağıracağını belirtir. Yukarıdaki gövdede de görüldüğü gibi, `Option` değeri `Some` ise `f` hiç çağrılmaz. `None` ise `f` bir kez çağrılır. Tüm kapanışlar `FnOnce` trait’ini uyguladığından, `unwrap_or_else` fonksiyonu tüm kapanış türlerini kabul edebilir ve oldukça esnektir.

> Not: Eğer yapmak istediğimiz şey ortamdan herhangi bir değer yakalamayı gerektirmiyorsa, `Fn` trait’lerinden birini uygulayan bir kapanış yerine doğrudan bir fonksiyon ismi de kullanabiliriz. Örneğin, bir `Option<Vec<T>>` değeri üzerinde `unwrap_or_else(Vec::new)` çağırabiliriz; eğer değer `None` ise bu çağrı boş bir vektör döndürür. Derleyici, fonksiyon tanımına uygun olan `Fn` trait’ini otomatik olarak uygular.

Şimdi dilimlerde (`slice`) tanımlanmış olan standart kütüphane fonksiyonu `sort_by_key`’e bakalım. Bu fonksiyonun `unwrap_or_else`’den nasıl farklı çalıştığını ve neden `FnOnce` yerine `FnMut` trait sınırını kullandığını göreceğiz. Bu fonksiyonda kapanış, dilimdeki her öğe için bir referans alır ve karşılaştırılabilir (`Ord`) bir `K` türü döndürür. Bu, örneğin her öğenin belli bir niteliğine göre sıralama yapmak istediğimizde kullanışlıdır. Liste 13-7’de, bir dizi `Rectangle` örneğimiz var ve bunları `width` (genişlik) değerine göre küçükten büyüğe sıralıyoruz.

<Listing number="13-7" file-name="src/main.rs" caption="Dikdörtgenleri genişliğe göre sıralamak için `sort_by_key` kullanımı">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Bu kodun çıktısı şudur:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key` fonksiyonunun bir `FnMut` kapanışı alacak şekilde tanımlanmasının nedeni, kapanışın dilimdeki her öğe için bir kez çağrılacak olmasıdır. Bu örnekte kullanılan kapanış `|r| r.width` ortamdan hiçbir değer yakalamaz, değiştirmez veya dışarı taşımaz. Bu nedenle `FnMut` trait sınırını karşılamaktadır.

Buna karşılık, Liste 13-8’de gösterilen örnekte, yalnızca `FnOnce` trait’ini uygulayan bir kapanış yer alır çünkü bu kapanış ortamdan bir değeri dışarı taşır. Derleyici, bu kapanışın `sort_by_key` ile kullanılmasına izin vermez.


<Listing number="13-8" file-name="src/main.rs" caption="`FnOnce` kapanışını `sort_by_key` ile kullanmaya çalışmak">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

Bu, `sort_by_key` fonksiyonunun kapanışı kaç kez çağırdığını saymaya çalışmanın dolaylı ve karmaşık (ve çalışmayan) bir yoludur. Bu kod, kapanışın ortamından gelen bir `String` olan `value` değerini `sort_operations` vektörüne ekleyerek sayma işlemini gerçekleştirmeyi dener. Kapanış, `value` değerini yakalar ve ardından `value`’yu ortamdan alarak sahipliğini `sort_operations` vektörüne devreder. Bu kapanış yalnızca bir kez çağrılabilir; ikinci kez çağrılmaya çalışıldığında `value` artık ortamda bulunmayacağı için tekrar `sort_operations` vektörüne eklenemez! Bu nedenle, bu kapanış yalnızca `FnOnce` trait’ini uygular. Bu kodu derlemeye çalıştığımızda, kapanışın `FnMut` olması gerektiği ancak `value`’nun ortamdan taşındığı için `move` edilemediğini belirten şu hatayı alırız:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Bu hata, kapanış gövdesinde `value`’nun ortamdan taşındığı satırı işaret eder. Bu hatayı düzeltmek için kapanış gövdesini, ortamdan değerleri taşımayacak şekilde değiştirmemiz gerekir. Ortamda bir sayaç tutmak ve kapanış içinde bu değeri arttırmak, kapanışın kaç kez çağrıldığını saymak için daha doğrudan bir yaklaşımdır. Liste 13-9’daki kapanış, yalnızca `num_sort_operations` sayaç değişkenine mutlak bir referans yakaladığı için `sort_by_key` ile çalışır ve birden fazla kez çağrılabilir:

<Listing number="13-9" file-name="src/main.rs" caption="`sort_by_key` ile `FnMut` kapanışı kullanmak mümkündür">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

`Fn` trait’leri, kapanışları kullanan veya tanımlayan fonksiyon ya da türlerde oldukça önemlidir. Bir sonraki bölümde iteratörleri ele alacağız. Birçok iteratör metodu, parametre olarak kapanış alır; dolayısıyla bu kapanış detaylarını aklınızda tutmanız faydalı olacaktır!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
