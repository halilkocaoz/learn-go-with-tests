# Dependency Injection

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/di)**

Bunun için interfaceler hakkında biraz bilgi sahibi olmanız gerekeceğinden, structler bölümünü daha önce okuduğunuz varsayılmaktadır.

Programlama topluluklarında dependency injection konusunda _birçok_ yanlış anlama var. Umarım, bu kılavuz size nasıl yapılacağını gösterecektir.

* Framework'e ihtiyacınız yok
* Tasarımınızı karmaşıklaştırmaz
* Testi kolaylaştırır
* Kullanışlı, harika fonksiyonlar yazmanıza izin verir.

hello-world bölümünde yaptığımız gibi, birini selamlayan bir fonksiyon yazmak istiyoruz ama bu sefer varolan çıktıyı _(actual printing)_ test edeceğiz.

Özetlemek gerekirse, fonksiyon bu şekilde gözüküyor

```go
func Greet(name string) {
	fmt.Printf("Hello, %s", name)
}
```

Fakat bunu nasıl test edeceğiz? `fmt.Printf` çıktılarını stdout'a çağırmak, test frameworkü kullanarak yapmamız çok zor.

Yapmamız gereken, yazdırmanın bağımlılığını **inject etmektir** \(bu sadece içeri girmek için süslü bir kelime)\.

**Yazdırma işleminin **_**nerede**_** veya **_**nasıl**_** olduğuna fonksiyonumuz önemsememeli, Bu yüzden concrete tip yerine **_**interface**_** kabul etmeliyiz.**

Bunu yaparsak, uygulamayı kontrol ettiğimiz bir şeye yazdıracak şekilde değiştirebiliriz, böylece test edebiliriz."Gerçek hayatta" stdout'a yazan bir şey inject edersiniz.

