<<<<<<< HEAD
# Fonksiyonel Dil Özellikleri: İteratörler ve Kapanışlar

Rust’un tasarımı, birçok mevcut programlama dili ve teknikten ilham almıştır. Bu etkilerden biri de _fonksiyonel programlama_dır. Fonksiyonel tarzda programlama genellikle şunları içerir:

- Fonksiyonları değer gibi kullanmak,
- Fonksiyonları başka fonksiyonlara argüman olarak vermek,
- Fonksiyonlardan geri döndürmek,
- İleride çalıştırmak üzere değişkenlere atamak,
- ve benzeri işlemler.

Bu bölümde fonksiyonel programlamanın ne olup ne olmadığı tartışılmayacak; bunun yerine, genellikle fonksiyonel dillerle ilişkilendirilen bazı Rust özelliklerine odaklanacağız.

Daha özel olarak şunları ele alacağız:

- _Kapanışlar_, bir değişkene atanabilen fonksiyon benzeri yapılardır
- _İteratörler_, bir dizi öğeyi işlemeye yarayan yapılardır
- Kapanışlar ve iteratörleri kullanarak 12. bölümdeki G/Ç projesini nasıl geliştirebiliriz
- Kapanışlar ve iteratörlerin performansı (sürpriz: düşündüğünüzden hızlılar!)

Daha önce pattern matching (desen eşleme) ve enum’lar gibi Rust’un fonksiyonel tarzdan etkilenen bazı özelliklerini gördük. Kapanışları ve iteratörleri iyi öğrenmek, hızlı ve idiomatik Rust kodu yazmak için önemlidir. Bu bölümü tamamen bu konulara ayıracağız.
=======
# Fonksiyonel Dil Özellikleri: Yineleyiciler ve Kapanışlar

Rust'ın tasarımı, birçok mevcut dilden ve teknikten ilham almıştır ve önemli bir etkisi de _fonksiyonel programlama_ olmuştur. Fonksiyonel bir tarzda programlama genellikle fonksiyonları değer olarak kullanmayı, onları argüman olarak geçirmeyi, başka fonksiyonlardan döndürmeyi, daha sonra çalıştırmak üzere değişkenlere atamayı ve benzeri işlemleri içerir.

Bu bölümde, fonksiyonel programlamanın ne olup ne olmadığı tartışmasına girmeyeceğiz; bunun yerine, Rust'ta fonksiyonel olarak adlandırılan birçok dilde bulunan benzer özellikleri ele alacağız.

Daha spesifik olarak şunları ele alacağız:

- Bir değişkende saklayabileceğiniz fonksiyon benzeri bir yapı olan _kapanışlar_
- Bir dizi öğeyi işlemenin bir yolu olan _yineleyiciler_
- Kapanışları ve yineleyicileri 12. Bölümdeki G/Ç projesini geliştirmek için nasıl kullanacağımızı
- Kapanışların ve yineleyicilerin performansı (ipucu: düşündüğünüzden daha hızlılar!)

Daha önce, fonksiyonel tarzdan etkilenen desen eşleştirme ve enumlar gibi bazı Rust özelliklerini de ele aldık. Kapanışlar ve yineleyicilerde ustalaşmak, idiomatik ve hızlı Rust kodu yazmanın önemli bir parçası olduğundan, bu bölümün tamamını onlara ayıracağız.

>>>>>>> 8a72a61ab1ba8eaf2752cc86ae0e98b6b0c03d9e
