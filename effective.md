[Effective-go]("https://golang.org/doc/effective_go.html") sayfasının Türkçe özetidir.

## Formatlama

- Kod stili en çok tartışılan ama çok kritik olmayan sorunlardan biridir
- Go formatlama işini `gofmt` programıyla kendisi hallediyor
	
		type T struct {
    		name string // name of the object
    		value int // its value
		}

- `Gofmt`çalıştırıldıktan sonra:

		type T struct {
    		name    string // name of the object
    		value   int    // its value
		}

- Bütün go standart kütüphaneleri `gofmt` ile formatlanmıştır
- Go'da girintileme için tab kullanılıyor
- Herhangi bir satır uzunluğu limiti bulunmuyor
- C ve Java'ya göre söz diziminde daha az parantez kullanıyor


## Yorum

- C stili blok(/\*\*/) ve satır(//) şeklinde açıklama yazılabiliyor
- Blok açıklama satırları `paket` tanımlamalarında kullanılıyor
- `paket` içeriği ile ilgili dokümantasyon oluşturulurken blok açıklamalarından yararlanıyor


## İsimlendirme

- Değişken isimleri konvansiyon olarak `mixedCaps` ya da `MixedCaps` şeklinde olmalı
- Go'da metodların private ya da public olmasını isimleri belirliyor
- Büyük harfle başlayan metodlar public, küçük harfle başlayanlar ise private 

		 Topla(a int, b int)   // public
		 topla(a int, b int)   // private
### Paket isimleri

### Getter/Setter

-  Go'da getter ve setter yapısı yok 
- `owner` adında bir değişken için `getOwner` değil `Owner` adında bir metod yazmak yeterli

### Interface isimleri

- Tek metodluk interface'ler metod_ismi + `er` şeklinde isimlendirilebilir (Reader, Writer)

## Noktalı virgül

- C'deki gibi Go'da da satırlar noktalı virgül(;) ile sonlanıyor
- Tek fark Go'da bu işlem derleme öncesinde kaynak kod taranırken otomatik yapılıyor
- Bu özelliğin yan etkisi olarak kıvrık parantezler 1TBS olmak zorunda
- 1 True Brace Style:
		
		if i < f() {
    		g()
		}
## Kontrol yapıları

- `if` basit hali:

		if x > 0 {
    		return y
		}

- `if` içinde değişken ilklenebilir:

		if err := file.Chmod(0664); err != nil {
   			 log.Print(err)
    		 return err
		}

- Koşul ifadelerindeki gereksiz `else` ifadeleri kullanmak zorunda değilsiniz

		f, err := os.Open(name)
		if err != nil {
    		return err
		}
		codeUsing(f)

- Aşağıdaki örnek kod akışında koşullar sağlanmadığı durumda `return`'lar devreye girip akışı bitirecek.

		f, err := os.Open(name)
		if err != nil {
    		return err
		}
		d, err := f.Stat()
		if err != nil {
    		f.Close()
    		return err
		}
		codeUsing(f, d)


## Değişken tanımlama ve Atama 


## For

- Go'daki `for` C'deki `for`ve `while`'ı kapsıyor
- Go'da `do-while` yok 
- `for`'un üç tipi var

		// C'deki for tarzı
		for init; condition; post { }

		// C'deki while
		for condition { }

		// C'deki for(;;)
		for { }

- Kısa atama özelliği index değişkenini ilklendirme işini kolaylaştırıyor

		sum := 0
		for i := 0; i < 10; i++ {
    		sum += i
		}

- Array, map, slice, string üzerinde iterasyon yapmanız gerekiyorsa `range` fonksiyonu oldukça kullanışlı

		for key, value := range oldMap {
   			 newMap[key] = value
		}

- Sadece indeks ya da anahtar bilgisi gerekiyorsa

		for key := range m {
   			if key.expired() {
        	delete(m, key)
    	 	}
		}

- Sadece mapteki değerler lazımsa `_` kullanarak istenmeyen değeri boşa çıkarabilirsiniz

		sum := 0
		for _, value := range array {
    		sum += value
		}

- `range` string'lerin içinde de dolaşmaya olanak sağlıyor

		for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
   			 fmt.Printf("character %#U starts at byte position %d\n", char, pos)
		}

çıktısı
		
		character U+65E5 '日' starts at byte position 0
		character U+672C '本' starts at byte position 3
		character U+FFFD '�' starts at byte position 6
		character U+8A9E '語' starts at byte position 7

