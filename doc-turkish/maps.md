# Mapler

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/maps)**

[Arrayler & slicelar](arrays-and-slices.md) bölümünde, değerleri nasıl saklayacağımızı gördünüz. Şimd, `key` ile değerleri nasıl sakladığımıza ve hızlıca bakabildiğimiz inceleyeceğiz.

Mapler, öğeleri sözlüğe benzer şekilde saklamanızı sağlar. `key`'i kelime, `value`'yu tanım olarak düşünebilirsiniz. Mapleri öğrenmenin kendi sözlüğümüz oluşturmaktan daha iyi yolu nedir?

İlk olarak, kelimelerin ve anlamlarının olduğu bir sözlüğümüzün olduğunu varsayalım, eğer bir kelimeye bararsak sözlük bize onun anlamını dönmeli.

## İlk olarak test yaz

`dictionary_test.go` içerisinde

```go
package main

import "testing"

func TestSearch(t *testing.T) {
    dictionary := map[string]string{"test": "this is just a test"}

    got := Search(dictionary, "test")
    want := "this is just a test"

    if got != want {
        t.Errorf("got %q want %q given, %q", got, want, "test")
    }
}
```

Map tanımlamak  array tanımlamaya benziyor. Aradaki fark, `map` keywordu ile başlıyor ve iki tip gereklidir. İlki key tipi, `[]` içerisine yazılır. İkincisi value tipi,`[]`'in sağına yazılır.

