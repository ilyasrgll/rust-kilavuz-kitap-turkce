<!-- Eski başlık. Kaldırmayın, bağlantılar bozulabilir. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

<<<<<<< HEAD
## Kapanışlar: Ortamlarını Yakalayabilen Anonim Fonksiyonlar

Rust’taki kapanışlar, bir değişkene kaydedebileceğiniz veya başka fonksiyonlara argüman olarak verebileceğiniz anonim fonksiyonlardır. Kapanışı bir yerde tanımlayıp, daha sonra farklı bir bağlamda değerlendirmek üzere başka bir yerde çağırabilirsiniz. Fonksiyonlardan farklı olarak, kapanışlar tanımlandıkları kapsamdan değerleri yakalayabilirler. Bu bölümde kapanışların bu özelliklerinin kod tekrarını nasıl azaltabileceğini ve davranışları nasıl özelleştirebileceğini göstereceğiz.
=======
## Kapanışlar: Ortamlarını Yakalayabilen İsimsiz Fonksiyonlar

Rust'taki kapanışlar, bir değişkende saklayabileceğiniz veya başka fonksiyonlara argüman olarak geçirebileceğiniz isimsiz fonksiyonlardır. Kapanışı bir yerde oluşturup, daha sonra farklı bir bağlamda çağırarak değerlendirebilirsiniz. Fonksiyonlardan farklı olarak, kapanışlar tanımlandıkları kapsamdan değerleri yakalayabilirler. Bu kapanış özelliklerinin kodun yeniden kullanılmasını ve davranışın özelleştirilmesini nasıl sağladığını göstereceğiz.
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

<!-- Eski başlıklar. Kaldırmayın, bağlantılar bozulabilir. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

<<<<<<< HEAD
### Kapanışlarla Ortamın Yakalanması

Öncelikle kapanışları, tanımlandıkları ortamdan değerleri yakalayarak daha sonra kullanmak için nasıl kullanabileceğimizi inceleyeceğiz. Senaryo şöyle: Tişört şirketimiz belirli aralıklarla, posta listemizdeki bir kişiye promosyon amacıyla özel üretim, sınırlı sayıda bir tişört hediye ediyor. Posta listesinde yer alan kişiler, isteğe bağlı olarak profillerine favori renklerini ekleyebiliyor. Eğer seçilen kişi favori rengini belirtmişse, o renkte tişört alıyor. Eğer bir renk belirtilmemişse, şirketin o an en çok stokta bulunan rengi gönderiliyor.


Bunu uygulamanın birçok yolu vardır. Bu örnekte, `ShirtColor` adında bir `enum` kullanacağız; bu `enum` yalnızca `Red` ve `Blue` varyantlarına sahiptir (basitlik olması açısından renk sayısını sınırlıyoruz). Şirketin envanterini, içinde `shirts` adında bir alan bulunan `Inventory` adlı bir `struct` ile temsil edeceğiz. Bu alan, şu anda stokta bulunan tişört renklerini içeren bir `Vec<ShirtColor>` olacaktır. `Inventory` yapısı üzerinde tanımlı olan `giveaway` metodu, ücretsiz tişört kazanacak kişinin isteğe bağlı tişört rengi tercih bilgilerini alır ve hangi rengin verileceğini döndürür. Bu kurulum Liste 13-1’de gösterilmektedir.

<Listing number="13-1" file-name="src/main.rs" caption="Tişört şirketinin promosyon dağıtım durumu">
=======
### Kapanışlarla Ortamı Yakalamak

Öncelikle, kapanışların tanımlandıkları ortamdan değerleri daha sonra kullanmak üzere nasıl yakalayabileceğimizi inceleyeceğiz. Senaryo şu: Zaman zaman, tişört şirketimiz promosyon olarak posta listemizden birine özel, sınırlı sayıda bir tişört hediye ediyor. Posta listesindeki kişiler profillerine isteğe bağlı olarak favori renklerini ekleyebiliyor. Eğer seçilen kişinin favori rengi ayarlanmışsa, o renkte tişört alıyor. Eğer favori renk belirtilmemişse, şirkette en çok bulunan renkten bir tişört veriliyor.

Bunu uygulamanın birçok yolu var. Bu örnekte, basitlik için yalnızca `Kırmızı` ve `Mavi` varyantlarına sahip bir `ShirtColor` enum'u kullanacağız. Şirketin stoğunu ise, içinde mevcut tişört renklerini tutan bir `Vec<ShirtColor>` içeren `shirts` alanına sahip bir `Inventory` yapısı ile temsil ediyoruz. `Inventory` üzerinde tanımlı `giveaway` metodu, hediye tişört kazanan kişinin isteğe bağlı renk tercihini alır ve kişiye verilecek tişört rengini döndürür. Bu kurulum, Liste 13-1'de gösterilmiştir.

<Listing number="13-1" file-name="src/main.rs" caption="Tişört şirketi hediye durumu">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

<<<<<<< HEAD
`main` fonksiyonu içerisinde tanımlanan `store`, bu özel üretim promosyon için iki mavi ve bir kırmızı tişörte sahiptir. `giveaway` metodunu, biri kırmızı tişört tercih eden ve diğeri herhangi bir tercih belirtmeyen iki kullanıcı için çağırıyoruz.

