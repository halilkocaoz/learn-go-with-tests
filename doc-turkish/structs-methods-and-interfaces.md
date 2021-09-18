# Structlar, methodlar & interfaceler

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/structs)**

Varsayalım ki uzunluğu ve yüksekliği verilen bir dörtgenin çevresini hesaplayan geometri koduna ihtiyacımız var. Dönüş tipi `float64`, 123.45 gibi, küsüratlı sayı (floating-point) olan `Perimeter(width float64, height float64)` isminde fonksiyon yazarız.

TDD aşamasına artık oldukça aşinasınızdır.

## İlk olarak test yaz

```go
func TestPerimeter(t *testing.T) {
    got := Perimeter(10.0, 10.0)
    want := 40.0

    if got != want {
        t.Errorf("got %.2f want %.2f", got, want)
    }
}
```

Yeni string formatını farkettiniz mi? `f` tanımı `float64` içindir ve `.2` virgülden sonra kaç basamağın yazılacağını belirtir.

## Testi çalıştırmayı dene

`./shapes_test.go:6:9: undefined: Perimeter`

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

```go
func Perimeter(width float64, height float64) float64 {
    return 0
}
```

Sonuç `shapes_test.go:10: got 0.00 want 40.00`.

## Testi geçecek kadar kod yaz

```go
func Perimeter(width float64, height float64) float64 {
    return 2 * (width + height)
}
```

Şimdiye kadar çok kolaydı. Şimdi `Area(width, height float64)` isminde dörtgenin alanını dönen fonksiyon yazalım.

TDD aşamalarını takip ederek kendiniz denemeye çalışın.

En sonunda testleriniz bunu gibi olacaktır.

```go
func TestPerimeter(t *testing.T) {
    got := Perimeter(10.0, 10.0)
    want := 40.0

    if got != want {
        t.Errorf("got %.2f want %.2f", got, want)
    }
}

func TestArea(t *testing.T) {
    got := Area(12.0, 6.0)
    want := 72.0

    if got != want {
        t.Errorf("got %.2f want %.2f", got, want)
    }
}
```

Kodumuz da buna benzeyecektir

```go
func Perimeter(width float64, height float64) float64 {
    return 2 * (width + height)
}

func Area(width float64, height float64) float64 {
    return width * height
}
```

## Refactor

Kodumuz işini yapıyor ancak dikdörtgenler hakkında açık bir şey içermiyor. Dikkatsiz bir geliştirici, yanlış cevap elde edeceğini fark etmeden bu fonksiyonlara bir üçgenin genişliğini ve yüksekliğini vermeye çalışabilir.

Fonksiyonlara `RectangleArea` gibi daha spesifik isimler verebilir. Daha iyi bir çözüm ise bu konsepti bizim için kapsayan kendi _tipimiz_ olan `Rectangle` tanımlamaktır.

