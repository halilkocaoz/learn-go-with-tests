# Arrayler ve slicelar

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/arrays)**

Arrayler aynı tipte birden çok elementi bir değişken içerisinde
belirli bir sırada saklamanızı sağlar.

Arrayler üzerinde iterasyon yapmanız çok yaygındır.Hadi, `Sum` fonksiyonunu yapmak için [yeni öğrendiğimiz `for`'u](iteration.md) kullanalım. `Sum` sayıların olduğu bir array alacak ve toplamı dönecek.

Hadi, TDD yeteneklerimizi kullanalım.

## İlk olarak test yaz

Çalışmak için yeni bir klasör oluşturun.`sum_test.go` isminde bir dosya oluşturun ve aşağıdakileri ekleyin:

```go
package main

import "testing"

func TestSum(t *testing.T) {

	numbers := [5]int{1, 2, 3, 4, 5}

	got := Sum(numbers)
	want := 15

	if got != want {
		t.Errorf("got %d want %d given, %v", got, want, numbers)
	}
}
```

Arraylar _sabit bir kapasiteye_ sahiptirler, bir array değişkenini tanımlarken kapasitesini de belirtiriz.
Bir arrayi iki yolla oluşturabiliriz:

* \[N\]type{value1, value2, ..., valueN} e.g. `numbers := [5]int{1, 2, 3, 4, 5}`
* \[...\]type{value1, value2, ..., valueN} e.g. `numbers := [...]int{1, 2, 3, 4, 5}`

Bazen hata mesajlarında girdileri yazdırmak faydalıdır.
Burada arraylerde de iyi çalışan `%v` fotmatlayıcısını değerleri "varsayılan" formatta yazdırması için kullandık.

[String formatları hakkında daha fazla bilgi edinin](https://golang.org/pkg/fmt/)

## Testi çalıştırmayı dene

`go test` komutunu çalıştırdığınızda derleyici `./sum_test.go:10:15: undefined: Sum` hatasını verecek.

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`sum.go` içerisine

```go
package main

func Sum(numbers [5]int) int {
	return 0
}
```

Şimdi testiniz _temiz bir hata mesajı_ verecek.

`sum_test.go:13: got 0 want 15 given, [1 2 3 4 5]`

## Testi geçecek kadar kod yaz

```go
func Sum(numbers [5]int) int {
	sum := 0
	for i := 0; i < 5; i++ {
		sum += numbers[i]
	}
	return sum
}
```

Bir arrayin belirli bir indeksindeki değere ulaşmak için `array[index]`
sözdizimini kullanın. Bu durumda, `for` kullanarak array üzerinde 5 kez iterasyon yaparak array içerisindeki her değeri `sum` değişkenine ekliyoruz.

## Refactor

Kodumuzu temizlemeye yardımcı olması için [`range`](https://gobyexample.com/range) anahtar kelimesini tanıtalım

```go
func Sum(numbers [5]int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}
	return sum
}
```

`range` array üzerinde iterasyon yapmanızı sağlar. `range`, her iterasyonda indeksı ve indeksin değerini döner.
([Blank Identifier](https://golang.org/doc/effective_go.html#blank)) `_` karakterini kullanarak indeks değerini yok sayabiliriz.

### Arrayler ve tipleri

Arraylarin ilginç bir özelliği, boyutun kendi tipinde kodlanmış olmasıdır. Eğer `[4]int` arrayini `[5]int` bekleyen bir fonksiyona gönderirseniz, kod derlenmeyecektir.
Bunlar farklı tiplerdir bu işlem aynı `int` bekleyen bir fonksiyona `string` göndermek gibidir.

Arraylarin sabit bir uzunluğa sahip olmasının oldukça zahmetli olduğunu düşünüyor olabilirsiniz ve muhtemelen çoğu zaman onları kullanmayacaksınız!

Go, koleksiyonunun boyutunu önemsemediği hatta herhangi bir boyutta olabilecek olan _slice_ tipine sahiptir.

Bir sonraki koşul, farklı boyutlardaki koleksiyonları toplamak olacak.

## İlk olarak test yaz

Herhangi bir boyutta koleksiyonlara sahip olmamızı sağlayan [slice][slice] tipini kullancağız. Sözdizimi arraylere çok benziyor sadece değişkeni tanımlarken boyutunu kaldırın.

`myArray := [3]int{1,2,3}` yerine `mySlice := []int{1,2,3}`

```go
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := [5]int{1, 2, 3, 4, 5}

		got := Sum(numbers)
		want := 15

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

}
```

## Dene ve testi çalıştır

Bu derlenmeyecektir

`./sum_test.go:22:13: cannot use numbers (type []int) as type [5]int in argument to Sum`

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

Bizimde yapabileceğimiz, buradaki problem

* `Sum` ın argümanını arrayden slicelara alarak var olan API'yı bozmak.  Bunu yaptığımızda muhtemelen başkasının gününü mahfedeceğiz çünkü _diğer_ testimiz derlenmeyecek!
* Yeni bir fonksiyon oluşturmak

Bizim durumumuzda, fonksiyonumuzu başka hiç kimse kullanmıyor, bu yüzden bakımı yapılacak iki fonksiyona sahip olmak yerine, sadece bir fonksiyona sahip olalım.

```go
func Sum(numbers []int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}
	return sum
}
```

Eğer testleri çalıştırmayı denerseniz derlenmeyecektir, ilk testin çalışabilmesi için array yerine slice'a çevirmelisiniz.

## Testi geçecek kadar kod yaz

Burada yapmamız gereken tek şey derleyici hatalarını gidermek ve testlerin geçmesini sağlamak!

## Refactor

`Sum` fonksiyonunu önceden düzenlemiştik  - Tek yaptığımız arrayler yerine slicelar yazmak, ekstra bir değişikliğe gerek yok.
Yeniden düzenleme aşamasında test kodumuzu ihmal etmememiz gerektiğini unutmayın - `Sum` testlerimizi daha da geliştirebiliriz.

```go
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := []int{1, 2, 3, 4, 5}

		got := Sum(numbers)
		want := 15

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

}
```

Testlerimizin değerini sorgulamak önemlidir. Mümkün olduğu kadar çok test olması bir amaç olmamalı ancak kod tabınında (code base) mümkün olduğu kadar tatminkarlık olmalı. Çok fazla testin olması gerçek bir probleme dönüşebilir ve bakımı için ekstra yük olabilir. **Her testin bir maliyeti vardır**.

Bizim durumumuzda, bu fonksiyon için iki test olması gereksiz.
Eğer bir slice için belirli bir boyutta çalışıyorsa \(makul ölçüde\) her boyuttaki slice için çalışacaktır.

Go'nun dahili test aracı [kapsama aracı (coverage tool)](https://blog.golang.org/cover) özelliğini içerir.
%100 kapsama için çabalamak nihai hedefiniz olmamalıdır, kapsama aracı kodunuzun testler tarafından kapsanmayan alanlarını belirlemenizde size yardımcı olabilir. TDD konusunda katıysanız,
100%'e yakın kapsama oranına sahip olmanız çok olasıdır.

Çalıştırmayı deneyin

`go test -cover`

Görmelisiniz

```bash
PASS
coverage: 100.0% of statements
```

Şimdi testlerden birini silin ve kapsama oranını bir daha kontrol edin.

Artık iyi test edilmiş bir fonksiyonumuz olduğu için mutluyuz sonraki mücadeleye girmeden önce yapmış olduğunuz harika işi teslim (commit) etmelisiniz.

`SumAll` isminde yeni bir fonksiyona ihtiyacımız var. Bu fonksiyon çeşitli sayıda sliceı parametre olarak alacak, yeni bir slice dönecek ve bu slice her sliceın toplamını içerecek.

Örneğin

`SumAll([]int{1,2}, []int{0,9})` fonksiyonu `[]int{3, 9}` döndürmeli

veya

`SumAll([]int{1,1,1})` fonskiyonu  `[]int{3}` döndürmeli

## İlk olarak test yaz

```go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if got != want {
		t.Errorf("got %v want %v", got, want)
	}
}
```

## Dene ve testi çalıştır

`./sum_test.go:23:9: undefined: SumAll`

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

Bizim testimize göre `SumAll` fonksiyonunu tanımlamalıyız.

Go çeşitli sayıda argüman alan [_(variadic functions)_](https://gobyexample.com/variadic-functions) yazmanıza izin verir.

```go
func SumAll(numbersToSum ...[]int) (sums []int) {
	return
}
```

Bu geçerlidir ancak testlerimiz hala derlenmeyecektir!

`./sum_test.go:26:9: invalid operation: got != want (slice can only be compared to nil)`

Go slicelar ile eşittir operatörünü kullanmanıza izin vermez. Her `got` ve `want` slicelarını iterate etmesi için bir fonksiyon yazabilirsiniz ancak  kolaylık olması için, herhangi iki değişkenin aynı olup olmadığını görmemizde kullanışlı olan, [`reflect.DeepEqual`][deepEqual] kullanabiliriz.

```go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

\(`DeepEqual` fonksiyonuna erişebilmek için dosyanın en yukarısında `import reflect` ekli olduğundan emin olun\)

Bunu belirtmek önemli, `reflect.DeepEqual` "type safe" güvenli tipte değildir - saçma bir şey yapsanızda kodunuz derlenecektir. Uygulamada bunu görmek için geçici olarak testi değiştirin:

```go
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := "bob"

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

Burada yaptığımız şey bir `slice` ile bir `string`'ı kıyaslamak. Bu kıyaslamayı yapmak mantıklı değil ancak test derleniyor! `reflect.DeepEqual` slicelar ile diğer şeyleri kıyaslamak için kullanışlı,  kullanırken dikkatli olmalısınız.

Testi eski haline getirin ve çalıştırın. Aşağıdakine benzer bir test çıktısı elde etmelisiniz

`sum_test.go:30: got [] want [3 9]`

## Testi geçecek kadar kod yaz

Burada yapmamız gerekn varargs üzerinde iterate etmek, `Sum` fonksiyonu ile toplamı hesaplamak, sonucu döneceğimiz olan slice'a eklemek

```go
func SumAll(numbersToSum ...[]int) []int {
	lengthOfNumbers := len(numbersToSum)
	sums := make([]int, lengthOfNumbers)

	for i, numbers := range numbersToSum {
		sums[i] = Sum(numbers)
	}

	return sums
}
```

Öğrenilecek birçok yeni şey!

Slice oluşturmak için yeni bir yol. `make`, üzerinde çalışacağımız `numbersToSum`'ın `len` ile elde ettiğimiz uzunluğunu kullanarak slice'ın kapasitesini belirlememizi sağlar.

Slicelar da arrayler gibi `mySlice[N]` indeksteki değere erişebilir veya
`=` ile yeni bir değer atayabilirsiniz.

Testler şimdi geçmeli.

## Refactor

Daha önce bahsettiğimiz gibi, sliceların kapasitesi vardır. Eğer slicesınızın boyutu 2 ve `mySlice[10] = 1` işlemini yapmaya çalışırsanız  _çalışma sırasında (runtime)_ hatası alırsınız.

Bunun yanı sıra, `append` fonksiyonuna slice ve yeni bir değer göndererek yeni bir slice elde edebilirsiniz.

```go
func SumAll(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		sums = append(sums, Sum(numbers))
	}

	return sums
}
```

Bu uyarlamada, kapasite hakkında daha az endişeliyiz. Boş bir slice olan `sums` ile başlıyoruz ve `Sum` fonksiyonunun sonucunu ekliyoruz.

Sıradaki koşulumuz `SumAll` fonksiyonunu `SumAllTails`'a çevirmek, bu sayede her sliceın "tails" değerini hesaplayacağız. Tail (kuyruk),
koleksiyonun ilk değeri \("kafa"\) haric tüm değerlerdir.

## İlk olarak test yaz

```go
func TestSumAllTails(t *testing.T) {
	got := SumAllTails([]int{1, 2}, []int{0, 9})
	want := []int{2, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

## Dene ve testi çalıştır

`./sum_test.go:26:9: undefined: SumAllTails`

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

Fonksiyonu`SumAllTails` olarak yeniden isimlendir ve testi tekrar çalıştır

`sum_test.go:30: got [3 9] want [2 9]`

## Testi geçecek kadar kod yaz

```go
func SumAllTails(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		tail := numbers[1:]
		sums = append(sums, Sum(tail))
	}

	return sums
}
```

Slicelar dilimlenebilir! Sözdizimi `slice[low:high]`. Eğer `:` bir tarafında ki değeri kaldırırsanız o taraf haric her değeri kapsar. Bizin durumumuzda, `numbers[1:]` ile "1. indeksten sona kadar" almasını söylüyoruz. Slicelar hakkında başka testler yazmak için biraz zaman harcamak ve daha aşina olmak için slice operatörüyle denemeler yapmak isteyebilirsiniz.

## Refactor

Bu sefer refactor için fazla bir şey yok.

Fonksiyonumuza boş bir slice gönderdiğimizde ne olacağını düşündünüz mü? Boş bir slice'ın "tail" (kuyruğu) nedir? Go'ya boş bir slice'ın `myEmptySlice[1:]` bütün elemanlarını almasını söylerseniz ne olur?

## İlk olarak test yaz

```go
func TestSumAllTails(t *testing.T) {

	t.Run("make the sums of some slices", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}

		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})

	t.Run("safely sum empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}

		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})

}
```

## Dene ve testi çalıştır

```text
panic: runtime error: slice bounds out of range [recovered]
    panic: runtime error: slice bounds out of range
```

O hayır! Test _derlendi_ ancak çalışma sırasında (runtime) hata verdi.
Derleme zamanı hataları bizim dostumuzdur çünkü çalışan yazılımlar yapmamıza yardım ediyorlar, çalışma zamanı hatalar düşmanımız çünkü kullanıcıları etkiliyorlar.

## Testi geçecek kadar kod yaz

```go
func SumAllTails(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		if len(numbers) == 0 {
			sums = append(sums, 0)
		} else {
			tail := numbers[1:]
			sums = append(sums, Sum(tail))
		}
	}

	return sums
}
```

## Refactor

Testimizde assertion etrafında yine tekrarlı kodlar var, öyleyse bunları bir fonksiyona çıkaralım.

```go
func TestSumAllTails(t *testing.T) {

	checkSums := func(t testing.TB, got, want []int) {
		t.Helper()
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	}

	t.Run("make the sums of tails of", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}
		checkSums(t, got, want)
	})

	t.Run("safely sum empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}
		checkSums(t, got, want)
	})

}
```

Bunun kullanışlı bir yan etkisi, kodumuza biraz tip güvenliği eklemesidir. Eğer geliştirici kazara yeni bir test eklerse `checkSums(t, got, "dave")` derleyici onların çalışmasını durduracak.

```bash
$ go test
./sum_test.go:52:21: cannot use "dave" (type string) as type []int in argument to checkSums
```

## Özetlersek

Ele alınanlar

* Arrayler
* Slicelar
  * Slice oluşturmanın çeşitli yolları
  * _sabit_ kapasiteli oldukları ama `append` kullanarak nasıl yenisini oluşturulacağı
  * Sliceları nasıl dilimleyeceğini!
* `len` ile array veya slice'ın uzunluğunu elde etmeyi
* Test kapsama aracını (coverage tool)
* `reflect.DeepEqual` kullanmanın neden kullanışlı olduğu ama kodumuzun tip güvenliğini (type-safety) düşürebileceği

Sliceları ve arraylari integerlar ile kullandık ancak diğer tiplerle de çalışabilirler, arrayler/sliceların kendileri de dahil. Eğer ihtiyacınız varsa `[][]string` ile tanımlayabilirsiniz.

Slicelar hakkında derinlemesine bir bakış için [Go blog'una][blog-slice] bakabilirsiniz. Okuyarak öğrendiklerinizi pekiştirmek için daha fazla test yazmayı deneyin.

Test yazmaktan başka Go ile deneme yapmanın bir başka kullanışlı yolu da Go playgrounddır. Soru sormanız gerekirse kodunuzu kolayca paylaşabilirsiniz ve çoğu şeyi deneyebilirsiniz. [Slice ile deneyler yapabilmeniz için Go playground'u hazırladım.](https://play.golang.org/p/ICCWcRGIO68)

Array'in dilimlenmesi ve slice'ın değiştirilmesinin orjinal diziye nasıl etkilediğinin [örneği](https://play.golang.org/p/bTrRmYfNYCp); "Kopya" slice orjinal arraye etki etmeyecektir.
Neden büyük bir slice'ı dilimledikten sonra slice'ın kopyasını almak neden iyi bir fikirdir [başka bir örnek](https://play.golang.org/p/Poth8JS28sc).

Bu sayfa [@bariscanyilmaz](https://github.com/bariscanyilmaz) tarafından çevrildi.

[for]: ../iteration.md#
[blog-slice]: https://blog.golang.org/go-slices-usage-and-internals
[deepEqual]: https://golang.org/pkg/reflect/#DeepEqual
[slice]: https://golang.org/doc/effective_go.html#slices
