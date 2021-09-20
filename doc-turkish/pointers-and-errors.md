# Pointers & errors

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/pointers)**


Son bölümde, bir dizi değerleri tutmamızı sağlayan structları öğrendik.

Bazı nedenlerden dolayı, bir durumu yönetmek için structları kullanmak isteyebilirsiniz, sizin kontrol ettiğiniz, istediğiniz şekilde kullanıcıların bir durumu değiştirmesine izin veren metotları da açığa çıkarabilirsiniz.

**Fintech, Go'yu seviyor** ya bitcoinler? Öyleyse, ne kadar harika bir bankacılık sistemi yapabildiğimizi görelim .

`Bitcoin`lerimizi barındırabildiğimiz bir `Wallet` structı yapalım.

## İlk olarak test yaz

```go
func TestWallet(t *testing.T) {

    wallet := Wallet{}

    wallet.Deposit(10)

    got := wallet.Balance()
    want := 10

    if got != want {
        t.Errorf("got %d want %d", got, want)
    }
}
```

Bir önceki [örnekte](./structs-methods-and-interfaces.md), structın bileşenlerine, alanlarına doğrudan eriştik. Şimdiyse, çok güvenli olması gereken `wallet`ımızı dünyaya açmak istemiyoruz ve bunu metotlar aracılığı ile yapacağız.

## Testi çalıştırmayı dene

`./wallet_test.go:7:12: undefined: Wallet`

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

Derleyici(compiler), `Wallet`'ı bilmiyor. Öyleyse tanımlayalım.

```go
type Wallet struct { }
```

Cüzdanımızı tanıttık, tekrar testi çalıştırmayı deneyin.

```go
./wallet_test.go:9:8: wallet.Deposit undefined (type Wallet has no field or method Deposit)
./wallet_test.go:11:15: wallet.Balance undefined (type Wallet has no field or method Balance)
```

Wallet için tanımlanmamış metotlar ve şimdiyse onları tanımlamamız gerekiyor

Sadece testleri çalıştırmaya yetecek kadar kod yazmayı unutmayın! Şimdilik sadece testimizin anlaşılır bir hata mesajı ile doğru şekilde başarısız olduğunu görmeliyiz.

```go
func (w Wallet) Deposit(amount int) {

}

func (w Wallet) Balance() int {
    return 0
}
```

Bu sözdizimi tanıdık gelmediyse, geri dönün ve struct bölümünü okuyun.

Yazılımımız derlenmeli ve testler çalışmalıdır.

`wallet_test.go:15: got 0 want 10`

## Testi geçecek kadar kod yaz

_balance_ durumunu saklamak için bir çeşit değişkene ihtiyacımız olacak.

```go
type Wallet struct {
    balance int
}
```

Go'da bir bileşenin(variablelar, typelar, fonksiyonlar ve benzerleri) isimlendirmesi küçük harfle başlıyorsa, bu _private_ acces modifier'a sahiptir ve o paketin dışından ulaşılamaz.

Biz bu değeri manipüle etmek istiyoruz fakat sadece paket içindeki metotlar yardımı ile yapmak istiyoruz yani paket dışında herhangi bir yerde bunun mümkün olmasını istemiyoruz.

Hatırlayın, internal `balance` fieldına "receiver" ile verilen değişkenini kullanarak ulaşabiliriz.

```go
func (w Wallet) Deposit(amount int) {
    w.balance += amount
}

func (w Wallet) Balance() int {
    return w.balance
}
```

Şimdi tekrardan test paketini çalıştırın ve  güvence altında olan fintech sektöründeki kariyerinizin tadını geçen testleri izleyerek çıkarın

`wallet_test.go:15: got 0 want 10`

### ????

Ne oldu, kodumuzun çalışması gerekiyordu.
Yeni bakiyemizi ekliyoruz ve `Balance` metotu ile eklediğimiz yeni değeri döndürmesi gerekiyordu

Go'da **bir fonksiyonu veya yöntemi çağırdınızda onun aldığı argümanlar** _**kopyalanır**_

`func (w Wallet) Deposit(amount int)` metotunu çağırdığımızda buradaki **`w`** argümanı çağrıldığı yerdeki yapının bir kopyası olarak gelir.

Bir tanımı oluşturduğunuzda, bu bellekte bir yerlerde saklanır ve her tanımın bir adresi vardır. `&` kullanarak `&myVal` şeklinde bir çağrı ile bellekte saklandığı adrese ulaşabilirsiniz.

