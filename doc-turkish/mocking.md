# Mocklama

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/mocking)**

3'ten geriye sayan bir program yazmanız istendi, her bir numarayı yeni bir satırda (1 saniye aralıklarla) yazdıracak ve sıfır olduğunda "Go!" yazacak.
```
3
2
1
Go!
```

Bunu çözmek için `CountDown` isminde fonksiyon yazacağız daha sonra `main` metodunun içine koyacağız. Sonra bunun gibi gözükecek:

```go
package main

func main() {
    Countdown()
}
```

Bu oldukça önemsiz bir program olsa da, tamamen test etmek için her zaman olduğu gibi _yinelemeli (iterative)_, _test odaklı (test driven)_ bir yaklaşım benimsememiz gerekecek.

Yinelemeli derken neyi kastettim? _Kullanışlı yazılıma_ sahip olmak için en küçük adımları attığımızdan emin oluruz.

Bazı hacking tekniklerinden sonra teorik olarak çalışacak bir kodla uzun zaman harcamak istemiyoruz çünkü geliştiriciler genellikle bu şekilde tavşan deliklerine düşüyorlar. **_Çalışan bir yazılıma_ sahip olabilmeniz için gereksinimleri olabildiğince küçük parçalara ayırabilmek önemli bir beceridir.**

Çalışmamızı şu şekilde bölebilir ve üzerinde yineleyebiliriz:

- 3'ü Yazdır
- 3, 2, 1 ve Go! Yazdır
- Her satır arasında 1 saniye bekle

## İlk olarak test yaz

Yazılımımızın stdout'a yazdırması gerekiyor ve bunu DI bölümünde test etmeyi kolaylaştırmak için DI'yi nasıl kullanabileceğimizi gördük.

```go
func TestCountdown(t *testing.T) {
    buffer := &bytes.Buffer{}

    Countdown(buffer)

    got := buffer.String()
    want := "3"

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

`buffer` gibi şeyler tanıdık değilse, bir önceki bölümü tekrar okuyun [the previous section](dependency-injection.md).

`Countdown` fonksiyonumuzun bir yere veri yazmasını istediğimizi biliyoruz ve `io.Writer`  Go'da bu yolu fiilen interface olarak yapmakta.

- `main` içerisinde  `os.Stdout`'u göndereceğeğiz bu sayde kullanıcılarımız gerisayımın çıktılarını terminalde görebilecek.
- Test içerisinde `bytes.Buffer` göndereceğiz bu sayede testimiz üretilmiş veriyi yakalayabilecek.

## Dene ve testi çalıştır

`./countdown_test.go:11:2: undefined: Countdown`

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`Countdown`'ı tanımla

```go
func Countdown() {}
```

Tekrar dene

```go
./countdown_test.go:11:11: too many arguments in call to Countdown
    have (*bytes.Buffer)
    want ()
```

The compiler is telling you what your function signature could be, so update it.

```go
func Countdown(out *bytes.Buffer) {}
```

`countdown_test.go:17: got '' want '3'`

Harika!

## Testi geçecek kadar kod yaz

```go
func Countdown(out *bytes.Buffer) {
    fmt.Fprint(out, "3")
}
```

(`*bytes.Buffer` gibi olan) `io.Writer`'ı parametre alan `fmt.Fprint` kullanıyoruz ve ona `string` gönderiyoruz. Şimdi test geçmeli

## Refactor

`*bytes.Buffer` çalışırken bunun yerine genel amaçlı bir interface kullanmanın daha iyi olacağını biliyoruz.

```go
func Countdown(out io.Writer) {
    fmt.Fprint(out, "3")
}
```

Testleri tekrar çalıştırın ve şimdi geçiyor olmalılar.

Konuları tamamlamak için, fonksiyonumuzu `main`'e bağlayalım, böylece ilerleme kaydettiğimizden emin olmak için çalışan bazı yazılımlarımız olur.

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func Countdown(out io.Writer) {
    fmt.Fprint(out, "3")
}

func main() {
    Countdown(os.Stdout)
}
```

Programı deneyin ve çalıştırın ve el emeğinize hayran kalın.

Evet, bu önemsiz görünüyor ama bu yaklaşım, herhangi bir proje için tavsiye edeceğim şeydir. **İnce bir işlevsellik alın ve testlerle desteklenen uçtan uca çalışmasını sağlayın.**

Daha sonra 2,1 ve "Go!" yazdırabilecek.