Bu kod elbette başka şekillerde de uygulanabilirdi, fakat burada odağımız kapanışlar (closures) olduğu için, halihazırda öğrendiğiniz kavramlara sadık kalıyoruz; yalnızca `giveaway` metodunun gövdesi farklıdır çünkü burada bir kapanış kullanılmıştır. `giveaway` metodunda, kullanıcı tercihini `Option<ShirtColor>` türünde bir parametre olarak alıyoruz ve `user_preference` üzerinde `unwrap_or_else` metodunu çağırıyoruz. [`Option<T>` üzerindeki `unwrap_or_else` metodu][unwrap-or-else]<!-- ignore --> standart kütüphane tarafından tanımlanmıştır. Bu metot, bağımsız değişken olarak argümansız bir kapanış alır ve bu kapanış `T` türünde (bu örnekte `ShirtColor`) bir değer döndürür. Eğer `Option<T>` değeri `Some` ise, `unwrap_or_else` içerisindeki değeri döndürür. Eğer değer `None` ise, kapanışı çalıştırır ve onun döndürdüğü değeri verir.

`unwrap_or_else` metoduna argüman olarak `|| self.most_stocked()` kapanış ifadesini veriyoruz. Bu kapanış, herhangi bir parametre almaz (eğer alsaydı, parametreler iki dikey çizgi arasına yazılırdı). Kapanış gövdesi `self.most_stocked()` fonksiyonunu çağırır. Kapanışı burada tanımlıyoruz, ama `unwrap_or_else` metodunun implementasyonu kapanışı yalnızca gerekli olduğunda çalıştıracaktır.


Bu kodu çalıştırdığınızda aşağıdaki çıktıyı verir:
=======
`main`'de tanımlanan `store`'da bu promosyon için dağıtılacak iki mavi ve bir kırmızı tişört kalmıştır. `giveaway` metodunu, kırmızı tişört tercihi olan bir kullanıcı ve tercihi olmayan bir kullanıcı için çağırıyoruz.

Bu kod birçok şekilde uygulanabilirdi, fakat burada kapanışlara odaklanmak için, öğrendiğiniz kavramlar dışında yalnızca `giveaway` metodunun gövdesinde bir kapanış kullandık. `giveaway` metodunda, kullanıcı tercihini `Option<ShirtColor>` türünde bir parametre olarak alıyoruz ve `user_preference` üzerinde `unwrap_or_else` metodunu çağırıyoruz. [`Option<T>` üzerindeki `unwrap_or_else` metodu][unwrap-or-else]<!-- ignore --> standart kütüphane tarafından tanımlanmıştır. Bir argüman alır: argüman almayan ve bir `T` (bu durumda `ShirtColor`) döndüren bir kapanış. Eğer `Option<T>` `Some` varyantıysa, `unwrap_or_else` içindeki değeri döndürür. Eğer `Option<T>` `None` ise, `unwrap_or_else` kapanışı çağırır ve kapanışın döndürdüğü değeri döndürür.

`unwrap_or_else`'e argüman olarak `|| self.most_stocked()` kapanış ifadesini veriyoruz. Bu, kendisi parametre almayan bir kapanıştır (eğer kapanışın parametreleri olsaydı, iki dikey çizgi arasına yazılırdı). Kapanışın gövdesi `self.most_stocked()` fonksiyonunu çağırır. Kapanışı burada tanımlıyoruz ve `unwrap_or_else`'in implementasyonu gerekirse kapanışı daha sonra çalıştıracak.

Bu kodu çalıştırdığınızda şunu yazdırır:
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

<<<<<<< HEAD
Burada ilginç olan şeylerden biri, mevcut `Inventory` örneği üzerinde `self.most_stocked()` metodunu çağıran bir kapanışı `unwrap_or_else` metoduna aktarmış olmamızdır. Standart kütüphane, tanımladığımız `Inventory` veya `ShirtColor` türleri hakkında hiçbir şey bilmek zorunda değildir; bu senaryoda kullanmak istediğimiz mantığı da bilmez. Kapanış, `self` adlı `Inventory` örneğine değiştirilemez (immutable) bir referans yakalar ve bu referansla birlikte tanımladığımız kodu `unwrap_or_else` metoduna aktarır. Öte yandan, normal fonksiyonlar bu şekilde ortamlarını yakalayamazlar.

### Kapanışlarda Tür Çıkarımı ve Açık Tür Belirtimi

Fonksiyonlar ile kapanışlar arasında başka farklar da vardır. Kapanışlar genellikle parametrelerin veya döndürdükleri değerin türünü belirtmenizi gerektirmez; oysa `fn` anahtar sözcüğüyle tanımlanan fonksiyonlarda bu tür açıklamaları yapmak zorunludur. Fonksiyonlarda tür belirtimi zorunludur çünkü bu türler kullanıcılarınıza açık bir arayüz olarak sunulur. Bu arayüzü net biçimde tanımlamak, fonksiyonun ne tür değerler alıp ne tür değerler döndüreceği konusunda herkesin hemfikir olmasını sağlar. Kapanışlar ise böyle dışa açık arayüzlerde kullanılmazlar; genellikle değişkenlerde saklanırlar ve kütüphanemizi kullanan dış kodlara ad verilerek sunulmazlar.


Kapanışlar genellikle kısa olur ve geniş çaplı durumlar yerine dar kapsamlı özel senaryolarda kullanılır. Bu sınırlı bağlamlar içerisinde, derleyici (compiler) parametrelerin ve dönüş değerinin türlerini tıpkı değişkenlerde olduğu gibi kendiliğinden çıkarabilir. (Nadir durumlarda kapanışlar için açık tür tanımlamaları da gerekebilir.)

Tıpkı değişkenlerde olduğu gibi, daha açık ve net olması adına tür açıklamaları da ekleyebiliriz; fakat bu, gerekli olmayan fazladan sözdizimsel ayrıntı anlamına gelir. Bir kapanış için tür açıklamaları eklemek, Liste 13-2’de gösterildiği gibi görünür. Bu örnekte, kapanışı doğrudan argüman olarak geçirmek yerine bir değişkene atayarak tanımlıyoruz (Liste 13-1’deki gibi doğrudan geçmiyoruz).

