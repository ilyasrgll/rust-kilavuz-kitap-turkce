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