## İlk olarak test yaz

Genel tesisatın doğru çalışmasına yatırım yaparak, çözümümüzü güvenli ve kolay bir şekilde yineleyebiliriz. Tüm logic test edildiğinden, çalıştığından emin olmak için programı durdurup yeniden çalıştırmamız gerekmeyecek..

```go
func TestCountdown(t *testing.T) {
    buffer := &bytes.Buffer{}

    Countdown(buffer)

    got := buffer.String()
    want := `3
             2
             1
             Go!`

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

backtick syntax `string` oluşturmak için başba bir yoldur ancak testimiz için mükemmel olan yeni satırlar gibi şeyleri koymanıza izin verir. 

## Dene ve testi çalıştır

```
countdown_test.go:21: got '3' want '3
        2
        1
        Go!'
```
## Testi geçecek kadar kod yaz

```go
func Countdown(out io.Writer) {
    for i := 3; i > 0; i-- {
        fmt.Fprintln(out, i)
    }
    fmt.Fprint(out, "Go!")
}
```

`i--` ile geriye doğru sayan bir `for` döngüsü kullan ve `fmt.Println` kullanarak numaramızın ardından bir satır sonu karakteri ile `out` çıktısını alın. Sonunda "Go!" göndermek için `fmt.Fprint`'i kullan.

## Refactor

Bazı sihirli değerleri adlandırılmış sabitlere yeniden düzenlemekten başka yeniden düzenleme yapacak pek bir şey yok.

```go
const finalWord = "Go!"
const countdownStart = 3

func Countdown(out io.Writer) {
    for i := countdownStart; i > 0; i-- {
        fmt.Fprintln(out, i)
    }
    fmt.Fprint(out, finalWord)
}
```

Eğer programı çalıştırırsanız, istediğiniz çıktıyı almalısınız ancak 1 saniyelik duraklamalı dramatik geri sayımımız yok.

Go bunu `time.Sleep` ile başarmamızı sağlar. Koda eklemeyi dene.

```go
func Countdown(out io.Writer) {
    for i := countdownStart; i > 0; i-- {
        time.Sleep(1 * time.Second)
        fmt.Fprintln(out, i)
    }

    time.Sleep(1 * time.Second)
    fmt.Fprint(out, finalWord)
}
```

Eğer programı çalıştırırsan istediğimiz gibi çalışacaktır.

## Mocklama

Testler hala geçiyor ve yazılım beklenildiği gibi çalışıyor ancak bazı problemlerimiz var:
- Testimizin çalışması 4 saniye sürüyor.
    - Yazılım geliştirme hakkındaki her ileri görüşlü yazı, hızlı geri bildirim döngülerinin önemini vurgular.
    - **Yavaş tesler geliştiricinin üretkenliğini mahveder**.
    - Daha fazla test için gereksinimlerin daha karmaşık hale geldiğini hayal edin. Her yeni `Countdown` testi için test çalışmasına eklenen 4s'den memnun muyuz?
- Fonksiyonumuzun önemli bir özelliğini test etmedik.

`Sleep`'e bağımlılığımız var ve bunu testlerimizde kontrol edebilmemiz için çıkarmamız gerekli.

Eğer `time.Sleep`'i _mocklayabilirsek_,  "gerçek" `time.Sleep` yerine  _dependency injection_ kullanabiliriz ve onlar hakkında iddialarda bulunmak için **çağrıları gözetleyebilir**.

## İlk olarak test yaz

Bağımlılığımız interface olarak tanımlayalım. Bu, daha sonra testlerimizde `main`'de _gerçek_ Sleeper ve testlerimizde _spy_ sleeper kullanmamızı sağlar. Interface kullanarak ``Countdown` fonksiyonumuz habersiz olur ve çağrıyı yapan için biraz esneklik ekler.

```go
type Sleeper interface {
    Sleep()
}
```

`Countdown` fonksiyonu uyku süresinden sorumlu olmayacak bir tasarıma karar verdim. Bu, kodumuzun en azından şimdilik biraz basitleştirir ve fonksiyonumuzun bir kullanıcısının bu uykuyu istedikleri gibi yapılandırılabileceği anlamına gelir.

Şimdi testlerimizin kullanması için _mock_ yapmalıyız

```go
type SpySleeper struct {
    Calls int
}

func (s *SpySleeper) Sleep() {
    s.Calls++
}
```