Kodunuzda bazı noktalara çıktı almak için `print` ekleyerek bu durumu gözlemleyin

```go
func TestWallet(t *testing.T) {

    wallet := Wallet{}

    wallet.Deposit(10)

    got := wallet.Balance()

    fmt.Printf("address of balance in test is %v \n", &wallet.balance)

    want := 10

    if got != want {
        t.Errorf("got %d want %d", got, want)
    }
}
```

```go
func (w Wallet) Deposit(amount int) {
    fmt.Printf("address of balance in Deposit is %v \n", &w.balance)
    w.balance += amount
}
```

Bir tanımın başına `&` karakterini koyarak onun bellekteki adresini, point ettiği adresi alabiliriz.

Testleri tekrar çalıştıralım

```text
address of balance in Deposit is 0xc420012268
address of balance in test is 0xc420012260
```

Görebildiğiniz üzere, `balance`ların adresleri farklı. `Deposit` metotunu kullanırken, test kodundan gelen yapının bir kopyasını kullanıyoruz. Bu durumdan dolayı, testteki `balance` değişmiyor.

Bu problemi pointerları kullanarak aşabiliriz. [Pointerlar](https://gobyexample.com/pointers), bazı değerlerin adreslerini almamıza ve onları değiştirmemize izin verir. Dolayısıyla, cüzdanının bir kopyasını almak yerine içindeki orjinal değerleri değiştirebilmemiz için metotlarda argüman olarak pointer almalıyız.

```go
func (w *Wallet) Deposit(amount int) {
    w.balance += amount
}

func (w *Wallet) Balance() int {
    return w.balance
}
```

`*Wallet` ve `Wallet` arasındaki fark,
`*Wallet`'ın çağrıldığı yerdeki cüzdana point eden bir argüman olacağını belirtiyoruz
`Wallet` ise çağrıldığı yerden gelen bir kopya.

Testleri tekrardan çalıştırın, geçmiş olmaları gerekir.

Testlerin neden geçtiğini merak ediyor olabilirsiniz. Metotdaki işaretcinin refaransını şu şekile getirmedik:

```go
func (w *Wallet) Balance() int {
    return (*w).balance
}
```

ve doğrudan nesneye adres etti. Doğrusu yukarıdaki `(*w)` şeklinde olan kullanım kesinlikle geçerlidir. Yine de, Go'nun yapımcıları bu sözdizimini gereksiz buldular, bu sebeple açık bir referanslama olmadan `w.balance` şeklinde kullanıma izin verdiler. Yapılara yönelik bu point etme şeklinin kendine has ismi bile var: **_struct pointers_**. - [automatically dereferenced](https://golang.org/ref/spec#Method_values)

Technically you do not need to change `Balance` to use a pointer receiver as taking a copy of the balance is fine. However, by convention you should keep your method receiver types the same for consistency.

## Refactor

Bitcoin için bir cüzdan yapacağımızı söyledik fakat bitcoin'e dair herhangi bir şeyden bahsetmedik. Şimdilik sadece `int` tipini kullandık, çünkü `int` birşeyleri saymak için iyidir.

Bunun için sıfırdan bir `struct` oluşturmak biraz abartılı olabilir, `int` yeterlidir fakat bu durum için açıklayıcı bir veri tipi değil.

Go mevcut olan bir veri tipinden başka bir veri tipi oluşturmanıza olanak sağlar,

Şu sözdizimi ile varolan bir veri tipinden, başka bir veri tipi türetebilirsiniz: `type MyName OriginalType`

```go
type Bitcoin int

type Wallet struct {
    balance Bitcoin
}

func (w *Wallet) Deposit(amount Bitcoin) {
    w.balance += amount
}

func (w *Wallet) Balance() Bitcoin {
    return w.balance
}
```

```go
func TestWallet(t *testing.T) {

    wallet := Wallet{}

    wallet.Deposit(Bitcoin(10))

    got := wallet.Balance()

    want := Bitcoin(10)

    if got != want {
        t.Errorf("got %d want %d", got, want)
    }
}
```

`Bitcoin` oluşturmak için `Bitcoin(999)` sözdizimini kullanabilirsin.

Bunu yaparak, yeni bir tür oluşturuyoruz ve bu tipe özel _methodlar_ tanımlayabiliriz. Tipe özel methodlar oluşturmamız, alanlara özgü başı işlevler eklemek istediğimizde çok yararlı olabilir.

Bitcoin için [Stringer](https://golang.org/pkg/fmt/#Stringer)'ı implemente edelim.

```go
type Stringer interface {
        String() string
}
```

Stringer interfaceı, `fmt` paketinde tanımlanmıştır ve string olarak çıktı alırken `%s` gibi bir formatlayıcı ile nasıl yazdıralacağını tanımlamamıza olanak sağlar.

```go
func (b Bitcoin) String() string {
    return fmt.Sprintf("%d BTC", b)
}
```

Gördüğünüz gibi, tipler için metot oluşturma sözdizimi, structlarda olduğu gibidir.

Şimdiyse, testleri güncelleyerek bu oluşturduğumuz formatlı olarak string dönüşü yapan `String()` metotunu kullanmasını sağlayacağız.

```go
    if got != want {
        t.Errorf("got %s want %s", got, want)
    }
```

Testlerin çalışmasını kasıtlı olarak bozarak, yeni çıktıyı görelim

`wallet_test.go:18: got 10 BTC want 20 BTC`

Bu testlerimizde ve değiştirdiğimiz kodlar ile neler olduğunu daha anlaşılır olacak.

Sonraki yapacağımız geliştirme, `Withdraw` için olacak.

## İlk olarak test yaz

`Withdraw` tam olarak `Deposit`'ın tam tersi olacak

```go
func TestWallet(t *testing.T) {

    t.Run("Deposit", func(t *testing.T) {
        wallet := Wallet{}

        wallet.Deposit(Bitcoin(10))

        got := wallet.Balance()

        want := Bitcoin(10)

        if got != want {
            t.Errorf("got %s want %s", got, want)
        }
    })

    t.Run("Withdraw", func(t *testing.T) {
        wallet := Wallet{balance: Bitcoin(20)}

        wallet.Withdraw(Bitcoin(10))

        got := wallet.Balance()

        want := Bitcoin(10)

        if got != want {
            t.Errorf("got %s want %s", got, want)
        }
    })
}
```

## Testi çalıştırmayı dene

`./wallet_test.go:26:9: wallet.Withdraw undefined (type Wallet has no field or method Withdraw)`

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

```go
func (w *Wallet) Withdraw(amount Bitcoin) {

}
```

`wallet_test.go:33: got 20 BTC want 10 BTC`

## Testi geçecek kadar kod yaz

```go
func (w *Wallet) Withdraw(amount Bitcoin) {
    w.balance -= amount
}
```

## Refactor

Test kodlarımızda, tekrar eden kodlar var. Testlerimizi refactor edelim

```go
func TestWallet(t *testing.T) {

    assertBalance := func(t testing.TB, wallet Wallet, want Bitcoin) {
        t.Helper()
        got := wallet.Balance()

        if got != want {
            t.Errorf("got %s want %s", got, want)
        }
    }

    t.Run("Deposit", func(t *testing.T) {
        wallet := Wallet{}
        wallet.Deposit(Bitcoin(10))
        assertBalance(t, wallet, Bitcoin(10))
    })

    t.Run("Withdraw", func(t *testing.T) {
        wallet := Wallet{balance: Bitcoin(20)}
        wallet.Withdraw(Bitcoin(10))
        assertBalance(t, wallet, Bitcoin(10))
    })

}
```

Hesapta kalan paradan daha fazlasını `Withdraw` etmeye çalışırsanız ne olacak? Şimdiki ihtiyacımız, kredili bir mevduat hesabının olmadığını varsaymak olacak

`Withdraw` metotunu kullanırken geriye nasıl bir problem olduğunu bildiririz?

Go'da, bir hata oluştuğunu belirtmek istiyorsanız "idiomatic" bir seçeneğiniz var. Sizin yazdığınız fonksiyonunu çağırıldığı yere bir `err` döndürmesini ve onu kontrol ederek harekete geçmesini sağlayabilirsiniz.

Bunu bir test ile yapmayı deneyelim,

## İlk önce testi yaz

```go
t.Run("Withdraw insufficient funds", func(t *testing.T) {
    startingBalance := Bitcoin(20)
    wallet := Wallet{startingBalance}
    err := wallet.Withdraw(Bitcoin(100))

    assertBalance(t, wallet, startingBalance)

    if err == nil {
        t.Error("wanted an error but didn't get one")
    }
})
```

Hesap sahibinin, sahip olduğundan daha fazla parayı çekmeye çalıştığında bunu yapamamasını ve bir hata dönmesini istiyoruz.

We then check an error has returned by failing the test if it is `nil`.

`nil`, diğer programlama dillerinde olan `null` ile eşanlamlıdır. `Withdraw`'ın dönüşü `nil` olabilir çünkü return type olarak `error` kullanılacak ve bu bir interfacedır. Interface dönen veya parametre olarak alan fonksiyonlar görürseniz, bilinki bunların aldığı veya döndüğü değerler `nil` olabilir.

`null`'da olduğu gibi, `nil` bir değere ulaşmaya çalışırsanız **runtime panic** hatası fırlatılacaktır. Bunun için, `nil` olabilecek değerlere ulaşmaya çalışmadan önce `nil` olup, olmadığını kontrol etmelisiniz.

## Dene ve testi çalıştır

`./wallet_test.go:31:25: wallet.Withdraw(Bitcoin(100)) used as value`

Hata ifadesi biraz anlaşılmaz fakat amacımız sadece `Withdraw`'ı çağırmaktı, şuan herhangi bir değer döndürmüyor. Değer döndürmesini sağlamak için metotu dönüş tipine sahip olacak şekilde değiştirmeliyiz.

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

```go
func (w *Wallet) Withdraw(amount Bitcoin) error {
    w.balance -= amount
    return nil
}
```

Tekrardan compile etmesini sağlayacak kadar kod yazmalıyız, `Withdraw` metotunu bir `error` döndüreceği şekile getirdik. Şimdilik sadece `nil` dönmesini sağlıyoruz.

## Testi geçecek kadar kod yaz

```go
func (w *Wallet) Withdraw(amount Bitcoin) error {

    if amount > w.balance {
        return errors.New("oh no")
    }

    w.balance -= amount
    return nil
}
```

`errors` paketini kodunuza import etmeyi unutmayın.


`errors.New`, bizim oluşturduğumuz bir mesaj ile `error` oluşturuyor.

## Refactor

Testin okunabilirliğini arttırmak için, önceden yaptığımız gibi assert ile bir helper oluşturalım.

```go
assertError := func(t testing.TB, err error) {
    t.Helper()
    if err == nil {
        t.Error("wanted an error but didn't get one")
    }
}
```

```go
t.Run("Withdraw insufficient funds", func(t *testing.T) {
    startingBalance := Bitcoin(20)
    wallet := Wallet{startingBalance}
    err := wallet.Withdraw(Bitcoin(100))

    assertError(t, err)
    assertBalance(t, wallet, startingBalance)
})
```

Umarım, "oh no" hata mesajını tekrardan düzenleyeceğimizi düşünüyorsunuzdur çünkü bu hata mesajını dönmenin pek bir faydası olmaz, boş bir hata mesajı dönmekten pek bir farkı yok.

Hatanın kullanıcıya döndüğünü varsayarak, testlerimizi bir hatanın varlığından ziyade hatanın mesajı üzerinden iddia oluşturacak şekilde güncelleyelim.

## İlk önce testi yaz

`string` ile karşılaştırmak yapmak için test helperimizi güncelleyelim

```go
assertError := func(t testing.TB, got error, want string) {
    t.Helper()
    if got == nil {
        t.Fatal("didn't get an error but wanted one")
    }

    if got.Error() != want {
        t.Errorf("got %q, want %q", got, want)
    }
}
```

Çağırma yöntemimizi değiştirelim,

```go
t.Run("Withdraw insufficient funds", func(t *testing.T) {
    startingBalance := Bitcoin(20)
    wallet := Wallet{startingBalance}
    err := wallet.Withdraw(Bitcoin(100))

    assertError(t, err, "cannot withdraw, insufficient funds")
    assertBalance(t, wallet, startingBalance)
})
```

`t.Fatal` çağrıldığı zaman testlerimiz duracak, bunu yapmamızın sebebi hata almamız gereken yerde hata almadık fakat burada kesinlikle hata almamız gerekiyordu ve verdiğimiz hata mesajı ile neden durulduğunundan bahsettik. Bu olmazsa, gelen değer `nil` olduğu için panic atacaktı ve yine duracaktı fakat panic attığı zaman hatanın ne olduğunu anlamak için kafa patlatmamız gerekirdi. `nil` yüzünden panic almamak için önceden mudahale edip `didn't get an error but wanted one` cümlesi ile sorunun ne olduğunu bildirdik.

## Testi çalıştırmayı dene

`wallet_test.go:61: got err 'oh no' want 'cannot withdraw, insufficient funds'`

## Testi geçecek kadar kod yaz

```go
func (w *Wallet) Withdraw(amount Bitcoin) error {

    if amount > w.balance {
        return errors.New("cannot withdraw, insufficient funds")
    }

    w.balance -= amount
    return nil
}
```

## Refactor

`Withdraw` ve bunun için yazılmış test kodlarından hata mesajlarının kopyası var.

Birisi hata mesajını tekrardan düzenlemek isterse, testler üzerinde de gidip bunu değiştirmesi çok can sıkıcı olabilir. Hata mesajının tam olarak neyi ifade ettiği umurumuzda değil, sadece belilir koşullarda para çekme işlemi yapıldığında bir hata mesajı alacağız.

Go'da errorlar birer değerdir. Yani bunları bir değişkene atayıp tek bir kaynaktan edinmeyi sağlayabiliriz.

```go
var ErrInsufficientFunds = errors.New("cannot withdraw, insufficient funds")

func (w *Wallet) Withdraw(amount Bitcoin) error {

    if amount > w.balance {
        return ErrInsufficientFunds
    }

    w.balance -= amount
    return nil
}
```

`var` keywordu, global olarak paket içinde bir değişken tanımlamamıza izin veriyor.

Bu şekilde bir değişiklik ile `Withdraw` metotumuz çok daha anlaşılır görünüyor.

Ardından, test kodlarımızda belirli string tanımlamaları yerine, yeni yaptığımız hata tanımlamasını kullanabiliriz ve bunun dışında helperlar ile ilgili bir refactorde uygulayalım.

```go
func TestWallet(t *testing.T) {

    t.Run("Deposit", func(t *testing.T) {
        wallet := Wallet{}
        wallet.Deposit(Bitcoin(10))
        assertBalance(t, wallet, Bitcoin(10))
    })

    t.Run("Withdraw with funds", func(t *testing.T) {
        wallet := Wallet{Bitcoin(20)}
        wallet.Withdraw(Bitcoin(10))
        assertBalance(t, wallet, Bitcoin(10))
    })

    t.Run("Withdraw insufficient funds", func(t *testing.T) {
        wallet := Wallet{Bitcoin(20)}
        err := wallet.Withdraw(Bitcoin(100))

        assertError(t, err, ErrInsufficientFunds)
        assertBalance(t, wallet, Bitcoin(20))
    })
}

func assertBalance(t testing.TB, wallet Wallet, want Bitcoin) {
    t.Helper()
    got := wallet.Balance()

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}

func assertError(t testing.TB, got, want error) {
    t.Helper()
    if got == nil {
        t.Fatal("didn't get an error but wanted one")
    }

    if got != want {
        t.Errorf("got %q, want %q", got, want)
    }
}
```

Test kodlarını takip etmek ve anlamak çok daha kolay oldu.

Helperları ana test fonksiyonunun dışına çıkardık, böylece biri bu dosyayı açtığında yardımcıları görmek yerine testlerin geçmesi için iddia ettiklerimizi görecek.

Test yazmanın faydaları arasında kodumuzun gerçek kullanımının nasıl olduğunu başka geliştiricilerinde anlamasına yardımcı olması vardır.

### Kontrol edilmemiş hatalar

Go derleyicisi size çok fazla yardımcı olmasına rağmen gözden kaçırabileceğiniz şeyler olur ve işlenecek hataları bulmak, görmek bazen zor olabilir.

Test etmediğimiz bir senaryo daha var, bunu bulmak için Go'da mevcut birçok linterdan biri olan `errcheck`'i kurabilirsiniz. `errcheck`'ı edinmek için terminalde şunu çalıştırın:

`go get -u github.com/kisielk/errcheck`

`errcheck` paketini edindikten sonra, kontrol etmek istediğiniz dizine gidin ve terminalde `errcheck .` komutunu çalıştırın.

Şu şekilde bir şey göreceksin:

`wallet_test.go:17:18: wallet.Withdraw(Bitcoin(10))`

Gördüğünüz şeyin bize söylediği, `wallet_test.go` dosyasında belirtilen satırdaki error checki yapmadığımız. Bu senaryo içinde err check'in yapılmasını sağlayabiliriz.

Test kodlarımızın en son hali şu şekilde olacak:

```go
func TestWallet(t *testing.T) {

    t.Run("Deposit", func(t *testing.T) {
        wallet := Wallet{}
        wallet.Deposit(Bitcoin(10))

        assertBalance(t, wallet, Bitcoin(10))
    })

    t.Run("Withdraw with funds", func(t *testing.T) {
        wallet := Wallet{Bitcoin(20)}
        err := wallet.Withdraw(Bitcoin(10))

        assertNoError(t, err)
        assertBalance(t, wallet, Bitcoin(10))
    })

    t.Run("Withdraw insufficient funds", func(t *testing.T) {
        wallet := Wallet{Bitcoin(20)}
        err := wallet.Withdraw(Bitcoin(100))

        assertError(t, err, ErrInsufficientFunds)
        assertBalance(t, wallet, Bitcoin(20))
    })
}

func assertBalance(t testing.TB, wallet Wallet, want Bitcoin) {
    t.Helper()
    got := wallet.Balance()

    if got != want {
        t.Errorf("got %s want %s", got, want)
    }
}

func assertNoError(t testing.TB, got error) {
    t.Helper()
    if got != nil {
        t.Fatal("got an error but didn't want one")
    }
}

func assertError(t testing.TB, got error, want error) {
    t.Helper()
    if got == nil {
        t.Fatal("didn't get an error but wanted one")
    }

    if got != want {
        t.Errorf("got %s, want %s", got, want)
    }
}
```

## Özetlersek

### Pointerlar

* Go metotlara veya fonksiyonlara ilettiğiniz değerlerin bir kopyasını(`pass by value`) gönderir. Bu nedenle, bir değerin gerçek durumunun değiştirilmesi gerektiği durumalarda pointer kullanmalısınız.
* Go'nun değerlerin kopyasını göndermesi bir çok durumda yararlıdır fakat bazı durumlarda sisteminizin bir verinin kopyası üzerinde çalışmasını veya gerçek veriler dışında bir kopya oluşturması istemezsiniz, bu durumlarda o değeri referans edecek bir pointer kullanmalısınız. Örnek olarak, çok büyük veri yapıları ile çalışırken veya yalnızca bir verinin sadece bir örneği olması gerektiği durumlar mesela: veritabanı bağlantıları.

### nil

* Pointerlar nil olabilir
* Bir fonksiyon bir şeye point eden bir değer döndürdüğü zaman, nil olup olmadığını kontrol ettiğinizden emin olmalısınız veya **runtime panic** oluşturursunuz ve derleyici size burada yardımcı olamaz.
* Unutulacak bir değeri tanımlamak istediğiniz zaman biçilmiş bir kaftandır.

### Errorlar

* Hatalar bir fonksiyonun veya metot çağırırken, çağrıldığı yerde olan kod bloklarında bir hatanın olduğunu belirtmek için kullanılır.
* Test kodlarımızı gözlemleyerek, bir `string` üzerinden hata kontrolu yapmamız tuhaf bir şey olacağını varsaydık. Bu nedenle, hata döndürme implementasyonumuzda daha anlamlı bir şekilde hata dönmesini sağladık ve bu kodun daha da kolay test edilebilmesini sağladı. Ayrıca bu yaptığımız, bizim kodumuz bir API'ya olarak kullanacak başka geliştiriciler için de daha kolay anlaşılabilir olacağına karar verdik.
* Bu hata işleme ile ilgili hikayenin sadece başlangıcıydı. Daha karmaşık şeylerde yapabiliriz. Sonraki bölümlerde hata yakalama ile ilgili daha fazla strateji ele alınacak.
* [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)

### Mevcüt olan tiplerden, yenisini türetmek

* Değerler için çalışılan alana özgü daha anlaşılır bir anlam yüklendirebileceğiniz durumlarda kullanışlıdır.
* Arayüzleri implemente etmenize izin verir

Go dili ile yazılımlar üretirken hataları ve pointerları bolca kullanacaksınız, bu duruma dair bir önyargınızın, çekincenizin olmaması lazım. Go derleyicisi, size genellikle yardımcı olur, bir hata yaptığınızda acele etmeden hata mesajını okuyun, hata mesajları sizi çözüme götürecektir.
