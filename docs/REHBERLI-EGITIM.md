> Bu belge NVlabs/motionbricks deposundaki `docs/guided_tutorial.md` dosyasının Türkçe çevirisidir (CC-BY-4.0). Değişiklik: Türkçeleştirme.

# MotionBricks rehberli eğitim

Bu eğitim MotionBricks'in temel kavramlarını tanıtır ve onları eksiksiz bir SOMA iş akışında uygular. Alıştırmalar tek bir üretimden başlayıp tamamen etkileşimli bir sahneye kadar ilerler:

1. **Temeller:** Inbetweener (ara kare üretici) ile yürürken eğilip engelin altından geçme etkileşimini hedef pozlardan yeniden kurun.
2. **Smart Primitive:** Aynı pozları yeniden kullanılabilir, poza dayalı bir etkileşime dönüştürün.
3. **Sahne:** Birkaç Smart Primitive'i bir etkileşim parkurunda bir araya getirin.
4. **Scene Preview:** Sonucu gerçek zamanlı oynatın ve nasıl işlediğini inceleyin.

Bu eğitimdeki bütün komutlar ve dosya yolları depo köküne göredir.

![Eğitimin sonucu](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tutorial_final_results.gif)

*Rehberli eğitimin son hali.*

> **Unitree G1:** Aynı iş akışı Unitree G1 için de geçerli. **Character** seçicisinden **Unitree G1**'i seçin. SOMA dosyaları yerine G1 CSV kliplerini ve G1 Smart Primitive belgelerini kullanın.


## Temel kavramlar

### Pozlar arasında hareket üretimi

MotionBricks, karakterin son hareketinden bir ya da daha fazla hedef poza (target pose) uzanan bir geçiş üretir.

![Yeşil renkte dört bağlam pozu, üretilen pozlar ve mavi renkte bir hedef poz](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/core_idea.gif)

Her üretim şu girdileri kullanır:

- **Dört bağlam pozu (context pose):** Karakterin en son hareketi. Birlikte, karakterin o anki duruşunu ve geçişe hangi hızla girdiğini anlatırlar.
- **Bir ila dört hedef poz:** Karakterin ulaşması gereken pozlar. Tek hedef poz, karakterin durduğu yerdeki duruşu, konumu ve yönelimi belirler. İkinci poz hedefe bir hız ekler. Üçüncü ve dördüncü poz ivme ve momentum ekler, böylece karakter hedefin içinden hareket ederek geçer.

Geçişin süresini model belirler. Kök (root) yörüngesini ve gövde hareketini üretir. Hedefe ulaşmak için gereken ayak yerleşimini ve momentum değişimlerini de o üretir.

**Sırada ne olacağına** ana uygulama karar verir. MotionBricks ise **karakterin o anki hareketinden bu amaca nasıl gideceğini** belirler.

Bu eğitimde dört örnek uygulama var. MotionBricks'in nasıl entegre edileceğini gösterirler ve üretim için hazırlanmış bir araç takımı değildirler. Eğitim, modelin nasıl çalıştığını açıklar. Modelin girdilerinin ve çıktılarının bir animasyon sisteminde nereye oturduğunu da gösterir. Bu kavrayışla geliştiriciler MotionBricks'i kendi motorlarına, araçlarına ve iş hatlarına bağlayabilir.

### Geçiş klipleri yerine anahtar pozlar

Klasik, klip tabanlı sistemlerde birçok durum çifti arasındaki geçişlerin tek tek hazırlanması gerekir. Bu kliplerin seçilmesi, eşzamanlanması ve karıştırılması gerekir. MotionBricks'te ise durum makinesi bunların yerine hedef pozları seçer ve sahneye yerleştirir.

![Bağlam pozları, hedef pozlar ve bir karışım uzayıyla sürülen SOMA lokomosyonu](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_targets_soma.gif)
*Bağlam pozları mor, o anki hedef pozlar mavi, karışım uzayının (blend space) canlı sonucu yeşil gösterilir.*

### Üretimleri zincirleme

Tek bir üretim bilerek kısa tutulur. Sürekli hareket bir döngüden doğar. Uygulama bir geçiş üretir ve onu geçiş noktasına kadar oynatır. Gösterdiği her pozu kaydeder. Oynatılan son dört poz, bir sonraki üretimin bağlam pozlarına dönüşür.

![Animasyon üretimlerini zincirleme](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/target_b_flow.png)

Özbağlanımlı (autoregressive) derken bu döngüyü kastediyoruz. Her üretim, bir önceki üretimin ortaya koyduğu hareketten başlar. Model animasyonun tamamını hiçbir zaman bir kerede görmez. Yalnız oynatılan son dört pozu ve sıradaki hedefi görür.

Hem Inbetweener'ın hem de Scene Preview'un temel entegrasyon kalıbı budur. Üretilen hareket, lokomosyon değişimleri ve kliplere giriş ya da kliplerden çıkış geçişlerinin hepsi bir sonraki bağlamı besleyebilir.

## Bölüm 1: Inbetweener ile animasyon üretin

Inbetweener, MotionBricks üretim döngüsünün en küçük eksiksiz örneğidir. Kliplerden pozlar örnekler ve bu pozları görünüme yerleştirmenizi sağlar. Ardından bu pozların içinden sırayla geçen bir hareket üretir.

Bu bölümde şu poz dizisini oluşturacaksınız:

![Kaçınma için hedef pozlar](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/duck_under_flow.png)

### Inbetweener'ı başlatın

Inbetweener'ı şu komutla başlatın:

```sh
uv run python -m app.inbetweener_demo --model soma
```

Newton görüntüleyicisi Inbetweener ile açılır.

![Newton görüntüleyicisinde açılmış Inbetweener](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_01_open.png)

Arayüzün iki ana alanı var:

