# Concurrency

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/concurrency)**

Kurgu şu şekilde:Bir meslektaşınız, URL'lerin durumunu kontrol eden `CheckWebsites` isminde bir fonksiyon yazar.

```go
package concurrency

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		results[url] = wc(url)
	}

	return results
}
```

Her bir URL'in kontrolünü boolean değer olarak tutan bir map döner - başarılı cevaplar için `true`, başarısız cevaplar için `false`.

Ayrıca, parametre olarak URL alan ve boolean değer dönen `WebsiteChecker`'a parametre göndermelisiniz. 
Bu, bütün web sitelerini kontrol etmek için kullanılacak.

[Dependency Injection][DI] kulanmak, gerçek HTTP çağrıları yapmadan fonksiyonu test etmemizi sağlar, güvenilir ve hızlı hale getirir

İşte, onların yazdığı test :

```go
package concurrency

import (
	"reflect"
	"testing"
)

func mockWebsiteChecker(url string) bool {
	if url == "waat://furhurterwe.geds" {
		return false
	}
	return true
}

func TestCheckWebsites(t *testing.T) {
	websites := []string{
		"http://google.com",
		"http://blog.gypsydave5.com",
		"waat://furhurterwe.geds",
	}

	want := map[string]bool{
		"http://google.com":          true,
		"http://blog.gypsydave5.com": true,
		"waat://furhurterwe.geds":    false,
	}

	got := CheckWebsites(mockWebsiteChecker, websites)

	if !reflect.DeepEqual(want, got) {
		t.Fatalf("Wanted %v, got %v", want, got)
	}
}
```

Fonksiyon kullanımda ve yüzlerce web sitesinin kontrolünde kullanılıyor. Ancak meslektaşınız
fonksiyonun yavaş oluduğuna dair şikayetler almaya başladı, bu yüzden sizden hızlandırmak
için yardım istedi.

## Test yaz

`CheckWebsites`'ın hızını test etmek için benchmark kullanalım bu sayede 
yaptığımız değişikliklerin etkilerini görürüz

```go
package concurrency

import (
	"testing"
	"time"
)

func slowStubWebsiteChecker(_ string) bool {
	time.Sleep(20 * time.Millisecond)
	return true
}

func BenchmarkCheckWebsites(b *testing.B) {
	urls := make([]string, 100)
	for i := 0; i < len(urls); i++ {
		urls[i] = "a url"
	}

	for i := 0; i < b.N; i++ {
		CheckWebsites(slowStubWebsiteChecker, urls)
	}
}
```
Benchmark, yüz tane url'in bulunduğu slice'ı ve `WebsiteChecker`'ın 
sahte bir implementasyonunu kullanarak `CheckWebsite`'ı test eder. `slotStubWebsiteChecker` kasten yavaş.
Tam olarak yirmi milisaniye beklemek için `time.Sleep` kullanır ve sonra true döner.

