# Hello, World

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/hello-world)**

Yeni bir programlama dilinde, ilk programınızın [Hello, World](https://en.m.wikipedia.org/wiki/%22Hello,_World!%22_program) olması artık gelenekseldir.

- İstediğiniz yere bir klasör oluşturun
- İçine `hello.go` adında yeni bir dosya oluşturun ve oluşturduğunuz dosyanın içine aşağıdaki kodu yazın.

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world")
}
```

Çalıştırmak için `go run hello.go` yazın.

## Nasıl çalışıyor

Go'da bir program yazmaya başladığınızda, içinde `main` fonksiyonu olan bir `main` paketine sahip olacaksınız. Paketler, ilgili Go kodunu birlikte gruplandırmanın yoludur.

`func` anahtar kelimesi, adı ve gövdesi olan bir fonksiyon tanımlamınızı sağlar.

`import "fmt"` ile ekrana yazdırma fonksiyonu olan `Println`'ı içeren bir paketi içe aktarıyoruz.

## Nasıl test edilir

Bunu nasıl test edersin? "Domain" kodunuzu dış dünyanın vereceği yan etkilerden ayırarak başlayabiliriz. `fmt.Println` bizim yazmadığımız bir fonksiyon ve dış dünyadan gelen bir kod, "Hello World" stringini göndermek ise domain(bizim yazdığımız) kod.

Bunları birbirinden ayıralım ve test edilebilir olsun

```go
package main

import "fmt"

func Hello() string {
	return "Hello, world"
}

func main() {
	fmt.Println(Hello())
}
```

`func` ile başlayan yeni bir fonksiyon oluşturduk ama bu sefer tanımlarken `string` keywordunu de kullandık. Yaptığımız tanım ile bu yeni fonksiyonun `string` bir değer döneceğini belirttik.

Şimdiyse, `hello_test.go` adında bir dosya oluşturalım ve bu dosya ile `hello.go` dosyasının içindeki `func Hello()` fonksiyonunu test edelim.

```go
package main

import "testing"

