# Install Go, set up environment for productivity

[Buradan](https://golang.org/doc/install) Go'nun resmi kurulum yönlendirmelerine ulaşabilirsiniz.

Yada OSX veya Ubuntu için yazdığımız yönlendirmeyi takip edebilirsiniz.

## Kurulum

### OSX

OSX sisteminize Go'yu yüklemek için yapmanız gerekenlerden ilki, Homebrew'ı yüklemek, fakat Homebrew'ı yükleme işlemi Xcode ile bağlantılıdır. Yani ilk önce Xcode'un yüklü olduğundan emin olun.

```sh
xcode-select --install
```

Ardından Homebrew'ı yüklemek için bu komutu çalıştırın

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Bundan sonra brew ile Go'yu yükleyebilirsiniz

```sh
brew install go
```

### Ubuntu

Ubuntu sisteminize Go'yu yüklemek için APT'nın sisteminizde yüklü olduğundan emin olun ve APT'ye bir depo tanımlayarak başlayalım

```sh
sudo add-apt-repository ppa:longsleep/golang-backports
```

Sonraysa APT'nin gerekli güncellemeleri yapmasını sağlayalım,

```sh
sudo apt update
```

Şimdiyse, Go programlama dilini yükleyebilirsiniz

```sh
sudo apt install golang-go
```

### Kurulumu doğrulamak

Kurulumu doğrulamak için aşağıdaki komutu çalıştırabilirsiniz.

```sh
go version
```

## Go Environment

### Go Modules

[Moduleler](https://github.com/golang/go/wiki/Modules) Go 1.11 ile duyuruldu. Bu yaklaşım, Go 1.16'dan beri varsayılan derleme moduludur, artık `GOPATH` kullanılması önerilmez.

Moduleler dependency management, version selection ve reproducible buildler ile ilgili sorunları çözmeyi amaçlar. Ayrıca yazılımcıların `GOPATH` dışında yazdığı Go kodunu çalıştırmalarına yardımcı olur.

Go Modullerini kullanmak oldukça basittir. Projenizin kök dizini olarak `GOPATH` dışında herhangi bir dizini seçebilirsiniz ve o dizinde `go mod init` komutu ile yeni bir modül oluşturarak kullanmaya başlayabilirsiniz.

Başarılı bir derleme için `go.mod` dosyası otomatik olarak oluşturulacaktır ve bu dosya gerekli olan diğer modüllerin yolunu Go sürümleri ile ve diğer bağımlılık gereksinimlerini içerir.

Eğer `<modulepath>` belirtilmediyse, `go mod init` komutu dizin yapısına bakarak modül yolunu tahmin etmeye çalışacaktır fakat siz yine de bir argüman sağlayarak `<modulepath>`'ı kendiniz belirtebilirsiniz.

```sh
mkdir my-project
cd my-project
go mod init <modulepath>
```

`go.mod` şu şekilde görünecek:

```
module cmd

go 1.16

```

`go help` ile mevcut tüm `go mod` komutları genel bir bakış atabilirsiniz.

```sh
go help mod
go help mod init
```

## Go Editör

Editör seçimi oldukça kişiseldir, zaten Go'yu destekleyen bir editör seçiminiz olabilir. Eğer yoksa, [Visual Studio Code](https://code.visualstudio.com)'u seçebilirsiniz.

Aşağıdaki komutu kullanarak VS Code'u yükleyebilirsiniz:

```sh
brew install --cask visual-studio-code
```

VS Code'un yüklendiğini aşağıdaki komutu kullanarak doğrulayabilirsiniz:

```sh
code .
```

VS Code otomatik olarak herhangi bir yazılım diline destek vermez fakat siz VS Code uzantıları yükleyerek, istediğiniz herhangi bir programlama dili için VS Code'un destek sağlamasını edinebilirsiniz. Go programlama dili için çeşitli seçenekleriniz var, en iyilerinden biri [Luke Hoban VSCode Go](https://github.com/golang/vscode-go). Bu uzantıyı yüklemek için:

```sh
code --install-extension golang.go
```

VS Code ile ilk defa bir Go dosyası açtığınız zaman, VS Code analiz araçlarının eksik olduğunu söyleyecek. Eksik olan araçları yüklemek için VS Code'un sizi yönlendirdiği butona tıklamanız gerekli. VS Code tarafından kurulan ve kullanılan araçların listesine [buradan](https://github.com/golang/vscode-go/blob/master/docs/tools.md) ulaşabilirsin.

## Go Debugger

Go programlarınızı debug etmek için en iyi seçeneklerden biri Delve'dır. Bunu yüklemek için aşağıdaki komutu kullanabilirsiniz,

```sh
go get -u github.com/go-delve/delve/cmd/dlv
```

VS Code'da Go programlarınızı debug yapılandırması ve çalıştırma konusunda ek yardım için lütfen [buraya](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) bakın.

## Go Linting

Varolan linter'ı [GolangCI-Lint](https://golangci-lint.run) kullanarak yapılandırabilirsiniz, GolangCI-Lint'ı yüklemek için [buraya](https://golangci-lint.run/usage/install/) bakabilirsiniz.

## Refactoring ve araçlarınız

Bu kitabın en fazla değindiği refactoring'in önemidir.

Araçlarınız, güven içinde daha büyük bir parçaya etki eden refactoringler yapmanıza yardımcı olur.

Kullandığınız editöre(VS Code), bir kaç tuş dokunuşu ile aşağıdakileri gerçekleştirecek kadar aşina olmalısınız:

- **Extract/Inline variable**. Being able to take magic values and give them a name lets you simplify your code quickly
- **Extract method/function**. Bir kod bölümünü alarak, başka bir metot veya fonksiyon olarak çıkarabilmek önemlidir.
- **Yeniden isimlendirme**. Yaptığınız isimlendirmeleri, bütün dosyalar için kolayca ve güvenli bir şekilde yeniden yapabiliyor olmalısınız.
- **go fmt**. Go, `go fmt` adında bir biçimlendiriciye, formatlayıcıya sahip. Editörunuz siz dosyayı her kaydettiğinde bunu çalıştırıyor olmalıdır.
- **Testleri çalıştır**. Yukarıdakilerin hepsini yapabiliyor olmalı ve herhangi bir şeyi bozup, bozmadığınızı anlamak için testlerinizin hızla yeniden çalışıyor olması lazım.

Ek olarak, çalışma yaparken şunlara sahip olmalısınız

- **Fonksiyon imzalarını görüntülemek** - Bir fonksiyonu nasıl çağıracağınızdan emin olmadığınız durumlar olabilir. Editörunuzun, bir fonksiyonu çağırırken, aldığı parametreleri ve ne döndürdüğünü açıkca belirtiyor olması lazım.

- **Fonksiyonu görüntüleme** - İmzayı görüntülemenize rağmen kodun tam olarak ne yaptığına emin değilseniz, fonksiyonunun içeriğini görüntülemek için yazıldığı yere kolayca erişebiliyor olmalısınız.

- **Nerede kullanıldığını görmek** - Bir kod öğesinin(fonksiyon, değişken, metot, yapı) tam olarak nerede veya nerelerde kullanıldığını kolayca bulabilmelisiniz, kod üzerinden değişiklik yaparken veya bir kod bloğunu çıkarırken, hangi bölümlerin etkileneceğini görmelisiniz.

Araçlarınıza hakim olmak, sadece koda konsantre olmanıza ve bağlamları değiştirmeyle uğraşacağınız süreyi kısar.

## Özetlersek

Şuanda, Go'nun bilgisayarınızda kurulu olması, Go ile kod yazacak bir düzenleyeciye sahip olmanız ve bazı temel araçların mevcut olması gerekir. Go, üçüncü taraf ürünlerden oluşan çok geniş bir ekosisteme sahiptir. Burada bir kaçına değindik, daha eksiksiz bir liste için <https://awesome-go.com> adresine bakın.

Bu sayfa [@halilkocaoz](https://github.com/halilkocaoz) tarafından çevrildi.
