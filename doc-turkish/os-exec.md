# OS Exec

**[Bu bölümün bütün kodlarını burada bulabilirsiniz](https://github.com/quii/learn-go-with-tests/tree/main/q-and-a/os-exec)**

[keith6014](https://www.reddit.com/user/keith6014), [reddit](https://www.reddit.com/r/golang/comments/aaz8ji/testdata_and_function_setup_help/)'te soruyor

> `os/exec.Command()` kullanarak XML verisi üreten bir komut çalıştırıyorum. Bu komut, `GetData()` adlı bir fonksiyon içinde çalıştırılacak.

> GetData()'yı test etmek için kendi oluşturduğum bir test verisine sahibim.

> _test.go dosyamda TestGetData adında bir testim var ve bu test, GetData() fonksiyonunu çağırıyor, ancak bu fonksiyon os.exec kullanıyor. Bunun yerine test verilerimi kullanmasını istiyorum.

> Bunu başarmanın iyi bir yolu nedir? GetData'yı çağırırken bir "test" bayrak moduna sahip olmalı mıyım, böylece GetData(mode string) gibi bir dosyayı okuyacak mı?

Bir kaç şey;

- Bir şeyin test edilmesi zor olduğunda, bu genellikle endişelerin ayrılmasının pek doğru olmamasından kaynaklanır.
- Kodunuza "test modları" eklemeyin, yerine [Dependency Injection](/dependency-injection.md) kullanın, böylelikle endişeleri ayırabilir ve kendi bağımlılıklarınızı modelleyebilirsiniz.

Kodun neye benzeyeceğini tahmin etmeye çalıştım:

```go
type Payload struct {
	Message string `xml:"message"`
}

func GetData() string {
	cmd := exec.Command("cat", "msg.xml")

	out, _ := cmd.StdoutPipe()
	var payload Payload
	decoder := xml.NewDecoder(out)

	// these 3 can return errors but I'm ignoring for brevity
	cmd.Start()
	decoder.Decode(&payload)
	cmd.Wait()

	return strings.ToUpper(payload.Message)
}
```

- Sürece harici bir komut yürütmenize izin veren `exec.Command`ı kullanır.
- Çıktıyı `cmd.StdoutPipe`ta yakalıyoruz ve bu bıze bır `io.ReadCloser` döndürüyor (bu ileride önemli olacak).
- Kodun az çok geri kalan kısmı [mükemmel dokümantasyondan](https://golang.org/pkg/os/exec/#example_Cmd_StdoutPipe) kopyalanıp yapıştırılmıştır.
	- Stdout'tan herhangi bir çıktıyı `io.ReadCloser` içine alıyoruz ve komutu `başlatıyoruz`, ardından `Wait`i çağırarak tüm verilerin okunmasını bekliyoruz. Bu iki çağrı arasında verinin kodunu 'Payload' yapımızda çözüyoruz.

İşte `msg.xml` içinde bulunanlar:

```xml
<payload>
    <message>Happy New Year!</message>
</payload>
```

Bunu uygulamalı olarak göstermek için basit bir test yazdım:

```go
func TestGetData(t *testing.T) {
	got := GetData()
	want := "HAPPY NEW YEAR!"

	if got != want {
		t.Errorf("got %q, want %q", got, want)
	}
}
```

## Test edilebilir kod

Test edilebilir kod ayrıştırılmış ve tek amaçlıdır. Bana, bu kod için iki ana endişe var gibi geliyor:

1. Ham XML verilerini alma
2. XML verilerinin kodunun çözülmesi ve iş mantığımızın uygulanması (bu senaryoda `<message>`daki `strings.ToUpper`)

İlk kısım sadece standart kütüphanedeki örneği kopyalıyor.

İkinci kısım iş mantığımızın oluştuğu yerdir ve koda bakarak mantığımızdaki "dikişin" nerede başladığını görebiliriz; `io.ReadCloser`ımızı aldığımız yer burasıdır. Bu mevcut soyutlamayı endişeleri ayırmak ve kodumuzu test edilebilir hale getirmek için kullanabiliriz.

**GetData ile ilgili sorun, iş mantığının XML alma araçlarıyla bağlantılı olmasıdır. Tasarımımızı daha iyi hale getirmek için onları ayırmamız gerekiyor**

`TestGetData`mız iki endişemiz arasındaki entegrasyon testimiz olarak hareket edebilir, bu yüzden çalışmaya devam ettiğinden emin olmak için onu elimizde tutacağız.

İşte yeni ayırılmış kod böyle gözüküyor:

```go
type Payload struct {
	Message string `xml:"message"`
}

func GetData(data io.Reader) string {
	var payload Payload
	xml.NewDecoder(data).Decode(&payload)
	return strings.ToUpper(payload.Message)
}

func getXMLFromCommand() io.Reader {
	cmd := exec.Command("cat", "msg.xml")
	out, _ := cmd.StdoutPipe()

	cmd.Start()
	data, _ := io.ReadAll(out)
	cmd.Wait()

	return bytes.NewReader(data)
}

func TestGetDataIntegration(t *testing.T) {
	got := GetData(getXMLFromCommand())
	want := "HAPPY NEW YEAR!"

	if got != want {
		t.Errorf("got %q, want %q", got, want)
	}
}
```

Now that `GetData` takes its input from just an `io.Reader` we have made it testable and it is no longer concerned how the data is retrieved; people can re-use the function with anything that returns an `io.Reader` (which is extremely common). For example we could start fetching the XML from a URL instead of the command line.
Artık `GetData` girdisini yalnızca `io.Reader`dan aldığından onu test edilebilir hale getirdik ve artık verilerin nasıl alındığıyla ilgilenmiyoruz; insanlar bu işlevi `io.Reader` döndüren herhangi bir şeyle yeniden kullanabilirler (ki bu son derece yaygındır). Örneğin XML'i komut satırı yerine bir URL'den almaya başlayabiliriz.

```go
func TestGetData(t *testing.T) {
	input := strings.NewReader(`
<payload>
    <message>Cats are the best animal</message>
</payload>`)

	got := GetData(input)
	want := "CATS ARE THE BEST ANIMAL"

	if got != want {
		t.Errorf("got %q, want %q", got, want)
	}
}

```

İşte `GetData` için bir birim testi örneği.

Go testinde endişeleri ayırarak ve mevcut soyutlamaları kullanarak önemli iş mantığımızı kullanmak çocuk oyuncağıdır.

Bu sayfa [@rasimthegrey](https://github.com/rasimthegrey) tarafından çevrildi.