=======
Buradaki ilginç bir nokta, mevcut `Inventory` örneği üzerinde `self.most_stocked()` çağıran bir kapanış geçirmiş olmamızdır. Standart kütüphanenin bizim tanımladığımız `Inventory` veya `ShirtColor` türleri ya da bu senaryoda kullanmak istediğimiz mantık hakkında hiçbir şey bilmesine gerek yoktu. Kapanış, `self` `Inventory` örneğine değiştirilemez bir referans yakalar ve belirttiğimiz kodu `unwrap_or_else` metoduna iletir. Fonksiyonlar ise ortamlarını bu şekilde yakalayamazlar.

### Kapanışlarda Tür Çıkarımı ve Açık Tür Bildirimi

Fonksiyonlar ve kapanışlar arasında başka farklar da vardır. Kapanışlar genellikle parametrelerin veya dönüş değerinin türünü, `fn` fonksiyonlarında olduğu gibi belirtmenizi gerektirmez. Fonksiyonlarda tür bildirimleri gereklidir çünkü türler, kullanıcılarınıza açıkça sunulan bir arayüzün parçasıdır. Bu arayüzü katı şekilde tanımlamak, herkesin bir fonksiyonun hangi türde değerler kullandığı ve döndürdüğü konusunda hemfikir olmasını sağlamak için önemlidir. Kapanışlar ise böyle açık bir arayüzde kullanılmaz: değişkenlerde saklanır ve isimlendirilmeden, kütüphanenizin kullanıcılarına sunulmadan kullanılır.

Kapanışlar genellikle kısa ve yalnızca dar bir bağlamda geçerlidir. Bu sınırlı bağlamlarda, derleyici parametrelerin ve dönüş değerinin türünü, çoğu değişkenin türünü çıkarabildiği gibi çıkarabilir (nadir durumlarda kapanış tür bildirimlerine de ihtiyaç duyulabilir).

Değişkenlerde olduğu gibi, açıklığı ve netliği artırmak için tür bildirimleri ekleyebiliriz, ancak bu, gereğinden fazla ayrıntılı olmamıza neden olur. Bir kapanış için tür bildirimleri eklemek, Liste 13-2'de gösterildiği gibi olur. Bu örnekte, kapanışı argüman olarak geçtiğimiz yerde tanımlamak yerine, bir değişkende saklıyoruz.

>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e
<Listing number="13-2" file-name="src/main.rs" caption="Kapanışta parametre ve dönüş türlerinin isteğe bağlı olarak belirtilmesi">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

<<<<<<< HEAD
Tür açıklamaları eklendiğinde, kapanışların sözdizimi fonksiyonlarınkine daha çok benzemeye başlar. Burada, bir parametreye 1 ekleyen bir fonksiyon ve aynı davranışı gösteren bir kapanış tanımlıyoruz. İlgili bölümleri hizalamak için bazı boşluklar ekledik. Bu örnek, kapanış sözdiziminin fonksiyon sözdizimine ne kadar benzediğini; farkların yalnızca boru çizgileri (`| |`) ve bazı sözdizim öğelerinin opsiyonel olmasında olduğunu gösteriyor:
=======
Tür bildirimleri eklendiğinde, kapanışların sözdizimi fonksiyonlarınkine daha çok benzer. Burada, parametresine 1 ekleyen bir fonksiyon ve aynı davranışa sahip bir kapanış tanımlıyoruz. İlgili kısımları hizalamak için bazı boşluklar ekledik. Bu, kapanış sözdiziminin fonksiyon sözdizimine ne kadar benzediğini, ancak boru işaretleri ve isteğe bağlı sözdizimi miktarı dışında nasıl farklılaştığını gösterir:
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

<<<<<<< HEAD
İlk satır, klasik bir fonksiyon tanımıdır. İkinci satırda tam tür belirtilmiş bir kapanış tanımı yer alır. Üçüncü satırda tür açıklamaları kaldırılmıştır. Dördüncü satırda ise süslü parantezler (`{}`) kaldırılmıştır — bu, gövde yalnızca bir ifade içerdiğinde opsiyoneldir. Tüm bu tanımlar geçerlidir ve çağrıldıklarında aynı davranışı gösterirler. `add_one_v3` ve `add_one_v4` satırlarında, türlerin kullanım bağlamından çıkarılabilmesi için kapanışların gerçekten değerlendirilmesi gerekir. Bu durum, Rust’un `let v = Vec::new();` ifadesinde de geçerlidir — derleyici `Vec`’in türünü çıkarabilmek için ya tür açıklaması ister ya da içine eklenecek öğelere bakar.

Kapanış tanımlarında, derleyici her parametre ve dönüş değeri için tek bir kesin tür çıkarımı yapar. Örneğin, Liste 13-3’te yalnızca aldığı değeri aynen geri döndüren kısa bir kapanış tanımı yer alır. Bu kapanış, yalnızca örnek amacıyla gösterilmiştir ve gerçek kullanım açısından çok anlamlı değildir. Dikkat ederseniz, tanımda hiçbir tür açıklaması yapılmamıştır. Bu nedenle kapanışı ilk kez `String` türüyle çağırdığımızda bu tür kapanışa yerleşir. Daha sonra aynı kapanışı bir tamsayı (integer) ile çağırmaya çalışırsak, hata alırız.


<Listing number="13-3" file-name="src/main.rs" caption="Türleri çıkarımla belirlenmiş bir kapanışı iki farklı türle çağırma girişimi">
=======
İlk satırda bir fonksiyon tanımı, ikinci satırda ise tam tür bildirimli bir kapanış tanımı var. Üçüncü satırda, kapanış tanımından tür bildirimlerini kaldırıyoruz. Dördüncü satırda ise, kapanış gövdesi yalnızca bir ifade içerdiği için süslü parantezleri kaldırıyoruz. Bunların hepsi geçerli tanımlardır ve çağrıldıklarında aynı davranışı gösterirler. `add_one_v3` ve `add_one_v4` satırlarında, kapanışların kullanılacağı yerden türlerin çıkarılması gerekir. Bu, `let v = Vec::new();` ifadesinin de tür çıkarımı için ya tür bildirimi ya da vektöre bir değer eklenmesini gerektirmesine benzer.

