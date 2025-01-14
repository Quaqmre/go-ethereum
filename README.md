## Giriş

**Mehmet Akif Tatar etherium-go source code incelemesi**

Çalışmakta olan bir blockchain networküne , Transaction objesi içerisine yeni bir prop eklenmesi case'i düşünüldüğünde;

- Hash olarak data miner'lara iletileceği için minerlar bu transactionı bir block içerisine alarak daha sonra blockchain'e ekleyebilir.
  Ancak burada transaction objesinin serilize sorunu ortaya çıkmakta.Node'lar bu alanı göremiyor olacaktır.Bu nedenle yeni bir genesis block oluşturarak yeni bir networkte ve güncellenen nodelar ile bu type ekleme işlemi yapılması gerekmektedir.

## Yeni Network içerisinde

Güncellenen Nodelar ile yeni type alanı eklenebilir.Bunun için Source Code içerisinde,

**_//#!#_** imleci ile belirttiğim alanlarda değişiklikler yapılması gerekmektedir.

- **transaction.go**
- transaction_args.go
- transaction_signing.go
- transaction_marshalling.go
- encode.go
- access_list_tx.go
- dynamic_fee_tx.go
- legacy_tx.go
- typecache.go

#### Transaction.go

Transaction içerisinde _inner_ olarak yer alan TxnData interface'ine yeni alanımız için metodumuzu eklememiz gerekir.

**newField() string**

- TxnData'yı implemente eden tüm objelerin de bu field'ı ekleyerek ilgili metot içerisinde newField datasını dönmesini sağlamalıyız.

**NewTxn()** metodu yeni transaction oluşturmak için kullanılmalıdır.

**NewTransaction**(nonce uint64, to common.Address, amount *big.Int, gasLimit uint64, gasPrice *big.Int, data []byte) \*Transaction

- Deprecated edilmiş **NewTransaction** metodu geriye dönük versiyonlar için bir sorun teşkil etmekte. Yeni eklenen alan bu versiyonlarda çalışmayacaktır.

**Transaction.go** içeirinde en önemli değişikliklerden biri _Message_ structinda olmalıdır.Çünkü node içerisinde yapılan değişiklikler clientlar tarafındanda implemente edilmelidir.

Message struct'ini ve Metodlarını değiştirerek _newField_ alanımızı eklememiz gerekir.

- **NewMessage** , **AsMessage** , ve **NewField** metodları.

##### Transaction.go file'ında yapılan bu değişiklikler bu Transaction ve Message objesini kullanan bir çok metotunda değiştirilmesini beraberinde getirir.

#### transaction_args.go

Transactionların asıl argümanlarını oluşturan kısım,
Clienlardan gelen requestleri karşılayan api modeli.
ve TxnData interfacesine eklediğimiz
_newField_ metodunu implemente etmesi gerekir.

Solidity ile akıllı kontrat oluştururken _newField_ alanını tanıması için düzenleme yapılması gerektiğini düşünüyorum.

#### transaction_signing.go

İmzalama kısmında yeni alanında eklenmesi gerekmektedir, eklenmez ise imza eksik data ile oluşturulur.

**Hash(tx \*Transaction)** metodu düzenlenerek _newField_ alanı eklenmelidir.

#### transaction_marshalling.go

Onaylanan bir objenin kısaca serilize ve deserilize edilmesi için marshalling metodu içerisinde yeni alanımızın bulunması gerekmektedir. yoksa deserilize kısmında hatalara sebep olacaktır.

#### encode.go

encode.go ve typecache.go içerisinde yer alan metotlar ile
_newField_ alanımızın optional olduğunu belirleyebiliriz.
Tam olarak nasıl belirlendiğini bulamadım ancak typecache içerisinde optional tag'i yer almaktadır.

#### access_list_tx.go

Onaylanan transactionların bir reprezantasyon'u bu nedenle bu objeninde transaction_args'ta olduğu gibi tekrar güncellenmesi gerekmektedir.

#### dynamic_fee_tx.go

Aynı şekilde bir transaction reprezantasyon'u olduğu için bununda aynı şekilde güncellenmesi gerekir.

#### legacy_tx.go

Reprezantasyon objenin güncellenmesi gerekir.

### public -private key

Public ve Private key mantığı https'teki mantık ile tamamen bağımsız bir şekilde işlemekte,
buradaki mantık aslında private key ile işaretlenen bir transactionın public key ile hangi adrese ait olduğununun validate edilebilmesini sağlamaktdır.

### EIP-2718

RLP-encoded - EIP-2718 güncellemesiyle yeni type'lar transaction içerisinde geriye dönük onaylanabilmesi sağlanmıştır. Bu güncelleme yeni alanların transaction içerisine eklenebilmesine olanak sağlıyor olabilir.