_Spylar_, bağımlılıkların nasıl kullanıldığını kaydedebilen bir tür _mockturlar_. Gönderilen argümanı kaç kez çağırıldıkları vb. kaydedebilirler.  Bizim durumumuzda, `Sleep()`'in kaç kez çağırıldığını kaydediyoruz bu sayede testimizde kontrol edebiliyoruz.

Spy'ımıza bağımlılık inject etmek için testleri güncelle ve sleepin 4 kez çağırıldığını doğrula.

```go
func TestCountdown(t *testing.T) {
    buffer := &bytes.Buffer{}
    spySleeper := &SpySleeper{}

    Countdown(buffer, spySleeper)

    got := buffer.String()
    want := `3
2
1
Go!`

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }

    if spySleeper.Calls != 4 {
        t.Errorf("not enough calls to sleeper, want 4 got %d", spySleeper.Calls)
    }
}
```

## Dene ve testi çalıştır

```
too many arguments in call to Countdown
    have (*bytes.Buffer, *SpySleeper)
    want (io.Writer)
```

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`Sleeper`'ı kabul etmesi için `Countdown`'u güncellemeliyiz

```go
func Countdown(out io.Writer, sleeper Sleeper) {
    for i := countdownStart; i > 0; i-- {
        time.Sleep(1 * time.Second)
        fmt.Fprintln(out, i)
    }

    time.Sleep(1 * time.Second)
    fmt.Fprint(out, finalWord)
}
```

Eğer tekrar denerseniz, `main` fonksiyonunuz tekrar aynı sebeplerden derlenmeyecektir

```
./main.go:26:11: not enough arguments in call to Countdown
    have (*os.File)
    want (io.Writer, Sleeper)
```

Hadi ihtiyacımız olan interfacei implemente eden _gerçek_ sleeperı oluşturalım

```go
type DefaultSleeper struct {}

func (d *DefaultSleeper) Sleep() {
    time.Sleep(1 * time.Second)
}
```

Daha sonra gerçek uygulamamızda şöyle kullanabiliriz 

```go
func main() {
    sleeper := &DefaultSleeper{}
    Countdown(os.Stdout, sleeper)
}
```

## Testi geçecek kadar kod yaz

Test derleniyor ancak geçmiyor çünkü inject edilmiş bağımlılık yerine hala `time.Sleep`'i çağırıyoruz. Hadi bunu düzeltelim.

```go
func Countdown(out io.Writer, sleeper Sleeper) {
    for i := countdownStart; i > 0; i-- {
        sleeper.Sleep()
        fmt.Fprintln(out, i)
    }

    sleeper.Sleep()
    fmt.Fprint(out, finalWord)
}
```

Şimdi test geçmeli ve 4 saniye sürmemeli

### Hala problemler var

Hala test etmediğimiz önemli bir özellik var.

`Countdown` her çıktıdan önce uyumalı, örn:

- `Uyu`
- `N'i yazdır`
- `Uyu`
- `N-1'i yazdır`
- `Uyu`
- `Go!'yu yazdır`
- vb

Son değişikliğimiz sadece 4 kez uyuduğunu iddia ediyor ancak bu uyumalar sıra dışı gerçekleşebilir.

Testleri yazarken, testlerinizin size yeterli güven verdiğinden emin değilseniz, testlerinizi bozun! (öncelikle değişikliklerinizi kaynak denetimine adadığınızdan emin olun). Kodu aşağıdaki şekilde değiştirin

```go
func Countdown(out io.Writer, sleeper Sleeper) {
    for i := countdownStart; i > 0; i-- {
        sleeper.Sleep()
    }

    for i := countdownStart; i > 0; i-- {
        fmt.Fprintln(out, i)
    }

    sleeper.Sleep()
    fmt.Fprint(out, finalWord)
}
```

Testleri çalıştırırsanız, implementasyon yanlış olsa da geçecektir.

İşlem sırasının doğru olup olmadığını kontrol etmek için yeni bir testle spyı tekrar kullanalım.

İki farklı bağımlılığımız var ve tüm işlemlerini tek bir listeye kaydetmek istiyoruz. Böylece _ikisi için de bir spy_ yaratacağız.

```go
type SpyCountdownOperations struct {
    Calls []string
}

func (s *SpyCountdownOperations) Sleep() {
    s.Calls = append(s.Calls, sleep)
}

func (s *SpyCountdownOperations) Write(p []byte) (n int, err error) {
    s.Calls = append(s.Calls, write)
    return
}

const write = "write"
const sleep = "sleep"
```

