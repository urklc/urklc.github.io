---
layout: post
title: Kariyer ikilemi: Native mi yoksa Cross Platform mu?
author: Uğur Kılıç
---

Bugünlerde artık native developer olmanın geçerliliğini yitirdiği, iş bulmanın zorlaşacağı, ABD'nin parasını piyasadan toplamasının sonucu olarak da herkesin her işi yapacağı bir dönemin geleceği üzerine bir beklenti oluştuğunu görüyorum. Özellikle iş arayışlarında zorlanan junior adaylar da arayış süreçleri uzadıkça bu fikre kapılıveriyorlar.

> Bu yazı bir uygulama cross platform mu geliştirilmeli yoksa native mi üzerine degil. Bu epey klişe olurdu, yine de bu klişeyi başka bir yazıya saklayacağım.

## 1. Dünyamız

Öncelikle biraz kendi hikayemden bahsetmek istiyorum.

2009 yılında Symbian (native) ile mobil programlamaya başladım. Yakın zamanda iPhone OS'un yarattığı rüzgar sayesinde kendimi orada buluverdim. O dönemlerde aşağıdaki tweet'te bahsettiğim birçok cross-platform uygulama geliştirme ortamları sürekli gündemdeydi. İronik olan şu ki; kariyerimin ilk döneminde cross-platform uygulama geliştirme framework'ü (C++, Objective-C ile) yazıyordum. Bu işin ne kadar heyecan verici olduğunu yaşayarak görüyordum.

<blockquote class="twitter-tweet"><p lang="tr" dir="ltr">Phonegap (Cordova), Ionic, Kony, Appcelerator Titanium, Sencha, Xamarin, Qt, ReactNative, son zamanlarda Flutter, bir de çok heyecanlanmışsak Kotlin Multiplatform...<br><br>Umarım ne demek istediğimi anlamışsınızdır. 😊🙂😶‍🌫️</p>&mdash; Uğur Kılıç (@urklc_) <a href="https://twitter.com/urklc_/status/1663938203085053956?ref_src=twsrc%5Etfw">May 31, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
<br />

Birden çok platforma aynı anda uygulama geliştirme fikri ve iOS'in ne zamana kadar bu şekilde popüler olacağı şüphesi içimdeki cross-platform meşalesini sürekli yanık tuttu. Buna dünya devi Nokia'nın batmasının getirdiği korku da eklenince sürekli bir panik halinde kariyerimi sürdürüyordum. Bir keresinde bu panikle iOS'i tamamen bırakıp, Windows Mobile'a geçme çabalarım oldu. Neyse ki Nokia Lumia modelleri o kadar yaygınlaşmadı da beni böylesi büyük bir hatadan kurtardı. Bunun dışında, bazı dönemlerde Android ve Backend ile de ilgilendim. Amacım acil bir durumda geçiş yapabileceğim bir alanın hazır bulunmasıydı. Pandemi döneminde ise daha çok teknik altyapısını incelemek için 1 yıl kadar Flutter ile uygulama geliştirdim (ve çok eğlendim). Şimdi de React Native bir uygulamanın içine gömülü olan Swift framework'ü üzerinde çalışıyorum.