- **Görünüm (View):** Karakteri, o anki pozları ve üretilen hareketi gösterir.
- **Inbetweener penceresi:** **Character** seçicisini, **Target Poses** listesini ve üretim ayarlarını içerir. **Character** seçicisinde **SOMA** yazdığından emin olun.

Gezinmek için imleci görünümün üzerine getirin ve şu kontrolleri kullanın:

- **W**, **A**, **S** ve **D** ya da **ok tuşları** kamerayı yatay olarak taşır.
- **Q** ve **E** kamerayı aşağı ve yukarı taşır.
- **Orta fare tuşuyla sürükleme** kamera pivotunun çevresinde döndürür.
- **Shift + orta fare tuşuyla sürükleme** kaydırır.
- **Ctrl + orta fare tuşuyla sürükleme** kamera pivotuna yaklaştırır ya da uzaklaştırır.
- **Fare tekerleği** kamera pivotuna yaklaştırır ya da uzaklaştırır. **Ctrl + fare tekerleği** kameranın görüş alanını değiştirir.

### End hedefini oluşturun ve yerleştirin

**Target Poses** listesinde **Target 1**'e tıklayarak onu seçin ve özelliklerini görüntüleyin:

![Seçili Target 1 ve özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_02_Pose1.png)

**Target 1**'e çift tıklayın ve adını **End** yapın:

![Adı End olarak değiştirilmiş poz](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_03_End.png)

Görünümde, mavi pozun üzerindeki taşıma tutamacıyla (gizmo) karakteri yeşil (Y) eksen boyunca taşıyın:

![End pozunu taşıma](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_05_EndMove.png)

Karakteri biraz daha ileri taşıyın:

![Karakteri taşıma](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_04_Move.png)

**Generate**'e tıklayın:

![Generate düğmesi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_06_generate.png)

İlk MotionBricks geçişiniz böylece üretilir:

![İki poz arasında animasyon üretimi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tutorial_first_anim.gif)

### Hedef ve bağlam pozlarını inceleyin

**End** hedefi hala seçiliyken **Visibility** altında **Show Skeleton(s)** ve **Show Velocity** seçeneklerini açın. **Show Mesh** zaten açık:

![Açılmış görünürlük seçenekleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_07_visibility.png)

Görünüm artık bu hedef için klipten örneklenen dört pozu gösteriyor. Bu pozlar birlikte hedefi ve hedefin hızını tanımlar. Kırmızı ok hedefin hızını temsil eder:

![Hedef pozun kareleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_08_targetpose.png)

İncelemek için **Start**'ı seçin ve aynı seçenekleri açın:

![Bağlam pozunun kareleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_09_startpose.png)

**Start** için görünüm, seçili karedeki pozu ve ondan önceki üç pozu gösterir. Pozlar 30 Hz'de örneklenir, yani 120 Hz'lik bu BVH klibinde her dördüncü kareye denk gelirler. Bu dört poz üretimin bağlam pozlarıdır.

### Ara hedefleri ekleyin

**Target Poses** listesinin altındaki **+** düğmesine tıklayarak iki yeni hedef oluşturun:

![Eklenen iki yeni hedef](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_10_addposes.png)

**Target 2** ve **Target 3**'ü seçip listede yukarı sürükleyin:

![Listede yukarı taşınmış yeni hedefler](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_11_moveposes.png)

Hedeflere şu adları verin:

- **Entry**
- **Duck-under**

**Entry** hedefini seçin ve tutamaçla onu **Start** ile **End** arasına, mesafenin yaklaşık üçte birine yerleştirin:

![Entry pozunu taşıma](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_12_movepose.png)

**Duck-under** hedefini seçin ve mesafenin yaklaşık üçte ikisine yerleştirin:

![Kaçınma pozunu taşıma](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_13_movepose.png)

### Klibi ve kareleri atayın

**Entry** hedefini seçin ve **Clip** alanının yanındaki **...** düğmesine tıklayın:

![Klip seçme düğmesi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_14_clip.png)

Şu BVH klibini bulup açın:

```text
assets/motions/soma/avoidance/neutral_avoid_obstacle_bend_down_walk_ff_180_R_001__A543.bvh
```

Görünüm artık klibi o hedefin üzerinde gösteriyor:

![İlk poz için yeni klip](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_15_newclippose.png)

**Clip** alanının altındaki **Seconds** düğmesine tıklayarak **Frames** kaydırıcısına geçin:

![Seconds düğmesi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_16_seconds.png)

**199**. kareyi seçin:

![Karelere geçiş](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_17_frames.png)

> **Not:** Değeri doğrudan yazmak için kaydırıcıya Ctrl + tıklayın.

Görünümde döndürme tutamacıyla (en büyük çember) hedefi, çevresindeki hedeflerle uyumlu olacak biçimde yönlendirin:

![Pozu yönlendirme](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_18_rotate.png)

Hedefe şu ayarları uygulayın:

![Entry hedefinin ayarları](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_19_settings.png)

