# İndeksleme

İndeksleme, veritabanında bir alanın sıralı şekilde tutulmasıdır. Bu işlem için
indekslenen sutunun bulunduğu bir tablo oluşturulur ve bu alandaki veriler sıralanır.
İndekslemenin amacı, veriye hızlı ulamaktır.

Veritabanında verilerin aranması için 4 farklı yöntem vardır. Bunlar şu şekildedir;

* Table Scan
* Clustered Index Scan
* Index Seek
* Key Lookup

### Table Scan

Tabloda herhangi bir indeks bulunmadığında uygulanan yöntemdir. Tabloda primary key dahi bulunmaz.
Normalde tablolardaki veriler sayfalar/parçalar halinde satır satır tutulur.
Bu sayfalar karışık halde bulunur. Aranan değeri bulmak için tüm veriler gezilir.
Table scan veri arama yöntemlerinden en kötüsü ve maliyetli olanıdır.


<img src="https://cs186berkeley.net/notes/assets/images/02-DisksFiles/PageVisual.png"  width="600"/>

### Clustered Index

Primary key bir **clustered index**'tir. Bir tabloda en fazla bir tane clustered index olabilir.
Clustered index verinin diske nasıl yazılacağını belirtir. Genelde artan sırayla kullanılır(yazdırılır).
Clustered index bulunan tablolarda **Clustered Index Scan** yöntemi kullanılır. Örneğin id ile arama
yapıldığında ikiye bölme algoritması ile sıralı id üzerinden veriyi bulmak çok daha hızlı olacaktır.


<img src="https://miro.medium.com/v2/resize:fit:858/1*ZMC3u0m5r830C2VDbaTa7g.jpeg"  width="600"/>

### Non-Clustered Index

Primary Key dışında oluşturulan indekstir. Bir müşteri tablomuz olduğunu düşünelim.
Bu müşteri tablomuzda sadece primary key olduğunu düşünelim. Burada id üzerinde arama yaptığımızda sıkıntı yok.
Ancak Müşterinin adını kullanarak bir sorgu yazıp aramayaptığımızda, tekrardan eşleşen verileri
bulmak için tüm tabloyu gezecektir. Haliyle daha fazla güç ve zaman harcanacaktır. Sorgudan cevap dönme süremiz de
verinin büyüklüğüne göre oldukça uzayabilir. Yani veri okuma sürecimiz uzayacaktır.
Bunun için "name" alanı için index oluşturmalıyız. 

![name_index](https://miro.medium.com/v2/resize:fit:931/1*sz3PldJHpc_cCTbk1YR7LA.png)

Görselde görüldüğü gibi artık name alanımızın alfabetik olarak sıralı halinin ve id bilgisinin bulunduğuı
yeni bir tablo oluşturulur. Artık name kullanılarak bir veri arama yapıldığında
name indeks tablosundan hozlıca aranan isim bulunur ve Id bilgisi alınır.
Bize burada sadece Name ve Id biligileri verilir çünük index tablosunda sadece bu alanlar vardır.
Bu arama türüne **Index Seek (Non-Clustered)** denir.

Ardından ulaşılan id verisi ile ana tabloya gilidilerek ilgili kişiye ait tüm bilgiler getirilir.
Bu arama türüne **Key Lookup** denir. Eğer bizim indeksimiz, **included column**
içermiyorsa önce **Index Seek** sonrasında **Key Lookup** yaparak veriyi bize getirir.
Eğer sadece Index Seek kullanılmasını istersek, indeks oluşturuken ana tabloda diğer tüm sütünları
burada "**included column**" olarak vermemiz gerekir.


>[!WARNING] 
> 
>Indeksleme yaparken mevcut duruma ve ihtiyaca göre hareket edilmelidir.
> Indekse tüm sütunları, included column olarak eklersek okuma performansından kazanç
> sağlarız ancak depolama alanından feragat ederiz. Çünkü indeks tablomuzun boyutu büyür.
> 
>Ayrıca çok fazla alanı indeks olarak tutmak okumada işleminde hız kazanmayı sağlasada,
> yazma işlemi için daha çok performans harcamaya neden olur ve yavaşlık sağlar.

> [!NOTE]
> 
> Id alanı olarak UUID kullanırken dikkatli olunmalıdır. Çok fazla verinin olduğu
> durumlarda performans kaybına neden olabilir. UUID'YE Aşırı derece ihtiyaç duyulmuyor ise 
> kullanmaktan kaçınmalı, artan ve intager değer kullanılmalıdır.


### Indeks Bozulması (Erasmentation)

Yeni gelen veriler indeks tablosunda sıralı olarak tutulamaz.
Yeni veriler sona eklenir. Böyle durumlarda düzenli aralıklarla bakım yapılarak indeksleme güncellenmelidir.

Index bozulma sürecinin süresini uzaltmak için **Fill Factor** değeri tanımlanabilir.
Fill Factor değeri sayesinde indeks sayfaları belirli bir oranda doldurulması sağlanır.
Örneğin indeks tablomuzu diskte tutan page'ler 10 satırlı verilerden oluştuğunu düşünelim.
Fill Factor değerimizi de %80 olarak verelim. Yani bu page'ler %80 oranında doldurulacak, %20'lik 
kısım ise yeni gelen veriler için boş bırakılacak. Bu sayede indeksleme yapıldıktan sonra gelen veriler sıraya göre aralara 
koyularak diğer veriler kaydırılabilecek. Bu sayede indeks yapısı(sıralama) bozulmayacak.
Tabiki eninde sonunda yeni eklenen veriler %20 lik kısımı dolduracaktır. İşte o zaman tekrar
indeks bozulmaları baş gösterecektir. Fill Factor ile index bozulmasını tamamen engelleyemiyoruz.
Sadece geciktiriyoruz.

> [!NOTE]
>
> Fill Factor değerini düşük tutarsak diskte tutlan page sayısı artar ve 
> hafızadan daha fazla harcamış oluruz ancak index bozulması gecikeceğinden dolayı hız kazanırız.
>


<img src="https://media.geeksforgeeks.org/wp-content/uploads/20230416145322/FIll-Factor.png"  height="400" width="600"/>