Okuduğunuz üzere cross platform'a karşı herhangi bir ön yargım, nefretim yok. Aksine kendimi cross platform'un bir neferi olarak görüyorum. Kariyerime cross platform framework geliştirerek -çekirdekten- başlamak ve daha sonra bu platformları dönem dönem kullanma fırsatı bulmak benim için heyecan verici bir yolculuktu. Bu hikayeyi anlatmaktaki asıl amacım süreci göstererek sizi bir fikre hazırlamaktı. Eğer bu işi yapıyorsanız kafanızın bir köşesinde bu değişim belirsizliği, bu panik her zaman olacak. Sanıyor musunuz ki Backend geliştirmeye başlarsanız emekli olana kadar Spring yazacaksınız, ya da cloud teknolojileri 20 yıl boyunca bugünkü kavramlar üzerinden konuşulacak? Çalıştığımız alan exponential bir şekilde gelişiyor, buna zihnimizin adapte olması çok zor. Onun için -biraz spiritüel bir mesaj olacak ama- zihninizi rahat bırakın ve gerçekçi olup kendinize yeni yetenekler katmaya çalışın. Bu illa ki oturup başka teknolojileri birebir öğrenmek demek değil, daha çok orada yapılanlarla ilgili kabaca bir fikre ve gidebileceği yer hakkında ön görüye sahip olmak. Popüler bir örnek vermek istiyorum: AI ile ilgili hiçbir şey yapamıyorsanız, gidip birkaç tutorial takip ederek kendi profil resminizi oluşturun. Bunu 10 yıl önce yazıyor olsaydım şunu derdim: Bitcoin alın. Çünkü o zamanlar bu bir challenge'dı. Siz bunu gerçekleştirirken mutlaka blockchain'in nasıl çalıştığı üzerine bir şeyler okuyacaktınız. Bu deneyimlerinizin bazılarının getirisi çok yüksek olacak, bazıları ise size zaman kaybıymış gibi gelecek. Ama gözden kaçırmamanız gereken tek gerçek, bunların sizi olası bir değişime hazır tutacağı.

> Degişim istemeyen gitsin çay ocağı işletsin.
> Uğur Kılıç

## 2. Teknik Gerçekler

Konuya teknik bir perspektiften ve bunun sektöre etkisinden bakmadan geçersek hata yaparız.

* Kim ne derse desin cross platform hiçbir zaman performans açısından native ile karşılaştırılamaz. Facebook'un yıllardır geliştirdiği React Native'in bu süreçte çok yıprandığını ve o yüzden bunu başaramadığını söylesek; Google'in Flutter ile bunu neden başaramadığı sorusu karşımıza çıkar. Üstelik Skia gibi bir UI framework kullanmalarına rağmen. 

* Performansın dışında, iki ayrı dünyanın çatıştığı bazı noktalar vardır ki, sadece çıktıyı değil; süreci de geliştiriciler için çok farklılaştırıyor.

    * Bunlardan bir tanesi projeye eklenen üçüncü taraf kütüphanelerdir. Web geliştiricilerinin alışkanlığı (zorunluluğu) bir ihtiyaçları olduğunda bunu üçüncü taraf kütüphane ekleyerek gidermek. Bu da maalesef beraberinde birçok riski getiriyor. Mobilde ise durum tam tersi. Birçok fonksiyonalite zaten işletim sistemi ile birlikte sunuluyor.

    * İkincisi projenin build aşamasında gereken bağımlılıklar, paket yöneticileri, node versiyonları...

Başta söylediğim gibi cross platform vs native yazısı olmadığından burada kesiyorum; bu kadarını yazmamın nedeni iki ortam hakkında kafamızda bir fikir oluşmasını sağlamaktır. Bu iki ortamdan hangisini çalışanların öncelikli olarak tercih edeceği bariz. İş verenlerin de kaynak bulma konusunda yaşadıkları sorunları düşünürsek, burada karar verici olan yine çalışan tercihi olacaktır. Diğer taraftan iş verenler de kaynak kısma amacıyla yaptıkları denemelerin sonuçlarını değerlendirerek bu iki seçenekten hangisinin uzun vadede daha verimli olduğuna kendileri karar verecekler. Bazı şirketler için cross platform doğru bir seçenek olabilecekken, birçoğu (özellikle teknoloji odaklı olanlar) native odaklı ilerleyecektir. Birçoğunun native tercih edeceğini tahmin etmemin sebeplerini özetlersek: Çalışan mutluluğu, performans, üründe yeniliklere hızlı adaptasyon ve sonuç: müşteri memnuniyeti. O yüzden native hem çalışan hem de işveren açısından daha çekici bir tercih olarak yaşamaya devam edecek diye düşünüyorum.

> Piyasada para kalıp kalmadığı konusunda fikir edinmek isteyenler bu yazıyı yazdığım tarihteki -teknoloji ağırlıklı- NASDAQ hisselerini inceleyebilir. 😊