Kapanış tanımlarında, derleyici parametrelerin ve dönüş değerinin her biri için birer somut tür çıkarır. Örneğin, Liste 13-3'te yalnızca aldığı değeri döndüren kısa bir kapanış tanımı gösterilmiştir. Bu kapanış, örnek amacı dışında çok kullanışlı değildir. Tanımda tür bildirimi yoktur. Tür bildirimi olmadığı için, kapanışı herhangi bir türle çağırabiliriz; burada ilk olarak `String` ile çağırıyoruz. Sonra, `example_closure`'ı bir tamsayı ile çağırmaya çalışırsak hata alırız.

<Listing number="13-3" file-name="src/main.rs" caption="Türleri çıkarılan bir kapanışı iki farklı türle çağırmaya çalışmak">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Derleyici bize şu hatayı verir:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

<<<<<<< HEAD
`example_closure` adlı kapanışı ilk kez bir `String` değeriyle çağırdığımızda, derleyici `x` değişkeninin türünü ve kapanışın dönüş türünü `String` olarak çıkarımlar. Bu türler artık `example_closure` kapanışına sabitlenmiş olur ve aynı kapanışı daha sonra farklı bir türle (örneğin bir tamsayıyla) çağırmaya çalıştığımızda tür uyumsuzluğu hatası alırız.

### Referansları Yakalama veya Sahipliği Taşıma

Kapanışlar, tanımlandıkları ortamdan değerleri üç farklı yolla yakalayabilirler. Bu yakalama yöntemleri, bir fonksiyonun parametre alırken kullandığı üç temel yönteme karşılık gelir: değiştirilemez (immutable) ödünç alma, değiştirilebilir (mutable) ödünç alma ve sahiplenme (ownership).

Liste 13-4’te, yalnızca değeri yazdırmak için değiştirilemez referansa ihtiyaç duyan bir kapanış tanımlıyoruz. Bu kapanış, `list` adlı vektöre değiştirilemez bir referans yakalar.

<Listing number="13-4" file-name="src/main.rs" caption="Değiştirilemez referans yakalayan bir kapanış tanımlama ve çağırma">
=======
İlk kez `example_closure`'ı bir `String` değeriyle çağırdığımızda, derleyici `x`'in ve kapanışın dönüş değerinin türünü `String` olarak çıkarır. Bu türler, `example_closure` kapanışında kilitlenir ve aynı kapanışla farklı bir tür kullanmaya çalıştığımızda tür hatası alırız.

### Referansları Yakalamak veya Sahipliği Taşımak

Kapanışlar, ortamlarından değerleri üç şekilde yakalayabilir: değiştirilemez ödünç alma, değiştirilebilir ödünç alma ve sahipliği alma. Kapanış, gövdesinde yakalanan değerlerle ne yaptığına göre hangisini kullanacağına karar verir.

Liste 13-4'te, yalnızca değeri yazdırmak için değiştirilemez bir referansa ihtiyaç duyduğu için, `list` adlı vektöre değiştirilemez bir referans yakalayan bir kapanış tanımlıyoruz.

<Listing number="13-4" file-name="src/main.rs" caption="Değiştirilemez referans yakalayan bir kapanış tanımlamak ve çağırmak">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

<<<<<<< HEAD
Bu örnek aynı zamanda bir değişkenin bir kapanışı tutabileceğini ve kapanışı daha sonra, tıpkı bir fonksiyon gibi, değişken ismini ve parantezleri kullanarak çağırabileceğimizi gösterir.

Aynı anda birden fazla değiştirilemez referans bulundurabileceğimiz için `list` vektörü; kapanıştan önce, kapanış tanımlandıktan sonra ama çağırılmadan önce, ve kapanış çağrıldıktan sonra da hâlâ erişilebilir durumdadır. Bu kod derlenir, çalıştırılır ve şu çıktıyı verir:

=======
Bu örnek ayrıca, bir değişkenin bir kapanış tanımına bağlanabileceğini ve daha sonra kapanışı, değişken adı ve parantez kullanarak bir fonksiyon adıymış gibi çağırabileceğimizi gösterir.

Aynı anda birden fazla değiştirilemez referansa sahip olabileceğimiz için, `list` kapanış tanımından önce, tanımdan sonra ama kapanış çağrılmadan önce ve kapanış çağrıldıktan sonra da erişilebilir. Bu kod derlenir, çalışır ve şunu yazdırır:
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

<<<<<<< HEAD
Sıradaki örnek olan Liste 13-5’te, kapanışın gövdesini değiştiriyoruz ve `list` vektörüne bir eleman eklemesini sağlıyoruz. Bu durumda kapanış artık `list`'e değiştirilebilir (mutable) bir referans yakalıyor.

<Listing number="13-5" file-name="src/main.rs" caption="Değiştirilebilir referans yakalayan bir kapanış tanımlama ve çağırma">
=======
Sonraki örnekte, Liste 13-5'te, kapanış gövdesini `list` vektörüne bir eleman ekleyecek şekilde değiştiriyoruz. Kapanış artık değiştirilebilir bir referans yakalıyor.

<Listing number="13-5" file-name="src/main.rs" caption="Değiştirilebilir referans yakalayan bir kapanış tanımlamak ve çağırmak">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