- Go'da `++` ve `--` operatörleri yok
- Paralel atamaya olanak sağlıyor

		// 'ters' --> 'srst'
		for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    		a[i], a[j] = a[j], a[i]
		}

## Switch 

		func unhex(c byte) byte {
    		switch {
    		case '0' <= c && c <= '9':
        		return c - '0'
    		case 'a' <= c && c <= 'f':
        		return c - 'a' + 10
    		case 'A' <= c && c <= 'F':
        		return c - 'A' + 10
    		}
    		return 0
		}

- Koşul durumları liste olarak verilebilir

		func shouldEscape(c byte) bool {
    		switch c {
    		case ' ', '?', '&', '=', '#', '+', '%':
        		return true
    		}
    		return false
		}

- Go'da `break`, `switch`'i sonlardırmak için kullanılabilir
- Döngü ya da `switch`'in hangisinin sonlanacağına etiket kullanılarak karar verilebilir
		
		Loop:
			for n := 0; n < len(src); n += size {
				switch {
				case src[n] < sizeOne:
					if validateOnly {
						break
					}
					size = 1
					update(src[n])

				case src[n] < sizeTwo:
					if n+1 >= len(src) {
						err = errShortInput
						break Loop
					}
					if validateOnly {
					break
					}
					size = 2
					update(src[n] + src[n+1]<<shift)
			}
		}

- `continue`da etiket kabul ediyor fakat sadece döngüler için kullanılabiliyor
- `switch` ile iki byte `slice` karşılaştıran fanksiyon

		// Compare returns an integer comparing the two byte slices,
		// lexicographically.
		// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
		func Compare(a, b []byte) int {
    		for i := 0; i < len(a) && i < len(b); i++ {
        		switch {
        		case a[i] > b[i]:
            		return 1
        		case a[i] < b[i]:
            		return -1
        		}
    		}
    		switch {
    		case len(a) > len(b):
        		return 1
    		case len(a) < len(b):
        	return -1
    		}
    		return 0
		}

## Tip switch'i

- `switch` kullanarak bir değişkenin tipini öğrenebilirsiniz

		var t interface{}
		t = functionOfSomeType()
		switch t := t.(type) {
			default:
    		fmt.Printf("unexpected type %T", t)       // %T prints whatever type t has
		case bool:
    		fmt.Printf("boolean %t\n", t)             // t has type bool
		case int:
    		fmt.Printf("integer %d\n", t)             // t has type int
		case *bool:
    		fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
		case *int:
    		fmt.Printf("pointer to integer %d\n", *t) // t has type *int
		}

## Fonksiyonlar

- Fonksiyonlar birden fazla değer dönebilir
- Örneğin, `Write` metodu hem satır sayısı hemde hata mesajı dönebilir

		func (file *File) Write(b []byte) (n int, err error)

- Dönüş değerleride aynı fonksiyon paremetleri gibi isimlendirilebilir
- Tanımlanan değişkenler tiplerine göre sıfırla ilklendirilir

		func ReadFull(r Reader, buf []byte) (n int, err error) {
    		for len(buf) > 0 && err == nil {
        		var nr int
       			nr, err = r.Read(buf)
        		n += nr
        		buf = buf[nr:]
    		}
    		return
		}

## Defer

- `defer` verilen fonksiyonun, `defer`'i çağıran fonksiyondan `return` edilince çağrılmasını sağlar

		// Dosya içeriğini string olarak döner.
		func Contents(filename string) (string, error) {
			f, err := os.Open(filename)
			if err != nil {
				return "", err
			}
			defer f.Close()  // işimiz biter bitmez f.Close çalışacak.
			var result []byte
			buf := make([]byte, 100)
			for {
				n, err := f.Read(buf[0:])
				result = append(result, buf[0:n]...) // append daha sonra inelecek.
				if err != nil {
					if err == io.EOF {
						break
					}
					return "", err  // burdan çıkarsa f kapatılacak.
				}
			}
			return string(result), nil // burdan çıkarsada f kapatılacak.
		}

- Yukarıdaki örnekte `defer`kullanmanın iki avantajı var:
	- dosyayı kapatmayı unutma gibi bir durum kalmıyor
	- açma ve kapama işlemleri birbirine yakın olduğundan okunması rahat hale geliyor

- `defer` çağrıları LIFO şeklinde çalışır

		for i := 0; i < 5; i++ {
			defer fmt.Printf("%d ", i) // 4 3 2 1 0 sırasıyla yazacak
		}



## `new` ile bellek ayırma

