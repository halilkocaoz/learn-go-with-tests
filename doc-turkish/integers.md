# Integerlar

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/integers)**

Integerlar beklediğiniz gibi çalışır.Bir şeyler denemek için `Add` fonksiyonu yazalım.`adder_test.go` adında test dosyası oluşturalım ve bu kodu yazalım.

**Not:** Go kaynak dosyaları her dizinde sadece bir tane `package` a sahip olabilir, dosyalarınızı ayrı ayrı düzenlediğinizden emin olun. [Burada bunu ile ilgili bir açıklama var.](https://dave.cheney.net/2014/12/01/five-suggestions-for-setting-up-a-go-project)

## İlk olarak test yaz

```go
package integers

import "testing"

func TestAdder(t *testing.T) {
	sum := Add(2, 2)
	expected := 4

	if sum != expected {
		t.Errorf("expected '%d' but got '%d'", expected, sum)
	}
}
```

Stringleri formatlarken `%q` yerine `%d` kullandığımızı farketmişsinizdir. Bunun sebebi string yerine integer yazdırmak istememiz.

Ayrıca `main` paketini kullanmadığımızı da unutmayın, bunu yerine adından da anlaşılacağı üzere `integers` adında bir paket tanımladık. Integerlar ile çalışırken `Add` gibi fonksiyonları burada gruplandıracağız.

## Dene ve testi çalıştır

Testi çalıştır `go test`

Derleme hatasını incele

`./adder_test.go:6:9: undefined: Add`

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

Derleyici hata vermeyecek kadar kod yaz. Testlerimizin doğru nedenle başarısız olup olmadığını kontrol etmek istediğimizi unutma.

```go
package integers

func Add(x, y int) int {
	return 0
}
```

Aynı tipte iki tane argüman olduğunda (bizim durumumuzda iki integer) `(x int, y int)` yazmak yerine `(x, y int)` yazarak kısaltabilirsin.

Şimdi testleri çalıştır ve testin neyin yanlış olduğunu doğru şekilde raporladığını görün.

`adder_test.go:10: expected '4' but got '0'`

Farkettiyseniz [son](hello-world.md#one...last...refactor?) bölümde _named return value_ öğrenmiştik ama burada aynısını kullanmadık. Bu durum genelde bağlamın sonucu anlaşılır olmadığında kullanılmalıdır. Bizim durumumuzda ise `Add` fonksiyonunun parametrelerini toplayacağı oldukça açıktır. Daha fazla detay için [buraya](https://github.com/golang/go/wiki/CodeReviewComments#named-result-parameters) bakabilirsiniz.

## Testi geçecek kadar kod yaz

TDD'nin tam manası ile _testi geçecek kadar minimum kod_ yazmalıyız. Belki ukala bir programcı bu şekilde yapardı.

```go
func Add(x, y int) int {
	return 4
}
```

Aha! Tekrar engellendi, TDD sahtekarlık değil mi?

Testin başarısız olması için farklı numaralarla başka test yazabiliriz ancak bu [kedi fare oyununa](https://en.m.wikipedia.org/wiki/Cat_and_mouse) benzeyecek.

Go'nun söz dizimine aşina olduğumuzda geliştiricileri sıkmayan ve bug bulmalarına yardımcı olan _"Property Based Testing"_ isimli tekniği anlatacağım.

Şimdilik düzgünce yapalım

```go
func Add(x, y int) int {
	return x + y
}
```

Eğer testi çalıştırırsanız geçecektir.

## Refactor

Var olan kodda iyileştirme yapabileceğimiz bir şey yok.

Dönüş argümanını adlandırarak bunun belgelerde ve aynı zamanda çoğu geliştiricinin editöründe nasıl göründüğünü daha önce keşfetmiştik.

Bu harika çünkü yazdığınız kodun kullanılabilirliğine yardımcı oluyor. Kullanıcı sadece kodunuzun imzasına([type signature](https://en.wikipedia.org/wiki/Type_signature)) ve dokümanına bakarak kullanımını anlayabilir.

Fonksiyonunuza yorum ekleyerek onlara doküman ekleyebilirsiniz. Standart kütüphanede olduğu gibi bunlar da Go Doc'ta gözükecektir.

```go
// Add takes two integers and returns the sum of them.
func Add(x, y int) int {
	return x + y
}
```

### Örnekler

Gerçekten ekstra mesafe katetmek istiyorsanız [örnekleri](https://blog.golang.org/examples) yapabilirsiniz. Standart kütüphanenin dokümanında bir çok örnek bulabilirsiniz.

Codebasein dışında kalan, readme dosyası gibi dosyaların kod örnekleri kontrol edilmedikleri için eski olmuş olabilir ve var olan kod ile kıyaslandığında yanlış olabilirler.

Go örnekleri tıpkı testler gibi yürütülür, böylece örneklerin kodun gerçekte ne yaptığını yansıttığından emin olabilirsiniz.

Örnekler derlenir ve isteğe bağlı olarak bir paketin test takımının parçası olarak çalıştırılır.

Tipik testlerde olduğu gibi örnekler paketin `_test.go` dosyasında olan fonksiyonlardır. `ExampleAdd` fonksiyonunu `adder_test.go` dosyasına ekleyin.

```go
func ExampleAdd() {
	sum := Add(1, 5)
	fmt.Println(sum)
	// Output: 6
}
```

(Eğer editörünüz sizin yerinize paketleri otomatik olarak eklemiyorsa, derleme adımı çalışmayacaktır çünkü `import "fmt"` kodu `adder_test.go` dosyası içerisnde eksiktir. Hangi editörü kullanırsanız kullanın, bu tür hataların sizin için otomatik olarak nasıl düzeltileceğini araştırmanız şiddetle tavsiye edilir.)

Eğer kodunuz geçerli olmayacak şekilde değişirse, derlemeniz başarısız olacaktır.

Paketin testini çalıştırırsak, örnek fonksiyonun bizden başka bir düzenleme yapılmasına gerek kalmadan çalıştığını görebiliriz:

```bash
$ go test -v
=== RUN   TestAdder
--- PASS: TestAdder (0.00s)
=== RUN   ExampleAdd
--- PASS: ExampleAdd (0.00s)
```

Lütfen not edin, eğer `// Output: 6` yorumunu kaldırırsak örnek fonksiyon çalışmayacaktır. Fonksiyon derlenecektir ancak çalışmayacaktır.

Bu kodu ekleyerek örneğimiz `godoc` içerisindeki dokümanda görünecek ve kodumuz daha da erişilebilir hale gelecektir.

Bunu denemek için `godoc -http=:6060` komutunu çalıştırın ve `http://localhost:6060/pkg/` adresini ziyaret edin.

Burada `$GOPATH` içerisindeki tüm paktelerin listesini göreceksiniz, bu kodu `$GOPATH/src/github.com/{your_id}` gibi bir yere yazdığınızı varsayarsak, örnek dokümanlarınızı bulabileceksiniz.

Eğer kodunuzu örnekler ile açık bir adreste paylaşırsanız, kodunuzun dokümanını [pkg.go.dev](https://pkg.go.dev/) adresinde paylaşabilirsiniz. Örneğin bu bölüm için nihai API [burada](https://pkg.go.dev/github.com/quii/learn-go-with-tests/integers/v2). Bu web arayüzü standart kütüphane ve 3. parti kütüphanelerinin dokümanını incelemenize olanak sağlar.

## Özetlersek

Ele alınanlar:

-   TDD iş akışının daha fazla uygulanması
-   Integerlar, ekleme
-   Kodumuzun kullanıcılar tarafından hızlı bir şekilde anlayabilmeleri için daha iyi doküman yazmak
-   Testlerimizin bir parçası olarak kontrol edilen kodumuzu nasıl kullanacağımıza dair örnekler
