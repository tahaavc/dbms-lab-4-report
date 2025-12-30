# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?

Aşağıda kutucuk (checkbox) ile gösterilen maddelerden en az birini seçtiğiniz açık kaynak kodlu bir VT kaynak kodları üzerinde göstererek açıklayınız. Açıklama bölümüne kısaca metninizi yazıp, kod üzerinde gösterim videonuzun linkini en altta belirtilen kutucuğa yerleştiriniz.

- [X]  Seçtiğiniz konu/konuları bu şekilde işaretleyiniz. **!**
    
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [X]  **Blok bazlı disk erişimi** → block_id + offset
- [ ]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [X]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [X]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [X]  LRU / CLOCK gibi algoritmaları
- [X]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [X]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [ ]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [ ]  WAL (Write Ahead Log) İlkesi
- [ ]  Log disk (fsync vs write) sistem çağrıları farkı

---

# Özet Tablo

| Kavram      | Bellek          | Disk / DB      |
| ----------- | --------------- | -------------- |
| Adresleme   | Pointer         | Page + Offset  |
| Hız         | O(1)            | Page IO        |
| PK          | Yok             | Index anahtarı |
| Veri yapısı | Array / Pointer | B+Tree         |
| Cache       | CPU cache       | Buffer Pool    |

Bu tablo, bellek tabanlı sistemler ile disk tabanlı veritabanı sistemleri arasındaki
temel farkları ve performans açısından kullanılan veri yapıları ile önbellekleme
yaklaşımlarını özetlemektedir.

# Video [Linki](https://youtu.be/v-0owcyHP4s) 

# Açıklama
Veritabanı sistemlerinde performans, büyük ölçüde disk erişim maliyetlerinin nasıl yönetildiğine ve verinin disk üzerinde hangi yapılarla organize edildiğine bağlıdır. Disk tabanlı depolama birimleri, bellek erişimine kıyasla oldukça yavaş çalıştığından, her sorguda diske erişilmesi sistem performansını ciddi ölçüde düşürmektedir. Bu nedenle modern veritabanı yönetim sistemleri, disk erişimlerini minimize etmeye yönelik çeşitli mekanizmalar geliştirmiştir. Bu çalışmada, PostgreSQL açık kaynak veritabanı sisteminin kaynak kodları incelenerek performansı doğrudan etkileyen iki temel yapı ele alınmıştır: Buffer Pool mekanizması ve B+ Tree veri yapısı.
Buffer Pool, veritabanlarında sık kullanılan veri sayfalarının ana bellek (RAM) üzerinde tutulmasını sağlayan bir önbellekleme (caching) mekanizmasıdır. Veriler diskten satır bazında değil, sayfa (page) bazında okunur. PostgreSQL’de varsayılan sayfa boyutu 8 KB olup, bir sorgu çalıştırıldığında ilgili veri sayfası diskten okunarak Buffer Pool içerisine alınır. Aynı sayfaya daha sonra tekrar ihtiyaç duyulması durumunda, disk erişimi yapılmaksızın doğrudan bellekteki kopya kullanılır. Bu yaklaşım, disk I/O sayısını azaltarak sorgu sürelerinin kısalmasını ve sistemin genel verimliliğinin artmasını sağlar.
Buffer Pool mekanizmasının etkin bir şekilde çalışabilmesi için bellek yönetimi büyük önem taşır. Ana bellek sınırlı bir kaynak olduğu için tüm veri sayfalarının sürekli olarak RAM’de tutulması mümkün değildir. Bu nedenle veritabanı sistemleri, hangi sayfaların bellekte kalacağına ve hangilerinin bellekten çıkarılacağına karar veren sayfa değiştirme algoritmaları kullanır. PostgreSQL kaynak kodlarında incelenen StrategyGetBuffer fonksiyonu, bu karar sürecinin uygulandığı temel noktalardan biridir. Bu fonksiyon, CLOCK ve LRU benzeri algoritmalar yardımıyla uzun süredir kullanılmayan veya erişim sıklığı düşük olan sayfaların bellekten çıkarılmasını sağlar. Böylece Buffer Pool daha verimli kullanılır ve gereksiz disk erişimlerinin önüne geçilir.
Diskten veri okuma sürecinin başlatılması ise StartReadBuffer gibi fonksiyonlar aracılığıyla gerçekleştirilmektedir. Bu fonksiyonlar, ilgili veri sayfasının diskten okunarak Buffer Pool’a alınması sürecini başlatır. Bu aşama, verinin doğrudan bellekte erişilebilir hale gelmesini sağlayan sürecin ilk adımıdır ve Buffer Pool mekanizmasının disk I/O’yu azaltmadaki rolünü açıkça göstermektedir.
Ancak veritabanı performansı yalnızca belleğe alınan veri miktarıyla sınırlı değildir. Verinin disk üzerinde nasıl organize edildiği de performans açısından kritik bir öneme sahiptir. Bu noktada B+ Tree veri yapıları devreye girmektedir. B+ Tree, disk tabanlı sistemler için özel olarak tasarlanmış dengeli bir ağaç veri yapısıdır ve veritabanı indeksleme mekanizmalarında yaygın olarak kullanılmaktadır. Bu yapı sayesinde veriler sıralı bir şekilde tutulur ve arama, ekleme ve silme işlemleri logaritmik zaman karmaşıklığında gerçekleştirilebilir.
PostgreSQL kaynak kodlarında yer alan btinsert fonksiyonu, B+ Tree indeks yapısına yeni kayıtların eklenmesini sağlayan temel fonksiyonlardan biridir. Bir tabloya yeni bir kayıt eklendiğinde, ilgili indeks yapısı bu fonksiyon aracılığıyla güncellenir. Böylece indeks yapısı her zaman tutarlı ve güncel kalır. B+ Tree’nin bu yapısı sayesinde veritabanı, çok büyük veri kümeleri üzerinde bile sınırlı sayıda disk erişimi ile arama işlemlerini gerçekleştirebilir.
Sonuç olarak Buffer Pool ve B+ Tree mekanizmaları, modern veritabanı sistemlerinde performansın temel bileşenleri arasında yer almaktadır. Buffer Pool, sık kullanılan veri sayfalarını bellekte tutarak disk I/O maliyetini azaltırken; B+ Tree veri yapısı, verinin disk üzerinde verimli ve erişilebilir bir biçimde organize edilmesini sağlar. PostgreSQL kaynak kodları üzerinde yapılan inceleme, bu iki yapının birlikte çalışarak veritabanı sistemlerinin yüksek performanslı ve ölçeklenebilir olmasına doğrudan katkı sağladığını açıkça ortaya koymaktadır.

## VT Üzerinde Gösterilen Kaynak Kodları

(PostgreSQL Buffer Pool – diskten veri sayfasının okunması ve Buffer Pool’a alınması, ReadBuffer) [Linki](https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/bufmgr.c)  


(Buffer Pool strateji yönetimi – LRU / CLOCK benzeri algoritmalar, StrategyGetBuffer) [Linki](https://github.com/postgres/postgres/blob/master/src/backend/storage/buffer/freelist.c)  


(B+ Tree indeks yapısı – indeks ekleme işlemleri, btinsert) [Linki](https://github.com/postgres/postgres/blob/master/src/backend/access/nbtree/nbtree.c)  
