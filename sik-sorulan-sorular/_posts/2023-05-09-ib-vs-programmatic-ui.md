---
layout: post
title: Interface Builder mı yoksa Programmatic UI mı?
author: Uğur Kılıç
---

Bu konuyu iki ana başlıkta değerlendirmek istiyorum:

## Geliştirme Aşaması

* Görsel arayüzü çok daha hızlı bir şekilde oluşturabilirsiniz. Son çıktıyı görebildiğiniz için hata yapma olasılığı düşüktür.
* Programatik yöntemde ise arayüzün son halini göremediğiniz için kafanızda canlandırmanız gerekir ve orta seviye karmaşık ekranlarda biraz deneme yanılma yapmanız gerekebilir.

## Code Review ve Bakım Aşaması

* Storyboard üzerinde aynı anda çalışan birden fazla kişi varsa, code review sırasında çok fazla çakışma yaşanabilir.
* Interface builder kodunu okumak zor olduğu için insanlar layout review'i yapamayabilirler. *(Ama programatik layout kodunu da çoğu insan review etmez. :))*
* Programatik olarak yazılan layout kodunu okuyup anlamak, nasıl bir ekran oluşacağını hayal etmek ve gerektiğinde kodu değiştirmek daha fazla zaman alır.

## Sonuç

Önemli olan her amaç için tek bir **"source of truth"** belirlemek, yani layout'u IB'den oluşturup; property'leri koddan değiştirerek ilerleyebiliriz. Bazı property'leri IB'den, bazılarını koddan değiştirme yoluna gitmemeliyiz. Her şeyi IB'den yapamazsınız; animasyon vb. işlemler için zaten programatik kullanmak zorundasınız, o yüzden bazı layout kodu her türlü koda sızacak.

Storyboard kullanıyorsanız içereceği view controller sayısına dikkat etmeniz gerekir. Storyboard referansları kullanarak akışları farklı storyboard'lara bölmelisiniz. Farklı ekiplerle çalışıyorsanız akışları bu ekiplere paylaştırmanız önemlidir.

Performans konusunda da programatik yazmak daha faydalı olabilir, ancak göz ardı edilebilir.

Sonuç olarak, *proje büyüklüğü, akışların birbirinden ayrılabilirliliği, çalışacak kişi sayısı, projenin bütçesi ve en önemlisi zaman içinde hangi yöntemde daha rahat hissettiğiniz* en önemli kriterler olacak.