Eğer [`fmt.Printf`](https://pkg.go.dev/fmt#Printf) kaynak koduna bakarsanız bağlanabileceğimiz bir yol görebilirsiniz

```go
// yazılan byteların sayısını ve gerçekleşmiş write error hatasını döner.
func Printf(format string, a ...interface{}) (n int, err error) {
	return Fprintf(os.Stdout, format, a...)
}
```

İlginç! `Printf` altında sadece `Fprintf` çağırıyor ve `os.Stdout`'u parametre olarak gönderiyor.

 Tam olarak `os.Stdout` nedir? `Fprintf`, 1. argüman için kendisine ne iletilmesini bekliyor?

```go
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
	p := newPrinter()
	p.doPrintf(format, a)
	n, err = w.Write(p.buf)
	p.free()
	return
}
```

Bir `io.Writer`

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

Buradan `os.Stdout`'un `io.Writer` implemente ettiğini sonucunu çıkarabiliriz; `Printf`, `os.Stdout`'u `io.Writer` bekleyen `Fprintf`'e gönderir.

Daha fazla Go kodu yazdıkça, bu arayüzün çok fazla ortaya çıktığını göreceksiniz çünkü "bu verileri bir yere koymak" için harika, kullanışlı bir interface.

Dolayısıyla, greetingi örtülü bir şekilde bir yere göndermek için nihayetinde `Writer`'ı kullandığımızı biliyoruz. Kodumuzu test edilebilir ve daha fazla yeniden kullanılabilir hale getirmek için bu mevcut soyutlamayı kullanalım.

## İlk olarak test yaz

```go
func TestGreet(t *testing.T) {
	buffer := bytes.Buffer{}
	Greet(&buffer, "Chris")

	got := buffer.String()
	want := "Hello, Chris"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

`bytes` paketindeki `Buffer` tipi,`Writer` interfaceini implemente eder, çünkü `Write(p []byte) (n int, err error)` metoduna sahip.

Onu `Writer` olarak göndermek için testimizde kullanacağız `Greet`'i çağırdıktan sonra ona ne yazıldığını kontrol edebiliriz.

## Dene ve testi çalıştır

Test derlenmeyecektir

```text
./di_test.go:10:7: too many arguments in call to Greet
    have (*bytes.Buffer, string)
    want (string)
```

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

_Derleyiciyi dinle_ ve problemi düzelt.

```go
func Greet(writer *bytes.Buffer, name string) {
	fmt.Printf("Hello, %s", name)
}
```

`Hello, Chris di_test.go:16: got '' want 'Hello, Chris'`

Test başarısız oluyor. Adın yazdırıldığına dikkat edin, ancak stdout'a gidecek.

## Testi geçecek kadar kod yaz

Testimizde, greetingi buffera göndermek için writerı kullan. `fmt.Fprintf`'in `fmt.Printf` gibi olduğunu hatırla ancak string göndermek için `Writer`'ı alır, oysa `fmt.Printf` varsayılan olarak stdouttur.

```go
func Greet(writer *bytes.Buffer, name string) {
	fmt.Fprintf(writer, "Hello, %s", name)
}
```

Şimdi testten geçiyor.

## Refactor

Daha önce derleyici bize 'bytes.Buffer' için pointer göndermemizi söyledi. Bu teknik olarak doğru ama çok kullanışlı değil.

Bunu göstermek için, `Greet` fonksiyonunu stdout'a yazdırmasını istediğimiz bir Go uygulamasına bağlamayı deneyin.

```go
func main() {
	Greet(os.Stdout, "Elodie")
}
```

`./di.go:14:7: cannot use os.Stdout (type *os.File) as type *bytes.Buffer in argument to Greet`

Daha önce tartışıldığı gibi, `fmt.Fprintf`, hem `os.Stdout` hem de `bytes.Buffer` implementationını bildiğimiz bir `io.Writer` göndermenize izin verir.

Eğer kodumuzu daha kullanışlı bir interface çevirirsek onu hem testlerde hem de uygulamada kullanabilirizi..

```go
package main

import (
    "fmt"
    "os"
    "io"
)

func Greet(writer io.Writer, name string) {
    fmt.Fprintf(writer, "Hello, %s", name)
}

func main() {
	Greet(os.Stdout, "Elodie")
}
```

## io.Writer hakkında daha fazla

`io.Writer` kullanarak başka hangi yerlere veri yazabiliriz? ``Greet`` fonksiyonumuz ne kadar kullanışlı?

### İnternet

Aşağıdakileri çalıştırın

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
)

func Greet(writer io.Writer, name string) {
    fmt.Fprintf(writer, "Hello, %s", name)
}

func MyGreeterHandler(w http.ResponseWriter, r *http.Request) {
    Greet(w, "world")
}

func main() {
    log.Fatal(http.ListenAndServe(":5000", http.HandlerFunc(MyGreeterHandler)))
}
```

Programı çalıştırın ve [http://localhost:5000](http://localhost:5000) adresine gidin. Greeting fonksiyonunun kullanıldığını göreceksiniz.

HTTP serverlar  sonraki konularda bahsedilecek o yüzden detaylar hakkında çok fazla endişe etmeyin.

HTTP handler yazdığınızda, size `http.ResponseWriter` istek (request) yapmak için kullanılan `http.Request` verilir. Sunucuyu implemente ettiğinizde, writer kullanarak cevabınızı _yazarsınız_.

`http.ResponseWriter`'ın `io.Writer`'ı implemente ettiğini tahmin ediyorsunuzdur, bu sayede handlerımızda `Greet` fonksiyonumuzu kullanabiliriz.

## Özetlersek

Kodumuzun ilk halini test etmek kolay değildi çünkü kontrol edemediğimiz bir yere veri yazıyordu.

_Testlerimizden motive olarak_ kodu refactor ettik, böylece verilerin **bir bağımlılık inject ederek** _nerede_ yazıldığını kontrol edebildik, bu da bize şunları yapmamızı sağladı:

* **Kodu test etme* Eğer bir fonksiyonu _kolayca_ test edemiyorsanız, bunun nedeni genellikle fonksiyona veya global bir duruma bağımlılıklarıdır. Örneğin, bir tür hizmet katmanı tarafından kullanılan global bir veritabanı bağlantı havuzunuz varsa, test edilmesi muhtemelen zor olacak ve çalıştırması yavaş olacaktır. DI sizi, testlerinizi kontrol edebileceğiniz, mocklayabileceğiniz veritabanı bağımlılığı \(bir interface araclığı ile)\ inject etmeye motive eder.
* **Separate our concerns (Bağlantıların ayırma)**, _verilerin nasıl oluştuğu ve nereden nereye gittiği_ ayrılması.Bir fonksiyonun/metodun çok fazla sorumluluğu olduğunu düşünüyorsanız \(veri oluşturma  _ve_ db'ye yazma? HTTP isteklerini handle etme _ve_ domain seviyesinde mantık?\) DI ihtiyaç duyacağınız araçtır.
* **Kodumuzun farklı contextlerde yeniden kullanılması** Kodumuzun kullanabileceği ilk "yeni" context, testlerin içindedir. Biri sizin fonksiyonlarınıza yeni bir şey denemek isterse, kendi bağımlılıklarını inject edebilir.

### Mocking'e ne dersin? DI için buna ihtiyacın olduğunu ve aynı zamanda bunun kötü olduğunu duydum

Mocking detaylıca ileride ele alınacaktır \(ayrıca kötü değil\). Inject ettiğiniz gerçek şeyleri, testlerinizde kontrol edebileceğiniz ve inceleyebileceğiniz taklit bir sürümle değiştirmek için mockingi kullanırsınız. Bizim durumumuzda, standart kütüphanenin bizim için kullanıma hazır bir şeyi vardı.

### Go standard kütüphanesi gerçekten faydalı, üzerinde çalışmak için zaman ayır

`io.Writer` interfaceine aşina oldukça, testimizde `bytes.Buffer`'ı `Writer` olarak kullanabileceğiz ve fonksiyonumuzu komut satırı uygulamasında veya web sunucusunda kullanmak için standart kütüphaneden diğer `Writer`ları kullanabiliriz.

Standart kitaplığa ne kadar aşina olursanız, yazılımınızı çeşitli contextlerinde yeniden kullanılabilir hale getirmek için kendi kodunuzda yeniden kullanabileceğiniz bu kullanışlı interfaceleri o kadar çok görürsünüz.

Bu örnek, [The Go Programming language](https://www.amazon.co.uk/Programming-Language-Addison-Wesley-Professional-Computing/dp/0134190440) kitabından oldukça etkilenmiştir, eğer beğendiyseniz satın alın!

Bu sayfa [@bariscanyilmaz](https://github.com/bariscanyilmaz) tarafından çevrildi.