**Struct** kullanarak basitçe kendi tipimizi oluşturabiliriz. Verilerimizi depoladığımız fieldların kolksiyonuna [Struct](https://golang.org/ref/spec#Struct_types) denir.

Aşağıdaki gibi bir struct tanımlayın

```go
type Rectangle struct {
    Width float64
    Height float64
}
```

`float64` yerine `Rectangle` kullanmak için testlerimizi değiştirelim.

```go
func TestPerimeter(t *testing.T) {
    rectangle := Rectangle{10.0, 10.0}
    got := Perimeter(rectangle)
    want := 40.0

    if got != want {
        t.Errorf("got %.2f want %.2f", got, want)
    }
}

func TestArea(t *testing.T) {
    rectangle := Rectangle{12.0, 6.0}
    got := Area(rectangle)
    want := 72.0

    if got != want {
        t.Errorf("got %.2f want %.2f", got, want)
    }
}
```

Düzeltmeye çalışmadan önce testlerinizi çalıştırmayı unutmayın. Testler aşağıdakine benzer faydalı bir hata mesajı göstermeli.

```text
./shapes_test.go:7:18: not enough arguments in call to Perimeter
    have (Rectangle)
    want (float64, float64)
```

Structın fieldlarına `myStruct.field` sözdizimi ile erişebilirsiniz.

Testi düzeltmek için iki fonksiyonu değşitirin.

```go
func Perimeter(rectangle Rectangle) float64 {
    return 2 * (rectangle.Width + rectangle.Height)
}

func Area(rectangle Rectangle) float64 {
    return rectangle.Width * rectangle.Height
}
```

Bir fonksiyona `Rectangle` vermenin amacımızı daha net ifade ettiğini kabul edeceğinizi umuyorum. Strcut kullanmanın avantajlarını ileride ele alacağız.

Sıradaki koşulumuz ise çemberler için `Area` fonksiyonu yazmak.

## İlk testi yaz

```go
func TestArea(t *testing.T) {

    t.Run("rectangles", func(t *testing.T) {
        rectangle := Rectangle{12, 6}
        got := Area(rectangle)
        want := 72.0

        if got != want {
            t.Errorf("got %g want %g", got, want)
        }
    })

    t.Run("circles", func(t *testing.T) {
        circle := Circle{10}
        got := Area(circle)
        want := 314.1592653589793

        if got != want {
            t.Errorf("got %g want %g", got, want)
        }
    })

}
```

Görebileceğiniz gibi `f` iyi bir nedenle `g` ile yer değiştirdi.
`g` kullanımı, hata mesajında daha kesin bir ondalık sayı yazdıracaktır \([fmt seçenekleri](https://golang.org/pkg/fmt/)\).
Örneğin, yarıçapı 1.5 olan bir çemberin alanı hesaplandığında, `f`'in yazdıracağı değer `7.068583` iken `g` 'nin yazdıracağı değe `7.0685834705770345` olacaktır.

## Testi çalıştırmayı dene

`./shapes_test.go:28:13: undefined: Circle`

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`Circle` tipini tanımlamamız gerekiyor.

```go
type Circle struct {
    Radius float64
}
```

Şimdi testi çalıştırmayı tekrar dene

`./shapes_test.go:29:14: cannot use circle (type Circle) as type Rectangle in argument to Area`

Bazı programlama dilleri bunun gibi bir şeyler yapmanıza izin verir:

```go
func Area(circle Circle) float64 { ... }
func Area(rectangle Rectangle) float64 { ... }
```

Ancak Go'da yapamazsınız

`./shapes.go:20:32: Area redeclared in this block`

İki seçeneğimiz var:

-   Farklı _paketlerde_ aynı fonksiyonlar tanımlamak.`Area(Circle)` yeni bir pakette oluşturabilir ancak bu fazla abartılmış olur.
-   Bunun yerine yeni tanımladığımız tipler üzerine [_metotlar_](https://golang.org/ref/spec#Method_declarations) tanımlayabiliriz.

### Metotlar nedir?

Şimdiye kadar sadece _fonksiyonlar_ yazdık ancak metotları kullanıyorduk . `t.Errorf` çağırdığımızda aslında `t` \(`testing.T`\)'nin instancını `Errorf` metodunu çağırıyorduk.

Metotlar, receiverı olan fonksiyonlardır.
Metot tanımı, bir tanımlayıcıyı, metot adını bir metota bağlar ve metodu receiverın temel türüyle ilişkilendirir.

Metotlar fonksiyonlara çok benzerdir ancak onlar belirli bir tipin instanceından çağırılır. `Area(rectangle)` gibi Fonksiyonları ise istedğiniz yerden çağırabilirsiniz ancak metotları sadece "instance" üzerinden çağırabilirsiniz.

Örnek yardımcı olacaktır, bu yüzden önce testlerimizi fonksiyon yerine metotları çağırmak için değiştirelim ve ardından kodu düzeltelim.

```go
func TestArea(t *testing.T) {

    t.Run("rectangles", func(t *testing.T) {
        rectangle := Rectangle{12, 6}
        got := rectangle.Area()
        want := 72.0

        if got != want {
            t.Errorf("got %g want %g", got, want)
        }
    })

    t.Run("circles", func(t *testing.T) {
        circle := Circle{10}
        got := circle.Area()
        want := 314.1592653589793

        if got != want {
            t.Errorf("got %g want %g", got, want)
        }
    })

}
```

Eğer testleri çalıştırmayı denersek aşağıdakileri elde ederiz

```text
./shapes_test.go:19:19: rectangle.Area undefined (type Rectangle has no field or method Area)
./shapes_test.go:29:16: circle.Area undefined (type Circle has no field or method Area)
```

> type Circle has no field or method Area

Derleyicinin burada ne kadar harika olduğunu tekrarlamak istiyorum. Aldığınız hata mesajlarını yavaş yavaş okumak için zaman ayırmanız o kadar önemlidir ki uzun vadede size yardımcı olacaktır.

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

Tiplerimize biraz metot ekleyelim

```go
type Rectangle struct {
    Width  float64
    Height float64
}

func (r Rectangle) Area() float64  {
    return 0
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64  {
    return 0
}
```

Metot tanımlamanın sözdizimi neredeyse fonksiyonlar ile aynıdır bu yüzden oldukça benzerlerdir. Tek fark metotların recieverıdır `func (receiverName ReceiverType) MethodName(args)`.

Metodunuz bu tip bir değişken üzerinde çağrıldığında, onun verilerine referansınızı `receiverName` değişkeni aracılığıyla alırsınız. Diğer birçok programlama dilinde bu örtük olarak yapılır ve alıcıya `this` aracılığıyla erişirsiniz.

Go'da receiver değişkeninin türün ilk harfi olması bir kuraldır.

```go
r Rectangle
```

Eğer testleri tekrardan çalıştırmayı denerseniz şimdi derlenecektirler ve hata çıktısı vereceklerdir.

## Testi geçecek kadar kod yaz

Şimdi yeni metodumuzu düzelterek dikdörtgen testlerimizi geçelim

```go
func (r Rectangle) Area() float64  {
    return r.Width * r.Height
}
```

Eğer dörgen testlerini tekrardan çalıştırırsanız testi geçeceklerdi ancak çember hala başarısız olacaktır.

To make circle's `Area` function pass we will borrow the `Pi` constant from the `math` package \(remember to import it\).

```go
func (c Circle) Area() float64  {
    return math.Pi * c.Radius * c.Radius
}
```

## Refactor

Testlerimizde bazı tekrarlar var.

Yapmak istediğimiz, _şekillerin_ koleksiyonunu almak ,her birinin `Area()` metotlarını çağırmak ve sonuçlarını kontrol etmek.

`Rectangle` ve `Circle` parametre olarak gönderebileceğimiz `checkArea` fonksiyonu yazmak istiyoruz ama bu şekillerin dışında bir şey gönderdiğimizde derlenmemesini istiyoruz.

Go'da bu işi **interfacelar** ile yapabiliyoruz

[Interfaceler](https://golang.org/ref/spec#Interface_types), Go gibi statik tipli dillerde çok güçlü konseptlerdir çünkü fonksiyonların farklı tipler ile kullanımasına izin verir ve tip güvenliğini korurken highly-decoupled kod oluşturur.

Testlerimizi düzenleyerek interfaceleri tanıtalım.

```go
func TestArea(t *testing.T) {

    checkArea := func(t testing.TB, shape Shape, want float64) {
        t.Helper()
        got := shape.Area()
        if got != want {
            t.Errorf("got %g want %g", got, want)
        }
    }

    t.Run("rectangles", func(t *testing.T) {
        rectangle := Rectangle{12, 6}
        checkArea(t, rectangle, 72.0)
    })

    t.Run("circles", func(t *testing.T) {
        circle := Circle{10}
        checkArea(t, circle, 314.1592653589793)
    })

}
```

Diğer alıştırmalarda olduğu gibi yardımcı fonksiyon oluşturuyoruz ama bu sefer paramtere için `Shape` istiyoruz. `Shape` olmayan bir değer ile çağırdığımızda derlenmeyecektir.

Bir şeyler nasıl `Shape` oluyor? Interface tanımı ile Go'ya neyin `Shape` olduğunu söylüyoruz

```go
type Shape interface {
    Area() float64
}
```

`Rectangle` ve `Circle`'da olduğu gibi yeni bir `type` oluşturuyoruz ama bu sefer `struct` yerine `interface` kullanarak.

Birkere bunu eklediğinizde, tesler geçecektir.

### Bekle, ne?

Burdaki interfacelar çoğu programlama dillerinden farklılar. Normalde `My type Foo implements interface Bar` buna benzer bir kod yazmanız gerekir.

Ancak bizim durumumuzda

-   `Rectangle`, `Area` isminde `float64` dönen bir metoda sahip, `Shape` interfacini karşılıyor
-   `Circle`, `Area` isminde `float64` dönen bir metoda sahip, `Shape` interfacini karşılıyor
-   `string` böyle bir methoda sahip değil, bu yüzden interfaci karşılamıyor
-   vb.

Go'da **interface çözünürlüğü örtüktür**. Eğer paramtere olarak gönderdiğiniz değer interface ile eşleşirse derlenir.

### Decoupling

Yardımcımızın şeklin `Rectangle`, `Circle` veya `Triangle` olup olmadığıyla ilgilenmesine gerek olmadığına dikkat edin.Bir interface tanımlayarak, yardımcı somut türlerden _ayrışmış (decoupled)_ olur ve yalnızca işini yapması için ihtiyaç duyduğu metota sahip olur.

**Yalnızca ihtiyacınız olanı** tanımlamak için interfaceleri kullanma yaklaşımı, yazılım tasarımında çok önemlidir ve sonraki bölümlerde daha ayrıntılı olarak ele alınacaktır.

## Daha fazla refactoring

Artık structları anladığınıza göre, "table driven testlere" girebiliriz.

[Table driven testler](https://github.com/golang/go/wiki/TableDrivenTests), aynı şekilde test edilebilecek bir test senaryoları listesi oluşturmak istediğinizde kullanışlıdır.

```go
func TestArea(t *testing.T) {

    areaTests := []struct {
        shape Shape
        want  float64
    }{
        {Rectangle{12, 6}, 72.0},
        {Circle{10}, 314.1592653589793},
    }

    for _, tt := range areaTests {
        got := tt.shape.Area()
        if got != tt.want {
            t.Errorf("got %g want %g", got, tt.want)
        }
    }

}
```

Buradaki tek yeni söz dizimi "anonymous structtır", `areaTests`. `shape` ve `want` fieldlarına sahip `[]struct` ile struct slice'ı tanımlıyoruz. Daha sonra sliceları durumlar ile dolduruyoruz.

Daha sonra, testlerimizi çalıştırmak için struct fieldları kullanarak, diğer dilimlerde yaptığımız gibi bunları iterate ediyoruz.

Bir geliştiricinin yeni bir shape tanıtmasının, `Area` uygulamasını ve ardından onu test senaryolarına eklemesinin ne kadar kolay olacağını görebilirsiniz. Ek olarak, `Area` ile ilgili bir hata bulunursa, düzeltmeden önce alıştırma yapmak için yeni bir test senaryosu eklemek çok kolaydır.

Table driven testler alet çantanızda harika bir araç olabilir ancak testlerde ekstra gürültüye ihtiyacınız var.
Bir interfacin çeşitli uygulamalarını test etmek istediğinizde veya bir fonksiyona iletilen verilerin test edilmesi gereken birçok farklı gereksinimi varsa, bunlar çok uygundur.

Tüm bunları başka bir şekil ekleyerek ve test ederek gösterelim; bir üçgene.

## İlk olarak test yaz

Yeni şeklimiz için test eklemek çok kolay. Sadece listemize `{Triangle{12, 6}, 36.0},` ekle.

```go
func TestArea(t *testing.T) {

    areaTests := []struct {
        shape Shape
        want  float64
    }{
        {Rectangle{12, 6}, 72.0},
        {Circle{10}, 314.1592653589793},
        {Triangle{12, 6}, 36.0},
    }

    for _, tt := range areaTests {
        got := tt.shape.Area()
        if got != tt.want {
            t.Errorf("got %g want %g", got, tt.want)
        }
    }

}
```

## Testi çalıştırmayı dene

Testi çalıştırmayı denemeye devam etmeyi ve derleyicinin bir çözüme rehberlik etmesine izin vermeyi hatırla.

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`./shapes_test.go:25:4: undefined: Triangle`

Henüz `Triangle`'ı tanımlamadık

```go
type Triangle struct {
    Base   float64
    Height float64
}
```

Tekrar dene

```text
./shapes_test.go:25:8: cannot use Triangle literal (type Triangle) as type Shape in field value:
    Triangle does not implement Shape (missing Area method)
```

Bize `Triangle`'ı shape olarak kullanamayacağımızı çünkü `Area()` metoduna sahip olmadığını söylüyor, o zaman testin çalışması için boş bir uygulamasını ekleyelim.

```go
func (t Triangle) Area() float64 {
    return 0
}
```

Sonunda kod derleniyor ve hata kodumuzu alıyoruz

`shapes_test.go:31: got 0.00 want 36.00`

## Testi geçecek kadar kod yaz

```go
func (t Triangle) Area() float64 {
    return (t.Base * t.Height) * 0.5
}
```

Ve testimiz geçiyor!

## Refactor

Tekrar, uyarlamamız iyi ancak testlerimiz biraz iyileştirme yapabilir.

Bunu taradığınızda

```go
{Rectangle{12, 6}, 72.0},
{Circle{10}, 314.1592653589793},
{Triangle{12, 6}, 36.0},
```

Testlerinizin kolayca anlaşılmasını amaçlamalısınız, burada tüm sayıların neyi temsil ettiği hemen belli olmuyor.

Şimdiye kadar size yalnızca `MyStruct{val1, val2}` structunun örneklerini oluşturmak için sözdizimi gösterildi, ancak isteğe bağlı olarak fieldları adlandırabilirsiniz.

Şimdi nasıl gözüktüğüne bakalım

```go
        {shape: Rectangle{Width: 12, Height: 6}, want: 72.0},
        {shape: Circle{Radius: 10}, want: 314.1592653589793},
        {shape: Triangle{Base: 12, Height: 6}, want: 36.0},
```

[Test-Driven Development by Example](https://g.co/kgs/yCzDLF) kitabında Kent Beck, bazı testleri bir noktaya kadar yeniden düzenler ve şunları iddia eder:

> Eğer test, **bir dizi işlemin değil**, doğruluk iddiasında bulunursa, bize daha anlaşılır olur.

\(altındaki vurgu bana aittir\)

Şimdi testlerimiz - daha doğrusu test senaryolarının listesi - şekiller ve alanları hakkında doğruluk iddiasında bulunuyor.

## Test çıktılarınızı yardımcı hale getirin

`Triangle` implemente ettiğimizde testimiz başarısız olmuştu hatırladınız mı? Hata mesajı olarak `shapes_test.go:31: got 0.00 want 36.00` yazdırmıştı.

Bunun `Triangle` ile ilgili olduğunu biliyorduk çünkü sadece onunla çalışıyorduk.
Peki ya tablodaki 20 vakadan birinde sistemde bir bug olursa?
Hangi durumun hatalı olduğunu geliştirici nasıl bilecek?
Geliştirici için harika bir deneyim değil, hangi durumun başarısız olduğunu bulmak için manuel olarak hepsine bakmak zorundalar.

Hata mesajımızın formatını değiştiriyoruz `%#v got %g want %g`. `%#v` formatı structın fieldlarındaki veriler ile birlikte yazdıırı bu sayede gelşitirici test edilen properyleri bir bakışta görebilir.

Testlerimizin okunabilirliğini daha da arttırmak için `want` fieldını daha açıklayıcı olan `hasArea` olarak değiştirebiliriz.

Table driven testler ile ilgili son ipucu `t.Run` kullanmak ve test senaryolarını adlandırmak.

Her senaryoyu `t.Run` ile sarmalarsanız testleriniz başarısız olduğunda daha temiz bir çıktı elde edersiniz

```text
--- FAIL: TestArea (0.00s)
    --- FAIL: TestArea/Rectangle (0.00s)
        shapes_test.go:33: main.Rectangle{Width:12, Height:6} got 72.00 want 72.10
```

`go test -run TestArea/Rectangle` komutu ile tablonuz içerisinde specific testleri çalıştırabilirsiniz.

Bu durumu kapsayana final test kodumuz

```go
func TestArea(t *testing.T) {

    areaTests := []struct {
        name    string
        shape   Shape
        hasArea float64
    }{
        {name: "Rectangle", shape: Rectangle{Width: 12, Height: 6}, hasArea: 72.0},
        {name: "Circle", shape: Circle{Radius: 10}, hasArea: 314.1592653589793},
        {name: "Triangle", shape: Triangle{Base: 12, Height: 6}, hasArea: 36.0},
    }

    for _, tt := range areaTests {
        // using tt.name from the case to use it as the `t.Run` test name
        t.Run(tt.name, func(t *testing.T) {
            got := tt.shape.Area()
            if got != tt.hasArea {
                t.Errorf("%#v got %g want %g", tt.shape, got, tt.hasArea)
            }
        })

    }

}
```

## Özetlersek

Bu daha çok TDD uygulamasıydı, temel matematik problemlerine yönelik çözümlerimizi yineliyor ve testlerimizle motive edilen yeni dil özelliklerini öğreniyordu.

-   Struct tanımlayarak ilgili verileri bir araya getirmenize ve kodunuzun amacını daha net hale getirmenize olanak tanıyan kendi veri türlerinizi oluşturma
-   Interface tanımlayarak farklı tiplerle fonksiyonların kullanma \([parametric polymorphism](https://en.wikipedia.org/wiki/Parametric_polymorphism)\)
-   Metotlar ekleyerek veri tiplerine fonksiyonalite ekleme ve interfaceleri implemente etme
-   Table driven testler ile assertionları daha temiz yapma ve test takımlarını genişletme ve bakımını kolaylaştırma

Bu önemli bir bölümdü çünkü artık kendi türlerimizi tanımlamaya başlıyoruz. Go gibi statik olarak yazılan dillerde, kendi türlerinizi tasarlayabilmek, anlaşılması, birleştirilmesi ve test edilmesi kolay yazılımlar oluşturmak için çok önemlidir.

Interfaceler, sistemin karmaşıklığını gizlemek için harika bir araç. Bizim durumumuzda, test yardımcı _kodumuz_, iddia ettiği shapi tam olarak bilmek zorunda değildi, sadece alanı için nasıl "soracağını" bilmek zorundaydı.

Go'ya daha aşina oldukça, arayüzlerin ve standart kütüphanenin gerçek gücünü görmeye başlayacaksınız. _Her yerde_ kullanılan, standart kütüphanede tanımlı interfaceleri öğreneceksiniz ve bunların kendi türlerinize karşı uygulamalarını öğreneceksiniz,çok sayıda harika işlevi çok hızlı bir şekilde yeniden kullanabileceksiniz.