func TestHello(t *testing.T) {
	got := Hello()
	want := "Hello, world"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

## Go moduleleri?

Bir sonraki yapacağımız şey, yazdığımız test kodunu çalıştırmak. Terminalde `go test` komutunu çalıştırın. Eğer testler geçerse, büyük ihtimalle Go'nun eski bir sürümünü kullanıyorsun. 1.16 veya üstü bir versiyon kullanıyorsanız testler çalışmayacaktır ve terminalde şöyle bir hata göreceksiniz

```shell
$ go test
go: cannot find main module; see 'go help modules'
```

Problem nedir? Tek kelime ile, [module](https://blog.golang.org/go116-module-changes). Şanslısın ki, bu problemi çözmek oldukça kolay.
Terminalde `go mod init hello` komutunu çalıştırın ve bu komut `go.mod` dosyasını şu içeriklerle oluşturacaktır

```go
module hello

go 1.16
```

Bu dosya, `go` tarafından kullanılan araçlara(örn: cli) kodunuzla alakalı temel bilgileri verir. Yazdığınız kodu veya uygulamayı dağıtmayı düşünüyorsanız kodun indirilebileceği yerleri ve bağımlılıklarını burada bildirirsiniz. Şimdilik module dosyanız minimum düzeyde ve bu şekilde olması normal. Moduller hakkında daha fazla bilgi edinmek için [bu referansa](https://golang.org/doc/modules/gomod-ref) bakabilirsin. Şimdi test etmeye ve Go öğrenmeye dönebiliriz.

Sonraki bölümlerde, `go test` veya `go build` gibi go cli komutlarını çalıştırmadan önce her yeni projeniz için `go mod init SOMENAME` komutunu çalıştırmanız gerekecek.

## Tekrardan test etme

Terminalde `go test` komutunu tekrardan çalıştırın. Testleri geçmiş olmalı. `want` string'inin içerisindeki değeri değiştirerek kasıtlı olarak testlerin başarısız olmasını sağlayabilirsiniz.

Şunu göz ardı etmeyin, birden çok test çatısı arasından bir seçim yapmanıza veya nasıl kurulacağını öğrenmenize gerek yoktur. İhtiyacınız olan her şey dilde yerleşik olarak bulunur ve test yazarken kullandığınız sözdizimi(syntax) yazacağınız kodun geri kalanında olduğu gibidir.

### Test yazmak

Test yazmak, bir kaç kurala dayanarak bir fonksiyon yazmaktan farksızdır.

*  `xxx_test.go` gibi adlandırılmış bir .go dosyasının içinde bulunmalıdır
* Test fonksiyonlarının isimlendirmesi, `Test` keywordu ile başlamalıdır
* Test fonksiyonları sadece bir argüman alır ve oda `t *testing.T`'dır.
* `*testing.T` tipini kullanabilmek için, "testing" paketini eklemelisin `import "testing"`


Şimdilik, `*testing.T` tipindeki `t` argümanınızın test çerçevesine bir "hook" olduğunu bilmeniz yeterli. Testi başarısız kılmak için, `t.Fail()` gibi metotları kullanabilirsin.

Bazı yeni konulara değindik

#### `if`
Go dilindeki if ifadeleri, diğer dillere çok benzer.

#### Variable tanımlama

`varName := value` sözdizimi ile tekrar kullanabildiğimiz ve okunabilirliği arttırmak için bazı variablelar tanımladık.

#### `t.Errorf`

`t` üzerinde `Errorf` _method_ çağırıyoruz, bu metot mesaj yazdıracak ve testi başarsız kılacak. 'f', '%q' gibi yer tutucular ile yer tutucuların yerine gelecek yeni bir string oluşturmamıza izin veren biçimi ifade eder. Testin başarısız olmasını sağladınız da hangi sebeplerden dolayı başarısız olduğunu açıkca belirtmeniz gerekir.

Yer tutucu stringler ile alakalı daha ayrıntılı bilgiye [fmt go](https://golang.org/pkg/fmt/#hdr-Printing) belgesinden ulaşabilirsiniz. `q`, testler için oldukça kullanışlı bir yer tutucudur.

`Errorf` gibi metotlar ve bizim yazdıklarımız gibi fonksiyonlar arasındaki farka ileride değineceğiz.

### Go doc

Go'nun hayatınızın kalitesini artıracak bir diğer özelliği ise dokümantasyondur. `godoc -http :8000` komutunu çalıştararak dökümanlara yerelde göz atabilirsiniz. [localhost:8000/pkg](http://localhost:8000/pkg) adresine gittiğinizde, sisteminizde yüklü bütün paketleri göreceksiniz.

Standart kütüphanelerin büyük bir çoğunluğu örneklerle açıklanmış mükemmel dokümantasyona sahiptir. [http://localhost:8000/pkg/testing/](http://localhost:8000/pkg/testing/) adresine giderek Testing konusunda neler yapabileceğinizi görmek faydalı olacaktır.

Eğer `godoc` komutuna ulaşamıyorsanız, `go get golang.org/x/tools/cmd/godoc` yazarak edinebilirsiniz.

### Hello, SEN

Artık bir testimiz olduğuna göre, yazılımımızı güven içinde yineleyip, geliştirebiliriz.

En son örnekte, testin nasıl yazılacağına ve bir fonksiyon bildireceğine dair bir örnek yapmak için ilk önce kodumuzu yazıp, sonra o koda test kodu yazdık. Artık, ilk önce testimizi yazacağız.

Şimdiki ihtiyacımız, selamlamayı spesifik olarak kime yapacağını belirlemesini sağlamak.

Bu ihtiyacımızı bir test yazarak başlayalım. Bu temel Test Driven Development yaklaşımıdır ve testinizin gerçekten istediğimiz şeyi test ettiğinden emin olmamızı sağlar. İlk önce kodu sonra o koda ait testi yazdığınızda, kod istendiği gibi çalışmasa bile testinizin geçmeye devam etme riski vardır.

```go
package main

import "testing"

func TestHello(t *testing.T) {
	got := Hello("Chris")
	want := "Hello, Chris"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```
`go test` komutunu çalıştırınca, bir derleme hatası almalısınız

```text
./hello_test.go:6:18: too many arguments in call to Hello
    have (string)
    want ()
```

Go gibi statik olarak yazılmış bir programlama dili kullanırken _derleyiciye kulak vermek_ önemlidir. Derleyici(compiler) kodunuzun nasıl bir araya gelip çalışması gerektiğini anlar, böylece sizin bunu düşünmenize gerek kalmaz.

Bu durumda, derleyici size ne yapmanız gerektiğini söylüyor. `Hello` fonksiyonunu değiştirmeliyiz ve bir argüman almasını sağlamalıyız.

`Hello` fonksiyonunu, string türünde bir argümanı kabul edecek şekilde düzenleyin.

```go
func Hello(name string) string {
	return "Hello, world"
}
```

Bunu yapar ve tekrardan testleri çalıştırırsanız, `hello.go` dosyası compile olmayacak. Bunu, `main` fonksiyonununda olan `Hello` fonksiyonunun kullanımını güncelleyerek çözebilirsin.

```go
func main() {
	fmt.Println(Hello("world"))
}
```

Testlerinizi çalıştırdığınızda şöyle bir şey görmelisiniz.

```text
hello_test.go:10: got 'Hello, world' want 'Hello, Chris''
```

Sonunda compile olan bir programımız var fakat bu program testte yazdığımız gereksinimleri karşılamıyor.

`Hello` fonksiyonun aldığı `name` argümanını `Hello,` ile birleştirerek kullanalım ve testi geçelim.

```go
func Hello(name string) string {
	return "Hello, " + name
}
```

Testleri tekrar çalıştırdığınızda, geçmiş olmaları gerekir. Normalde Test Driven Development döngüsünün bir parçası olarak şimdi _refactor_ yapmalıyız.

### Source control ile ilgili bir not

Şimdi, test ile desteklenmiş ve çalışan bir yazılımımız var. Eğer bir source control sistemi kullanıyorsanız(kesinlikle kullanıyor olmalısınız), kodu olduğu gibi `commit` ederdim.

Yinede default(main veya master) branch'e pushlamazdım commit'i, çünkü sonradan refactor etmeyi düşünüyorum. Tekrardan refactor edeceğim için herhangi bir karışıklık oluşursa elimizde geri dönebileceğimiz bir versiyonun bulunması güzel, onun için `commit` atabiliriz.

Burada gözden geçirecek, tekrardan düzenleyecek pek bir şey yok. Şimdiyse, başka bir programlama dili özelliği olan _constant_'ı tanıyabiliriz.

### Constantlar

Constantlar şu şekilde tanımlanır

```go
const englishHelloPrefix = "Hello, "
```

Kodumuzu constantlar ile refactor edebiliriz

```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
	return englishHelloPrefix + name
}
```
Refactor ettikten sonra hiçbir bir şeyi bozmadığımızdan emin olmak için testleri tekrar çalıştırın.

Bu constant tanımı, her seferinden tekrardan `"Hello, "`'un oluşturulmasından kurtaracağı için uygulamanızın performansını arttırır.

Açık olmak gerekirse, çok ahım şahım bir performans artışı olmayacak yani bunu göz ardı edebilirsiniz. Fakat bazı değerleri anlamladırmak ve bazen de performansa artışına yardımcı olması için constantları kullanmayı düşünebilirsiniz.

## Tekrardan "Hello, world"

Şimdiki ihtiyacımız ise fonksiyonumuz boş bir string değeri ile çağrıldığında varsayılan olarak "Hello, " yerine "Hello, World" yazması.

Yeni bir başarısız olacak test yazarak başlayalım.

```go
func TestHello(t *testing.T) {
	t.Run("saying hello to people", func(t *testing.T) {
		got := Hello("Chris")
		want := "Hello, Chris"

		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	})
	t.Run("say 'Hello, World' when an empty string is supplied", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"

		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	})
}
```

Burada subtestleri görüyorsunuz, bu testing için kullandığımız yapının içindeki başka bir araçtır. Bazen testleri bir şeyler etrafında gruplamak ve ardından farklı senaryolarda subtestler yazmaya ihtiyaç duyarız.

Bu yaklaşımın yararlarından biri, diğer testlerde de kullanılabilecek paylaşımlı kodlar barındırabilmek.

Tekrar eden bir kod var gördüğünüz gibi, bu kod mesajın nasıl olması gerektiğini kontrol ettiğimiz kod.

Refactor etmek sadece production kodları için geçerli değildir, test kodlarımızı da refactor ederiz.

Testlerinizde bulunan kodların da _açık ve net_ olması önemlidir.

Test kodlarımızı refactor edelim,

```go
func TestHello(t *testing.T) {
	assertCorrectMessage := func(t testing.TB, got, want string) {
		t.Helper()
		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	}

	t.Run("saying hello to people", func(t *testing.T) {
		got := Hello("Chris")
		want := "Hello, Chris"
		assertCorrectMessage(t, got, want)
	})
	t.Run("empty string defaults to 'World'", func(t *testing.T) {
		got := Hello("")
		want := "Hello, World"
		assertCorrectMessage(t, got, want)
	})
}
```

Şimdi biz burada ne yaptık?

Testlerin geçmesi için olması gerektiği durumları, assertion olarak bir fonksiyona dönüştürdük. Bu yaptığımız tekrarlamayı azaltıyor ve test kodlarımızın okunabilirliğini arttırıyor. Go'da fonksiyonların içinde başka bir fonksiyon tanımlaması yapabilirsiniz ve bunları değişkenlere atıyabilirsiniz. Fonksiyonların içinde tanımlanan fonksiyonları, normal tanımlanan fonksiyonlar gibi çağırabilirsiniz. Gerektiğinde test kodunun başarısız olduğunu söylemek için `t *testing.T`'i parametre olarak pass etmeliyiz.

Helper fonksiyonunda kullandığımız `testing.TB`, `testing.T` veya `testing.B` tipindeki interfaceleri kabul etmesi için oluşturulmuş başka bir interfacedir. Böylelikle benchmark veya test fonksiyonlarını `testing.TB` ile çağırabiliriz.

`t.Helper()`, test kodlarına bu yöntemin bir yardımcı olduğunu söylemek için gereklidir. Bunu yaparak, testler başarısız olduğunda hangi satır ile testin başarısız olduğu bildirilirken, yardımcı fonksiyonun içindeki satırlar değil, fonksiyon çağrımızı yaptığımız satırlarda hata olduğu bildirilecektir. Bu sayede, geliştiriciler hangi testin başarısız olduğunu kolayca takip edebilecektir. Eğer tam olarak anlamadıysanız, herhangi bir testi başarısız olacak şekile getirin(şuan zaten öyle) ve test çıktısını gözlemleyin ve sonrasında aynı hatalı test kodunu `t.Helper()` kodunun başına `//` ekleyerek gözlemleyin.

Şimdi `Hello()` fonksiyonunu tekrardan refactor ederek, istediğimize ulaşabiliriz.

```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
	if name == "" {
		name = "World"
	}
	return englishHelloPrefix + name
}
```

Testlerimizi çalıştırırsak, yeni eklediğimiz gereksinimi karşıladığını ve yanlışlıkla başka fonksiyonları bozmadığımızı görebiliriz.

### Tekrardan source control

Şimdi yazdığımız koddan memnunuz, tekrardan bu kodları `commit` edebiliriz.

### Disiplin

TDD disiplini ile ilgili döngüyü tekrar gözden geçirelim,

* Bir test yazın
* Yazdığımız test ile birlikte kodumuzun compile edilmesini sağlayacak duruma getirin
* Testi çalıştırın, başarısz olduğunu görün ve hata mesajının anlamlı olup, olmadığını kontrol edin
* Testi geçecek kadar kod yazın
* Refactor edin

İlk bakışta bu sıkıcı görünebilir, fakat bu döngüye bağlı kalmak önemlidir.

Bu yaptığımız sadece ihtiyacınız olan testlere sahip olmanızı sağlamakla kalmaz, aynı zamanda testlerin verdiği güven ile refactorler yaparak çok daha iyi yazılım tasarımlarına sahip olmanızı sağlar.

Testlerin başarısız olduğunu da görmek gerekir, çünkü bu bizlere hata mesajlarının nasıl göründüğü gösterir. Başarısız test mesajlarının sorun ile ilgili net bir fikir vermediği durumlarda, sorunu anlamak ve çözüme kavuşturmak çok zor olur ve hiçbir geliştirici böyle bir code base'de çalışmak istemez.

By ensuring your tests are _fast_ and setting up your tools so that running tests is simple you can get in to a state of flow when writing your code.

Test yazmayarak uzun vadede çok büyük zaman kayıpları yaşarsınız, çünkü herhangi bir değişiklik yaptığınızda yaptığınız değişikliği sizin manuel olarak test etmeniz gerekir ve bu uzun vadede size çok büyük zaman kayıpları yaşatır.

## Devam et! Daha fazla gereksinim

Kodumuzun yapması gereken daha fazla gereksinim var, şimdiyse selamlamanın hangi dilde olacağını belirleyen ikinci bir parametre daha eklemeliyiz. Eğer destek verilmeyen, tanınmayan bir dil geçilirse varsayılan olarak İngilizce'yi kullanacağız.

Bu özelliği, TDD kullanarak kolayca geliştirebiliyor olmamızdan emin olmalıyız.

Mevcut test kodlarına Ispanyolca ile selamlama için bir test yazın.

```go
	t.Run("in Spanish", func(t *testing.T) {
		got := Hello("Elodie", "Spanish")
		want := "Hola, Elodie"
		assertCorrectMessage(t, got, want)
	})
```
Hile yapmak yok! Unutma **ne olursa olsun, ilk önce test yazılacak**. Şimdi test kodlarınızı çalıştırdığınızda, compiler hata verecek bunun sebebi, `Hello()` fonksiyonu şuanda tek bir parametre alıyor fakat biz iki parametre gönderdik.

```text
./hello_test.go:27:19: too many arguments in call to Hello
    have (string, string)
    want (string)
```
Derleme sorunlarını `Hello()` fonksiyonuna bir string parametre daha almasını sağlayarak çözebiliriz.

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}
	return englishHelloPrefix + name
}
```

Testi tekrar çalıştırdığınızda compiler, bu seferse önceden yazılmış testlerin tek parametre ile çağrıldığından şikayetci olacak.

```text
./hello.go:15:19: not enough arguments in call to Hello
    have (string)
    want (string, string)
```

Diğer testlerdeki fonksiyonu çağrılarına, ikinci bir parametre olarak boş bir string yollayın ve artık yazılımınız derlenecek, yeni senaryomuz dışındaysa bütün testler başarılı olacaktır.

```text
hello_test.go:29: got 'Hello, Elodie' want 'Hola, Elodie'
```

Gönderilen dilin İspanyolcaya eşit olup, olmadığını kontrol etmek için `if` kullanıp, o dile göre selamlama yapabiliriz.

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == "Spanish" {
		return "Hola, " + name
	}
	return englishHelloPrefix + name
}
```

Testlerin hepsi şuanda başarılı olmalıdır.

Şimdiyse yeniden refactor etme zamanı. Görmüş olmalısınız, kodumuzda kendini tekrar eden kısımlar var. İlk önce bu kodu kendiniz düzenlemeye çalışın ve her düzenleme de herhangi bir şeyi bozmadığınızdan emin olmak için testleri çalıştırın.

```go
const spanish = "Spanish"
const englishHelloPrefix = "Hello, "
const spanishHelloPrefix = "Hola, "

func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == spanish {
		return spanishHelloPrefix + name
	}
	return englishHelloPrefix + name
}
```

### French

* Dil parametresi için `"French"` yollandığı zaman, `"Bonjour, "` ile selamlanacağını iddia ettiğiniz bir test yazın
* Testin başarısız olduğunu görün ve hata mesajının anlaşılabilir olup, olmadığını kontrol edin
* Sonra Fransızca için yazılan testi geçmek için, kodda en az değişikliği yapın.

Aşağı yukarı aşağıdaki koda benzer bir şeyler yazmış olmalısınız.

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	if language == spanish {
		return spanishHelloPrefix + name
	}
	if language == french {
		return frenchHelloPrefix + name
	}
	return englishHelloPrefix + name
}
```

## `switch`

Belirli, ortak bir değeri kontrol ederken birçok `if` ifadesi kullanmak yerine, `switch` kullanın. Daha sonra farklı dillerde de destek vermek istersek okumayı kolaylaştırması ve genişletilebilir olması için `switch` kullanmak çok daha iyi olur.

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	prefix := englishHelloPrefix

	switch language {
	case french:
		prefix = frenchHelloPrefix
	case spanish:
		prefix = spanishHelloPrefix
	}

	return prefix + name
}
```

Şimdi seçtiğiniz bir konuşma dili için test yazın, mesela Türkçe desteği ekleyin ve yeni dile destek vermek için kodunuzu değiştirmenin, genişletmenin ne kadar kolay olduğunu görün.

### son...bir...refactor?

`Hello()` fonksiyonunun çok kalabalıklaştığını, okumanın zorlaştığını iddia edebilirsiniz ve haklısınız. Ortak olan kısımları alarak yeni bir fonksiyon haline getirerek ona güzel bir isimlendirme yapabilirsiniz.

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}

	return greetingPrefix(language) + name
}