<<<<<<< HEAD
Bu kod derlenir, çalışır ve şu çıktıyı verir:
=======
Bu kod derlenir, çalışır ve şunu yazdırır:
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

<<<<<<< HEAD
Dikkat edin, `borrows_mutably` kapanışının tanımı ile çağrılması arasında artık bir `println!` satırı yoktur. Bunun nedeni, `borrows_mutably` kapanışı tanımlandığında `list`'e değiştirilebilir bir referans yakalamasıdır. Kapanış çağrıldıktan sonra tekrar kullanılmadığı için değiştirilebilir ödünç alma (borrow) sona ermiş olur. Ancak kapanış tanımı ile çağrılması arasına bir `println!` eklemeye çalışırsanız, bu mümkün olmayacaktır çünkü Rust, bir değişken değiştirilebilir olarak ödünç alınmışken başka hiçbir ödünç alıma izin vermez. Bunu kendiniz deneyerek hata mesajını görebilirsiniz!

Eğer kapanışın gövdesi sahipliği (ownership) gerçekten gerektirmese bile, çevresindeki ortamdan yakaladığı değerlere sahip olmasını istiyorsanız, kapanış parametre listesinin önüne `move` anahtar kelimesini koyabilirsiniz.

Bu teknik genellikle bir kapanışı yeni bir iş parçacığına (thread) aktarırken kullanılır. Böylece yakalanan veriler, yeni iş parçacığına ait olur. İş parçacıklarını ve neden ihtiyaç duyduğumuzu 16. Bölüm’de ayrıntılı olarak ele alacağız. Ancak şimdilik, `move` anahtar kelimesi gerektiren bir kapanış kullanarak yeni bir iş parçacığı başlatmaya kısaca bakalım. Liste 13-6, Liste 13-4’ün bir iş parçacığında çalışacak şekilde değiştirilmiş hâlidir; vektörü ana iş parçacığı yerine yeni iş parçacığında yazdırır.


<Listing number="13-6" file-name="src/main.rs" caption="Bir iş parçacığına verilen kapanışın `list`'in sahipliğini alması için `move` kullanılması">
=======
Artık `borrows_mutably` kapanışının tanımı ile çağrılması arasında bir `println!` yok: `borrows_mutably` tanımlandığında, `list`'e değiştirilebilir bir referans yakalar. Kapanış çağrıldıktan sonra tekrar kullanılmadığı için değiştirilebilir ödünç alma sona erer. Kapanış tanımı ile çağrısı arasında, yazdırmak için değiştirilemez ödünç alma artık izinli değildir çünkü değiştirilebilir ödünç alma varken başka ödünç almalar yapılamaz. Oraya bir `println!` ekleyip alacağınız hata mesajını görebilirsiniz!

Kapanışın gövdesi sahipliği doğrudan gerektirmese bile, ortamındaki değerlerin sahipliğini kapanışa zorla taşımak isterseniz, parametre listesinin başına `move` anahtar kelimesini ekleyebilirsiniz.

Bu teknik, genellikle bir kapanışı yeni bir iş parçacığına geçirirken, verinin yeni iş parçacığı tarafından sahiplenilmesini sağlamak için kullanılır. İş parçacıklarını ve neden kullanmak isteyebileceğinizi 16. Bölümde ayrıntılı olarak ele alacağız, ancak şimdilik, bir kapanışın sahipliği almasını gerektiren bir iş parçacığı başlatmayı kısaca inceleyelim. Liste 13-6, Liste 13-4'ün vektörü ana iş parçacığı yerine yeni bir iş parçacığında yazdıracak şekilde değiştirilmiş halini gösterir.

<Listing number="13-6" file-name="src/main.rs" caption="İş parçacığı için kapanışın `list`'in sahipliğini almasını zorlamak için `move` kullanmak">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

<<<<<<< HEAD
Yeni bir iş parçacığı (thread) başlatıyoruz ve bu iş parçacığına çalıştırması için bir kapanış veriyoruz. Kapanışın gövdesi, `list` vektörünü yazdırır. Liste 13-4’teki örnekte, kapanış sadece `list`'e değiştirilemez (immutable) bir referans yakalıyordu çünkü yazdırmak için bu kadarlık erişim yeterliydi. Bu örnekte, kapanış gövdesi hâlâ yalnızca değiştirilemez referansa ihtiyaç duymasına rağmen, `list`'in sahipliğini kapanışa aktarmamız gerekir. Bunu sağlamak için kapanış tanımının başına `move` anahtar kelimesi ekleriz.

Eğer ana iş parçacığı, `join` çağrılmadan önce başka işlemler yapıyor olsaydı, yeni iş parçacığı kalan işlemlerden önce tamamlanabilir veya tam tersi olabilir. Ana iş parçacığı `list`’in sahipliğini koruyup, yeni iş parçacığı tamamlanmadan önce sonlanır ve `list`’i düşürürse (drop), iş parçacığında kalan referans geçersiz hâle gelir. Bu nedenle derleyici, `list`’in sahipliğinin iş parçacığına verilen kapanışa taşınmasını zorunlu kılar. `move` anahtar kelimesini kaldırmayı veya kapanış tanımından sonra `list`'i ana iş parçacığında kullanmayı deneyerek derleyici hatalarını kendiniz görebilirsiniz!
=======
Yeni bir iş parçacığı başlatıyoruz ve iş parçacığına argüman olarak çalıştırması için bir kapanış veriyoruz. Kapanış gövdesi listeyi yazdırıyor. Liste 13-4'te, kapanış yalnızca yazdırmak için `list`'i değiştirilemez bir referansla yakalamıştı. Bu örnekte, kapanış gövdesi hala yalnızca değiştirilemez bir referansa ihtiyaç duysa da, kapanış tanımının başına `move` anahtar kelimesini ekleyerek `list`'in kapanışa taşınmasını belirtmemiz gerekiyor. Ana iş parçacığı, yeni iş parçacığı çağrılmadan önce daha fazla işlem yaparsa, yeni iş parçacığı ana iş parçacığından önce veya sonra bitebilir. Ana iş parçacığı `list`'in sahipliğini koruyup yeni iş parçacığından önce biterse ve `list`'i bırakırsa, iş parçacığındaki değiştirilemez referans geçersiz olurdu. Bu nedenle, derleyici, yeni iş parçacığına verilen kapanışa `list`'in taşınmasını zorunlu kılar. `move` anahtar kelimesini kaldırmayı veya kapanış tanımından sonra ana iş parçacığında `list`'i kullanmayı deneyin, hangi derleyici hatalarını alacağınızı göreceksiniz!
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