`SpyCountdownOperations`,her çağrıyı bir slicea kaydederek `io.Writer` ve `Sleeper`'ı implemente eder. Bu testte sadece operasyonların sırası bizi alakadar etmekte bu yüzden sadece operasyonların ismini kadetmek yeterli.

Test suitemize uyuma ve yazma operasyonlarımızın umduğumuz gibi doğur sırada gerçeklestiğini doğrulaması için alt test ekleyebiliriz

```go
t.Run("sleep before every print", func(t *testing.T) {
    spySleepPrinter := &SpyCountdownOperations{}
    Countdown(spySleepPrinter, spySleepPrinter)

    want := []string{
        sleep,
        write,
        sleep,
        write,
        sleep,
        write,
        sleep,
        write,
    }

    if !reflect.DeepEqual(want, spySleepPrinter.Calls) {
        t.Errorf("wanted calls %v got %v", want, spySleepPrinter.Calls)
    }
})
```

Bu test başrısız olmalı. Testi düzeltmek için `Countdown`'ı eski haline getirelim.

`Sleeper` üzerinde spylık yapan iki testimiz var, testimizi yeniden düzenleyebiliriz böylece biri yazılanları test eder diğeri de yazılanlar arasında uyuduğumuzdan emin olur. Sonunda, ilk spyımuzu kullanılmadığı için silebiliriz.

```go
func TestCountdown(t *testing.T) {

    t.Run("prints 3 to Go!", func(t *testing.T) {
        buffer := &bytes.Buffer{}
        Countdown(buffer, &SpyCountdownOperations{})

        got := buffer.String()
        want := `3
2
1
Go!`

        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    })

    t.Run("sleep before every print", func(t *testing.T) {
        spySleepPrinter := &SpyCountdownOperations{}
        Countdown(spySleepPrinter, spySleepPrinter)

        want := []string{
            sleep,
            write,
            sleep,
            write,
            sleep,
            write,
            sleep,
            write,
        }

        if !reflect.DeepEqual(want, spySleepPrinter.Calls) {
            t.Errorf("wanted calls %v got %v", want, spySleepPrinter.Calls)
        }
    })
}
```

Artık fonksiyonumuz ve iki önemli özelliği düzgünce test edildi.

## Sleeper'ı düzenlenebilir hale getirme

`Sleeper`'ın düzenlenebilir olması harika bir özellik olurdu. Bunun anlamı ana programımız içerisinden uyuma zamanını ayarlayabiliriz.

### İlk olarak test yaz

Ayarlama ve test için ne gerekiyorsa kabul eden `ConfigurableSleeper` tipini oluşturalım.

```go
type ConfigurableSleeper struct {
    duration time.Duration
    sleep    func(time.Duration)
}
```

 Uyuma süresini ayarlamak ve sleep fonksiyonuna parametre göndermek için `duration`'ı kullanıyoruz. `sleep`'in metod imzası (signature) `time.Sleep` ile aynı olması bize `time.Sleep`'i gerçek uygulamada kullanmamızı, `sleep`'i ise testlerimizde kullanmamızı sağlıyor:

```go
type SpyTime struct {
    durationSlept time.Duration
}

func (s *SpyTime) Sleep(duration time.Duration) {
    s.durationSlept = duration
}
```

Spyımız yerindeyken, ayarlanablir sleeper için yeni test oluşturabiliriz.

```go
func TestConfigurableSleeper(t *testing.T) {
    sleepTime := 5 * time.Second

    spyTime := &SpyTime{}
    sleeper := ConfigurableSleeper{sleepTime, spyTime.Sleep}
    sleeper.Sleep()

    if spyTime.durationSlept != sleepTime {
        t.Errorf("should have slept for %v but slept for %v", sleepTime, spyTime.durationSlept)
    }
}
```

Testte yeni bir şey olmamalı ve önceki mock testler ile kurulumu çok benzer olmalı.

### Dene ve testi çalıştır
```
sleeper.Sleep undefined (type ConfigurableSleeper has no field or method Sleep, but does have sleep)

```

`ConfigurableSleeper`'ımızda `Sleep` metodunun olmadığını ima eden çok anlaşılabilir bir hata mesajı görmelisiniz.

### Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et
```go
func (c *ConfigurableSleeper) Sleep() {
}
```

`Sleep` fonksiyonumuzu implemente ettikten sonra başarısı olan bir testimiz var.

