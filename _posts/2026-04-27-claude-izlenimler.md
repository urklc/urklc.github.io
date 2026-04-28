---
layout: post
title: "Claude + Design izlenimlerim"
categories: [Türkçe]
author: Uğur Kılıç
---

Son dönemde *Claude* ile yaşadığım uygulama geliştirme sürecini ve aldığım notları paylaşmak istedim. Önceki [Cursor yazımın](../cursor-izlenimler) üzerine, bu sefer çıtayı biraz daha yukarı taşıyıp *Claude* ile sıfırdan bir ürün ayağa kaldırdım.

Sürecin özeti şu: **SwiftUI** ile bir **iOS** uygulaması, **FastAPI (Python)** ile bir backend ve *Hetzner* üzerinde sıfırdan bir VPS kurulumu. Arka planda çeşitli yapay zeka modellerine çağrılar yapan kompleks bir yapı. İşin en garip kısmı; ne **SwiftUI** ne de backend tarafında tek bir satır koda dönüp bakmadım, IDE açmadım. Sadece *App Store Connect* tarafındaki süreçlerle ilgilendim.

# Süreç Nasıl İşledi?
Uygulamanın fikrinden tutun da mimari kararlarına kadar tamamen *Claude*’un yönlendirmeleriyle ilerledim. Özellikle benim kolay kolay kurgulayamayacağım, ciddi matematiksel ve teknik konseptleri çok hızlı bir şekilde çözdü ve doğru bir yapı kurmamı sağladı.

*Kullanım alışkanlığım ise şöyleydi:*

* *Terminal Odaklı:* Xcode'u sadece proje kurulumu için kullandım. Geri kalan tüm geliştirme sürecini terminal üzerinden [Superpowers](https://github.com/obra/superpowers) skill'i ile yürüttüm.

* *TDD*: Kodun içine girip bakmasam da, arka planda testlerin yazılmasını ve her adımda o testlerin tek tek yeşile dönmesini izlemek müthiş bir güven veriyor. Yazdığı kodun çalıştığından emin olarak ilerlemek süreci çok hızlandırdı.

Bu sürecin aksayan yanlarından birisi yapay zeka agentlarının o meşhur aşırı özgüveniydi. Projeyi geliştirirken veya bittikten sonra use-case’leri tek tek verip mutlaka doğrulatmak gerekiyor. Bazen öyle basit senaryoları atlayabiliyor ki, siz sormadan bunları kendi kendine düzeltmiyor. Burada çözüm olarak ya çeşitli 'review skill'leri devreye sokulabilir ya da benim şu aşamada tercih ettiğim gibi manuel sorgulamalarla ilerlenebilir.

Diğer kritik nokta da backend ve app security konularında Claude'i beslemek gerekliliği. Bu konularda hiç bilgi sahibi olmasak bile, endişelerimizi gündeme getirip agentı çözüm bulması için biraz dürtmek gerekiyor. Daha önce bahsettiğim şu grafiği ekliyorum: https://x.com/urklc_/status/2039460672077996308?s=20

# Claude Design ve "Polish" Aşaması

Tasarım konusu benim için her zaman biraz sancılı olmuştur. *Claude Design* burada devreye girince işin rengi değişti:

* Bana üçer tane uygulama ikonu önerdi. Seçim yapmakta zorlandım çünkü hepsi çok profesyonel duruyordu.
* Ardından üç farklı tema (design system) sundu ve bunları birkaç sayfamda gösterdi. İçlerinden birini seçtim. Hepsi son derece detaylı ve profesyoneldi.
* Daha güzeli, seçtiğim tasarımı projedeki tüm mevcut SwiftUI view’larına kendisi otomatik olarak uyguladı.
* Nasıl ki kod yazarken plan yaparak gitmek kaliteyi çok etkiliyorsa, burada da ne istediğini bilip Claude Design'ı doğru yönlendirmek kritik. Zaten süreçte yol gösteriyor.

# Kullanım Limitleri

*Claude 5x Max* planı kullanıyorum ve 3-4 saatlik yoğun session'larda bile saatlik limitlere takılmadım. Tüm kurulum, brainstorming ve bug-fixing süreci haftalık limitimin sadece %20'sini tüketti.

Tasarım konusunda da; 7-8 sayfalık bir uygulamayı tüm detaylarıyla (ikonlar, temalar, view güncellemeleri) geliştirmeye yetti. Sadece dark mode için haftalık limitlerim yetmedi.

Ancak önemli bir uyarım var: Uygulama geliştikçe know-how yönetimi kritikleşiyor. Eğer projenin bilgisini bir özet halinde tutup yeni session'lara doğru şekilde beslemezseniz; uygulamayı %20 limit harcayarak oluştururken, basit bir bug'ı çözmek için %30 limit harcamaya başlarsınız. Bu "context yönetimi" ve oyunun kurallarını değiştiren tecrübelerimi daha sonra paylaşacağım.

# Sonuç

"AI kod yazamaz", "Büyük projede tıkanır", "UX ve kullanıcıdan anlamaz" diyenlerin argümanları her geçen gün geçerliliğini yitiriyor. AI sadece kod yazmıyor, projenin fikir aşamasından, tasarımına ve mimarisine kadar her an inisiyatif alıyor ve projeyi yönlendiriyor.

Şu an tek isteğim Claude Design limitlerimin bir an önce açılması. O limitin hemen açılması için Anthropic ne isterse vermeye hazırım. :)
