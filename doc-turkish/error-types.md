# Hata tipleri

**[Bu bölümün tüm kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/q-and-a/error-types)**

**Kendi hata tiplerinizi oluşturmak kodunuzu düzenlemenin zarif bir yolunu oluşturabilir, kodunuzu kullanmayı ve test etmeyi daha kolay hale getirebilir.**

Gopher Slack'ten Pedro soruyor:

> Eğer `fmt.Errorf("%s must be foo, got %s", bar, baz)` gibi bir hata oluşturursam, string değerlerini karşılaştırmadan eşitliği test etmenin bir yolu var mı?

Bu fikri keşfetmeye yardımcı bir fonksiyon oluşturalım:

```go
// DumbGetter will get the string body of url if it gets a 200
func DumbGetter(url string) (string, error) {
	res, err := http.Get(url)

	if err != nil {
		return "", fmt.Errorf("problem fetching from %s, %v", url, err)
	}

	if res.StatusCode != http.StatusOK {
		return "", fmt.Errorf("did not get 200 from %s, got %d", url, res.StatusCode)
	}

	defer res.Body.Close()
	body, _ := io.ReadAll(res.Body) // ignoring err for brevity

	return string(body), nil
}
```

Farklı nedenlerle başarısız olabilecek bir fonksiyon yazmak nadir bir durum değildir ve her senaryoyu doğru şekilde ele aldığımızdan emin olmak istiyoruz.

Pedro'nun dediği gibi, durum hatası için şu şekilde bir test yazabiliriz:

```go
t.Run("when you don't get a 200 you get a status error", func(t *testing.T) {

	svr := httptest.NewServer(http.HandlerFunc(func(res http.ResponseWriter, req *http.Request) {
		res.WriteHeader(http.StatusTeapot)
	}))
	defer svr.Close()

	_, err := DumbGetter(svr.URL)

	if err == nil {
		t.Fatal("expected an error")
	}

	want := fmt.Sprintf("did not get 200 from %s, got %d", svr.URL, http.StatusTeapot)
	got := err.Error()

	if got != want {
		t.Errorf(`got "%v", want "%v"`, got, want)
	}
})
```

Bu test her zaman `StatusTeapot` döndüren bir sunucu oluşturur ve ardından `DumbGetter` URL'sini argüman olarak kullanırız. Böylece `200` olmayan yanıtları doğru şekilde işlediğini görebiliriz.

## Bu test etme yönteminin sorunları

Bu kitap, _testlerinizi dinleyin_'i vurgulamaya çalışıyor ve bu test iyi _hissettirmiyor_:

- Bunu test etmek için, üretim kodunun yaptığı gibi aynı string'i oluşturuyoruz.
- Bu okumak ve yazmak için rahatsız edici.
- Tam hata mesajı string'i, aslında _ilgilendiğimiz_ şey mi?

Bu bize ne anlatıyor? Testimizin ergonomisi, kodumuzu kullanmaya çalışan başka bir kod parçasına yansıyacaktır.

Kodumuzun kullanıcısı, döndürdüğümüz belirli hata türlerine nasıl tepki veriyor? Yapabilecekleri en iyi şey, son derece hataya meyilli ve yazması korkunç olan hata mesajına bakmak.

## Ne yapmalıyız

TDD ile aşağıdaki düşünce yapısına girmenin faydasına sahip oluruz:

> _Ben_ bu kodu nasıl kullanmak isterdim?

`DumbGetter` için yapabileceğimiz şey, kullanıcıların tip sistemini kullanarak ne tür bir hata oluştuğunu anlamaları için bir yol sağlamaktır.

Peki `DumbGetter` bize, aşağıdakilere benzer bir şey döndürebilse:

```go
type BadStatusError struct {
	URL    string
	Status int
}
```

Büyülü bir string yerine, çalışmak için gerçek _verilere_ sahibiz.

Mevcut testimizi bu ihtiyacı karşılayacak şekilde değiştirelim:

```go
t.Run("when you don't get a 200 you get a status error", func(t *testing.T) {

	svr := httptest.NewServer(http.HandlerFunc(func(res http.ResponseWriter, req *http.Request) {
		res.WriteHeader(http.StatusTeapot)
	}))
	defer svr.Close()

	_, err := DumbGetter(svr.URL)

	if err == nil {
		t.Fatal("expected an error")
	}

	got, isStatusErr := err.(BadStatusError)

	if !isStatusErr {
		t.Fatalf("was not a BadStatusError, got %T", err)
	}

	want := BadStatusError{URL: svr.URL, Status: http.StatusTeapot}

	if got != want {
		t.Errorf("got %v, want %v", got, want)
	}
})
```