- Bellek ayırma iki fonksiyon yardımı ile yapılabilir. `new` ve `make`
- `new` ile ayırılan alan verilen tipin sıfırı ile ilklendirilir
- `new` ayrılan alanın adresini döner

	p := new(SyncedBuffer)  //  *SyncedBuffer
	var v SyncedBuffer      //   SyncedBuffer

- Ayrılan alanın ilklendirilmesi gereken durumlarda olabilir

		func NewFile(fd int, name string) *File {
		    if fd < 0 {
			return nil
		    }
		    f := new(File)
		    f.fd = fd
		    f.name = name
		    f.dirinfo = nil
		    f.nepipe = 0
		    return f
		}

- En basit şekilde(`composite literal`) kullanarakta aynı işlemi gerçekleştirebiliriz

		func NewFile(fd int, name string) *File {
		    if fd < 0 {
			return nil
		    }
		    f := File{fd, name, nil, 0}
		    return &f
		}

- C'nin tersine Go'da yerel değişkenin adresini dönebilirsiniz
- `composite literal` her seferinde bellekten yeni bir alan ayırır.
- Değişken fonksiyondan çıkıldıktan sonrada hafızada kalacaktır

		return &File{fd, name, nil, 0}

- Yukarıdaki şekilde fonksiyonu daha da sade bir hale getirebiliriz
- Paremeteler verilirken sıralı şekilde verilmelidir
- Ya da `değişken_ismi: değer` şeklinde istediğiniz sırada verebilirsiniz

		return &File{fd: fd, name: name}

- Aynı şekilde array, slice, map gibi veri türlerini de oluşturabilirsiniz

		a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
		s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
		m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}

## `make` ile bellek ayırma

- `make(T, args)`, `new(T)`'den farklı olarak ayırdığı alanı ilkler ve T tipinde bir değer döner(\*T değil)
- Örneğin:
	
		make([]int, 10, 100)

- Bellekte 100 int'lik bir alan ayırır, sonra 100 intlik dizinin ilk 10 elemanını işaret eden, boyutu 10 olan ve 100 kapasiteli bir `slice` oluşturur
- Aşağıdaki örnekte `new` ile `make`'in arasındaki farkı daha iyi anlayabiliriz:

		var p *[]int = new([]int)       // slice veriyapısı oluşturuyor; *p == nil; nadiren kullanışlı
		var v  []int = make([]int, 100) // 100 karakterlik bir int dizisine işaret eden bir v slice'ı oluşturur

		// Gereksiz karmaşık bellek ayırma:
		var p *[]int = new([]int)
		*p = make([]int, 100, 100)

		// Idiomatic:
		v := make([]int, 100)

- `make` sadece map, slice, channel için kullanılabilir. Ve pointer dönmez.

## Array

- Array'ler ne kadar hafızaya ihtiyaç duyulduğu bilinen durumlarda kullanışlı olabilirler
- Genelde bir sonraki bölümde anlatılacak slice veri yapısının alt yapısını oluştururlar
- C'deki array'lerden farklıdırlar
	* Array'ler birer değer olarak tutulur. Bir array'i diğerine atamak, array'in kopyalanmasına neden olur
	* Aynı şekilde bir fonksiyona array verirseniz, fonksiyon array'in bir kopyasını alacaktır(pointer değil)
	* Array'lerin boyutu tiplerinin bir parçasıdır. Yani [10]int ile [20]int farklı türlerdir

- C'deki gibi kullanmak isterseniz

		func Sum(a *[3]float64) (sum float64) {
		    for _, v := range *a {
			sum += v
		    }
		    return
		}

		array := [...]float64{7.0, 8.5, 9.1}
		x := Sum(&array)  // adres öperatörüne dikkat edin

- Yukarıdaki kullanımda idiomatic bir yaklaşım değil. Go'da yaygın olarak slice'lar kullanılır

## Slice

- Slice'lar dinamik array'ler gibi düşünülebilir
- Her slice arka tarafta bir array'e işaret eder
- Bir slice'ı diğer bir slice'a atarsanız ikiside aynı array'a işaret edeceğinden kopyalanma olmaz
- Aynı şekilde fonksiyona parametre olarak verilidiğinde adres olarak geçer
- Arkadaki array'in kapasitesi kadar dinamik olarak boyutları ihtiyaca göre artabilir
- Aşağıdaki kod parçasında `append` fonksiyonun nasıl çalıştığını  görebilirsiniz

		func Append(slice, data[]byte) []byte {
		    l := len(slice)
		    if l + len(data) > cap(slice) {  // veri kapasiteden daha büyükse belleği arttır
			// Gerekenin iki katı kadar alan ayır, gelcekte oluşabilecek ihtiyaçlar için
			newSlice := make([]byte, (l+len(data))*2)
			// copy, built-in bir fonskiyondur ve her slice türü için kullanılabilir.
			copy(newSlice, slice)
			slice = newSlice
		    }
		    slice = slice[0:l+len(data)]
		    for i, c := range data {
			slice[l+i] = c
		    }
		    return slice
		}