func greetingPrefix(language string) (prefix string) {
	switch language {
	case french:
		prefix = frenchHelloPrefix
	case spanish:
		prefix = spanishHelloPrefix
	default:
		prefix = englishHelloPrefix
	}
	return
}
```

Yeni kavramlar:

- Yeni fonksiyon tanımlarken ve string bir değer dönüşü yapacağını belirttik.
- Bu yapılan, yazdığımız fonksiyonunda `prefix` adında bir değişken oluşturacak.
  - Eğer herhangi bir değer seçilmezse, `string` tipinde bir dönüş için `""` değeri dönecek. Integer bir dönüş olsaydı bu `0` olacaktı.
    - Bu şekilde bir tanım ile bir değeri dönerken, bu şekilde sadece `return`'da kullanabilirsiniz isterseniz `return prefix`'de tercih edebilirsiniz.
  - Bu Go doc'ta fonksiyonunuzun için görüntülenebilecektir ve böylece koduzun ne yaptığı daha anlaşılır hale gelir.
- Gelen parametre değeri, caselerden hiç biri ile eşleşmezse `default` seçilecektir.
- Go'da `private`(sadece o paket altında kullanılabilir, dışarıya kapalı) programlama bileşenleri küçük harf ile, `public` (dışarıya açık) bileşenlerse büyük harf ile başlar. Algoritmalarımızın içindeki bileşenlerin hepsini dışarıya açmak istemediğimiz zaman `private` tanım yapmalıyız.

## Özetlersek

`Hello, world` programı yazarken bu kadar çok şey öğrenebileceğini kim bilebilirdi?

Şimdiye kadar şunları biraz anlamış olman lazım:

### Go'nun bazı sözdizimleri

- Test yazmak
- Parametre alan ve dönüş tiplerine sahip fonksiyon tanımlamak
- `if`, `const` ve `switch`
- Değişken ve constanlar tanımlama

### TDD süreci ve bu sürecin adımlarının neden önemli olduğu

- Başarısız olacağını bilmemize rağmen test yazmak ve o testin hata mesajını görerek herhangi bir eksiklik varsa hata mesajını düzeltmek, sonraysa kodumuzu bu testin gereksinimlerine göre değiştirmek, geliştirmek
- Testi geçmek için yazdığımız az miktarda kodlar sayesinde, çalışan bir yazılımımız olduğunu biliyoruz buda bize geliştirme yaparken doğru yolda olduğumuzu ve bu yolda devam etmemiz gerektiğini belirtiyor
- Ardından refactor etmek, çalıştığını bildiğimiz ve bize güven veren testler sayesinde bir önceki seferde en az seviyede yazıp bıraktığımız kodu, refactor ederek daha iyi bir hale getiriyoruz

Bu örnekte, `Hello()`'dan başlayarak, `Hello("name")` sonrasındaysa `Hello("name", "French")` e kadar ilerledik.

Bu elbette gerçek dünyada üretilen yazılımlar için önemsizdir ancak ilkeler hep aynıdır. Sizin burada öğrendiğiniz disiplini ve TDD akışını gerçek dünyada üretilen yazılımları yazan mühendislerde kullanıyor. TDD yaklaşımı ile geliştirme yapmak için bolca pratik yapmalısınız, fakat yazılımlarınızda sorunları test edebilmek için küçük parçalara bölmek işleri her zaman kolaylaştırır.

Bu sayfa [@halilkocaoz](https://github.com/halilkocaoz) tarafından çevrildi.
