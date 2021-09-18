# Iterasyon

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/for)**

Go'da tekrarlı işler için `for`'a ihtiyacınız var. Go içerisinde `while`, `do`, `until` anahtar kelimeleri yoktur sadece `for` kullanabilirsiniz ve bu iyi bir şey!

Hadi bir karakteri 5 kez tekrar eden bir fonksiyon için test yazalım

Şimdiye kadar yeni bir şey yok, bu yüzden pratik yapmak için kendiniz yazmaya çalışın.

## İlk olarak test yaz

```go
package iteration

import "testing"

func TestRepeat(t *testing.T) {
	repeated := Repeat("a")
	expected := "aaaaa"

	if repeated != expected {
		t.Errorf("expected %q but got %q", expected, repeated)
	}
}
```

## Dene ve testi çalıştır

`./repeat_test.go:6:14: undefined: Repeat`

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

_Disiplini koruyun!_ Testin düzgün bir şekilde başarısız olması için şu anda yeni bir şey bilmenize gerek yok.

Yapmanız gereken tek şey kodu derlemek için yeterli değişikliği yapmak, bu sayede kodunuzun iyi yazıldığını kontrol etmek için test edebilirsiniz.

```go
package iteration

func Repeat(character string) string {
	return ""
}
```

Bazı basit problemler için test yazacak kadar Go bilmek hoş değil mi? Bu, artık production koduyla istediğiniz kadar oynayabileceğiniz ve umduğunuz gibi davranmasını bildiğiniz anlamına gelir.

`repeat_test.go:10: expected 'aaaaa' but got ''`

## Testi geçecek kadar kod yaz

`for` sözdizimi çok dikkat çekici değildir ve çoğu C benzeri dillerde olduğu gibidir.

```go
func Repeat(character string) string {
	var repeated string
	for i := 0; i < 5; i++ {
		repeated = repeated + character
	}
	return repeated
}
```

C, Java veya JavaScript dillerinin aksine, for döngüsünün 3 bileşenini saran parantezler yoktur ve süslü parantezler `{ }` her zaman zorunludur. Muhtemelen şu satırda ne olduğunu merak ediyorsunuz.

```go
	var repeated string
```

Değişken tanımlamak için bir süre `:=` kullanmıştık ancak `:=` basitçe [her iki adım için kısa yol](https://gobyexample.com/variables). Burada sadece `string` değişkeni tanımladık. Aynı zamanda `var` ile fonksiyon tanımlayabildiğimizi ileride göreceğiz.

Testi çalıştırın ve geçtiğini görün.

for döngüsünün farklı kullanımlarını [burada](https://gobyexample.com/for) bulabilirsiniz.

## Refactor

Şimdi düzenlemenin ve yeni bir atama `+=` operatörünü tanıtma zamanı.

```go
const repeatCount = 5

func Repeat(character string) string {
	var repeated string
	for i := 0; i < repeatCount; i++ {
		repeated += character
	}
	return repeated
}
```

`+=` operatörü _"ekle ve ata operatörü"_ olarak bilinir. Sağdaki değeri soldaki depere ekler ve sonucu soldaki değere atar. Integerlar gibi diğer tiplerlede çalışır.

### Kıyaslama

Go'da [benchmark (kıyaslama)](https://golang.org/pkg/testing/#hdr-Benchmarks) yazmak dilin diğer birinci sınıf özelliklerindendir ve test yazmaya çok benzer.

```go
func BenchmarkRepeat(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Repeat("a")
	}
}
```

Kodun teste çok benzediğni sizde göreceksiniz.

`testing.B` şifreli olarak adlandırılan `b.N`'e erişmenizi sağlar.

Kıyaslama (benchmark) kodu çalıştırıldığında `b.N` kere çalışır ve ne kadar sürdüğünü ölçer.

Kodun çalışma süresni önemsememelisiniz, framework bu durum için neyin "iyi" bir değer olduğuna karar verecek ve bazı iyi sonuçlar elde etmeniz için size izin verecek.

Benchmarkı çalıştırmak için `go test -bench=.` (Windows Powershell'de iseniz `go test -bench="."`)

```text
goos: darwin
goarch: amd64
pkg: github.com/quii/learn-go-with-tests/for/v4
10000000           136 ns/op
PASS
```

`136 ns/op` fonksiyonumuzun çalışma süresinin ortalama 136 nanosaniye \(benim bilgisayarımda\) sürdüğünü gösteriri. Bu iyi bir değer! Bunu test etmek için 10000000 kere çalıştı.

_NOT_ Benchmarklar varsayılan olarak ardışık çalışır.

## Alıştırmaları yap

- Testi değiştir, bu sayede çağıran kişi karakterin kaç kez tekrarlanacağını belirleyebilir ve sonradan kodu düzeltebilir
- Fonksiyonunu dokümente etmek için `ExampleRepeat` yaz
- [Strings](https://golang.org/pkg/strings) paketine bir göz atın. Yararlı olabileceğini düşündüğünüz fonksiyonları bulun ve burada yaptığımız gibi testler yazarak bunları deneyin. Standard kütüphaneyi öğrenmek için zaman ayırmak ileride gerçekten işe yarayacak.

## Özetlersek

- Daha fazla TDD pratiği
- `for` öğrenildi
- Benchmarkın (kıyaslama) nasıl yazıldığı öğrenildi

Bu sayfa [@bariscanyilmaz](https://github.com/bariscanyilmaz) tarafından çevrildi.
