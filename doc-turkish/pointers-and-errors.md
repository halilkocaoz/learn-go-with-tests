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

`null`'da olduğu gibi, `nil` bir değere ulaşmaya çalışırsanız **runtime panic** hatası fırlatılacaktır. Bunun için, bu gibi değerlere ulaşmaya çalışmadan önce `nil` olup, olmadığını kontrol etmelisiniz.

## Dene ve testi çalıştır

`./wallet_test.go:31:25: wallet.Withdraw(Bitcoin(100)) used as value`

The wording is perhaps a little unclear, but our previous intent with `Withdraw` was just to call it, it will never return a value. To make this compile we will need to change it so it has a return type.

## Testin çalışması için için minimum kodu yaz ve başarısız test çıktılarını kontrol et

```go
func (w *Wallet) Withdraw(amount Bitcoin) error {
    w.balance -= amount
    return nil
}
```

Again, it is very important to just write enough code to satisfy the compiler. We correct our `Withdraw` method to return `error` and for now we have to return _something_ so let's just return `nil`.

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

Remember to import `errors` into your code.

`errors.New` creates a new `error` with a message of your choosing.

## Refactor

Let's make a quick test helper for our error check to improve the test's readability

```go
assertError := func(t testing.TB, err error) {
    t.Helper()
    if err == nil {
        t.Error("wanted an error but didn't get one")
    }
}
```

And in our test

```go
t.Run("Withdraw insufficient funds", func(t *testing.T) {
    startingBalance := Bitcoin(20)
    wallet := Wallet{startingBalance}
    err := wallet.Withdraw(Bitcoin(100))

    assertError(t, err)
    assertBalance(t, wallet, startingBalance)
})
```

Hopefully when returning an error of "oh no" you were thinking that we _might_ iterate on that because it doesn't seem that useful to return.

Assuming that the error ultimately gets returned to the user, let's update our test to assert on some kind of error message rather than just the existence of an error.

## Write the test first

Update our helper for a `string` to compare against.

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

And then update the caller

```go
t.Run("Withdraw insufficient funds", func(t *testing.T) {
    startingBalance := Bitcoin(20)
    wallet := Wallet{startingBalance}
    err := wallet.Withdraw(Bitcoin(100))

    assertError(t, err, "cannot withdraw, insufficient funds")
    assertBalance(t, wallet, startingBalance)
})
```

We've introduced `t.Fatal` which will stop the test if it is called. This is because we don't want to make any more assertions on the error returned if there isn't one around. Without this the test would carry on to the next step and panic because of a nil pointer.

## Try to run the test

`wallet_test.go:61: got err 'oh no' want 'cannot withdraw, insufficient funds'`

## Write enough code to make it pass

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

We have duplication of the error message in both the test code and the `Withdraw` code.

It would be really annoying for the test to fail if someone wanted to re-word the error and it's just too much detail for our test. We don't _really_ care what the exact wording is, just that some kind of meaningful error around withdrawing is returned given a certain condition.

In Go, errors are values, so we can refactor it out into a variable and have a single source of truth for it.

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

The `var` keyword allows us to define values global to the package.

This is a positive change in itself because now our `Withdraw` function looks very clear.

Next we can refactor our test code to use this value instead of specific strings.

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

And now the test is easier to follow too.

I have moved the helpers out of the main test function just so when someone opens up a file they can start reading our assertions first, rather than some helpers.

Another useful property of tests is that they help us understand the _real_ usage of our code so we can make sympathetic code. We can see here that a developer can simply call our code and do an equals check to `ErrInsufficientFunds` and act accordingly.

### Unchecked errors

Whilst the Go compiler helps you a lot, sometimes there are things you can still miss and error handling can sometimes be tricky.

There is one scenario we have not tested. To find it, run the following in a terminal to install `errcheck`, one of many linters available for Go.

`go get -u github.com/kisielk/errcheck`

Then, inside the directory with your code run `errcheck .`

You should get something like

`wallet_test.go:17:18: wallet.Withdraw(Bitcoin(10))`

What this is telling us is that we have not checked the error being returned on that line of code. That line of code on my computer corresponds to our normal withdraw scenario because we have not checked that if the `Withdraw` is successful that an error is _not_ returned.

Here is the final test code that accounts for this.

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

## Wrapping up

### Pointers

* Go copies values when you pass them to functions/methods, so if you're writing a function that needs to mutate state you'll need it to take a pointer to the thing you want to change.
* The fact that Go takes a copy of values is useful a lot of the time but sometimes you won't want your system to make a copy of something, in which case you need to pass a reference. Examples include referencing very large data structures or things where only one instance is necessary \(like database connection pools\).

### nil

* Pointers can be nil
* When a function returns a pointer to something, you need to make sure you check if it's nil or you might raise a runtime exception - the compiler won't help you here.
* Useful for when you want to describe a value that could be missing

### Errors

* Errors are the way to signify failure when calling a function/method.
* By listening to our tests we concluded that checking for a string in an error would result in a flaky test. So we refactored our implementation to use a meaningful value instead and this resulted in easier to test code and concluded this would be easier for users of our API too.
* This is not the end of the story with error handling, you can do more sophisticated things but this is just an intro. Later sections will cover more strategies.
* [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)

### Create new types from existing ones

* Useful for adding more domain specific meaning to values
* Can let you implement interfaces

Pointers and errors are a big part of writing Go that you need to get comfortable with. Thankfully the compiler will _usually_ help you out if you do something wrong, just take your time and read the error.