<!-- Eski başlıklar. Kaldırmayın, bağlantılar bozulabilir. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

<<<<<<< HEAD
### Yakalanan Değerleri Kapanışın Dışına Taşımak ve `Fn` Trait'leri

Bir kapanış, tanımlandığı ortamdan bir değere referans yakaladığında ya da onun sahipliğini aldığında (_closure içine_ ne taşınacağı etkilenmiş olur), kapanışın gövdesindeki kod kapanış daha sonra çalıştırıldığında (_closure dışına_ neyin taşınacağına) karar verir.

Bir kapanışın gövdesi şunları yapabilir: yakalanan değeri dışarı taşımak (move), yakalanan değeri değiştirmek (mutate), hiçbir şey taşımamak ya da değiştirmemek, veya hiçbir şey yakalamamak.

Bir kapanışın ortamdan değerleri nasıl yakaladığı ve bunlarla ne yaptığı, hangi trait’leri uyguladığını (implement ettiği) belirler. Trait’ler, fonksiyonların ve yapıların hangi türde kapanışları kullanabileceğini belirtmesini sağlar. Kapanışlar, gövdelerinin değerleri nasıl işlediğine bağlı olarak şu üç trait’ten birini, ikisini veya hepsini otomatik olarak uygularlar:

- `FnOnce`: En az bir kez çağrılabilen tüm kapanışlar bu trait’i uygular. Eğer kapanış, yakaladığı bir değeri gövdesinden dışarı taşıyorsa, yalnızca `FnOnce` trait’ini uygular ve diğer `Fn` trait’lerini uygulayamaz, çünkü yalnızca bir kez çağrılabilir.
- `FnMut`: Eğer kapanış değerleri dışarı taşımıyorsa ama değiştirebiliyorsa, `FnMut` uygulanır. Bu tür kapanışlar birden fazla kez çağrılabilir.
- `Fn`: Eğer kapanış hem değerleri dışarı taşımıyor hem de değiştirmiyorsa veya hiçbir şey yakalamıyorsa, `Fn` trait’ini uygular. Bu kapanışlar, ortamı değiştirmeden birçok kez çağrılabilir. Bu, özellikle kapanışın eşzamanlı (concurrent) olarak birden fazla kez çağrılması gereken durumlarda önemlidir.

Şimdi, Liste 13-1’de kullandığımız `Option<T>` tipi üzerindeki `unwrap_or_else` metodunun tanımına bakalım:

=======
### Kapanışlardan Değerleri Taşımak ve `Fn` Özellikleri

Bir kapanış ortamından bir referans yakaladığında veya bir değerin sahipliğini aldığında (yani kapanışa _ne taşındığını_ etkilediğinde), kapanış gövdesindeki kod, kapanış daha sonra çalıştırıldığında referanslara veya değerlere ne olacağını tanımlar (yani kapanıştan _ne çıkarılacağını_ etkiler).

Bir kapanış gövdesi şunları yapabilir: yakalanan bir değeri kapanıştan dışarı taşıyabilir, yakalanan değeri değiştirebilir, değeri ne taşıyabilir ne de değiştirebilir veya ortamdan hiçbir şey yakalamayabilir.

Bir kapanışın ortamdan değerleri nasıl yakaladığı ve işlediği, kapanışın hangi trait'leri uyguladığını etkiler ve trait'ler, fonksiyonların ve yapıların hangi tür kapanışları kullanabileceğini belirtmesini sağlar. Kapanışlar, gövdelerinin değerlerle ne yaptığına bağlı olarak, bu üç `Fn` trait'inden birini, ikisini veya üçünü birden otomatik olarak uygular:

* `FnOnce`, yalnızca bir kez çağrılabilen kapanışlar için geçerlidir. Tüm kapanışlar en azından bu trait'i uygular çünkü tüm kapanışlar çağrılabilir. Gövdesinden yakalanan değerleri dışarı taşıyan bir kapanış yalnızca `FnOnce`'ı uygular ve diğer `Fn` trait'lerini uygulamaz çünkü yalnızca bir kez çağrılabilir.
* `FnMut`, gövdesinden yakalanan değerleri dışarı taşımayan, ancak yakalanan değerleri değiştirebilen kapanışlar için geçerlidir. Bu kapanışlar birden fazla kez çağrılabilir.
* `Fn`, gövdesinden yakalanan değerleri dışarı taşımayan ve değiştirmeyen, ayrıca ortamdan hiçbir şey yakalamayan kapanışlar için geçerlidir. Bu kapanışlar, ortamlarını değiştirmeden birden fazla kez çağrılabilir, bu da örneğin bir kapanışın aynı anda birden fazla kez çağrılması gereken durumlarda önemlidir.

`Option<T>` üzerinde kullandığımız `unwrap_or_else` metodunun tanımına bakalım:
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

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