Key özeldir. Sadece kıyaslanabilir (comparable) tip olabilir. Eğer iki keyin eşitliği söyleyemezsek doğru değerin aldığımızdan emin olamayız.  Kıyaslanabilir (Comparable)  tipler [dil özellikleri (language spec)](https://golang.org/ref/spec#Comparison_operators) bölümünde detaylıca açıklanmıltır.

Value tipi herhangi bir tip olabilir.  Başka bir map bile olabilir.

Bu testteki her şey tanıdık gelmeli.

## Testi çalıştırmayı dene

`go test` komutnu çalıştırdığımızda derleyici `./dictionary_test.go:8:9: undefined: Search` hatası ile başarısız olacaktır.

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`dictionary.go` içerisinde

```go
package main

func Search(dictionary map[string]string, word string) string {
    return ""
}
```

Şimdi tesrin *temiz bir hata mesajı* verecektir 

`dictionary_test.go:12: got '' want 'this is just a test' given, 'test'`.

## Testi geçecek kadar kod yaz

```go
func Search(dictionary map[string]string, word string) string {
    return dictionary[word]
}
```

Map'ten value almak Array'den value almak ile aynıdır `map[key]`.

## Refactor

```go
func TestSearch(t *testing.T) {
    dictionary := map[string]string{"test": "this is just a test"}

    got := Search(dictionary, "test")
    want := "this is just a test"

    assertStrings(t, got, want)
}

func assertStrings(t testing.TB, got, want string) {
    t.Helper()

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

İmplementasyonu daha genel yapmak için `assertStrings` yardımcısını oluşturdum.

### Özel tip ile kullnma

Map etrafında yeni bir tür oluşturarak ve `Seach`'ü metod halien getirerek sözlüğümüzn kullanımını iyileştirebiliriz.

`dictionary_test.go` içerisinde:

```go
func TestSearch(t *testing.T) {
    dictionary := Dictionary{"test": "this is just a test"}

    got := dictionary.Search("test")
    want := "this is just a test"

    assertStrings(t, got, want)
}
```

Daha tanımlamadığımız, `Dictionary` tipini kullanmaya başladık. Sonra `Dictionary` insanceında `Search`'ü çağırdık.

`assertStrings` değiştirmemize gerek yok.

`dictionary.go` içerisinde:

```go
type Dictionary map[string]string

func (d Dictionary) Search(word string) string {
    return d[word]
}
```

`map`'i wrapper eden `Dictionary` tipini oluşturduk. Özel tip tanımı ile `Search` metodunu oluşturabiliriz.

## İlk olarak test yaz

Basit arama implemente etmesi oldukçta kolay ama aradığımız kelime sözlükte yoksa ne olacak?

Aslında hiçbir şey almayacağız. Bu iyi çünkü program çalışmaya devam edecek ama daha iyi bir yöntem var. Fonksiyon kelimenin sözlükte olmadığınu rapor edebilir. Bu şekilde, kullanıcı, kelimenin var olup olmadığını veya sadece bir tanımının olup olmadığını merak etmez (bu bir sözlük için pek kullanışlı görünmeyebilir. Ancak, bu, diğer kullanım durumlarında anahtar olabilecek bir durumdur).

```go
func TestSearch(t *testing.T) {
    dictionary := Dictionary{"test": "this is just a test"}

    t.Run("known word", func(t *testing.T) {
        got, _ := dictionary.Search("test")
        want := "this is just a test"

        assertStrings(t, got, want)
    })

    t.Run("unknown word", func(t *testing.T) {
        _, err := dictionary.Search("unknown")
        want := "could not find the word you were looking for"

        if err == nil {
            t.Fatal("expected to get an error.")
        }

        assertStrings(t, err.Error(), want)
    })
}
```

Go'da bu durumu handle etmek için ikinci bir argüman olan `Error` tipi döndürülür.

assertion'a gönderdiğimiz gibi `Error`ler `.Error()` metodu ile stringe dönüştürülebilir. Ayrıca `nil` durumdunda `assertStrings`'i `if` ile `.Error()` çağırmaktan koruyoruz.

## Dene ve testi çalıştır

Derlenmeyecektir

```
./dictionary_test.go:18:10: assignment mismatch: 2 variables but 1 values
```

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

```go
func (d Dictionary) Search(word string) (string, error) {
    return d[word], nil
}
```

Şimdi testiniz daha anlaşılır hata mesajı ile başarısız olacaktır.

`dictionary_test.go:22: expected to get an error.`

## Write enough code to make it pass

```go
func (d Dictionary) Search(word string) (string, error) {
    definition, ok := d[word]
    if !ok {
        return "", errors.New("could not find the word you were looking for")
    }

    return definition, nil
}
```

Bunu geçebilmek için mapin ilginç bir özelliğini kullanıyoruz. İki değer dönebilir. İkinci değer boolean yani  key'in başarılı bir şekilde döndüüğünü belirtir.

Bu özellik tanımı olmayan ve sözlükte olmayan kelimelerin farkını ayırt etmemizi sağlar.

## Refactor

```go
var ErrNotFound = errors.New("could not find the word you were looking for")

func (d Dictionary) Search(word string) (string, error) {
    definition, ok := d[word]
    if !ok {
        return "", ErrNotFound
    }

    return definition, nil
}
```

`Search` fonksiyonundaki sihirli hatayı değişkene atayarak kurtulabilirizi. Daha iyi bir testimizin olmasını sağlayacak.

```go
t.Run("unknown word", func(t *testing.T) {
    _, got := dictionary.Search("unknown")

    assertError(t, got, ErrNotFound)
})

func assertError(t testing.TB, got, want error) {
    t.Helper()

    if got != want {
        t.Errorf("got error %q want %q", got, want)
    }
}
```

Yeni bir yardımcı oluşturarak testlerimizi basitleştirdik ve `ErrNotFound` kullanmaya başladık bu sayede ileride hata mesajını değiştirsek bile testimiz başarısız olmayacaktır.

## İlk olarak test yaz

Sözlükte arama yapmak için harika bir yolumuz var. Ne yazık ki, sözlüğe yeni kelimeler ekleyebileceğimiz bir yol yok.

```go
func TestAdd(t *testing.T) {
    dictionary := Dictionary{}
    dictionary.Add("test", "this is just a test")

    want := "this is just a test"
    got, err := dictionary.Search("test")
    if err != nil {
        t.Fatal("should find added word:", err)
    }

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

Bu testte, sözlüğün geçerliğiliğini kolaylaştırmak için `Seach` fonksiyonunun kullanışlı hale getiriyoruz.

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`dictionary.go` içerisinde

```go
func (d Dictionary) Add(word, definition string) {
}
```

Şimdi testiniz başarısız olmalı 

```
dictionary_test.go:31: should find added word: could not find the word you were looking for
```

## Testi geçecek kadar kod yaz

```go
func (d Dictionary) Add(word, definition string) {
    d[word] = definition
}
```

Map'e ekleme yapmak array'a oldukça benzer. Sadece keyi belirlemeniz ve değerini value'ya atamanız gerekli.

### Pointerlar, kopyalar, et al

Maplerin ilginç bir özelliği de adresslerini göndermeden düzenleme yapabilmeniz(örn `&myMap`)

Bu onları "reference tip" gibi _hissettiriyor_ [ancak Dave Cheney anlattığı gibi](https://dave.cheney.net/2017/04/30/if-a-map-isnt-a-reference-variable-what-is-it) değiller.

> Map value'su runtime.hmap structurena bir pointerdır.

Bir mapi fonksiyon/metoda'a gönderdiğinizde sadece kopyalıyorsunuz, pointer kısmını, verileri içeren temel veri yapısını değil.

Mapler hakkında şaşırtıcı olan valueları `nil` olabilir. `nil` map okuğunuzda boş bir map gibi davranır ama `nil` mape yazma işlemi çalışma anı (runtime) hatasına sebep olur. Mapler hakkında daha fazlasını [burada](https://blog.golang.org/go-maps-in-action) okuyabilirsiniz.

Bu nedenle, asla boş bir map oluşturmayın:

```go
var m map[string]string
```

Bunun yerine, yukarıda yaptığımız gibi boş bir map oluşturabilirsinzi veya map oluşturmak için `make` keywordünü oluşturabilirsiniz:

```go
var dictionary = map[string]string{}

// VEYA

var dictionary = make(map[string]string)
```
 
İki yöntem de boş `hash map` oluşturur ve `dictionary`'e işaret eder. Bu sayede çalışma zamanı (runtime) hatası almazsınız.

## Refactor

Refactor yapmamız için çok fazla bir şey yok ancak test biraz basitleştirme kullanabilir.

```go
func TestAdd(t *testing.T) {
    dictionary := Dictionary{}
    word := "test"
    definition := "this is just a test"

    dictionary.Add(word, definition)

    assertDefinition(t, dictionary, word, definition)
}

func assertDefinition(t testing.TB, dictionary Dictionary, word, definition string) {
    t.Helper()

    got, err := dictionary.Search(word)
    if err != nil {
        t.Fatal("should find added word:", err)
    }

    if definition != got {
        t.Errorf("got %q want %q", got, definition)
    }
}
```

kelime ve tanım için değişkenler oluşturduk ve tanım assertionını kendi yardımcı fonksiyonuna taşıdık.

`Add` iyi gözüküyor. Ancak, Eklemeye çalıştığımız değer zaten mevcutsa ne olacağını düşünmedik!

Eğer value varsa map hata fırlatmayacak. Onun yerine, yeni değeri eskisinin üzerine yazacak. Pratikte kullanışlı ancak fonksiyon ismi doğru yapmaz . `Add` var olan değeri değiştirmemeli.  Sadece sözlüğe yeni kelimeyi eklemeli.

## İlk olarak test yaz

```go
func TestAdd(t *testing.T) {
    t.Run("new word", func(t *testing.T) {
        dictionary := Dictionary{}
        word := "test"
        definition := "this is just a test"

        err := dictionary.Add(word, definition)

        assertError(t, err, nil)
        assertDefinition(t, dictionary, word, definition)
    })

    t.Run("existing word", func(t *testing.T) {
        word := "test"
        definition := "this is just a test"
        dictionary := Dictionary{word: definition}
        err := dictionary.Add(word, "new test")

        assertError(t, err, ErrWordExists)
        assertDefinition(t, dictionary, word, definition)
    })
}
...
func assertError(t testing.TB, got, want error) {
	t.Helper()
	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

Bu test için, `Add` fonksiyonun hata ,`ErrWordExists`, döndürmesi için değiştirdik bu sayede yeni hata değişkenine karşı doğrulama yapıyoruz.   Ayrıca önceki testi, `assertError` fonksiyonunda olduğu gibi `nil` hatası için düzenledik.

## Testi çalıştırmayı dene

Derleyici hata verecektir çünkü `Add` için bir değer dönmüyoruz.

```
./dictionary_test.go:30:13: dictionary.Add(word, definition) used as value
./dictionary_test.go:41:13: dictionary.Add(word, "new test") used as value
```

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

`dictionary.go` içinde

```go
var (
    ErrNotFound   = errors.New("could not find the word you were looking for")
    ErrWordExists = errors.New("cannot add word because it already exists")
)

func (d Dictionary) Add(word, definition string) error {
    d[word] = definition
    return nil
}
```

Şimdi iki hata alıyoruz. Hala valueyu değiştiryor ve hata için `nil` dönüyoruz.

```
dictionary_test.go:43: got error '%!q(<nil>)' want 'cannot add word because it already exists'
dictionary_test.go:44: got 'new test' want 'this is just a test'
```

## Testi geçecek kadar kod yaz

```go
func (d Dictionary) Add(word, definition string) error {
    _, err := d.Search(word)

    switch err {
    case ErrNotFound:
        d[word] = definition
    case nil:
        return ErrWordExists
    default:
        return err
    }

    return nil
}
```

Hata ile eşleşmesi için `switch` ifadesini kullanıyoruz. `switch`e sahip olmak `Search`'ün `ErrNotFound`'tan başka hata dönmesi durumlar için ekstra güvenlik sağlar.

## Refactor

Çok fazla refactor yapmamıza gerek yok ancak hata kullanımlarımız arttıkça bir kaç değişiklik yapabiliriz.

```go
const (
    ErrNotFound   = DictionaryErr("could not find the word you were looking for")
    ErrWordExists = DictionaryErr("cannot add word because it already exists")
)

type DictionaryErr string

func (e DictionaryErr) Error() string {
    return string(e)
}
```

Hataları sabit yaptık; Bu `error` interfacinden implemente eden kendi `DictionaryErr` tipimizi zorunlu kılıyor. [Dave Cheney'nin hazırladığı muhteşem makaleden](https://dave.cheney.net/2016/04/07/constant-errors) daha fazlasını okuyabilirsiniz. Basitçe söylemek gerekirse, hataları tekrar kullanılabilir ve değiştirilemez hale getirir.

Sıradaki, kelimenin anlamını güncelleyen `Update` fonksiyonunu oluşturalım.

## İlk olarak test yaz

```go
func TestUpdate(t *testing.T) {
    word := "test"
    definition := "this is just a test"
    dictionary := Dictionary{word: definition}
    newDefinition := "new definition"

    dictionary.Update(word, newDefinition)

    assertDefinition(t, dictionary, word, newDefinition)
}
```

`Update`, `Add` çok yakından ilişkili ve sonraki implementasyonumuz olacak.

## Dene ve testi çalıştır

```
./dictionary_test.go:53:2: dictionary.Update undefined (type Dictionary has no field or method Update)
```

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

Bunun gibi bir hata ile nasıl baş edebileceğimizi biliyoruz. Fonksiyonumuzu tanımlamamız gerekli.

```go
func (d Dictionary) Update(word, definition string) {}
```

Bununla, kelimenin tanımını değiştirmemiz gerektiğini görebiliyoruz.

```
dictionary_test.go:55: got 'this is just a test' want 'new definition'
```

## Testi geçecek kadar kod yaz

`Add` ile olan issueyu düzelttiğimizde bunu nasıl yapacağımızı görmüştük. `Add`e gerçekten benzeyen bir şey implemente edelim.

```go
func (d Dictionary) Update(word, definition string) {
    d[word] = definition
}
```

Basit bir değişiklik olduğu için bu konuda yapmamız gereken herhangi bir refactoring yok. Ancak, `Add` ile aynı sorunu yaşıyoruz. Eğer yeni bir kelime gönderirsek `Update` onu sözlüğe ekleyecek.

## İlk olarak test yaz

```go
t.Run("existing word", func(t *testing.T) {
    word := "test"
    definition := "this is just a test"
    newDefinition := "new definition"
    dictionary := Dictionary{word: definition}

    err := dictionary.Update(word, newDefinition)

    assertError(t, err, nil)
    assertDefinition(t, dictionary, word, newDefinition)
})

t.Run("new word", func(t *testing.T) {
    word := "test"
    definition := "this is just a test"
    dictionary := Dictionary{}

    err := dictionary.Update(word, definition)

    assertError(t, err, ErrWordDoesNotExist)
})
```

Kelimenin olmadığı durumlar için başka bir error tipi ekledik. Ayrıca `error` değeri dönmesi için `Update`'i değiştirdik.

## Dene ve testi çalıştır

```
./dictionary_test.go:53:16: dictionary.Update(word, newDefinition) used as value
./dictionary_test.go:64:16: dictionary.Update(word, definition) used as value
./dictionary_test.go:66:23: undefined: ErrWordDoesNotExist
```

Bu sefer 3 hata aldık ama nasıl çözeceğimizi biliyoruz.

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

```go
const (
    ErrNotFound         = DictionaryErr("could not find the word you were looking for")
    ErrWordExists       = DictionaryErr("cannot add word because it already exists")
    ErrWordDoesNotExist = DictionaryErr("cannot update word because it does not exist")
)

func (d Dictionary) Update(word, definition string) error {
    d[word] = definition
    return nil
}
```

Yeni error tipimizi ekledik ve `nil` error dönüyoruz.

Bu değişiklikler sayesinde daha anlaşılır hata alıyoruz:

```
dictionary_test.go:66: got error '%!q(<nil>)' want 'cannot update word because it does not exist'
```

## Testi geçecek kadar kod yaz

```go
func (d Dictionary) Update(word, definition string) error {
    _, err := d.Search(word)

    switch err {
    case ErrNotFound:
        return ErrWordDoesNotExist
    case nil:
        d[word] = definition
    default:
        return err
    }

    return nil
}
```

Bu fonksiyon, `dictionary`i güncelleme ve hata döndürme dışında `Add` ile neredeyse aynı.

### Update için yeni bir hata tanımlamayla ilgili not

`ErrNotFound` tekrar kullanabilirdik and yenir bir hata eklemezdil. Ancak, bir güncelleme başarısız olduğunda kesin bir hataya sahip olmak genellikle daha iyidir.

Spesifik hatalar ne olup bittiği hakkında daha fazla bilgi verir. Web uygulamasında bir örnek:

>`ErrNotFound` oluştuğunda kullanıcıyı yeniden yönlendirebilirsiniz ama `ErrWordDoesNotExist` hatasını görüntülersiniz..

Sonraki, sözlükte bir kelime silmek için `Delete` fonksiyonu oluşturalım.

## İlk olarak test yaz

```go
func TestDelete(t *testing.T) {
    word := "test"
    dictionary := Dictionary{word: "test definition"}

    dictionary.Delete(word)

    _, err := dictionary.Search(word)
    if err != ErrNotFound {
        t.Errorf("Expected %q to be deleted", word)
    }
}
```

Testimiz `Dictionary` ve bir kelime oluşturu daha sonra silindiğini kontrol eder.

## Testi çalıştırmayı dene

`go test` komutunu çalıştırarak :

```
./dictionary_test.go:74:6: dictionary.Delete undefined (type Dictionary has no field or method Delete)
```

## Testin çalışması için minimum kodu yaz ve başarısız test çıktılarını kontrol et

```go
func (d Dictionary) Delete(word string) {

}
```

Bunu ekledikten sonra, test bize kelimeyi silmediğimizi söylüyor.

```
dictionary_test.go:78: Expected 'test' to be deleted
```

## Testi geçecek kadar kod yaz

```go
func (d Dictionary) Delete(word string) {
    delete(d, word)
}
```

Go built-in olarak mapler üzerinde çalışan `delete` fonksiyonuna sahip. İki argüman alor. İlki map ve ikincisi kaldırılacak olan key.

`delete` fonksiyonu bir şey döndürmez ve `Delete` metodunu aynı düşünceye dayandırdık. Olmayan bir değeri silmenin bir etkisi olmadığından, `Update` ve `Add` gibi API'yı hatalar ile karmaşıklaştırmamıza gerek yok.

## Özetlersek

Bu bölümde çok konudan bahsettik. Sözlüğümüz tam bir CRUD (Create, Read, Update ve Delete) API oluşturduk.  
Süreç boyunca öğrendiklerimiz:

* Map oluşturma
* Map içerisinde item arama
* Mape yeni item ekleme
* Mapte item güncelleme
* Mapten item silme
* Hatalar hakkında öğrendiklerimiz
  * Constant hatalar oluşturma
  * Error wrappers yazma
