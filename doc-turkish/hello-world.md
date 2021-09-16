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

`varName := value` söz dizimi ile tekrar kullanabildiğimiz ve okunabilirliği arttırmak için bazı variablelar tanımladık.

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

Açık olmak gerekirse, çok ahım şahım bir performans artışı olmayacak yani bunu göz ardı edebilirsiniz. Fakat bazı değerleri anlamladırmak ve bazen de performansa yardımcı olması için constantlar oluşturmayı düşünmeye değer.

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

Testlerinizde bulunan kodlarında _açık ve net_ olması önemlidir.

Test kodlarımızı refacto edelim,

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

`t.Helper()` is needed to tell the test suite that this method is a helper. By doing this when it fails the line number reported will be in our _function call_ rather than inside our test helper. This will help other developers track down problems easier. If you still don't understand, comment it out, make a test fail and observe the test output. Comments in Go are a great way to add additional information to your code, or in this case, a quick way to tell the compiler to ignore a line. You can comment out the `t.Helper()` code by adding two forward slashes `//` at the beginning of the line. You should see that line turn grey or change to another color than the rest of your code to indicate it's now commented out.

Now that we have a well-written failing test, let's fix the code, using an `if`.

```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
	if name == "" {
		name = "World"
	}
	return englishHelloPrefix + name
}
```

If we run our tests we should see it satisfies the new requirement and we haven't accidentally broken the other functionality.

### Tekrardan source control

Now we are happy with the code I would amend the previous commit so we only
check in the lovely version of our code with its test.

### Disiplin

Let's go over the cycle again

* Write a test
* Make the compiler pass
* Run the test, see that it fails and check the error message is meaningful
* Write enough code to make the test pass
* Refactor

On the face of it this may seem tedious but sticking to the feedback loop is important.

Not only does it ensure that you have _relevant tests_, it helps ensure _you design good software_ by refactoring with the safety of tests.

Seeing the test fail is an important check because it also lets you see what the error message looks like. As a developer it can be very hard to work with a codebase when failing tests do not give a clear idea as to what the problem is.

By ensuring your tests are _fast_ and setting up your tools so that running tests is simple you can get in to a state of flow when writing your code.

By not writing tests you are committing to manually checking your code by running your software which breaks your state of flow and you won't be saving yourself any time, especially in the long run.

## Devam et! Daha fazla gereksinim

Goodness me, we have more requirements. We now need to support a second parameter, specifying the language of the greeting. If a language is passed in that we do not recognise, just default to English.

We should be confident that we can use TDD to flesh out this functionality easily!

Write a test for a user passing in Spanish. Add it to the existing suite.

```go
	t.Run("in Spanish", func(t *testing.T) {
		got := Hello("Elodie", "Spanish")
		want := "Hola, Elodie"
		assertCorrectMessage(t, got, want)
	})
```

Remember not to cheat! _Test first_. When you try and run the test, the compiler _should_ complain because you are calling `Hello` with two arguments rather than one.

```text
./hello_test.go:27:19: too many arguments in call to Hello
    have (string, string)
    want (string)
```

Fix the compilation problems by adding another string argument to `Hello`

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "World"
	}
	return englishHelloPrefix + name
}
```

When you try and run the test again it will complain about not passing through enough arguments to `Hello` in your other tests and in `hello.go`

```text
./hello.go:15:19: not enough arguments in call to Hello
    have (string)
    want (string, string)
```

Fix them by passing through empty strings. Now all your tests should compile _and_ pass, apart from our new scenario

```text
hello_test.go:29: got 'Hello, Elodie' want 'Hola, Elodie'
```

We can use `if` here to check the language is equal to "Spanish" and if so change the message

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

The tests should now pass.

Now it is time to _refactor_. You should see some problems in the code, "magic" strings, some of which are repeated. Try and refactor it yourself, with every change make sure you re-run the tests to make sure your refactoring isn't breaking anything.

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

* Write a test asserting that if you pass in `"French"` you get `"Bonjour, "`
* See it fail, check the error message is easy to read
* Do the smallest reasonable change in the code

You may have written something that looks roughly like this

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

When you have lots of `if` statements checking a particular value it is common to use a `switch` statement instead. We can use `switch` to refactor the code to make it easier to read and more extensible if we wish to add more language support later

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

Write a test to now include a greeting in the language of your choice and you should see how simple it is to extend our _amazing_ function.

### son...bir...refactor?

You could argue that maybe our function is getting a little big. The simplest refactor for this would be to extract out some functionality into another function.

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

A few new concepts:

* In our function signature we have made a _named return value_ `(prefix string)`.
* This will create a variable called `prefix` in your function.
  * It will be assigned the "zero" value. This depends on the type, for example `int`s are 0 and for `string`s it is `""`.
    * You can return whatever it's set to by just calling `return` rather than `return prefix`.
  * This will display in the Go Doc for your function so it can make the intent of your code clearer.
* `default` in the switch case will be branched to if none of the other `case` statements match.
* The function name starts with a lowercase letter. In Go public functions start with a capital letter and private ones start with a lowercase. We don't want the internals of our algorithm to be exposed to the world, so we made this function private.

## Sonuç

Who knew you could get so much out of `Hello, world`?

By now you should have some understanding of:

### Some of Go's syntax around

* Writing tests
* Declaring functions, with arguments and return types
* `if`, `const` and `switch`
* Declaring variables and constants

### The TDD process and _why_ the steps are important

* _Write a failing test and see it fail_ so we know we have written a _relevant_ test for our requirements and seen that it produces an _easy to understand description of the failure_
* Writing the smallest amount of code to make it pass so we know we have working software
* _Then_ refactor, backed with the safety of our tests to ensure we have well-crafted code that is easy to work with

In our case we've gone from `Hello()` to `Hello("name")`, to `Hello("name", "French")` in small, easy to understand steps.

This is of course trivial compared to "real world" software but the principles still stand. TDD is a skill that needs practice to develop, but by breaking problems down into smaller components that you can test, you will have a much easier time writing software.