<<<<<<< HEAD
Burada `T`, `Option` türünün `Some` varyantında tutulan değerin türünü temsil eden jenerik (generic) türdür. Aynı `T` türü, `unwrap_or_else` fonksiyonunun dönüş türüdür. Örneğin, `Option<String>` üzerinde `unwrap_or_else` çağırırsak, geri dönüş değeri `String` olacaktır.

Ayrıca `unwrap_or_else` fonksiyonunun `F` adında ikinci bir jenerik tür parametresine sahip olduğuna dikkat edin. Bu `F` türü, fonksiyona `f` parametresiyle verilen kapanışın türünü temsil eder.

`F` türüne uygulanan trait sınırı `FnOnce() -> T` şeklindedir. Bu, `F` türünden bir değer (yani kapanış) en az bir kez çağrılabilir olmalı, argüman almamalı ve bir `T` değeri döndürmelidir. Trait sınırında `FnOnce` kullanılması, `unwrap_or_else` fonksiyonunun `f`'yi en fazla bir kez çağıracağını belirtir. Yukarıdaki gövdede de görüldüğü gibi, `Option` değeri `Some` ise `f` hiç çağrılmaz. `None` ise `f` bir kez çağrılır. Tüm kapanışlar `FnOnce` trait’ini uyguladığından, `unwrap_or_else` fonksiyonu tüm kapanış türlerini kabul edebilir ve oldukça esnektir.

> Not: Eğer yapmak istediğimiz şey ortamdan herhangi bir değer yakalamayı gerektirmiyorsa, `Fn` trait’lerinden birini uygulayan bir kapanış yerine doğrudan bir fonksiyon ismi de kullanabiliriz. Örneğin, bir `Option<Vec<T>>` değeri üzerinde `unwrap_or_else(Vec::new)` çağırabiliriz; eğer değer `None` ise bu çağrı boş bir vektör döndürür. Derleyici, fonksiyon tanımına uygun olan `Fn` trait’ini otomatik olarak uygular.

Şimdi dilimlerde (`slice`) tanımlanmış olan standart kütüphane fonksiyonu `sort_by_key`’e bakalım. Bu fonksiyonun `unwrap_or_else`’den nasıl farklı çalıştığını ve neden `FnOnce` yerine `FnMut` trait sınırını kullandığını göreceğiz. Bu fonksiyonda kapanış, dilimdeki her öğe için bir referans alır ve karşılaştırılabilir (`Ord`) bir `K` türü döndürür. Bu, örneğin her öğenin belli bir niteliğine göre sıralama yapmak istediğimizde kullanışlıdır. Liste 13-7’de, bir dizi `Rectangle` örneğimiz var ve bunları `width` (genişlik) değerine göre küçükten büyüğe sıralıyoruz.

<Listing number="13-7" file-name="src/main.rs" caption="Dikdörtgenleri genişliğe göre sıralamak için `sort_by_key` kullanımı">
=======
Burada, `T`, `Option`'ın `Some` varyantındaki değerin türünü temsil eden genel türdür. Bu tür, aynı zamanda `unwrap_or_else` fonksiyonunun dönüş türüdür: örneğin, bir `Option<String>` üzerinde `unwrap_or_else` çağıran kod bir `String` alacaktır.

Sonra, `unwrap_or_else` fonksiyonunun ek bir genel tür parametresi `F` olduğunu görüyoruz. `F` türü, `unwrap_or_else` çağrılırken sağladığımız kapanışın türüdür.

Genel tür `F` üzerindeki trait sınırı `FnOnce() -> T`'dir, yani `F` bir kez çağrılabilmeli, argüman almamalı ve bir `T` döndürmelidir. Trait sınırında `FnOnce` kullanmak, `unwrap_or_else`'in kapanışı en fazla bir kez çağıracağını belirtir. `unwrap_or_else`'in gövdesinde, eğer `Option` `Some` ise, `f` çağrılmaz. Eğer `Option` `None` ise, `f` bir kez çağrılır. Tüm kapanışlar `FnOnce`'ı uyguladığı için, `unwrap_or_else` üç tür kapanışı da kabul eder ve olabildiğince esnektir.

> Not: Eğer yapmak istediğimiz şey ortamdan bir değer yakalamayı gerektirmiyorsa, bir fonksiyonun adını, bir kapanış yerine, `Fn` trait'lerinden birini uygulayan bir yerde kullanabiliriz. Örneğin, bir `Option<Vec<T>>` değeri üzerinde, değer `None` ise yeni, boş bir vektör almak için `unwrap_or_else(Vec::new)` çağırabiliriz. Derleyici, fonksiyon tanımı için uygun olan `Fn` trait'ini otomatik olarak uygular.

Şimdi, dilimlerde tanımlı olan ve standart kütüphanede bulunan `sort_by_key` metoduna bakalım; bu metodun neden `FnOnce` yerine `FnMut` trait sınırı kullandığını görelim. Kapanış, dilimdeki mevcut öğeye bir referans olarak bir argüman alır ve sıralanabilir bir türde bir değer döndürür. Bu fonksiyon, bir dilimi her bir öğenin belirli bir özelliğine göre sıralamak istediğinizde kullanışlıdır. Liste 13-7'de, bir dizi `Rectangle` örneğimiz var ve bunları `width` alanına göre küçükten büyüğe sıralamak için `sort_by_key` kullanıyoruz.

<Listing number="13-7" file-name="src/main.rs" caption="Dikdörtgenleri genişliğe göre sıralamak için `sort_by_key` kullanmak">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

<<<<<<< HEAD
Bu kodun çıktısı şudur:
=======
Bu kod şunu yazdırır:
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

<<<<<<< HEAD
`sort_by_key` fonksiyonunun bir `FnMut` kapanışı alacak şekilde tanımlanmasının nedeni, kapanışın dilimdeki her öğe için bir kez çağrılacak olmasıdır. Bu örnekte kullanılan kapanış `|r| r.width` ortamdan hiçbir değer yakalamaz, değiştirmez veya dışarı taşımaz. Bu nedenle `FnMut` trait sınırını karşılamaktadır.