- **Poses to Use:** 4. Böylece seçili kareden başlayan pozlar hedefteki hızı, ivmeyi ve momentumu tanımlar. Bkz. [Poses to Use ayarı](ARA-KARE-URETICI.md#poses-to-use-ayarı).
- **Transition Early By (frames):** 8. Geçiş noktası 8 kare öne alınır, böylece bir sonraki üretim karakter bu hedefe varmadan başlar. Bu hedef o zaman bir kılavuz görevi görür. Karakteri bir sonraki hedefin istediği yönelime ve hıza yönlendirir. Bağlam pozları hala hareket halinde olduğundan model bu hızı ileri taşır. Hareketin hedefe olabildiğince yakından uyması gerektiğinde daha küçük bir değer kullanın. Hedef, karakteri durduran tek bir poz da olabilir, uyulacak hızı ve momentumu anlatan dört poz da. Karakter bu durumda hedefe kadar oynar. Bkz. [Üretimleri zincirleme](#üretimleri-zincirleme).
- **Tokens to Generate:** **Min** değerini 8, **Max** değerini 14 olarak bırakın. Bu aralık, modelin bu hedefe giden geçiş için seçebileceği uzunlukları belirler. Her hareket belirteci (token) dört ham kare verir, yani 8 ile 14 arası hareket belirteci 32 ile 56 arası ham kare eder. **Raw frames:** göstergesi bu aralığı gösterir. Hedefler arasındaki mesafe uzadıkça yolu kat etmek için daha çok belirteç gerekir. Mesafe kısaldıkça daha az belirteç gerekir, böylece karakter süreyi doldurmak için fazladan hareket eklemez.

**Duck-under** hedefini seçin ve BVH klibini atamak için aynı adımları tekrarlayın:

```text
assets/motions/soma/avoidance/neutral_avoid_obstacle_bend_down_walk_ff_180_R_001__A543.bvh
```

**328**. kareyi seçin ve bu hedefe şu ayarları uygulayın:

![Duck-under hedefinin ayarları](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_20_settings.png)

Karakteri görünümde aşağıdaki gibi yönlendirin:

![Eğilerek geçme pozu](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_inbetweener_21_pose2.png)

### Son animasyonu üretin

Hedef listesi artık tamam. Son animasyonu görmek için **Generate**'e tıklayın:

![Gerçek zamanlı animasyon üretimi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tutorial_end_inbetweener.gif)

Bu kurulumu şunları değiştirerek daha ayrıntılı keşfedebilirsiniz:

- Her hedef için karakterin konumu ve yönelimi.
- Her hedef için kullanılan klip karesi.
- **Poses to Use** ve **Transition Early By** değerleri.
- Üretilen her geçişin ne kadar uzun olabileceğini belirleyen **Tokens to Generate** aralığı. Altındaki **Raw frames:** göstergesi buna karşılık gelen uzunluğu kare olarak verir.

> **Not:** MotionBricks'i kendi uygulamanıza entegre etmenin temel kavramları bunlardır. Hedef pozları uygulama mantığınız gerçek zamanlı tanımlar, MotionBricks de onlara ulaşan hareketi üretir. Inbetweener kodunun arkasındaki kavramlar için **[Inbetweener](ARA-KARE-URETICI.md)** ve **[Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md)** sayfalarına bakın.

## Bölüm 2: Bir Smart Primitive oluşturun

Eğitimin kalan bölümleri, örnek uygulamaların etkileşimli bir MotionBricks deneyimi kurmak için nasıl birlikte çalıştığını gösterir:

- **Smart Primitive Editor:** Etkileşim alanlarının çevresinde poza dayalı basit bir mantık tanımlar.
- **Scene Editor:** Smart Primitive'leri bir ortamda bir araya getirir.
- **Scene Preview:** Sonucu gerçek zamanlı çalıştırır. Lokomosyonu, etkileşim algılamayı, davranışları ve üretilen hareketi birleştirir.

Bu uygulamalar birlikte, geliştiricilere MotionBricks iş akışı için uygulamaya dönük bir başvuru kaynağı sunar. Geliştiriciler her parçayı kendi entegrasyonlarına uyarlayabilir.

### Smart Primitive Editor'ı başlatın

Smart Primitive Editor'ı başlatın:

```sh
uv run python -m app.smart_primitive_editor
```

Newton görüntüleyicisi Smart Primitive Editor ile açılır:

![Newton görüntüleyicisinde açılmış Smart Primitive Editor](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_01_open.png)

Başlamadan önce **Character** seçicisinde **SOMA** yazdığından emin olun:

![SOMA karakteri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_02_SOMA.png)

> **Not:** Görünüm kontrolleri Inbetweener'dakiyle aynıdır ve Scene Editor de bunları kullanır. Scene Preview'un kendi [kontrolleri](#kontroller) var. Orada **W**, **A**, **S** ve **D** ya da **ok tuşları** karakteri hareket ettirir. Kamera karakteri izler.

### Düğüm hiyerarşisini oluşturun

Bir Smart Primitive (akıllı ilkel) düğümlerden oluşan bir hiyerarşiyle kurulur ve her düğümün belirli bir rolü vardır. Şu hiyerarşiyi oluşturun:

- Interaction box
- Pivot
  - Entry pose
  - Duck-under pose

**Root** düğümünü seçin ve hiyerarşinin üstündeki **+** düğmesine tıklayın:

![Düğüm ekleme düğmesi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_03_addnode.png)

Yeni düğüme çift tıklayın ve adını **Interaction box** yapın:

![Adı değiştirilmiş etkileşim kutusu](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_04_interactionbox.png)

Hiyerarşinin geri kalanını kurmak için bir düğüm seçin ve altına bir alt düğüm eklemek üzere **+**'ya tıklayın:

![Smart Primitive hiyerarşisi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_05_hierarchy.png)

Sonraki adımlar Smart Primitive'teki her düğümün rolünü tanımlar.

### Etkileşim kutusunu (Interaction Box) yapılandırın

**Interaction box** düğümünü seçin ve **Properties** penceresine gidin. Kullanılabilir bileşen türlerini görmek için pencerenin boş bir alanına sağ tıklayın ve **Add** > **Components** > **Interaction Box**'ı seçin:

![Interaction Box bileşeni ekleme](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_06_interactionbox.png)

**Width** ve **Depth** değerlerini 1 m, **Height** değerini 2 m yapın. Kutunun zemine oturması için **Translation**'ın Z değerini 1 m yapın:

![Etkileşim kutusunun özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_07_interactionboxset.png)

Kutuyu Z ekseni boyunca 1 m yukarı taşımak için 3B tutamacı da kullanabilirsiniz:

![Etkileşim kutusunu taşıma](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_08_interactionboxtransf.png)

### Pivotu yapılandırın

Hedef pozlar Smart Primitive içinde sabit konumlarda hazırlanır. Hazırlanan dizinin ilerlediği yöne bakarlar. Çalışma zamanında ise karakter etkileşim alanına her yönden gelebilir. Pivot olmadan, yandan gelen bir karakter önce hazırlanan Entry pozuna yürür. Sonra diziyi izlemek için döner ve etkileşimden kendi yönü yerine hazırlanan yönde çıkar.

Pivot bu sorunu çözer. Etkileşim başladığında pivot, hedef pozlardan oluşan dizinin tamamını döndürür ve kaydırır. Böylece dizi, karakterin etkileşim alanına girdiği yöne uyar. Karakter de rotasından sapmadan etkileşimin içinden geçer:

![Yandan gelen bir karakter, pivotsuz ve eğitimdeki Pivot ile](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_pivot_00_problem.gif)

*Karakter, hazırlanan diziye 90 derecelik açıyla gelir. Soldaki pivotsuz durumda hazırlanan Entry pozuna yürür ve döner. Sağdaki eğitim Pivot'uyla hedef pozlar döner ve karakterin rotasına kayar. Karakterin yolu bir çizimdir. Çalışma zamanında hareketi MotionBricks üretir.*

Pivotu eklemek için hiyerarşide **Pivot** düğümünü seçin. **Properties** penceresinin boş bir alanına sağ tıklayın ve **Add** > **Components** > **Pivot**'u seçin:

![Pivot ekleme](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_09_addpivot.png)

**Pivot** özellikleri düğüme eklenir:

![Pivot özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_10_pivotprops.png)

Bu eğitimde karakter her yönden gelebilir. İleri gitmeyi sürdürür ve etkileşim alanındaki bir engelin altından geçmek için gövdesini indirir. Sonra rotasına devam eder. Yalnız etkileşim alanının içinde gereken pozları hazırlamanız yeterli. Alanın dışında poz gerekmez. Etkileşim başladığında pivot önce diziyi, karakterin yaklaşma yönüne uyacak şekilde döndürür. Sonra diziyi karakterin giriş noktasına kaydırır.

Bu davranış için **Pivot** özelliklerine şu değerleri verin:

![Tamamlanmış pivot özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_11_pivotpropsset.png)

| Ayar | Değer | Amaç |
| --- | --- | --- |
| Motion Direction | 0, 1, 0 | Pivotun yerel +Y ekseni, hazırlanan dizinin ilerlediği yöndür. |
| Operation Order | Rotation -> Translation | Pivot önce döner, sonra dönmüş öteleme çizgisi boyunca kayar. |
| Translation | TranslateX, Line Projection, Min -0.50 m, Max 0.50 m | Pivot, karakterin yoluyla buluşmak için iki yöne de en çok 0.5 m yana kayabilir. |
| Rotation | RotateZ, Unconstrained | Pivot, her yaklaşma yönüne bakmak için dikey eksen çevresinde dönebilir. |

Her ayarın başka seçenekleri de var, örneğin dönüş için bir **Cone** sınırı ya da öteki **Operation Order**. Kaydetme adımından sonra gelen [Pivotu anlamak](#pivotu-anlamak) bölümü her seçeneği ayrıntılı açıklar ve karakterin hareketine etkisini resimlerle gösterir.

### Hedef pozları yapılandırın

Hiyerarşide **Entry pose** düğümünü seçin. **Properties** penceresinin boş bir alanına sağ tıklayın ve **Add** > **Components** > **Target Pose**'u seçin:

![Hedef poz ekleme](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_12_addpose.png)

**Target Pose** özellikleri, 1. Bölümdeki Inbetweener özellikleriyle aynıdır. Smart Primitive Editor bunlara **Constrain Root**, **IK Pins** ve **Show Trajectory** seçeneklerini ekler. **Tokens to Generate** varsayılanı 8-14 yerine 6-14'tür. Bir dizideki ardışık adımlar çoğu zaman birbirine yakın yerleştirilir ve daha düşük alt sınır, model aralarında daha kısa bir geçiş üretebilsin diye konmuştur:

![Target Pose özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_13_poseprops.png)

**Clip**'in yanındaki **...** düğmesine tıklayın ve 1. Bölümdeki klibi açın:

```text
assets/motions/soma/avoidance/neutral_avoid_obstacle_bend_down_walk_ff_180_R_001__A543.bvh
```

Hedef poz değerlerini aşağıdaki gibi ayarlayın. **Constrain Root**'u açın. Bu seçenek klibin kökünü zemin boyunca düğümün orijinine taşır ve yüksekliğini korur:

![Entry pozunun ayarlanmış özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_14_posepropsset.png)

3B tutamaçla **Entry pose**'u döndürüp etkileşim kutusunun kenarına yakın bir yere yerleştirin. Pozu sarı okla hizalayın. Bu ok pivotun **Motion Direction** değerini, yani hazırlanan dizinin ilerlediği yönü gösterir:

![Poz dönüşümleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_15_posetransf.png)

Aynı adımları **Duck-under pose** için tekrarlayın. Şu değerleri verin ve **Constrain Root**'u açın:

![Duck-under pozunun ayarlanmış özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_16_pose2propsset.png)

3B tutamaçla karakteri, kutunun ortasında duracak şekilde konumlandırıp döndürün:

![İkinci pozun dönüşümleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_18_pose2transf.png)

### Diziyi tanımlayın

Şimdi pozların hangi sırayla kullanılacağını tanımlayın.

**Sequences** penceresinde yeni bir dizi eklemek için **+** düğmesine tıklayın:

![Dizi ekleme düğmesi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_17_addsequence.png)

Dizinin **Pivot** alanını **Pivot** düğümüne ayarlayın. Dizi böylece bu pivotu kullanır:

![Dizinin pivotu](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_19_sequpivot.png)

Hedef pozları, eylemin geçtiği sırayla ekleyin. **+ Add Target Pose**'a tıklayın ve **Entry pose**'u seçin. Yeniden tıklayın ve **Duck-under pose**'u seçin. Bir düğümü **Hierarchy** penceresinden dizinin başlığına sürükleyerek de ekleyebilirsiniz:

![Tamamlanmış dizi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_smartprim_20_sequdone.png)

### Smart Primitive'i kaydedin

Smart Primitive artık tamam.

**File** > **Save As...** yolunu izleyin ve Smart Primitive belgesini şu adla kaydedin:

```text
assets/smart_primitives/soma/SOMA_duck_tutorial.smart_primitive.json
```

### Pivotu anlamak

Bu bölüm, yukarıda yapılandırılan **Pivot**'un hazırlanan pozları her yaklaşmaya nasıl uyarladığını açıklar. Ortaya çıkan pozların karakterin gelen hareketini neden sürdürdüğünü de anlatır.

Şekiller yukarıda kaydedilen `SOMA_duck_tutorial` Smart Primitive'ini kullanır. Pozları, Scene Preview'un çalıştırdığı pivot hizalama koduyla yerleştirir ve düzenleyicilerin kullandığı SOMA ağını gösterir. Pembe ve yeşil karakterler, modele gönderilen gerçek hedef pozlardır. Mavi karakter gelen karakteri temsil eder. O ve kesikli yol yalnız birer çizimdir. Çalışma zamanında yolu MotionBricks üretir.

#### Pivot neden gerekir

Bir Smart Primitive'in hedef pozları, Smart Primitive'e göre sabit konumlarda hazırlanır. Çalışma zamanında ise karakter etkileşim alanına pek çok farklı konumdan, yönden ve hızla gelebilir. Pozlar hazırlandıkları yerde kalsaydı ilk hedef karakterin yolunun bir yanında kalabilir ya da başka bir yöne bakabilirdi. Üretilen geçişin ona ulaşmak için ani bir dönüş ya da yana adım atması gerekirdi.

Pivot, dizinin tamamının yerleşimini yaklaşmaya uyarlar. **Pivot** düğümü, hedef poz düğümlerinin üst düğümüdür. Çalışma zamanı pivotu taşıyıp döndürdüğünde her hedef poz ona rijit bir grup olarak eşlik eder. Örneklenen her pozun yalnız kökü dönüştürülür. Eklem değerleri klipten örneklendiği gibi kalır, böylece hazırlanan gövde hareketi korunur.

#### Pivot ne zaman hizalanır

Scene Preview'da pivot her dizinin başında bir kez hizalanır:

1. Karakterin yoklama kutusu (probe box), karakterin önüne doğru hareket yönünde uzanır. Yoklama kutusu Smart Primitive'in bir etkileşim kutusuyla kesiştiğinde dizi başlar.
2. Davranış bir dizi seçer. `VelocityMatchingSequenceBehavior`, ilk hedef hızı karakterin o anki hızına en yakın olan diziyi seçer.
3. Dizi bir pivota başvuruyorsa pivot yaklaşmayla hizalanır. Hizalama karakterin konumunu ve istenen hızını kullanır. Yoklama kutusunun zeminde kapladığı çizgiyi de hesaba katar.
4. Dünya uzayındaki hedef pozlar hizalanmış hiyerarşiden hesaplanır ve MotionBricks'e gönderilir.

Sınırlar her zaman pivotun hazırlandığı yerleşimden ölçülür, bir önceki hizalamadan ölçülmez. Bu sayede Smart Primitive tekrarlanan etkileşimlerde kaymaz. Pivotu olmayan bir dizi o anki yerleşimini korur.

#### Her ayarın rolü

Aşağıdaki üstten görünüm, eğitimdeki Smart Primitive'in hazırlanan yerleşimini gösterir. Smart Primitive Editor'ın **Pivot** için çizdiği görsel kılavuzlar da görünür:

![Eğitimdeki Pivot'un hazırlanan yerleşimi, üstten görünüm](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_pivot_01_layout.png)

*Hazırlanan yerleşim, üstten görünüm. Sarı ok Motion Direction'dır, kırmızı kesikli çizgi de Line Projection aralığıdır.*

| Ayar | Rol | Düzenleyicideki kılavuz |
| --- | --- | --- |
| **Motion Direction** | Hazırlanan dizinin pivotun yerel eksenlerinde ilerlediği yön. Hizalama pivotu bu yön karakterin ilerleme yönüne uyacak şekilde döndürür. Hedef pozlar bu yüzden sarı ok boyunca yönlendirildi. | Sarı ok |
| **Rotation** | Pivotun çevresinde dönebileceği yerel eksen. **RotateZ** onu dikey eksen çevresinde döndürür. **Unconstrained** her yönelime izin verir. **Cone**, dönüşü koni merkezinin iki yanında **Half Angle** kadarla sınırlar. **Cone Side** bu merkezi belirler. **Motion Direction** koniyi Motion Direction üzerine ortalar. **Mirrored** ters yöne ortalar. **Both** iki yana da izin verir. | Kırmızı yay (yalnız Cone) |
| **Translation** | Pivotun boyunca kayabileceği yerel eksen. **Line Projection** ile pivot, **Min** ve **Max** uzaklıkları arasında, yoklama kutusunun çizgisine en yakın noktaya gider. Pratikte bu nokta, karakterin yolunun çizgiyi kestiği yerdir. **Unconstrained** eksen boyunca her uzaklığa izin verir. | Kırmızı kesikli çizgi |
| **Operation Order** | **Rotation -> Translation** önce pivotu döndürür. Dönüş, öteleme çizgisini de hazırlanan orijin çevresinde döndürür. Pivot sonra dönmüş çizgi boyunca kayar. **Translation -> Rotation** önce hazırlanan çizgi boyunca kayar, sonra döner. | Kılavuz çizilmez |

**Pivot** bileşeninin **Show Motion Direction**, **Show Translation Constraint** ve **Show Rotation Constraint** seçenekleri bu kılavuzları gösterir ya da gizler.

#### Farklı yaklaşımlarda sonuç

Aşağıdaki şekil, hedef pozların altı farklı yaklaşmadaki yerleşimini üstten görünümle karşılaştırır. Her panelde mavi karakter, etkileşimin başladığı andaki gelen karakterdir. Pembe ve yeşil karakterler Entry ve Duck-under hedef pozlarıdır. Soluk kesikli ok, hazırlanan Motion Direction'ı gösterir.

![Altı yaklaşma için pivot hizalaması](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_pivot_02_cases.png)

1. **Pivot yok:** Pozlar hazırlandıkları yerde kalır. Açılı yaklaşan bir karakter Entry pozuna ulaşmak için keskin bir dönüş yapar. Engelin altından geçmek için de bir kez daha döner.
2. **Yalnız öteleme:** Karakter merkezin 0.4 m yanından yaklaşır. Pivot kendi çizgisi boyunca 0.4 m kayar ve pozlar karakterin yoluna gelir.
3. **Yalnız dönüş:** Karakter 35 derecelik açıyla yaklaşır. Pivot Z çevresinde 35 derece döner ve pozlar karakterin ilerleme yönüne bakar.
4. **Eğitim ayarları:** Karakter açılı ve merkezin dışından yaklaşır. Pivot önce döner. Sonra dönmüş çizgi boyunca, karakterin yolunun çizgiyi kestiği noktaya kayar.
5. **Line Projection** sınırı: Karakter 0.85 m yandan yaklaşır. Kayma çizginin +0.5 m ucunda durur ve kalan 0.35 m'yi üretilen geçiş kapatır.
6. **Cone sınırı:** Koninin **Half Angle** değeri 30 derece, karakter de 55 derecelik açıyla yaklaşır. Dönüş 30 derecede durur ve kalan 25 dereceyi üretilen geçiş kapatır.

Aşağıdaki animasyon eğitim ayarlarını kullanır. Yaklaşmayı önce yanal uzaklıklar boyunca, sonra yaklaşma açıları boyunca, en sonda da ikisi birlikte değişecek şekilde tarar. Her adım hem üstten hem de eğilme duruşunun açıkça seçildiği perspektiften gösterilir. Alttaki gösterge, pivotun bunun sonucunda yaptığı dönüşü ve kaymayı bildirir:

![Yaklaşma uzaklıkları ve açıları boyunca canlandırılmış pivot hizalaması](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_pivot_03_alignment.gif)
*Eğitim ayarlarıyla canlı pivot hizalaması. İlk aşamada uzaklık çizgiyi aştığında kayma ±0.50 m'de sınırlanır.*

#### Gelen hareketle süreklilik

MotionBricks hareketi her zaman dört bağlam pozundan bir sonraki hedefe doğru üretir. Bu pozlar karakterin o anki hızını taşır. Pivot, hedefleri bu bağlamla tutarlı olacak şekilde yerleştirir:

- **Konum:** Ötelemeden sonra Entry pozu karakterin yolunun üzerindedir. Karakter yana adım atmadan, ileri gitmeyi sürdürerek ona ulaşır.
- **Yönelim:** Dönüşten sonra Entry hedefinin dört pozu, dolayısıyla hızı da, karakterin ilerleme yönünü gösterir. Bağlam pozları ile hedef pozlar aynı hareket yönünü anlatır. Üretilen geçiş, gelen yürüyüşü doğal biçimde sürdürür.
- **Dizinin içinde:** Bütün hedef pozlar birlikte hareket eder. Entry ve Duck-under pozları arasındaki aralık, yönelim ve zamanlama tam hazırlandığı gibi kalır. **Transition Early By** ile karakter momentumunu bir hedeften ötekine taşır.
- **Çıkış:** Son poz da dizinin geri kalanıyla aynı açıda döner. Dizi bu yüzden hizalanmış Motion Direction boyunca çıkar. Etkileşim bittiğinde karakter zaten ilk ilerleme yönünde hareket ediyordur ve lokomosyon düzeltici bir dönüşe gerek kalmadan sürer.

Sonuçta pivot, hazırlanan pozları sanki klip karakterin gerçek yolu boyunca kaydedilmiş gibi yeniden ifade eder. Bunun geçerli kaldığı aralığı sınırlar belirler. Bu Smart Primitive'te öteleme çizgisi ilerleme yönünü enine keser. Duck-under pozu pivot orijinine yakındır. Pivotun dönmesi bu yüzden karakterin etkileşim alanını kesme açısını değiştirirken eğilme duruşu alanın ortasına yakın kalır. Kendi Smart Primitive'lerinizi hazırlarken izin verilen her yerleşimi engel için geçerli tutan uzaklıklar ve koni açıları seçin.

### Varyasyonları keşfedin

Varyasyon yapmak için pozları değiştirin ya da diziye yeni pozlar ekleyin. Farklı hızlara ya da gelen hareketlere yanıt veren diziler de ekleyebilirsiniz. Smart Primitive'ler ve düzenleyicileri hakkında daha fazlası için **[Smart Primitive Editor](SMART-PRIMITIVE-EDITORU.md)** sayfasına bakın.

Karşılaştırmak için şu Smart Primitive belgesini Smart Primitive Editor'da açın. Eğitimdeki Smart Primitive'e benzer ve kaçınmanın birkaç varyasyonunu içerir:

`assets/smart_primitives/soma/avoidance.smart_primitive.json`

Eğitimdeki Smart Primitive'i şunlarla genişletir:

- İkinci bir dal, `Variation 1`.
- Başka bir klipten örneklenmiş bir `Jog` dizisi.
- `Walk` ve `Jog` dizilerinin paylaştığı, bu eğitimdekiyle aynı ayarlara sahip aynı pivot.
- Çalışma zamanında hıza göre dizi seçimi.
- Zeminden yükseltilmiş, yaklaşık 1.46 m ile 2.46 m arasında duran daha küçük bir etkileşim kutusu. Ayakta duran bir karakterin yoklama kutusu 1.6 m yüksekliğindedir, böylece etkileşim kutusuna erişir ve kaçınmayı başlatır. Karakter çömeldiğinde yoklama kutusu yaklaşık 0.96 m'ye iner. Karakter o zaman etkileşimi başlatmadan engelin altından geçebilir. Bu ince ayar, eğitimin konusu dışında kalır. Eğitimdeki etkileşim kutusu zeminden 2 m'ye kadar uzanır, böylece her lokomosyon modu etkileşimi başlatır.

![Hız eşleme akışı](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/velocity_matching_flow.png)

Poza dayalı etkileşimlerin esnekliği burada görülür. Daha hızlı bir yaklaşma için yeni bir durum makinesi dalı ya da bir dizi geçiş klibi gerekmez. Başka bir hedef dizisi yeterlidir. Davranış da karakterin o anki hızına en iyi uyan diziyi seçer.

## Bölüm 3: Kaçınma, atlama ve oturma içeren bir etkileşim sahnesi kurun

Sahne şu Smart Primitive'leri kullanır:

- 2. Bölümde oluşturulan `SOMA_duck_tutorial.smart_primitive.json`.
- Depoyla birlikte gelen `jump_0_5m.smart_primitive.json`.
- Depoyla birlikte gelen `sit.smart_primitive.json`.

### Scene Editor'ı başlatın

Scene Editor'ı başlatın:

```sh
uv run python -m app.scene_editor
```

Newton görüntüleyicisi Scene Editor ile açılır:

![Sahne düzenleyicisi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_01_open.png)

**Character** seçicisinde **SOMA** yazdığından emin olun.

Bir sahnedeki her Smart Primitive başvurusu aynı rig'i (iskelet düzenini) kullanmalı. `CharacterStart` öğesi, Scene Preview sahneyi başlattığında karakterin nerede belireceğini tanımlar.

### Kaçınma Smart Primitive'ini ekleyin

Scene Editor'da, en üstteki **+** düğmesine tıklayarak yeni bir sahne öğesi ekleyin:

![+ düğmesiyle Scene penceresi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_02_add.png)

**Smart Primitive**'i seçin:

![Smart Primitive içeren Add menüsü](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_03_addsmartprim.png)

Yeni öğeye çift tıklayın ve adını **avoid** yapın:

![Adı değiştirilmiş Smart Primitive örneği](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_04_addsavoid.png)

**Properties** penceresinde:

- **Reference** alanını 2. Bölümde oluşturulan Smart Primitive belgesine ayarlayın.
- Çalışma zamanı **Behavior** alanını `VelocityMatchingSequenceBehavior` yapın.

![Smart Primitive örneğinin özellikleri](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_05_primpropsset.png)

3B tutamaçla Smart Primitive örneğini sahnede döndürüp konumlandırın:

![Smart Primitive örneğinin yerleşimi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_06_primmove.png)

Engeli göstermek için sahneye bir **Solid Cube** öğesi ekleyin:

![Katı küp ekleme](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_07_solidcube.png)

3B tutamaç ve küpün özellikleriyle küpü, Smart Primitive'in **Duck-under pose**'unun üstünde duracak şekilde konumlandırıp ölçekleyin:

![Engel küpünü ekleme](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_08_obstacle.png)

### Parkuru tamamlayın

Engel parkurunu tamamlamak için önceki adımları tekrarlayarak şu Smart Primitive'leri ekleyin:

- `assets/smart_primitives/soma/jump_0_5m.smart_primitive.json`
- `assets/smart_primitives/soma/sit.smart_primitive.json`

**Sit** Smart Primitive'i için çalışma zamanı davranışı olarak `SittingBehavior`'ı seçin.

Smart Primitive örneklerini sahnede konumlandırıp döndürün:

![Sahne yerleşimi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_09_layout.png)

Bu Smart Primitive'lerin engellerini göstermek için Solid Cube'lar ekleyin:

![Engellerin yerleşimi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneedit_10_layoutobstacles.png)

### Sahneyi kaydedin

**File** > **Save As...** yolunu izleyin ve sahneyi şu adla kaydedin:

```text
assets/scenes/soma_tutorial.scene.json
```

Sahne, başvurularını sahne dosyasına göre göreli olarak saklar. Birkaç örnek aynı Smart Primitive belgesine başvurabilir ve her biri kendi yerleşimini ve davranışını taşır. Daha fazla Smart Primitive ekleyip kendi yerleşiminizi kurarak denemeyi sürdürebilirsiniz.

## Bölüm 4: Sahneyi oynatın

Scene Preview, derli toplu bir gerçek zamanlı karakter demosudur. Şu parçaları birleştirir:

- Bir durum makinesi.
- Lokomosyon karışım uzayları.
- Etkileşim algılama.
- Smart Primitive davranışları.
- Klip oynatma.
- MotionBricks üretim API'si.

### Scene Preview'u başlatın

Çalıştırın:

```sh
uv run python -m app.scene_preview --scene assets/scenes/soma_tutorial.scene.json
```

Newton görüntüleyicisi Scene Preview ile açılır:

![Eğitim sahnesiyle açılmış Scene Preview](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_sceneprev_01_start.png)

Sahne yüklenmezse üç şeyi denetleyin. Her Smart Primitive başvurusu çözümlenebilmeli. Başvurulan her Smart Primitive belgesi, sahnenin rig'i olan SOMA'yı kullanmalı. Her örneğin geçerli bir **Behavior** değeri olmalı.

Karakter bir bekleme döngüsünde başlar ve sahneyi keşfetmeye hazırdır:

![Son sonuç](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tutorial_end_scenepreview.gif)

### Kontroller

Karakteri hareket ettirmek ve kamerayı döndürmek için şu kontrolleri kullanın:

| Eylem | Klavye ve fare | Oyun kumandası |
| --- | --- | --- |
| İleri, sola, geri ya da sağa hareket | **W**, **A**, **S** ve **D** ya da **ok tuşları** | Sol çubuk ya da D-pad |
| Koşma | Hareket ederken **Space**'i basılı tutun | Sol çubuğu daha fazla itin |
| Elle etkileşim ya da oturduktan sonra kalkma | **I** | **X** |
| Forward ve Strafing modları arasında geçiş | **O** | **Y** |
| Crouching ve Strafing modları arasında geçiş | **P** | **B** |
| Skipping ve Zombie gibi stil modları arasında sırayla geçiş | **[** | **A** |
| Kamerayı karakterin çevresinde döndürme | **Orta fare tuşuyla** sürükleyin | Sağ çubuk |

**Auto Interact** varsayılan olarak açıktır. Scene Preview, kesişen en yakın Smart Primitive örneğinin etkileşimini her yaklaşmada bir kez başlatır. Her etkileşimi **I** (kumandada **X**) ile elle başlatmak için **Scene Preview** penceresinde **Auto Interact**'i kapatın.

Karakter oturduğunda `Stand` dizisini çalıştırıp lokomosyona dönmek için **I**'ya (kumandada **X**) basın.

### Çalışma zamanını inceleyin

**Scene Preview** penceresinde şu seçenekleri açın:

- **Show Context Poses**, bir sonraki üretime verilen son pozları gösterir.
- **Character** altındaki **Show Target Poses**, o anki hedef pozları gösterir.
- **Show Blend Space Poses**, karıştırılmış hedef pozu lokomosyonun onu yerleştirdiği yerde gösterir. Lokomosyon yeni harekete ihtiyaç duyduğunda bu hedefe doğru üretir.
- **Show Probe Box**, karakterin hareket yönünde öne uzanan yoklama kutusunu gösterir.
- **Show Interaction Boxes**, Smart Primitive davranışlarının başlayabileceği alanları gösterir.

Yoklama kutusu her etkileşim kutusuna ulaştığında **Status** bölümündeki **State:** değerinin Locomotion'dan Interaction'a geçişini izleyin. Yoklama kutusu etkileşim kutusuyla kesiştiği sürece etkileşim kutusu kırmızıdan yeşile döner. Kaçınma ya da atlama etkileşimi bittiğinde karakter lokomosyona döner. Hareket girdisi yoksa bekleme durumuna geçer. Oturma etkileşimi, kullanıcı `Stand` dizisini isteyene kadar döngüdeki klibinde kalır.

> **Not:** Ayaklar kayıyorsa ya da eğilerek geçme çok erken veya çok geç başlıyorsa Smart Primitive'i Smart Primitive Editor'da ayarlayın:
>
> - **Hedef pozlar arasındaki mesafe:** Karakterin engelden önce gövdesini indirmeye yetecek payı olsun diye **Entry pose**'u **Duck-under pose**'tan uzaklaştırın. Karakter mesafeyi kapatmak için fazladan adım atıyorsa pozları birbirine yaklaştırın.
> - **Transition Early By (frames):** **Entry pose** ve **Duck-under pose** değerlerini ayarlayın. Daha büyük bir değer geçiş noktasını öne alır ve karakter eğilmeye daha erken hazırlanır. Daha küçük bir değer, karakterin her hedefe daha yakından ulaşmasını sağlar.
>
> Smart Primitive'i kaydedin, sonra sonucu görmek için sahneyi Scene Preview'da yeniden açın. Bu değerlerin ince ayarı yinelemeli bir iştir. Her etkileşim kutusunun içindeki hareket doğru görünene kadar sürdürün.

## Özet: bu iş akışı neyi gösteriyor

Tek bir klipten alınan iki hedef poz önce doğrudan üretim hedefi, sonra da yeniden kullanılabilir bir etkileşim oldu. Depodaki hazır kaçınma Smart Primitive'i, çalışma zamanı durum makinesinde hiçbir değişiklik gerektirmeden bir Jog varyasyonu ekliyor. Sahne bu etkileşimi Jump ve Sit ile birleştirdi. Her biri aynı poza dayalı üretim kalıbını kullanıyor.

Temel entegrasyon modeli bu sorumluluk ayrımıdır:

- Uygulama ya da durum makinesi sıradaki hedefleri seçer.
- Smart Primitive'ler konuma özgü hedefleri düzenler ve her yaklaşmaya uyarlar.
- Davranışlar uygun hedef dizisini seçip oynatır.
- Sahneler bu etkileşimleri ve görünür geometrilerini yerleştirir.
- MotionBricks, karakterin o anki bağlamından sürekli hareket üretir.

Örneklerin arkasındaki yeniden kullanılabilir kütüphane çağrılarını incelemek için [Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md) ile devam edin. [Smart Primitive Editor](SMART-PRIMITIVE-EDITORU.md), [Scene Editor](SAHNE-EDITORU.md) ve [Scene Preview](SAHNE-ONIZLEME.md) sayfaları her katmanı daha ayrıntılı anlatır.

---

**Unitree G1 Türkiye:** Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org. [Unitree G1 ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat)