```
countdown_test.go:56: should have slept for 5s but slept for 0s
```

### Testi geçeek kadar kod yaz

Tek yapmamız gereken `ConfigurableSleeper` için `Sleep` fonksiyonunuzu implemente etmek.

```go
func (c *ConfigurableSleeper) Sleep() {
    c.sleep(c.duration)
}
```

Bu değişiklikle birlikte tüm testler tekrar geçmeli ve tüm bu rahatsızlığa rağmen main program neden hiç değişmediğini merak edebilirsiniz. Umarım sonraki bölümden sonra netleşir.

### Temizleme ve refactor

Yapmamız gereken son şey main fonksyionda `ConfigurableSleeper`'ı kullanmak.

```go
func main() {
    sleeper := &ConfigurableSleeper{1 * time.Second, time.Sleep}
    Countdown(os.Stdout, sleeper)
}
```

Eğer testi ve uygulamayu çalıştırırsak tüm davranışların aynı kaldığını görebiliriz. 

`ConfigurableSleeper`'ı kullandığımız için `DefaultSleeper`'ı silmek artık güvenli. Programımızı tamamlıyoruz ve daha [kapsamlı (generic)](https://stackoverflow.com/questions/19291776/whats-the-difference-between-abstraction-and-generalization) ve keyfi uzunlukta gerisayımları olan Sleeperımız oluyor.

## Mocklama şeytani değilmiydi ?

Mocklamanın şeytani olduğunu duymuşsundur. Yazılım geliştirmede herhangi bir şey gibi, kötülük için kullanılabilir, [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) gibi.

İnsanlar normalde _testlerini dinlemediklerinde_ ve _refactoring aşamasına uymadıklarında_ kötü bir duruma girerler.

Eğer mocking kodunuz karmaşıklaşıyor veya bir şeyi test etmek için çok fazla mockunuz varsa, bu kötü duyguyu _dinlemeli_ ve kodunuzu düşünmelisiniz. Genelde bir işarettir

- Test ettiğiniz şey çok fazla şey yapıyorsa(çünkü mocklamak için çok fazla bağımlılığı var)
  - Modülü parçalara ayırın, böylece daha az iş yapar
- Bağımlılıkları çok ince taneli
  - Bu bağımlılıklardan bazılarını tek bir anlamlı modülde nasıl birleştirebileceğinizi düşünün.
- Testiniz implementasyon detayları ile fazla fazla ilgili
  - Uygulamadan ziyade beklenen davranışı test etmeyi tercih edin

Normade, çok mocklama koduzunda _kötü soyutlamaya_ bir işarettir.

**İnsanların TDD'yi zayıflık olarak gördüğü şey aslında güçtür**, çoğu zaman zayıf test kodu kötü tasarımın sonucudur veya daha güzel bir şekilde ifade edilirse, iyi tasarlanmış kod kolay test edilir.

### Ama mocklar ve testler hala hayatımı zorlaştırıyor! 

Hiç bu durumla karşılaştın mı?

- Refactoring yapmak istemek 
- Bunu yapmak için bir çok testi değiştirmek 
- TDD'yi sorgularsınız ve Medium'da `Mocklama zararlı olarak kabul edilir` diye paylaşım yaparsınız

Bu genelde çok fazla _implementasyon detayını_ test ettiğinize dair bir işarettir. Sistemin nasıl çalıştığı implementasyon için gerçekten önemli olana kadar testlerinizin _kullanışlı davranışları_ test etmeye çalışın.

Tam olarak hangi seviyenin test edileceğini bilmek bazen zordur, ancak burada uymaya çalıştığım bazı düşünce süreçleri ve kurallar vardır:

- **Refactoringin tanımı, kodun değişmesi ancak davranışın aynı kalmasıdır**. Teoride bazı yeniden düzenleme yapmaya karar verdiyseniz, herhangi bir test değişikliği yapmadan taahhütte bulunabilmelisiniz. Bu yüzden bir test yazarken kendinize sorun
  - İstediğim davranışı mı test ediyorum yoksa implementasyon detaylarını mı?
  - Bu kodu refactor etseydim, testlerde çok fazla değişiklik yapmak gerekir miydi?
- Go, private fonksiyonları test etmenize izin verse de, private fonksiyonlar, genel davranışı desteklemek için implentasyon ayrıntısı olduğu için bundan kaçınırdım. Açık davranışları test et. Sandi Metz, private fonksiyonları "daha az kararlı" olarak tanımlar ve testlerinizi bunlarla birleştirmek istemezsiniz.
- Eğer bir test **3'ten fazla mock ile çalışıyorsa** tasarım hakkında yeniden düşünmek için kırmızı bayraktır diye hissediyorum. 
- Spyları dikkatli kullanın. Spylar, yazdığınız algoritmanın içini görmenizi sağlar ve bu çok faydalı olabilir, ancak bu, test kodunuz ile implementasyon arasında daha sıkı bir bağlantı anlamına gelir. **Onları gözetleyecekseniz, bu ayrıntıları gerçekten önemsediğinizden emin olun**

#### Mocklama için framework kullanamaz mıyım ?

Mocklama sihir gerektirmez ve nispeten basittir; Framework kullanmak, mocklamayı olduğunda daha karmaşık yapabilir. Bu bölümde otomatik mocklama kullanmayacağız, böylece:

- mock nasıl çalışır daha iyi anlarız
- interface implemente etme pratiği yaparız

İşbirlikçi (Collobrative) projelerde, otomatik oluşturulan mockların değeri vardır. Bir takımda, mocklama aracı test doubleları arasında tutarlılığı kodlar. Bu, tutarsız yazılmış testlere dönüşebilecek tutarsız yazılmış test kopyalarını önleyecektir..

Yalnızca bir interface karşı test doubleları oluşturan bir mock oluşturucu kullanmalısınız. Testlerin nasıl yazıldığını aşırı derecede belirleyen veya çok fazla 'sihir' kullanan herhangi bir araç denize girebilir.

## Özetlersek

### TDD yaklaşımı hakkında daha fazla bilgi

- Daha az önemsiz örneklerle karşılaştığınızda, sorunu "ince dikey dilimlere" bölün. Tavşan deliklerine girmekten ve "büyük patlama" yaklaşımı benimsemekten kaçınmak için, mümkün olan en kısa sürede _testlerle desteklenen çalışan bir yazılıma_ sahip olduğunuz bir noktaya gelmeye çalışın.
- Çalışan bir yazılımınız olduğunda, ihtiyacınız olan yazılıma ulaşana kadar _küçük adımlarla yinelemeniz_ daha kolay olacaktır.

> "Yinelemeli geliştirmeyi ne zaman kullanmalıyız? Yinelemeli geliştirmeyi sadece başarılı olmasını istediğiniz projelerde kullanmalısınız."

Martin Fowler.

### Mocklama

- **Kodunuzun önemli bir kısmı mocklama olmadan test edilemeyecektir**. Bizim durumumuzda, kodumuzun her yazdırma arasında durakladığını test edemeyiz, ancak sayısız başka örnek var. Başarısız _olabilecek_ bir servicei çağırmak mı istiyorsunuz? Sisteminizi belirli bir durumda test etmek mi istiyorsunuz? Mocklama olmadan bu senaryoları test etmek oldukça zor.
- Mocklar olmadan, sadece basit iş kurallarını test etmek için veritabanları ve diğer üçüncü taraf şeyleri kurmanız gerekebilir. Muhtemelen yavaş testler yapacaksınız ve bu da **yavaş geri bildirim döngülerine** yol açacaktır.
- Bir şeyi test etmek için bir veritabanını veya bir web servisini döndürmek zorunda kalırsanız, bu tür servislerin güvenilmezliği nedeniyle **hassas testler** yapmanız olasıdır.

Bir geliştirici mocklamayı öğrendiğinde, bir sistemin her bir yönünü, _ne yaptığından_ ziyade, _nasıl çalıştığı_ açısından aşırı test etmek çok kolay hale gelir. **Testlerinizin değeri** ve gelecekteki yeniden düzenlemede ne gibi etkileri olacağı konusunda daima dikkatli olun.

Mocklama ile ilgili bu yazıda yalnızca bir tür mock olan **Spyları** ele aldık. Mocklar için "uygun" terim, "test double" olsa da

[> Test Double, test amacıyla bir üretim nesnesini değiştirdiğiniz her durum için genel bir terimdir.](https://martinfowler.com/bliki/TestDouble.html)

Test doubleları altında, stublar, spylar ve gerçekten de mocklar gibi çeşitli türler var! Daha fazla ayrıntı için [Martin Fowler'ın gönderisine](https://martinfowler.com/bliki/TestDouble.html) göz atın.