- Slice'ler adress bazlı çalıştıkları için yeni oluşan slice'in fonksiyondan dönülmesi gerekiyor

## İki boyutlu slice'lar

- Go'daki array ve slice veri türleri tek boyutludur
- Çoklu boyutta slice veya array tanımlamak için:
		
		type Transform [3][3]float64  // 3x3 array.
		type LinesOfText [][]byte     // slice		

- Slice'ların boyutları tuttukları verilere göre değiştiği için iç içe olan slice'ların boyutları farklı olabilir

		text := LinesOfText{
			[]byte("Now is the time"),
			[]byte("for all good gophers"),
			[]byte("to bring some fun to the party."),
		}

- İki boyutlu slice'lar için bellek ayırma işlemini iki şekilde yapabilirsiniz
- İlk olarak en dıştaki slice için ayırıp, sonrasında her satır için tek tek ayırırsınız

		// En dıştaki slice için bellek ayır.
		picture := make([][]uint8, YSize) // her y birim için bir satır.
		// Her satır için dolaş ve ayırma işlemini yap.
		for i := range picture {
			picture[i] = make([]uint8, XSize)
		}

- İkinci yol 

		// En dıştaki slice için bellek ayır.
		picture := make([][]uint8, YSize) // her y birim için bir satır.
		// Bütün pixelleri tutması için büyük bir slice oluştur.
		pixels := make([]uint8, XSize*YSize) // []uint8 tipinde resim [][]uint8 tipinde olduğu halde.
		// Döngü ile dolaşıp pixels slice içindeki her satır için ayırma işlemini yap.
		for i := range picture {
			picture[i], pixels = pixels[:XSize], pixels[XSize:]
		}

## Maps

- Anahtar, değer ikilisini tutan veri yapısıdır
- Slice hariç eşitlik özelliğini destekleyen bütün veri tipleri anahtar olarak kullanılabilir
- Map'lerde Slice'lar gibi arka planda bir veri yapısını gösterirler
- Bundan dolayı bir fonksiyona map geçirilirse yapılan  değişiklik kalıcı olur
- Tanımlama:

		var timeZone = map[string]int{
		    "UTC":  0*60*60,
		    "EST": -5*60*60,
		    "CST": -6*60*60,
		    "MST": -7*60*60,
		    "PST": -8*60*60,
		}
şeklinde yapılabilir
- Değer atasması yapma ya da anahtara gelen değeri çekme işkemi slice ve array'lerde olduğu gibi yapılabilir

		offset := timeZone["EST"]

- Eğer map'te olmayan bir anahtar için değer çekilmeye çalışırsanız, veri türüne göre sıfır tipi dönecektir(Integer ise 0, string ise "" gibi)
- Set veri türü değer kümesi `bool` tipli olan bir map ile gerçeklenebilir


		attended := map[string]bool{
		    "Ann": true,
		    "Joe": true,
		    ...
		}

		if attended[person] { // eğer map'te person diye biri yoksa false dönecektir
		    fmt.Println(person, "toplantıdaydı")
		}

- Bazen anahtara karşılık gelen değer boş string("") ya da sıfır olabilir
- Bu durumda anahtar olmadığı için bir sıfır tipi dönüyor yoksa gerçekten tutulan değer mi sıfır diye bir ayrım yapmanız gerekcektir

		var seconds int
		var ok bool
		seconds, ok = timeZone[tz]

- Bu yapıya "virgül, ok" deniyor
- Eğer `tz` diye bir anahtar var ise ok true olacaktır, aksi takdirde false
- Aşağıdaki örnekte fonksiyon içinde nasıl kullanıldığını görebilirsiniz

		func offset(tz string) int {
		   if seconds, ok := timeZone[tz]; ok {
			return seconds
		    }
		    log.Println("Bilinmeyen zaman dilimi:", tz)
		    return 0
		}
- Değeri önemsemeksizin test amaçlı sorgulamak için `_` öperatörünü kullanabilirsiniz
		
		_, present := timeZone[tz]

- Map'ten bir değer silmek içinde yerel `delete` fonksiyonunu kullanabilirsiniz

		delete(timeZone, "PDT")