Buna karşılık, Liste 13-8’de gösterilen örnekte, yalnızca `FnOnce` trait’ini uygulayan bir kapanış yer alır çünkü bu kapanış ortamdan bir değeri dışarı taşır. Derleyici, bu kapanışın `sort_by_key` ile kullanılmasına izin vermez.


<Listing number="13-8" file-name="src/main.rs" caption="`FnOnce` kapanışını `sort_by_key` ile kullanmaya çalışmak">
=======
`sort_by_key`'in bir `FnMut` kapanış alacak şekilde tanımlanmasının nedeni, kapanışı dilimdeki her bir öğe için birden fazla kez çağırmasıdır. `|r| r.width` kapanışı ortamdan hiçbir şey yakalamaz, değiştirmez veya dışarı taşımaz, bu nedenle trait sınırı gereksinimlerini karşılar.

Buna karşılık, Liste 13-8'de yalnızca `FnOnce` trait'ini uygulayan bir kapanış örneği gösterilmiştir, çünkü ortamdan bir değeri dışarı taşır. Derleyici, bu kapanışı `sort_by_key` ile kullanmamıza izin vermez.

<Listing number="13-8" file-name="src/main.rs" caption="`sort_by_key` ile bir `FnOnce` kapanış kullanmaya çalışmak">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

<<<<<<< HEAD
Bu, `sort_by_key` fonksiyonunun kapanışı kaç kez çağırdığını saymaya çalışmanın dolaylı ve karmaşık (ve çalışmayan) bir yoludur. Bu kod, kapanışın ortamından gelen bir `String` olan `value` değerini `sort_operations` vektörüne ekleyerek sayma işlemini gerçekleştirmeyi dener. Kapanış, `value` değerini yakalar ve ardından `value`’yu ortamdan alarak sahipliğini `sort_operations` vektörüne devreder. Bu kapanış yalnızca bir kez çağrılabilir; ikinci kez çağrılmaya çalışıldığında `value` artık ortamda bulunmayacağı için tekrar `sort_operations` vektörüne eklenemez! Bu nedenle, bu kapanış yalnızca `FnOnce` trait’ini uygular. Bu kodu derlemeye çalıştığımızda, kapanışın `FnMut` olması gerektiği ancak `value`’nun ortamdan taşındığı için `move` edilemediğini belirten şu hatayı alırız:
=======
Bu, `sort_by_key` kapanışı sıralama sırasında kaç kez çağırdığını saymak için ortamdan bir `String` olan `value`'ı `sort_operations` vektörüne ekleyerek saymaya çalışan, yapay ve karmaşık bir örnektir (ve çalışmaz). Kapanış, ortamdan `value`'ı yakalar ve ardından sahipliğini `sort_operations` vektörüne aktararak dışarı taşır. Bu kapanış yalnızca bir kez çağrılabilir; ikinci kez çağrılmaya çalışılırsa, `value` ortamda artık bulunmayacağı için çalışmaz! Bu nedenle, bu kapanış yalnızca `FnOnce`'ı uygular. Bu kodu derlemeye çalıştığımızda, kapanışın ortamdan bir değeri dışarı taşıyamayacağına dair bir hata alırız çünkü kapanışın `FnMut` uygulaması gerekir:
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

<<<<<<< HEAD
Bu hata, kapanış gövdesinde `value`’nun ortamdan taşındığı satırı işaret eder. Bu hatayı düzeltmek için kapanış gövdesini, ortamdan değerleri taşımayacak şekilde değiştirmemiz gerekir. Ortamda bir sayaç tutmak ve kapanış içinde bu değeri arttırmak, kapanışın kaç kez çağrıldığını saymak için daha doğrudan bir yaklaşımdır. Liste 13-9’daki kapanış, yalnızca `num_sort_operations` sayaç değişkenine mutlak bir referans yakaladığı için `sort_by_key` ile çalışır ve birden fazla kez çağrılabilir:

<Listing number="13-9" file-name="src/main.rs" caption="`sort_by_key` ile `FnMut` kapanışı kullanmak mümkündür">
=======
Hata, kapanış gövdesinde ortamdan bir değerin dışarı taşındığı satırı işaret eder. Bunu düzeltmek için, kapanış gövdesini ortamdan değer taşımayacak şekilde değiştirmemiz gerekir. Ortamda bir sayaç tutup, kapanış gövdesinde bu sayacı artırmak, kapanışın kaç kez çağrıldığını saymanın daha basit bir yoludur. Liste 13-9'daki kapanış, yalnızca sayaç olan `num_sort_operations`'a değiştirilebilir bir referans yakaladığı için `sort_by_key` ile çalışır ve birden fazla kez çağrılabilir:

<Listing number="13-9" file-name="src/main.rs" caption="`sort_by_key` ile bir `FnMut` kapanış kullanmak mümkündür">
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

<<<<<<< HEAD
`Fn` trait’leri, kapanışları kullanan veya tanımlayan fonksiyon ya da türlerde oldukça önemlidir. Bir sonraki bölümde iteratörleri ele alacağız. Birçok iteratör metodu, parametre olarak kapanış alır; dolayısıyla bu kapanış detaylarını aklınızda tutmanız faydalı olacaktır!
=======
`Fn` trait'leri, kapanışları kullanan fonksiyonları veya türleri tanımlarken veya kullanırken önemlidir. Sonraki bölümde yineleyicileri tartışacağız. Birçok yineleyici metodu kapanış argümanları alır, bu nedenle bu kapanış ayrıntılarını aklınızda bulundurun!
>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