`go test -bench=.` (veya Windows Powershell'de iseniz `go test -bench="."`) kullanarak benchmark çalıştırdığımızda:

```sh
pkg: github.com/gypsydave5/learn-go-with-tests/concurrency/v0
BenchmarkCheckWebsites-4               1        2249228637 ns/op
PASS
ok      github.com/gypsydave5/learn-go-with-tests/concurrency/v0        2.268s
```

`CheckWebsites`, 2249228637 nanosaniye olarak ölçüldü - yaklaşık iki saniye.

Hadi bunu daha hızlı yapmayı deneyelim.

### Testi geçecek kadar kod yaz

Sonunda concurrency, bir kerede birden fazla işlem, hakkında konuşabiliriz.
Bu her gün doğal olarak yaptığımız bir şey.

Örneğin, bu sabah bir fincan çay yaptım. Su ısıtıcısını açtım ve kaynamasını beklerken, 
sütü buzdolabından çıkardım, çayı dolaptan çıkardım, favori bardağımı buldum, 
çay poşetini bardağa koydum, su ısıtıcısı kaynadığında, suyu bardağa koydum. 

_Yapmadığım_ şey ise, su ısıtıcısını açmak ve boş boş orada beklemekti, 
ardında su ısıtıcıs kaynadıktan sonra diğer her şeyi yapmaktı

Eğer ilk yöntem ile çay yapmak neden daha hızlı olduğunu anlarsanız, 
`CheckWebsites`'ı nasıl daha hızlı yapacağımızı anlayabilirsiniz.
Sıradaki web sitesine istek atmadan önce web sitesinden cevap beklemek yerine, 
bilgisayarımıza beklerken sıradaki isteği atmasını söyleyeceğiz. 

Normalde Go'da `doSomething()` fonksiyonunu çağırdığımızda dönüş yapması için bekleriz
(dönecek bir değeri olmasa bile yine de bitmesi için bekleriz). 
Bu işleme *blocking* işlemi diyoruz -Bitemsi için bizi bekletiyor. An operation
Go'da blocklamayan operasyon, *goroutine* olarak isimlendirilen ayrı bir *process*'te çalışır.
Process'i, Go kod sayfasının yukarıdan aşağıya okunmasi gibi düşünün, fonksiyonun ne yaptığını 
okumak için her bir fonksiyonun 'içine' gitmek gibi düşünün. Ayrı bir process başladığında,
başka bir okuyucu fonksiyonun içini okurken orjinal okuyucu sayfanın aşağısında gitmeye devam etmesine benziyor.

Go'ya yeni bir goroutine başlatmasını söylemek için fonksiyonun önüne `go` keywordu koyuyoruz: `go doSomething()`

```go
package concurrency

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		go func() {
			results[url] = wc(url)
		}()
	}

	return results
}
```

goroutine başlatmanın tek yolu fonksiyon çağrısının önüne `go` koymak, 
goroutine başlatmak istediğimizde genellikle *anonymus fonksiyonları* kullanıyoruz.
anonymus fonksiyon literali, normal fonksiyon tanımı ile aynı, sadece isimsiz(şaşırtmayacak şekilde). Yukarıda
`for` döngüsünün içinde görebilirsiniz.

Anonymus fonksiyonlar onları kullanışlı yapan bir dizi özelliklere sahipler, 
ikisini yukarıda kullandık. İlk olarak, tanımlandıkları anda çalıştırılabilmeleri - anonymus fonksiyonunun sonundaki 
`()` işareti bunu yapmakta. İkinci olarak, tanımlandıkları lexical scope'a erişim sağlarlar -
anonymus fonksiyonu tanımladığınuz noktada mevcut olan tüm değişkenler, 
aynı zamanda fonksiyonun body'sinden de erişilebilirler

Yukarıdaki anonymus fonksiyonun body'si, loop'un body'sinin önceki hali ile aynı. 
Tek fark, loop'un her iterasyonda, var olan process ile concurrent olan (`WebsiteChecker` fonksiyonu), 
her birinin sonucunu result map'e ekleyen yeni bir goroutine başlatması.

Ancak, `go test` çalıştırdığımızda:

```sh
--- FAIL: TestCheckWebsites (0.00s)
        CheckWebsites_test.go:31: Wanted map[http://google.com:true http://blog.gypsydave5.com:true waat://furhurterwe.geds:false], got map[]
FAIL
exit status 1
FAIL    github.com/gypsydave5/learn-go-with-tests/concurrency/v1        0.010s

```

### Paralel(izm) evrenine hızlı bir geçiş...

Bu sonucu alamayabilirsiniz. İleride konuşacağımız olan panic mesajını almış olabilirsiniz.
Bunu alırsanız endişe etmeyin, sadece yukarıdaki sonucu alana kadar testi çalıştırmaya devam edin.
Ya da yaptığınızı farz edin. Size kalmış. Concurrency'ye hoş geldiniz: doğru şekilde ele alınmadığında 
ne olacağını tahmin etmek zor. Endişe etmeyin - Bu nedenle test yazıyoruz, 

### ... ve, geri döndük.

Orijinal testler tarafından yakalandık `CheckWebsites` şimdi boş bir harita döndürüyor.

`for` loopumuz başladığında, goroutinelerin hiçibiri sonuçlarını `results` map'e ekleyecek kadar zaman bulamıyor; 
`WebsiteChecker` fonksiyonu onlar için çok hızlı, hala boş map dönüyor.

Bunu düzeltmek için, goroutinelerin tümünün işlerini yapmalarını bekleyebiliriz ve sonra geri dönebiliriz.
İki saniye bunu yapmalı, değil mi?

```go
package concurrency

import "time"

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		go func() {
			results[url] = wc(url)
		}()
	}

	time.Sleep(2 * time.Second)

	return results
}
```

Şimdi, testleri çalıştırdığımızda elde edeceğiniz (veya etmeyeceğiniz - yukarıya bakın):

```sh
--- FAIL: TestCheckWebsites (0.00s)
        CheckWebsites_test.go:31: Wanted map[http://google.com:true http://blog.gypsydave5.com:true waat://furhurterwe.geds:false], got map[waat://furhurterwe.geds:false]
FAIL
exit status 1
FAIL    github.com/gypsydave5/learn-go-with-tests/concurrency/v1        0.010s
```

Bu harika değil - neden sadece bir sonuç? Belki, bekleme süresini artırarak bunu 
düzeltebiliriz - isterseniz deneyin. Çalışmayacaktır. Problem şu, `url` değişkeni `for` loop'un her 
iterasyonunda tekrar tekrar kullanılmakta - her defasında `urls`'ten yeni bir değer alıyor. 
Ancak, her goroutine `url` değişkeninin referansına sahip - Kendi bağımsız koyalarına sahipr değiller.
Böylece, _hepsi_ iterasyonun sonundaki `url`'in sahip olduğu değeri yazdırıyor - son url.
Bu yüzden elimizdeki tek sonuç son url.

Bunu düzeltmek için:

```go
package concurrency

import (
	"time"
)

type WebsiteChecker func(string) bool

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)

	for _, url := range urls {
		go func(u string) {
			results[u] = wc(u)
		}(url)
	}

	time.Sleep(2 * time.Second)

	return results
}
```

Her bir anonymus fonksiyon için url parametresi vererek - `u` -  ve ardından `url` parametresi ile anonymus fonksiyonu çağırarak,
goroutini başlattığımız loop'un iterasyonu için `u` değerinin `url` değeri olarak sabitlendiğinden emin oluruz.
`u`, `url` değerinin kopyası, bu sayede değiştirilemez.

Eğer şanslı iseniz, aşağıdakini elde edeceksiniz:

```sh
PASS
ok      github.com/gypsydave5/learn-go-with-tests/concurrency/v1        2.012s
```

Eğer şanssızsanız (Benchmark çalıştırarak daha çok deneyeceğiniz için elde etmeniz daha olası)

```sh
fatal error: concurrent map writes

goroutine 8 [running]:
runtime.throw(0x12c5895, 0x15)
        /usr/local/Cellar/go/1.9.3/libexec/src/runtime/panic.go:605 +0x95 fp=0xc420037700 sp=0xc4200376e0 pc=0x102d395
runtime.mapassign_faststr(0x1271d80, 0xc42007acf0, 0x12c6634, 0x17, 0x0)
        /usr/local/Cellar/go/1.9.3/libexec/src/runtime/hashmap_fast.go:783 +0x4f5 fp=0xc420037780 sp=0xc420037700 pc=0x100eb65
github.com/gypsydave5/learn-go-with-tests/concurrency/v3.WebsiteChecker.func1(0xc42007acf0, 0x12d3938, 0x12c6634, 0x17)
        /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:12 +0x71 fp=0xc4200377c0 sp=0xc420037780 pc=0x12308f1
runtime.goexit()
        /usr/local/Cellar/go/1.9.3/libexec/src/runtime/asm_amd64.s:2337 +0x1 fp=0xc4200377c8 sp=0xc4200377c0 pc=0x105cf01
created by github.com/gypsydave5/learn-go-with-tests/concurrency/v3.WebsiteChecker
        /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:11 +0xa1

        ... korkutucu satırların olduğu daha fazla olduğu metinler ...
```

Bu uzun ve korkutucu, ama yapmamız gereken tek şey nefes almak ve stacktrace'i okumak:`fatal error: concurrent map writes`.
Bazeb, testlerimizi çalıştırdığımızda, goroutinelerin ikisi results map'e gerçkten de aynı anda yazar.
Go'da mapler, aynı anda birden fazla şeyin kendilerine yazmaya çalışmasını sevmez, sonuç olarak `fatal error`.

Buna _race condition_ denir, yazılımın çıktısının zamanlamaya 
ve kontorlümüzün olmadığı ardışık olayların bağlı olduğu buglardır.
Hangi goroutine'in results map'e ne zaman yazacağımızı kontrol edemediğimiz için,
aynı anda map'e yazan iki goroutine'e karşı savunmasızızı.

Go, built in [_race detector_][godoc_race_detector]'ü ile race conditionları testpit etmemize yardımcı olur. 
Bu özelliği etkinleştirmek için, testleri `race` flagi ile çalıştırın: `go test -race`

Buna benzer çıktılar elde etmelisiniz:

```sh
==================
WARNING: DATA RACE
Write at 0x00c420084d20 by goroutine 8:
  runtime.mapassign_faststr()
      /usr/local/Cellar/go/1.9.3/libexec/src/runtime/hashmap_fast.go:774 +0x0
  github.com/gypsydave5/learn-go-with-tests/concurrency/v3.WebsiteChecker.func1()
      /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:12 +0x82

Previous write at 0x00c420084d20 by goroutine 7:
  runtime.mapassign_faststr()
      /usr/local/Cellar/go/1.9.3/libexec/src/runtime/hashmap_fast.go:774 +0x0
  github.com/gypsydave5/learn-go-with-tests/concurrency/v3.WebsiteChecker.func1()
      /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:12 +0x82

Goroutine 8 (running) created at:
  github.com/gypsydave5/learn-go-with-tests/concurrency/v3.WebsiteChecker()
      /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:11 +0xc4
  github.com/gypsydave5/learn-go-with-tests/concurrency/v3.TestWebsiteChecker()
      /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker_test.go:27 +0xad
  testing.tRunner()
      /usr/local/Cellar/go/1.9.3/libexec/src/testing/testing.go:746 +0x16c

Goroutine 7 (finished) created at:
  github.com/gypsydave5/learn-go-with-tests/concurrency/v3.WebsiteChecker()
      /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:11 +0xc4
  github.com/gypsydave5/learn-go-with-tests/concurrency/v3.TestWebsiteChecker()
      /Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker_test.go:27 +0xad
  testing.tRunner()
      /usr/local/Cellar/go/1.9.3/libexec/src/testing/testing.go:746 +0x16c
==================
```

Detayları okuması, tekrardan, zor - ama `WARNING: DATA RACE` oldukça açık. 
Hatanın gövdesini okurken, bir harita üzerinde yazma 
gerçekleştiren iki farklı goroutin görebiliriz:

`Write at 0x00c420084d20 by goroutine 8:`

aynı memory bloğuna yazıyor

`Previous write at 0x00c420084d20 by goroutine 7:`

Bunun üzerinde, yazma işleminin hangi kod satırında olduğunu görebiliriz:

`/Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:12`

goroutine 7 ve 8'in başladığı kod satırı:

`/Users/gypsydave5/go/src/github.com/gypsydave5/learn-go-with-tests/concurrency/v3/websiteChecker.go:11`

Bilmeniz gereken her şey terminalinizte yazdırılır - tek yapmanız gereken 
onu okumak için sabırlı olmaktır.

### Channellar

Bu data race'i _channelları_ kullanarak goroutienleri koordine ederek çözebiliriz.
Channellar Go'da değer alan ve gönderen veri yapılarıdır. Bu operasyonar, detayları ile birlikte, 
farklı processler arasında haberleşemyi sağlar.

Bu durumda, parent process ile, url ile `WebsiteChecker` fonksiyonunu çalıştırma işini yapan, 
her bir goroutine'i düşünmek istiyorum.

```go
package concurrency

type WebsiteChecker func(string) bool
type result struct {
	string
	bool
}

func CheckWebsites(wc WebsiteChecker, urls []string) map[string]bool {
	results := make(map[string]bool)
	resultChannel := make(chan result)

	for _, url := range urls {
		go func(u string) {
			resultChannel <- result{u, wc(u)}
		}(url)
	}

	for i := 0; i < len(urls); i++ {
		r := <-resultChannel
		results[r.string] = r.bool
	}

	return results
}
```
`results` map'in yanında artık `resultChannel` var, aynı `make` ile oluşturduğumz gibi.
`chan result` channel tipi -`result` channel'ı.
Yeni tip, `result`, `WebsiteChecker`'ın dönüş değerini kontrol edilen url ile ilişkilendirmek için yapıldı 
- `string` ve `bool` yapısıdır. Adlandırılacak her iki değere de ihtiyacımız olmadığı için,
her bir yapı içinde anonimdir; Değerlerin isimlendirmek zor olduğunda oldukça kullanışlı olabiliyor.

Url'ler üzerinde iterate ettiğimizde, `map`'e doğrudan yazmak yerine, `wc`'ye yapılan her çağrıyı
`result` yapısını `resultChannel`'a _send statement_ ile gönderiyoruz. Bu `<-` operatörünü kullanır,
sol tarafına channel ve sağ tarafına değeri alır:

```go
// Send statement
resultChannel <- result{u, wc(u)}
```

Sıradaki `for` loop, her bir url'i iterate eder. İçerisinde, 
değeri bir channel'dan alan ve bir değişkene atayan , _receive expression_ kullanıyoruz. 
Bu da `<-` operatörünü kullanır ama iki operand yer değiştirdi:
channel sağda ve atayacağımız deişken de solda:

```go
// Receive expression
r := <-resultChannel
```

Daha sonra map'i güncellemek için elde edilen `result`'ı kullanıyoruz.

Sonuçları bir channel'a göndererek, results map'ine yapılan yazma işlemlerinin 
zamanlamasını kontrol ediyoruz, sadece bir kere gerçekleştiğinden emin oluyoruz.
`wc` çağrılarının her biri ve result channel'ına gönderilen her bir çağrı kendi 
processi içinde paralel olarak gerçekleşse de,
sonuç kanalından receive expression ile değerleri birer birer çıkarıyoruz. 

Kodun daha hızlı yapmak istediğimiz kısmını paralel hale getirdik ve paralel olarak
 gerçekleşmeyen kısmın hala lineer olarak gerçekleşmesini sağladık. Channelları kullanarak 
 çoklu processler arasında iletişim kurduk.

Benchmark'ı çalıştırdığımızda:

```sh
pkg: github.com/gypsydave5/learn-go-with-tests/concurrency/v2
BenchmarkCheckWebsites-8             100          23406615 ns/op
PASS
ok      github.com/gypsydave5/learn-go-with-tests/concurrency/v2        2.377s
```
23406615 nanosaniye - 0.023 saniye, orijinal fonksiyondan yüz kat daha hızlı. Harika bir başarı.

## Özetlersek

Bu egzersiz TDD'de normalden biraz daha hafif oldu. Bir bakıma, `CheckWebsites` 
fonksiyonunun uzun bir yeniden düzenlemesinde yer alıyoruz;
girdiler ve çıktılar asla değişmedi, sadece daha hızlı oldu. Ama yazdığımız testler, 
yazdığımız benchmark yanı sıra, `CheckWebsites`'ı hala çalıtşığına dair güveni sağlayacak 
şekilde refactor etmemizi sağladı, aslında daha hızlı olduğunu gösteriyor.

Daha hızlı yapmak için öğrendiklerimiz

- *goroutines*, Go'da concurrency'nin basit bir birimi, 
   aynı anda birden fazla web sitesi kontrol etmemizi sağladı.
- *anonymous functions*, web sitelerini kontrol eden concurrent processleri 
   başlatmamızda kullandık.
- *channels*, farklı processler arasında iletişimi kontrol ve organize etmemize yardımcı eder,
   *race condition* bug'ında  kaçınmamızı sağlar.
- *the race detector*  concurrent kodda problemleri debug etmemize yardım etti

### Daha hızlı yap

Yazılım geliştirmenin çevik bir yolunun formülasyonu, genellikle Kent Beck'e atfedilir:

> [Çalışır hale getir, doğru hale getir , hızlı hale getir][wrf]

'Çalışma' testleri geçmek, 'doğru' kodu refactor etmek ve 'hızlı' 
kodu çabucak çalıştırması için optimize etmektir.
Bir kere çalışır ve doğru hale getirdikten sonra sadece 'hızlı hale' getirebiliriz.
Bize verilen kodun zaten çalıştığını gösterdiği için şanslıydık ve refactor etmemize gerek yoktu
Önce diğer iki adım yapılmadıkça, asla 'hızlı hale getirmeyi' denememeliyiz

> [Vakitsiz optimizasyon tüm kötülüklerin kökenidir][popt]
> -- Donald Knuth

[DI]: dependency-injection.md
[wrf]: http://wiki.c2.com/?MakeItWorkMakeItRightMakeItFast
[godoc_race_detector]: https://blog.golang.org/race-detector
[popt]: http://wiki.c2.com/?PrematureOptimization