`BadStatusError` hata arabirimi uygulayacak şekilde değiştirmemiz gerekecek:

```go
func (b BadStatusError) Error() string {
	return fmt.Sprintf("did not get 200 from %s, got %d", b.URL, b.Status)
}
```

### Test ne yapar?

Tam hata string'ini kontrol etmek yerine hatanın bir `BadStatusError` olup olmadığını görmek için bir [tür belirtimi (type assertion)](https://tour.golang.org/methods/15) yapıyoruz. Belirtimin geçtiğini varsayarak, hatanın özelliklerinin doğru olduğunu kontrol edebiliriz.

Testi çalıştırdığımızda, bize doğru türde hata döndürmediğimizi söylüyor:

```
--- FAIL: TestDumbGetter (0.00s)
    --- FAIL: TestDumbGetter/when_you_dont_get_a_200_you_get_a_status_error (0.00s)
    	error-types_test.go:56: was not a BadStatusError, got *errors.errorString
```

Hata işleme kodumuzu kendi türümüzle güncelleyerek `DumbGetter`'ı düzeltelim:

```go
if res.StatusCode != http.StatusOK {
	return "", BadStatusError{URL: url, Status: res.StatusCode}
}
```


Bu değişiklik bazı _gerçek olumlu etkiler_ yarattı:

- `DumbGetter` işlevimiz daha basit hale geldi, artık bir hata string'inin incelikleri ile ilgilenmiyor, sadece bir `BadStatusError` oluşturuyor.
- Testlerimiz artık kodumuzun kullanıcısının, yalnızca günlüğe kaydetme dışında daha karmaşık hata işleme yapmaya karar vermesi durumunda neler _yapabileceğini_ yansıtıyor (ve belgeliyor). Sadece bir tür onayı yapın ve ardından hatanın özelliklerine kolayca erişin.
- Bu halen "sadece" bir `error` olduğu için, istediklerinde bunu call stack'e geçirebilirler veya diğer `error`'lar gibi kaydedebilirler."

## Özetlersek

Eğer kendinizi birçok hata durumunu test ederken bulursanız, hata mesajlarını karşılaştırma tuzağına düşmeyin.

Bu, istikrarsız ve okuması/yazması zor testlere yol açar ve kodunuzu kullananların, meydana gelen hataların türüne bağlı olarak işleri farklı şekilde yapmaya başlamaları gerektiğinde karşılaşacakları zorlukları yansıtır.

Testlerinizin her zaman kodunuzu nasıl kullanmak _istediğinizi_ yansıttığından emin olun. Bu nedenle, kendi hata türlerinizi özetlemek için hata türleri oluşturmayı düşünün. Bu, kodunuzu kullanan kullanıcılar için farklı türde hataların ele alınmasını kolaylaştırır ve aynı zamanda hata işleme kodu yazmayı daha basit ve okunması kolay hale getirir.

## Ek Bilgi

Go 1.13'ten itibaren, [Go Blog](https://blog.golang.org/go1.13-errors)'da standart kütüphanede hatalarla çalışmanın yeni yolları ele alınmaktadır.

```go
t.Run("when you don't get a 200 you get a status error", func(t *testing.T) {

	svr := httptest.NewServer(http.HandlerFunc(func(res http.ResponseWriter, req *http.Request) {
		res.WriteHeader(http.StatusTeapot)
	}))
	defer svr.Close()

	_, err := DumbGetter(svr.URL)

	if err == nil {
		t.Fatal("expected an error")
	}

	var got BadStatusError
	isBadStatusError := errors.As(err, &got)
	want := BadStatusError{URL: svr.URL, Status: http.StatusTeapot}

	if !isBadStatusError {
		t.Fatalf("was not a BadStatusError, got %T", err)
	}

	if got != want {
		t.Errorf("got %v, want %v", got, want)
	}
})
```

Bu durumda, hatayı denemek ve çıkarmak için [`errors.As`](https://pkg.go.dev/errors#example-As) kullanıyoruz ve hatamızı custom type'ımıza çıkarıyoruz. Bu, başarı durumunu temsil etmek için bir `bool` döndürür ve bizim için `got` içine çıkartır.

Bu sayfa [@rasimthegrey](https://github.com/rasimthegrey) tarafından çevrildi.