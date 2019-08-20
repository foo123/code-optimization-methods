## Kod Optimizasyon Yöntemleri

*Çeşitli kod optimizasyon yöntemlerinden oluşan derleme*

### Çeviriler

* [Türkçe](/tr.md) ([@umutphp](https://github.com/umutphp) tarafından)


### İçindekiler

- [Genel Prensipler](#general-principles)
- [Alt-seviye](#low-level)
- [Dile bağımlı optimizasyon](#language-dependent-optimization)
- [Dile bağımlı optimizasyon](#language-independent-optimization)
- [Veritabanları](#databases)
- [Web](#web)
- [Referanslar](#references)

**Algoritmaların Hesaplama Karmaşıklığı Teorisi ve Kod Optimizasyon Teknikleri arasındaki ilişkiye dair bir not**

Hesaplama Karmaşıklığı Teorisi <sup> <a href="#r2" title="Computational complexity theory, wikipedia">2</a> </sup> ve kod optimizasyon teknikleri <sup> <a href="#r1" title="Code optimization, wikipedia">1</a> </sup> ortak bir amaç olarak verimli problem çözmeyi hedeflerler. Birbirileri ile ortak paydaları olmasına ve aynı bağlamı paylaşmalarına rağmen, aralarındaki fark vurgu yaptıkları konulardadır.

Hesaplama karmaşıklığı teorisi performansı girdi boyutu göre araştırır.  Alt yatan mimariden bağımsız olarak girdi boyutuna en az bağımlı ve en hızlı algoritmik çözümü tasarlamaya çalışır. Kod optimizasyon teknikleri ise mimariye ve hesaplama karmaşıklığı teorisinin tahminlemede kullandığı belli sabitlere odaklanır.

Gerçek zamanlı çalışan sistemlerde bu ikisi de kritik faktör olabilir (Örneğin, [JavaScript ile Gerçek Zamanlı Resim İşleme Processing](https://github.com/foo123/FILTER.js) kütüphanesi, evet ben yazdım :) ).

### Genel Prensipler

- **`DRY` Prensibi ve Önbellek** : Genel önbellekleme kavramı, gerekli değilse bir sonucun yeniden hesaplanmasından / yeniden yüklenmesinden kaçınmayı içerir. Bu DRY prensibinin <sup> <a href="#r3" title="DRY principle, wikipedia">3</a> </sup> farklı bir çeşidi olarak da görülebilir. Ara sonuçları saklayıp tekrar hesaplama zaman ve kaynak tasarrufu sağlandığından dinamik programlama da önbelleklemenin bir çeşidi gibi görülebilir.

- **`KISS`, sade olan daha hızlı olabilir**  :  'Keep it simple' <sup> <a href="#r4" title="KISS principle, wikipedia">4</a> </sup> prensibi, bir çok tekniği uygulanabilir ve değiştirilebilir hale getirir. ( Bu konuda yardımcı olabilecek çok fazla yazılım mühendisliği tekniği vardır <sup> <a href="#r34" title="Software development philosophies, wikipedia">34</a>, <a href="#r35" title="97 Things every programmer should know">35</a>, <a href="#r55" title="Stream processing, wikipedia">55</a>, <a href="#r56" title="Dataflow programming, wikipedia">56</a> </sup>  )

- **Tek rardüzen lersenda hakolayola bilir, tekrar düzenlersen daha kolay olabilir** : İhtiyaç duyduğunuz an yeniden düzenleme yapmaktan çekinmeyin. Çoğu zaman bir şey var olan özellikleri korunarak daha hızlı ve basit bir yapıya sahip olacak şekilde tekrar düzenlenebilir ve yeniden yapılandırılabilir  (denkşekillilik kavramı <sup> <a href="#r22" title="Isomorphism, wikipedia">22</a> </sup> , temsil değişikliği <sup> <a href="#r23" title="Representation, wikipedia">23</a>, <a href="#r24" title="Symmetry, wikipedia">24</a>, <a href="#r36" title="Data structure, wikipedia">36</a> </sup>), ve bu durum beraberinde başka faydalar da getirir. Örneğin, the bu sayısal ifade `(10+5*2)^2` aslında `400` sabitine eşittir, bir başka örnek ise ` infix ` ifade notasyonundan daha hızlı ayrıştırılabilen ` prefix ` (Polish) notasyonuna olan dönüşümdür.

- **Alt problemlere Böl ve çözümü Fethet** : Alt problemler (küçük, önemsiz ve özel parçalar) daha kolayca ve hızlıca çözülerek büyük çözüme ulaşılabilir. Sıralama algoritmaları buna en güzel örneklerdir <sup> <a href="#r5" title="Divide and conquer algorithm, wikipedia">5</a>, <a href="https://github.com/foo123/SortingAlgorithms">sıralama algoritmaları</a> </sup>.

- **Yardım istemekten çekinme** : Mümkünse iş yükünü dağıtın, daha küçük parçalara bölüm, paylaşın ve paralel olarak yapılabilmesini sağlayın <sup> <a href="#r6" title="Parallel computation, wikipedia">6</a>, <a href="#r54" title="Heterogeneous computing, wikipedia">54</a>, <a href="#r68" title="A Practical Wait-Free Simulation for Lock-Free Data Structures">68</a>, <a href="#r69" title="A Highly-Efficient Wait-Free Universal Construction">69</a>, <a href="#r70" title="A Methodology for Creating Fast Wait-Free Data Structures">70</a> </sup> .

- **United we Stand and Deliver** : Veriyi farklı kaynaklarda dağınık halde değil de tek ve bitişik bir kaynaktan alma, yüklemeyi ve küçük parçalar halinde okumak yerine tek parça halinde işlemeyi hızlı hale getirir (ör. önbellek, vector/pipeline, veritabanı sorguları) <sup> <a href="#r51" title="Locality of reference, wikipedia">51</a>, <a href="#r52" title="Memory access pattern, wikipedia">52</a>, <a href="#r53" title="Memory hierarchy, wikipedia">53</a> </sup>.

- **Biraz tembellik etmek çok da kötü bir şey değildir** : Kesinlikle söyleyebiliriz ki, bir program her çalıştırıldığında, verilerinin ve işlevlerinin yalnızca bir kısmı kullanılır. Sadece gerektiğinde bütün veriyi ve özellikleri yükleme ya da çalıştırma (tembel olma) daha uzun bir yol almamızı sağlayabilir <sup> <a href="#r7" title="Lazy load, wikipedia">7</a> </sup> .

**Ek Notlar**

Optimizasyona başlamadan önce neyin optimizasyona ihtiyaç duyduğunu belirleyip hesaplamak gerekir. Hiç optimizasyon yapmamak kör "optimizasyondan" çoğunlukla iyidir.

Bununla beraber, kişi daima verimli çözümler üretmeye ve optimize etmeye çalışmalı. Verimli olmayan "çözümler" en fazla çözümsüz durum kadar iyi olabilir, o da tabi kötü değillerse.

**Eksik optimizasyon ancak eksik bilgi ile geçerli görülebilir**. Örneğin, bütün bir `class` yada `array` yüklemek aynı bilgi ile  sadece bir `integer` dönmekten daha yavaştır.

Bazı optimizasyon teknikleri otomatikleştiriebilir (örneğin derleyicilerde) ama çoğunluğun elle yapılması gerekir.

Bazen hafıza/zaman kaynakları arasında bir takas dengesi vardır. Hızı artırmak hafıza/alan ihtiyacını artırabilir (**caching** buna en güzel örnektir).

`90-10` (`80-20` veya diğer varyasyonlar) kuralına göre diyebiliriz ki kodun çalışması için harcanan zamanın **`90%`** kısmı yazılan kodun **`10%`** kısmı (örneğin bir döngü) tarafından kullanılır. Bu kısımda yapılan bir optimizasyonun etkisi çok büyük olacaktır. (Örnek olarak Knuth'a bakabilirsiniz <sup> <a href="#r8" title="An empirical study of Fortran programs">8</a> </sup> )

Bir optimizasyon yöntemi (örneğin sadeleştirme) diğer bir yöntemin (örneğin sabit değiştirme) uygulanmasına yol açabilir ve bu da bir zincir şeklinde birinci yöntemin (ya da başka bir yöntemlerin) uygulanmasına yol açabilir. Kapı kapıyı açar.

**Referanslar:** [9](#r9 "Compiler optimizations, wikipedia"), [11](#r11 "Compiler Design Theory"), [12](#r12 "The art of compiler design - Theory and Practice"), [46](#r46 "Optimisation techniques"), [47](#r47 "Notes on C Optimisation"), [48](#r48 "Optimising C++"), [49](#r49 "Programming Optimization"), [50](#r50 "CODE OPTIMIZATION - USER TECHNIQUES")

### Alt-seviye

#### Genel Konular<sup> <a href="#r44" title="What Every Programmer Should Know About Floating-Point Arithmetic">44</a>, <a href="#r45" title="What Every Computer Scientist Should Know About Floating-Point Arithmetic">45</a> </sup>

**Veri Kaydetme**

- Disk erişimi yavaştır (Ağ üzerinden erişim daha da yavaştır)
- Hafızaya (RAM) erişim diske erişimden daha hızlıdır
- CPU Cache Hafızası (eğer varsa) ana hafızadan daha hızlıdır
- CPU kayıtları en hızlısıdır

**İkili (Binary) Formatlar**

- Double kesirli sayı aritmetiği yavaştır
- Float kesirli sayı aritmetiği double kesirli sayı aritmetiğinden daha hızlıdır
- Büyük tamsayı (long integer) aritmetiği float kesirli sayı aritmetiğinden daha hızlıdır
- Küçük tamsayı (short integer), sabit noktalı aritmetik büyük tamsayı aritmetiğinden daha hızlıdır
- Bit temelli aritmetik en hızlısıdır

**Aritmetik İşlemler**

- Üslü işlemler yavaştır
- Bölme işlemi üslü işlemlerden daha hızlıdır
- Çarpma işlemi bölme işleminden daha hızlıdır
- Toplama ve çıkarma işlemleri çarpma işleminden daha hızlıdırlar
- Bit işlemleri en hızlısıdır

#### Metodlar

- **CPU veri yuvası kullanımı** : Veri yuvası hafızası çok kullanılan verilere ulaşmak için en hızlı erişim yöntemi olduğu için optimum bir mantıkla yüklü işlemler sırasında veri yuvalarında  veri saklamak uygun bir yoldur (örneğin derleyiciler, gerçek zamanlı sistemler). Bu tarz optimizasyona imkan veren farklı algoritmalar (gafik reklendirme problemini temel alan) vardır. Başka bir yöntem olarak da programcı yapılan işlemin bir bölümünde CPU veri yuvalarında saklanan değişkenleri tanımlayabilir <sup> <a href="#r10" title="Register allocation, wikipedia">10</a> </sup>

- **Tekil Atom Optimizasyonu** : Bu atom olarak adlandırılan bir tek CPU işleminin optimizasyonun içeren farklı işlemleri kapsıyor. Örneğin bir atomun içindeki bazı işlenenler değişken kullanmak yerine sabit sayılara çevirilebilir. Başka bir örnek ise bir sayının karesini hesaplarken üslü sayı (code1}2) kullanmak yerine çarpma işlemi kullanmaktır, vs

- **Atom Grupları Optimizasyonu** : Bir önceki başlığa benzer olarak, bu optimizasyon türünde atom ve atom gruplarının kontrol akışı incelenir, işlemlerin amacı korunarak tekrar düzenlenmesi ve daha az komut kullanılması sağlanmaya çalışılır. Örneğin, karmaşık bir `IF THEN` akışı incelenerek basit bir `Jump` cümlesine çevrilebilir, vs.

### Dil tabanlı optimizasyon

- Dilin sunduğu özellikleri sağlamak için kullandığı alta yatan mekanizmaları anlamak için **resmi belgelerini ve el kitabını** dikkatlice okumak ve kod yazarken ya da alternatifler üretirken bunu kullanmak lazım.

### Dile bağımlı optimizasyon

- **İfadelerin yeniden düzenlenmesi** : Bir ifadenin (veya bir işlemin hesaplanmasının) hesaplanması için daha verimli kod, ifadede gerçekleşen işlemleri farklı bir sırayla işleyerek  üretilebilir. Bunun nedeni, ifadelerin / işlemlerin yeniden düzenlenmesi, eklenen ve çarpma sayısı ve dolayısıyla her bir işlemin (toplam) göreceli (hesaplamalı) maliyetleri de dahil olmak üzere, neyin eklenip neyin çarpılarak değiştirileceğidir. Aslında, bu aritmetik işlemlerle sınırlı değildir, fakat simetrileri kullanan herhangi bir işlem (örn. Değişmeli yasalar, birleşme yasaları ve dağıtım yasaları, gerçekte ellerinde bulundukları zaman, gerçekte aritmetik operatör simetrilerinin örnekleridir) ve işlemcilerin yeniden düzenlenmesidir ve aynı zamanda diğer avantajlara sahiptir. Bu durum karmaşık cümlelerin aksine aslında çok basittir. Daha iyi anlamak için bu örneklere bakabilirsiniz; Horner Yasası <sup> <a href="#r13" title="Horner rule, wikipedia">13</a> </sup>, Karatsuba Çarpımı <sup> <a href="#r14" title="Karatsuba algorithm, wikipedia">14</a> </sup>, hızlı karmaşık çarpım <sup> <a href="#r15" title="Fast multiplication of complex numbers">15</a> </sup>, hızlı matris çarpımı <sup> <a href="#r18" title="Strassen algorithm, wikipedia">18</a></sup>, hızlı üs hesaplama <sup> <a href="#r16" title="Exponentiation by squaring, wikipedia">16</a>,  <a href="#r17" title="Fast Exponentiation">17</a> </sup>, hızlı gcd işlemi <sup> <a href="#r78" title="A Binary Recursive Gcd Algorithm">78</a> </sup>, hızlı faktöriyel/binom açılımı <sup> <a href="#r20" title="Comments on Factorial Programs">20</a></sup> , hızlı fourier çevirimi <sup> <a href="#r57" title="Fast Fourier transform, wikipedia">57</a> </sup>, hızlı fibonacci sayısı hesaplama <sup> <a href="#r76" title="Fast Fibonacci numbers">76</a> </sup>,  birleştirerek sıralama <sup> <a href="#r25" title="Merge sort, wikipedia">25</a> </sup>,  üsleri kullanarak sıralama <sup> <a href="#r26" title="Radix sort, wikipedia">26</a> </sup>.

- **Sabit Yer Değiştirme/Üretme** : Çoğu zaman bir ifade tüm durumlarda tek bir sabit olarak hesaplanıyor olabilir, daha karmaşık ve daha yavaş ifade yerine sabit değer ile yer değiştirebilir (bazen derleyiciler bunu yapar).

- ** Fonksiyon/Rutin Satır İçini Çağırma** : Bir fonksiyonu ya da rutini çağırma program durumunu yığının bir başka bölümne taşıma ve geri getirme gibi cpu'nun yapması gereken birçok işlem gerektirir. Bu yüklü işlemler yapılırken oldukça yavaş olabilir ve fonksiyonların gövdelerini kullanmak daha hızlı olabilir. Bazen bunu derleyiciler yapar, bazen de bunu bir programcı fonksiyon ya da rutin gövdelerini direk (`inline` ) kullanarak yapabilir. <sup> <a href="#r27" title="Function inlining, wikipedia">27</a> </sup>

- **Akış Geçişlerini Birleştirme** :  `IF/THEN` talimatları ve mantığı, özünde cpu ` dalı ` talimatlarıdır. CPU dal talimatları, program `işaretçisinin` değiştirilmesini ve yeni bir yere gitmesini içerir. Çok fazla `jump` talimatı içerirmesi durumunda bu çok yavaş olabilir.Bununla birlikte, `IF/THEN` ifadelerinin yeniden düzenlenmesi (ortak kodu çarpanlara ayırmak, De Morgan'ın mantık sadeleştirme kurallarını kullanarak vb.) verimli ve daha az mantıksal karmaşıklık içeren daha *izomorfik* işlevlere sebep olur ve sonuç olarak daha az ve daha verimli `dalı` talimatları oluşur.

- **Ölü Kod Eleme** : Çoğu zaman derleyiciler, asla erişilmeyen bir kodu belirleyebilir ve derlenmiş programdan kaldırabilir. Ancak tüm vakalar tespit edilememektedir. Önceki sadeleştirme şemalarını kullanarak, programcı "ölü kodu" (asla erişilmez) daha kolay belirleyebilir ve kaldırabilir. "Ölü kod elemesine" alternatif bir yaklaşım "canlı kod ekleme" veya "ağaç sallama" teknikleridir.

- **Ortak Alt İfadeler** : Bu optimizasyon, kodun çeşitli bölümlerinde ortak olan alt ifadelerin tanımlanmasını, yalnızca bir kez değerlendirilmesini ve değerinin sonraki tüm yerlerde kullanmasını içerir (bazen derleyiciler bunu yapar).

- **Ortak Kodu Ayrıştırma** : Çoğu zaman aynı kod bloğu farklı dallarda bulunur, örneğin program bazı ortak işlevler ve ardından bazı parametrelere bağlı olarak başka bir şey yapmak zorundadır. Bu ortak kod dallardan ayrılabilir ve böylece gereksiz fazlalık, gecikme ve büyüklüğü ortadan kaldırabilir.

- **Mukavemet Azaltma** : Bu, bir işlemin (örneğin bir ifadenin) daha hızlı olan bir eşdeğerine dönüştürülmesini içerir. Buna en genel iki örnek `üslü işlem`'i `çarpma` ile ve `çarpma`'yı `toplama` ile (örneğin bir döngü içinde) değiştirmektir. Bu tekniği uygulama, daha basit ancak eşdeğer işlemlerin, daha karmaşık eşdeğerlerinden (genellikle yazılımda uygulanan) daha hızlı (genellikle donanımda uygulanır) işlemci çevrimi içermesi gerçeğinden kaynaklanan verimlilik artışı ile sonuçlanabilir <sup><a href="#r28" title="Strength reduction, wikipedia">28</a></sup>.

- **Küçük/Özel Durumları Ele Almak** : Bazen karmaşık bir hesaplamanın daha basit/sade işlem türleriyle hesaplanabilecek daha küçük ya da özel durumları olabilir(örneğin, `a^b` hesaplaması, daha basit bir yöntemle ` a,b=0,1,2` için özel durumları ele alınabilir). Önemsiz durumlar uygulamalarda bazı sıklıklarla ortaya çıkar, bu nedenle basitleştirilmiş özel durumlar kodda oldukça yararlı olabilir.  <sup> <a href="#r42" title="Three optimization tips for C">42</a>, <a href="#r43" title="Three optimization tips for C, slides">43</a> </sup> .

- **Matematiksel Teoremleri/İlişkileri Kullanma** : Bazen bir hesaplama herhangi bir matematik teoremini, çevrimini, simetrisini <sup> <a href="#r24" title="Symmetry, wikipedia">24</a> </sup> ya da bilgisini kullanarak doğru ve daha verimli bir şekilde yapılabilir (Örneğin, lineer denklem sistemlerini çözmek için Gauss metodu <sup> <a href="#r58" title="Gaussian elimination, wikipedia">58</a> </sup>, Euclid Algoritması <sup> <a href="#r71" title="Euclidean Algorithm">71</a> </sup>, ya da ikisi <sup> <a href="#r72" title="Gröbner basis">72</a> </sup>, Fast Fourier Çevirimi <sup> <a href="#r57" title="Fast Fourier transform, wikipedia">57</a> </sup>, Fermat'ın Küçük Teorisi <sup> <a href="#r59" title="Fermat's little theorem, wikipedia">59</a> </sup>,  Taylor-Mclaurin Serileri Açılımı, Trigonometrik Bilgiler <sup> <a href="#r60" title="Trigonometric identities, wikipedia">60</a> </sup>, Newton'un Metodu <sup> <a href="#r73" title="Newton's Method">73</a>,<a href="#r74" title="Fast Inverse Square Root">74</a> </sup>, v.s.<sup> <a href="#r75" title="Methods of computing square roots">75</a> </sup>).

- **Verimli Veri Yapılarını Kullanmak** : Veri yapıları, algoritmaların karşılığıdır (tüm kapama alanındaki), her verimli algoritma, belirli bir görev için ilişkili bir verimli veri yapısına ihtiyaç duyar. Birçok durumda uygun bir veri yapısı (gösterimi) kullanmak tüm farkı oluşturabilir (örneğin; veritabanı tasarımcıları ve arama motoru geliştiricileri bunu çok iyi bilir) <sup> <a href="#r36" title="Data structure, wikipedia">36</a>, <a href="#r37" title="List of data structures, wikipedia">37</a>, <a href="#r23" title="Representation, wikipedia">23</a>, <a href="#r62" title="Dancing links algorithm">62</a>, <a href="#r63" title="Data Interface + Algorithms = Efficient Programs">63</a>, <a href="#r64" title="Systems Should Automatically Specialize Code and Data">64</a>, <a href="#r65" title="New Paradigms in Data Structure Design: Word-Level Parallelism and Self-Adjustment">65</a>, <a href="#r68" title="A Practical Wait-Free Simulation for Lock-Free Data Structures">68</a>, <a href="#r69" title="A Highly-Efficient Wait-Free Universal Construction">69</a>, <a href="#r70" title="A Methodology for Creating Fast Wait-Free Data Structures">70</a>, <a href="#r77" title="Fast k-Nearest Neighbors (k-NN) algorithm">77</a> </sup>.

**Döngü Optimizasyonu**

Belki de en önemli kod optimizasyon teknikleri, döngü içeren tekniklerdir.

- ** Kod Hareketi / Döngü Değişmezleri ** : Bazen döngünün içindeki bir kod bölümü döngü indeksinden bağımsızdır, döngü dışına alınabilir ve sadece bir kere çalıştırılabilir (bu bir döngü değişmezidir). Böylelikle döngü daha az işlem yapar (bazen derleyiciler bunu yapar) 
    <sup> <a href="#r29" title="Loop invariant, wikipedia">29</a>, <a href="#r30" title="Loop-invariant code motion, wikipedia">30</a> </sup>

**örnek:**

```javascript

// bu döngü geliştirilebilir
for (i=0; i<1000; i++)
{
    invariant = 100*b[0]+15; // bu bir döngü değişmezidir çünkü döngü indeksine bağlı değildir
    a[i] = invariant+10*i;
}

// döngünün daha verimli hali
invariant = 100*b[0]+15; // şimdi döngü dışına alındı
for (i=0; i<1000; i++)
{
    a[i] = invariant+10*i; // döngü artık daha az işlem yapıyor
}

```

- **Döngü Birleştime** : Bazen 2 ve ya daha fazla döngü birleştirilerek tek bir döngü haline getirilebilir ve böylece yapılan kontrol ve atırma işlemlerinin sayısı azaltıralabilir.

**örnek:**

```javascript

// 2 loops here
for (i=0; i<1000; i++)
{
    a[i] = i;
}
for (i=0; i<1000; i++)
{
    b[i] = i+5;
}


// one fused loop here
for (i=0; i<1000; i++)
{
    a[i] = i;
    b[i] = i+5;
}

```

- **Kontrolu Dışarı Alma** : Bazen bir döngü, herhangi bir zamanda yalnızca birinin çalıştırılması sağlanarak iki veya daha fazla döngüye ayrılabilir.

**örnek:**

```javascript

// döngünün her turunda kontrol tekrar tekrar çalıştırılır
for (i=0; i<1000; i++)
{
    if (X>Y)  // her turda tekrar tekrar çalıştırılır
        a[i] = i;
    else
        b[i] = i+10;
}

// Kontrol dışarıya alınır ve sadece bir kere çalışır hale gelir
if (X>Y)  // sadece bir kere çalışacak
{
    for (i=0; i<1000; i++)
    {
        a[i] = i;
    }
}
else
{
    for (i=0; i<1000; i++)
    {
        b[i] = i+10;
    }
}

```

- **Array Doğrusallaştırma** : Döngüye sokulan çok boyutlu bir diziyi sanki (daha basiti olan) tek boyutlu bir dizi gibi işlemektir. Genelde çok boyutlu diziler (örneğin `2D` diziler için `NxM`) hafızada kaydedilirken bir doğrusallaştırma şeması kullanırlar. Aynı şema mantığı dizideki veriye ulaşırken sanki `tek` boyutlu bir diziymiş gibi kullanılabilir. Bu da iç içe döngüler kullanmak yerine tek döngü kullanmayı sağlar 
    <sup> <a href="#r31" title="Array linearisation, wikipedia">31</a>, <a href="#r32" title="Vectorization, wikipedia">32</a>, <a href="#r61" title="The NumPy array: a structure for efficient numerical computation">61</a> </sup>.

**örnek:**

```javascript

// iç içe döngü
// N = M = 20
// toplam boyut = NxM = 400
for (i=0; i<20; i+=1)
{
    for (j=0; j<20; j+=1)
    {
        // aslında a[i, j] gösterimi a[i + j*N] gösterimine benziyor
        // çoğunlukla tek boyutlu olan daha hızlıdır
        a[i, j] = 0;
    }
}

// dizi doğrusallaştırıldığında döngü teke düşer
for (i=0; i<400; i++)
    a[i] = 0;  // önceki döngüye eşit ama tek döngü
    
    
```

- **Döngü Azaltma** : Döngü azaltma iki (ya da daha fazla) döngü turunu teke düşürerek yapılan işlem sayısını azaltmaktır. Bu aslında kısmı döngü azaltmaktır, bütün halinde azaltma ise döngüyü tamamen ortadan kaldırıp bütün işlemi açıkta yapmaktır (işlem sayısının sabit olduğu küçük döngüler). Döngü azaltma, döngünün (ve sonuç olarak her döngü yinelemeyle ilişkili tüm ek yüklerin) daha az çalışmasına neden olur. İşlem hattına veya paralel hesaplamaya izin veren işlemcilerde, döngü açma işleminin ek bir faydası olabilir, önceki yineleme bitmeden beklemeden, hesaplanırken veya yüklenirken bir sonraki kontrolsüz yineleme başlayabilir. Dolayısıyle döngü hızı beklendiğinden daha çok artabilir 
    <sup> <a href="#r33" title="Loop unrolling, wikipedia">33</a> </sup>.

**örnek:**

```javascript

// olağan bir döngü
for (i=0; i<1000; i++)
{
    a[i] = b[i]*c[i];
}    
    
// kısmen azaltılmış döngü
for (i=0; i<1000; i+=2)
{
    a[i] = b[i]*c[i];
    // bir sonraki tur bu turda hesaplanarak döngü artışı ikiye katlanır
    a[i+1] = b[i+1]*c[i+1];
}

// bu durum ele alınırken hassas davranmak gerekir
// örneğin azaltılmaya çalışılan döngü sayısı azaltılacak döngü sayısının tam katı olmayabilir
// bu durumda fazla olan miktar ayrıca hesaplanabilir

```

### Veritabanları

#### Genel Konular

Veritabanı erişimi pahalı olabilir, bu da gerekli veri sayısını mümkün olduğunca az sayıda DB bağlantısı ve çağrı kullanarak almak daha iyi olur.

#### Yöntemler

- **Tembel Yüklemesi** : Gerekli olmadıkça, DB erişiminden kaçınmak, uygulama ömrü boyunca fazladan veriye ihtiyaç duyulmadığı veya istenmediği durumlarda sıklık olması koşuluyla verimli olabilir

- **Önbellek** : Kritik değilse ve hafif gecikmeli bir güncellemeye izin veriliyorsa, önceki sorgulanmış veri sonuçlarını aynı sorgu için yeniden kullanmak

- **Verimli Sorgu Kullanma** : İlişkisel DB'ler için en etkili sorgu, verilerin DB'de benzersiz bir şekilde endekslendiği bir dizin (veya bir dizi dizin) kullanmaktır <sup> <a href="#r66" title="10 tips for optimising Mysql queries">66</a>, <a href="#r67" title="Mysql Optimisation">67</a> </sup>.

- **Yükü Dağıtma** : Yükü tek bir yerine taşımak için daha fazla yardım eli (DB) ekleyin. Aslında bu, toplam yükü bölmek ve bağımsız olarak ele almak için birden fazla yere verilerin kopyalanması (artıklık oluşturma) anlamına gelir

### Web

- **Daha Küçük Veri Akışı** : İnternet üzerinden veri (ve genellikle bir ağ üzerinden veri) iletilmesi biraz zaman alır. Özellikle veri daha büyükse, bu nedenle sadece gerekli verileri ve hatta bunları kompakt bir biçimde iletmek en iyisidir. Bu, `JSON` yapısının, web’de rastgele verilerin kodlanması için `XML` yapısının yerini almasının bir nedenidir.

- **En Küçük Sayıda İstek Yapma** : Bu prensip, önceki prensibin bir benzeri olarak görülebilir. Bu, her isteğin sadece gerekli verileri kompakt bir biçimde iletmesi gerektiği değil aynı zamanda istek sayısının en aza indirilmesi gerektiği anlamına gelir. Bu `.css` dosyalarını tek bir `.css` dosyasına (resim dosyalarını da gömülü hale getirmek),  `.js`  dosyalarını da tek bir `.js` dosyasına çevirme gibi işlemleri içerebilir. Bu bazen büyük boyutta dosyalar oluşturabilir ama buna rağmen bir sonraki prensip ile birlikte daha hızı bir hizmet sağlanabilir.

- **Öbellek, önbellek ve daha fazla önbellek** : Bu bütün sayfaları, `.css` dosyalarını, `.js` dosyaları, resimleri vs her şeyi önbelleğe almak anlamına gelir. Sunucuda çnbellek oluşturma, itemcide önbellek oluşturma, ikisinin arasında önbellek oluşturma, kısaca heryerde önbellek kullanmak..

- **Yükü Dağıtma** : Web uygulamaları için, bu genellikle birden fazla konumdan (bulut üzerinden) yüklenebilecek (statik) dosyaları depolamak amacıyla bazı [bulut mimarilerinden](http://en.wikipedia.org/wiki/Cloud_computing) yararlanılarak uygulanır. Diğer yaklaşımlar arasında [yük dengeleme](http://en.wikipedia.org/wiki/Load_balancing_%28computing%29) (yalnızca statik dosyalar için değil, sunucular için de yük dağıtma) bulunur.

- **Uygulama kodunu daha hızlı/sade yapın** : Bu, genel olarak kod optimizasyonu ile ilgili önceki ilkelerden alınmıştır. Verimli uygulama kodu hem sunucu hem de kullanıcı kaynaklarını koruyabilir. Facebook'un `HipHop VM` yaratmasında bir sebep vardır..

- **Minimalizm bir sanattır** : Web sayfalarını birçok html, resim dosyalarından (birçok reklam alanı olmasını saymıyorum) oluşturmak en iyi tasarım değildir ve tabiki sayfa yüklenmesini daha yavaşlatır. Dolayısıyla minimal sayfalara sahip olmak ve işlemleri  `AJAX` ve `JSON` kullanarak (web 2.0 tam da bu demek) yapmak her defasında büyük sayfaları yeniden yüklemekten daha iyidir. Bu [kalıp motorlarının](http://en.wikipedia.org/wiki/Template_engine) and [MVC Çatılarının](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) yaratılmalarının bir sebebidir. Minimalizm için sanatsal boyutu kurban etmeye gerek yok çünkü [Minimalizm](http://en.wikipedia.org/wiki/Minimalism) **de** bir sanattır. 

[το λακωνίζειν εστί φιλοσοφείν (konuşma ve eylemlerde kısa, basit ve konu odaklı olmak, felsefe sanatıdır.)](https://en.wikipedia.org/wiki/Laconic_phrase)

[![Zen Circle](/zen-circle.jpg)](http://en.wikipedia.org/wiki/Ens%C5%8D)

### Kaynaklar

1. <a id="r1" href="http://en.wikipedia.org/wiki/Code_optimization">Code optimization, wikipedia</a>
2. <a id="r2" href="http://en.wikipedia.org/wiki/Computational_complexity_theory">Computational complexity theory, wikipedia</a>
3. <a id="r3" href="http://en.wikipedia.org/wiki/Don%27t_repeat_yourself">DRY principle, wikipedia</a>
4. <a id="r4" href="http://en.wikipedia.org/wiki/KISS_principle">KISS principle, wikipedia</a>
5. <a id="r5" href="http://en.wikipedia.org/wiki/Divide_and_conquer_algorithm">Divide and conquer algorithm, wikipedia</a>
6. <a id="r6" href="http://en.wikipedia.org/wiki/Parallel_computation">Parallel computation, wikipedia</a>
7. <a id="r7" href="http://en.wikipedia.org/wiki/Lazy_load">Lazy load, wikipedia</a>
8. <a id="r8" href="http://www.cs.tufts.edu/~nr/cs257/archive/don-knuth/empirical-fortran.pdf">An empirical study of Fortran programs</a>
9. <a id="r9" href="http://en.wikipedia.org/wiki/Category:Compiler_optimizations">Compiler optimizations, wikipedia</a>
10. <a id="r10" href="http://en.wikipedia.org/wiki/Register_allocation">Register allocation, wikipedia</a>
11. <a id="r11" href="https://www.amazon.com/Compiler-Design-Theory-Systems-programming/dp/0201144557">Compiler Design Theory</a>
12. <a id="r12" href="https://www.amazon.com/Art-Compiler-Design-Theory-Practice/dp/B00DJAX3KC">The art of compiler design - Theory and Practice</a>
13. <a id="r13" href="http://en.wikipedia.org/wiki/Horner_rule">Horner rule, wikipedia</a>
14. <a id="r14" href="http://en.wikipedia.org/wiki/Karatsuba_algorithm">Karatsuba algorithm, wikipedia</a>
15. <a id="r15" href="http://www.embedded.com/design/real-time-and-performance/4007256/Digital-Signal-Processing-Tricks--Fast-multiplication-of-complex-numbers">Fast multiplication of complex numbers</a>
16. <a id="r16" href="http://en.wikipedia.org/wiki/Exponentiation_by_squaring">Exponentiation by squaring, wikipedia</a>
17. <a id="r17" href="http://homepages.math.uic.edu/~leon/cs-mcs401-f07/handouts/fastexp.pdf">Fast Exponentiation</a>
18. <a id="r18" href="http://en.wikipedia.org/wiki/Strassen_algorithm">Strassen algorithm, wikipedia</a>
19. <a id="r19" href="http://en.wikipedia.org/wiki/Coppersmith%E2%80%93Winograd_algorithm">Coppersmith-Winograd algorithm, wikipedia</a>
20. <a id="r20" href="http://www.cs.berkeley.edu/~fateman/papers/factorial.pdf">Comments on Factorial Programs</a>
21. <a id="r21" href="https://github.com/PeterLuschny/Fast-Factorial-Functions">Fast Factorial Functions</a>
22. <a id="r22" href="https://en.wikipedia.org/wiki/Isomorphism">Isomorphism, wikipedia</a>
23. <a id="r23" href="https://en.wikipedia.org/wiki/Representation_(mathematics)">Representation, wikipedia</a>
24. <a id="r24" href="https://en.wikipedia.org/wiki/Symmetry">Symmetry, wikipedia</a>
25. <a id="r25" href="https://en.wikipedia.org/wiki/Merge_sort">Merge sort, wikipedia</a>
26. <a id="r26" href="https://en.wikipedia.org/wiki/Radix_sort">Radix sort, wikipedia</a>
27. <a id="r27" href="http://en.wikipedia.org/wiki/Function_inlining">Function inlining, wikipedia</a>
28. <a id="r28" href="http://en.wikipedia.org/wiki/Strength_reduction">Strength reduction, wikipedia</a>
29. <a id="r29" href="http://en.wikipedia.org/wiki/Loop_invariant">Loop invariant, wikipedia</a>
30. <a id="r30" href="http://en.wikipedia.org/wiki/Loop-invariant_code_motion">Loop-invariant code motion, wikipedia</a>
31. <a id="r31" href="http://en.wikipedia.org/wiki/Row-major_order">Array linearisation, wikipedia</a>
32. <a id="r32" href="http://en.wikipedia.org/wiki/Vectorization_%28mathematics%29">Vectorization, wikipedia</a>
33. <a id="r33" href="http://en.wikipedia.org/wiki/Loop_unwinding">Loop unrolling, wikipedia</a>
34. <a id="r34" href="http://en.wikipedia.org/wiki/List_of_software_development_philosophies">Software development philosophies, wikipedia</a>
35. <a id="r35" href="http://programmer.97things.oreilly.com/wiki/index.php/97_Things_Every_Programmer_Should_Know">97 Things every programmer should know</a>
36. <a id="r36" href="http://en.wikipedia.org/wiki/Data_structure">Data structure, wikipedia</a>
37. <a id="r37" href="http://en.wikipedia.org/wiki/List_of_data_structures">List of data structures, wikipedia</a>
38. <a id="r38" href="http://en.wikipedia.org/wiki/Cloud_computing">Cloud computing, wikipedia</a>
39. <a id="r39" href="http://en.wikipedia.org/wiki/Load_balancing_%28computing%29">Load balancing, wikipedia</a>
40. <a id="r40" href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller">Model-view-controller, wikipedia</a>
41. <a id="r41" href="http://en.wikipedia.org/wiki/Template_engine">Template engine, wikipedia</a>
42. <a id="r42" href="https://www.facebook.com/notes/facebook-engineering/three-optimization-tips-for-c/10151361643253920">Three optimization tips for C</a>
43. <a id="r43" href="http://www.slideshare.net/andreialexandrescu1/three-optimization-tips-for-c-15708507">Three optimization tips for C, slides</a>
44. <a id="r44" href="http://floating-point-gui.de/">What Every Programmer Should Know About Floating-Point Arithmetic</a>
45. <a id="r45" href="http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html">What Every Computer Scientist Should Know About Floating-Point Arithmetic</a>
46. <a id="r46" href="http://dragonbook.stanford.edu/lecture-notes/Stanford-CS143/20-Optimization.pdf">Optimisation techniques</a>
47. <a id="r47" href="https://cs.brown.edu/courses/cs033/docs/guides/c_optimization_notes.pdf">Notes on C Optimisation</a>
48. <a id="r48" href="http://www.tantalon.com/pete/cppopt/main.htm">Optimising C++</a>
49. <a id="r49" href="http://www.azillionmonkeys.com/qed/optimize.html">Programming Optimization</a>
50. <a id="r50" href="http://www.ibiblio.org/pub/languages/fortran/ch1-9.html">CODE OPTIMIZATION - USER TECHNIQUES</a>
51. <a id="r51" href="https://en.wikipedia.org/wiki/Locality_of_reference">Locality of reference, wikipedia</a>
52. <a id="r52" href="https://en.wikipedia.org/wiki/Memory_access_pattern">Memory access pattern, wikipedia</a>
53. <a id="r53" href="https://en.wikipedia.org/wiki/Memory_hierarchy">Memory hierarchy, wikipedia</a>
54. <a id="r54" href="https://en.wikipedia.org/wiki/Heterogeneous_computing">Heterogeneous computing, wikipedia</a>
55. <a id="r55" href="https://en.wikipedia.org/wiki/Stream_processing">Stream processing, wikipedia</a>
56. <a id="r56" href="https://en.wikipedia.org/wiki/Dataflow_programming">Dataflow programming, wikipedia</a>
57. <a id="r57" href="https://en.wikipedia.org/wiki/Fast_Fourier_transform">Fast Fourier transform, wikipedia</a>
58. <a id="r58" href="https://en.wikipedia.org/wiki/Gaussian_elimination">Gaussian elimination, wikipedia</a>
59. <a id="r59" href="https://en.wikipedia.org/wiki/Fermat%27s_little_theorem">Fermat's little theorem, wikipedia</a>
60. <a id="r60" href="https://en.wikipedia.org/wiki/Trigonometric_identities">Trigonometric identities, wikipedia</a>
61. <a id="r61" href="http://arxiv.org/abs/1102.1523v1">The NumPy array: a structure for efficient numerical computation</a>
62. <a id="r62" href="https://arxiv.org/abs/cs/0011047">Dancing links algorithm</a>
63. <a id="r63" href="http://soft.vub.ac.be/~madewael/w-JITds/paper.pdf">Data Interface + Algorithms = Efficient Programs</a>
64. <a id="r64" href="http://brandonlucia.com/pubs/approx2014.pdf">Systems Should Automatically Specialize Code and Data</a>
65. <a id="r65" href="http://www.cs.le.ac.uk/ac/LACFunding/LACNewParDS/finalreport.pdf">New Paradigms in Data Structure Design: Word-Level Parallelism and Self-Adjustment</a>
66. <a id="r66" href="http://20bits.com/article/10-tips-for-optimizing-mysql-queries-that-dont-suck">10 tips for optimising Mysql queries</a>
67. <a id="r67" href="https://www.percona.com/blog/">Mysql Optimisation</a>
68. <a id="r68" href="http://www.cs.technion.ac.il/~erez/Papers/wf-simulation-ppopp14.pdf">A Practical Wait-Free Simulation for Lock-Free Data Structures</a>
69. <a id="r69" href="http://www.cs.uoi.gr/tech_reports/publications/TR2011-01.pdf">A Highly-Efficient Wait-Free Universal Construction</a>
70. <a id="r70" href="http://www.cs.technion.ac.il/~erez/Papers/wf-methodology-ppopp12.pdf">A Methodology for Creating Fast Wait-Free Data Structures</a>
71. <a id="r71" href="https://en.wikipedia.org/wiki/Polynomial_greatest_common_divisor#Euclid.27s_algorithm">Euclidean Algorithm</a>
72. <a id="r72" href="https://en.wikipedia.org/wiki/Gr%C3%B6bner_basis">Gröbner basis</a>
73. <a id="r73" href="https://en.wikipedia.org/wiki/Newton%27s_method">Newton's Method</a>
74. <a id="r74" href="https://en.wikipedia.org/wiki/Fast_inverse_square_root">Fast Inverse Square Root</a>
75. <a id="r75" href="https://en.wikipedia.org/wiki/Methods_of_computing_square_roots">Methods of computing square roots</a>
76. <a id="r76" href="https://www.nayuki.io/page/fast-fibonacci-algorithms">Fast Fibonacci numbers</a>
77. <a id="r77" href="https://booking.ai/k-nearest-neighbours-from-slow-to-fast-thanks-to-maths-bec682357ccd">Fast k-Nearest Neighbors (k-NN) algorithm</a>
78. <a id="r78" href="https://hal.inria.fr/file/index/docid/71533/filename/RR-5050.pdf">A Binary Recursive Gcd Algorithm</a>
