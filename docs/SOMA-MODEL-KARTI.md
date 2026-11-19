> Bu belge NVlabs/motionbricks deposundaki `docs/soma_model.md` dosyasının Türkçe çevirisidir (CC-BY-4.0).

# MotionBricks SOMA Model Kartı

## Model Açıklaması

MotionBricks SOMA, son hareketten ve hedef pozlardan 3B insan hareketi üretir. Bir uygulamanın sürekli animasyona birleştirdiği kısa geçişleri tahmin eder ve karakter sahnede ilerledikçe hedefleri günceller.

[MotionBricks](https://nvlabs.github.io/motionbricks/) modeli, 30 eklemli [SOMA](https://github.com/NVlabs/SOMA-X) iskeleti için hazırlanmış Bones RigPlay 1 (RP1) hareketiyle eğitildi.

Bu kart, MotionBricks v0.2.0'daki SOMA ONNX modellerini kapsar. Her ONNX modeli, `MotionBricks-SOMA-RP-v1` içindeki poz belirteçleyici (tokenizer) ve kod çözücüyü, poz tahmincisini ve kök tahmincisini birleştirir. FP32 ve karma FP16 türevleri aynı tensör arayüzünü paylaşır. FP32, 32 bit kayan nokta duyarlılığı kullanır. Karma FP16, seçili işlemler için 16 bit duyarlılık kullanır.

SafeTensors kontrol noktasının kendi model kartı ve bileşen arayüzleri vardır. [Unitree G1 modelleri](UNITREE-G1-MODEL-KARTI.md) ayrıca belgelenir. Eğitim hattı (training pipeline) bu sürümün kapsamı dışındadır.

Bu model, ticari ve ticari olmayan hareket üretimi uygulamaları için tasarlandı.

### Nereden Başlanır

- Demoyu çalıştırmak için [Kurulum](../README.md#-kurulum) adımlarını, ardından [Çıkarım](#çıkarım) altındaki komutları izleyin.
- Bir uygulamaya üretim eklemek için [çalışma zamanı API'sini](CALISMA-ZAMANI-API.md#motorlar) kullanın.
- SafeTensors kontrol noktasından ONNX modelleri dışa aktarmak için [dışa aktarıcı rehberini](ONNX-DISA-AKTARIM.md) izleyin.

| Özellik | Değer |
| --- | --- |
| Model ailesi | MotionBricks |
| Görev | Pozlara koşullu hareket üretimi ve ara kare üretimi (inbetweening) |
| İskelet | SOMA, 30 eklem |
| Parametreler | Kontrol noktasında yaklaşık 200 milyon |
| Açık poz gösterimi | Kök hareketi, eklem konumları ve 6B dönüşler |
| Model biçimi | ONNX, opset 17 |
| Duyarlılıklar | FP32 ve karma FP16 |
| Toplu iş boyutu | Değişken `N`. Demolar `N=1` kullanır. |
| FPS | 30 |
| Bağlam | Son dört poz |
| Hedefler | En çok dört yerleştirilmiş poz |
| Çıktı uzunluğu | 24 ile 64 ham kare, motor sınır karelerini çıkardıktan sonra 18 ile 58 kare |

## Lisans ve Kullanım Koşulları

Bu modelin kullanımı [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) ile yönetilir. Anlaşmanın koşullarına bağlı olarak ticari kullanıma ve değiştirilmiş modellerin yeniden dağıtımına izin verir.

- Birlikte gelen yazılım [Apache 2.0](https://github.com/NVlabs/motionbricks/blob/main/LICENSE) kullanır.
- Listelenen, Bones kaynaklı insan BVH ve G1 CSV örnekleri [NVIDIA Sample Data Evaluation License](https://github.com/NVlabs/motionbricks/blob/main/assets/motions/LICENSE.txt) kullanır. NVIDIA teknolojilerinin şirket içi değerlendirmesi ve testi içindir. Kapsanan dosyaların tam listesi [örnek bildiriminde](https://github.com/NVlabs/motionbricks/blob/main/assets/motions/README.md).
- Robot varlıkları ve üçüncü taraf bağımlılıklar [kendi lisanslarını](https://github.com/NVlabs/motionbricks/tree/main/licenses) korur.

## Dağıtım Coğrafyası

Küresel.

## Amaçlanan Kullanımlar

**Kullanım durumu:** Animasyon, robotik simülasyonu ve etkileşimli karakter uygulamaları için pozlara koşullu insan hareketi üretmek.

- Hazırlanmış ya da örneklenmiş insan pozları arasında geçişler üretin.
- Uygulamadan hedefler yerleştirerek etkileşimli yürüyüşü/hareketi (lokomosyon) sürün.
- Nesnelere yaklaşmayı ve hedef pozlarla tanımlanan etkileşimleri canlandırın.
- Oyunlarda, medya araçlarında ve simülasyon sahnelerinde karakter animasyonunun ilk örneklerini (prototype) hazırlayın.

Uygulama sahnedeki hedef konumlarını ve yönelimlerini sağlar. Üretilen hareket bir sonraki üretimin bağlamı olabilir.

Bu model, ticari ve ticari olmayan animasyon, robotik ve simülasyon iş akışları kuran geliştiriciler ve araştırmacılar için tasarlandı. Üretilen hareket, fiziksel robot kontrolü için doğrulanmış bir çıktı değildir.

## Kaynaklar

T. Wang, O. Dionne, M. De Ruyter, D. Minor, D. Rempe, K. Zhao, M. Petrovich, Y. Yuan, C. Li, Z. Luo, B. Robison, X. Blackwell, B. Antoniazzi, X. B. Peng, Y. Zhu, and S. Yuen. **MotionBricks: Scalable Real-Time Motions with Modular Latent Generative Model and Smart Primitives**. ACM Transactions on Graphics 45(4), 2026. [DOI 10.1145/3811334](https://doi.org/10.1145/3811334), [arXiv:2604.24833](https://arxiv.org/abs/2604.24833).

## Model Mimarisi

**Mimari türü:** Modüler gizil üretken model.

**Ağ mimarisi:** Kök hareketini ve hareket belirteçlerini tahmin eden dönüştürücüler (transformers), öğrenilmiş bir hareket kod çözücüsüyle birlikte.

| Bileşen | Görev | Parametreler |
| --- | --- | --- |
| Poz VQ-VAE (vektör nicemlemeli değişimsel otokodlayıcı) | Dört pozluk belirteçleri kodlar ve hareketi çözer. Kod kitabında 16 kodluk 5 başlık vardır. | 23,4 milyon |
| Poz modeli | Hareket belirteçlerini tahmin eden maskeli dönüştürücü, 1024 genişlikte 16 katman | 136,5 milyon |
| Kök modeli | Süreyi ve kök yolunu tahmin eder | 40,4 milyon |

Yöntem [MotionBricks makalesinde](https://arxiv.org/abs/2604.24833) anlatılır.

**Temel model:** SOMA bileşenleri RP1 hareketiyle sıfırdan eğitildi. ONNX dışa aktarımı, eğitilmiş bu bileşenleri bir dağıtım grafiğinde birleştirir ve istenen yerde duyarlılığı dönüştürür. Dışa aktarma onları yeniden eğitmez ve yeni bir eğitim veri seti eklemez.

## Girdiler

**Girdi kipi:** Eklem konumları ve 6B dönüşler olarak pozlar, kök hareketi kısıtları ve izin verilen belirteç sayıları. Sekiz koşullandırma yuvası, önce dört bağlam pozunu, ardından dört hedef pozu tutar ve 30 Hz'de örneklenir.

**Girdi biçimi:** Sayısal ONNX tensörleri. Hareket değerleri FP32, maskeler ve belirteç sayıları INT32 kullanır. Girdi tensörü tablosu her parametrenin adını, şeklini ve anlamını tanımlar.

`N` bağımsız istek sayısıdır ve tüm girdiler aynı `N` değerini kullanmalıdır. `soma_n1.json` model bildirimi (manifest), `N=1` için tensörleri tanımlar.

### Poz Gösterimi

Pozlar, SOMA iskeleti üzerinde eklem konumlarını ve 6B dönüşleri tanımlar. Kök eklem `Hips`'tir. Kök öznitelikleri konum, yönelim (heading), hız ve yükseklik kısıtlarını sağlar. SOMA model uzayında Y yukarıdır ve XZ zemin düzlemidir. Konumlar metre kullanır. Her 6B dönüş, bir eklemin SOMA model uzayındaki dönüş matrisinin ilk iki sütununu tutar. Bağlam ve hedef pozlar aynı uzayı paylaşır. [soma_n1.json](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/soma/soma_n1.json) model bildirimi, eklem sırasını ve her tensörün adını, şeklini ve veri türünü (dtype) kaydeder.

### Girdi Tensörleri

| Girdi | Şekil | Tür | Anlam |
| --- | --- | --- | --- |
| `global_root_values` | `[N,8,5]` | FP32 | Kök konumu (XYZ), ardından yönelimin kosinüsü ve sinüsü |
| `has_global_root_values` | `[N,8]` | INT32 | Kök konumu ve yönelimi maskesi |
| `local_root_values` | `[N,8,4]` | FP32 | rad/s cinsinden yönelim değişim hızı, m/s cinsinden X ve Z boyunca kökün zemin hızı ve kök yüksekliği |
| `has_local_root_values` | `[N,8]` | INT32 | Kök hızı ve yüksekliği maskesi |
| `local_poses` | `[N,8,267]` | FP32 | Kök dışındaki 29 eklemin kökün zemin konumuna göre konumları (87 değer), ardından 30 eklemin her biri için bir 6B dönüş (180 değer) |
| `has_local_poses` | `[N,8]` | INT32 | Gövde pozu maskesi |
| `allowed_pred_num_tokens` | `[N,11]` | INT32 | İzin verilen belirteç sayıları, 6 ile 16 |

Yedi girdinin hepsi zorunludur. Maskeler 0 ya da 1 içerir, önce bağlam yuvaları, sonra hedef yuvaları gelir. En az bir belirteç sayısına izin verilmelidir. `allowed_pred_num_tokens` dizisinin `i`. öğesi `6 + i` belirteç sayısına izin verir. Her hareket belirteci dört pozu temsil eder.

## Çıktılar

**Çıktı kipi:** Tahmin edilen ayak temaslarıyla birlikte iskelet hareketi öznitelikleri ve seçilen belirteç sayısı.

**Çıktı biçimi:** Sayısal ONNX tensörleri. Hareket öznitelikleri FP32, belirteç sayısı INT32 kullanır. Çıktı tensörü tablosu her parametrenin adını, şeklini ve anlamını tanımlar.

| Çıktı | Şekil | Tür | Anlam |
| --- | --- | --- | --- |
| `pred_global_motions` | `[N,64,366]` | FP32 | Kare başına, SOMA model uzayında: kök konumu ve yönelimi (5), eklem konumları (87), 6B dönüşler (180), eklem hızları (90) ve ayak temasları (4) |
| `pred_num_tokens` | `[N]` | INT32 | Modelin seçtiği belirteç sayısı |

Baştaki `4 * pred_num_tokens` kare üretilen hareketi içerir. Ham çıktı 24 ile 64 kare arasıdır. Her iki uçtan üçer sınır karesi çıkarıldığında 30 Hz'de 18 ile 58 geçerli kare kalır. Motor bu öznitelikleri kök konumuna ve eklem dönüşlerine çözer.

Her karenin son dört özniteliği sol topuk, sol parmak ucu, sağ topuk ve sağ parmak ucu için ayak temaslarıdır. Eklemleri `LeftFoot`, `LeftToeBase`, `RightFoot` ve `RightToeBase`'dir. Anlamları için [foot_contacts.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/animation/foot_contacts.py) dosyasına bakın.

## Yazılım Entegrasyonu

| Bileşen | Entegrasyon |
| --- | --- |
| Test edilen çıkarım çalışma zamanı | NVIDIA TensorRT 10.8 |
| Uygulama çatısı | Python ve NVIDIA Warp |
| Demo ortamı | Newton |
| Test edilen işletim sistemi | Windows |
| Belgelenen dağıtım donanımı | NVIDIA Ada Lovelace GeForce RTX 4090 |

Uygulama, yerleştirilmiş bağlam ve hedef pozları sağlar. Motor koordinat dönüşümünü yapar ve üretilen hareketi dünya uzayında döndürür.

Tablo test edilen kurulumu gösterir, tam bir uyumluluk matrisi sunmaz. Süre ölçümleri yalnız Windows altındaki GeForce RTX 4090 için verilir. Kontrol noktasından ONNX'e dışa aktarıcı CPU üzerinde çalışır ve bu GPU çıkarım yolundan ayrıdır. Kart, CPU ile GPU çıkarımını karşılaştırmaz.

Temel modellerin ve ince ayarlı modellerin yapay zeka sistemlerine entegrasyonu, güvenli ve etkili dağıtım için kullanım durumuna özgü verilerle ek test gerektirir. V-model yöntemine göre birim ve sistem düzeyinde yinelemeli test ve doğrulama, riskleri azaltmak, teknik ve işlevsel gereksinimleri karşılamak ve dağıtımdan önce güvenlik ve etik standartlarına uyumu sağlamak için gereklidir.

### Çalışma Zamanında Üretim

`MotionBricksSOMAEngine`, [Motorlar](CALISMA-ZAMANI-API.md#motorlar) bölümündeki üretim çağrılarını çalıştırır. `PoseToken` bağlamı ve hedefi yerel eklem dönüşümlerini tutar. Yüklenen BVH kliplerinin kullandığı `AnimationBuffer` türüne üretir.

`create_input()` ve `create_output()` fonksiyonlarına geçirilen iskelet, modelin eklem adlarını içermeli ve kökü sıfır indeksinde tutmalıdır. Motor model eklemlerini bu iskelete eşler. Dünya uzayı SOMA model uzayından farklıysa `model_to_runtime` geçirin.

Motor, çıkarımdan önce son bağlam pozunun zemin konumunu ve yönelimini çıkarır. Kök yüksekliğini korur. Çıkarımdan sonra dünya yerleşimini geri yükler ve her iki uçtan üçer sınır karesi çıkarır. Çıkarım varsayılan olarak son bağlam pozunu SOMA model uzayında +X yönüne çevirir.

## Model Sürümleri

**Model sürümü:** `MotionBricks-SOMA-RP-v1`. Birlikte gelen yazılım sürümü v0.2.0'dır.

| Duyarlılık | ONNX dosyası |
| --- | --- |
| FP32 | `mmm_v6_soma_fp32.onnx` |
| Karma FP16 | `mmm_v6_soma_fp16.onnx` |

Dosyalar [motionbricks/assets/onnx/soma](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/assets/onnx/soma) içinde saklanır.

Hareket girdileri ve çıktıları iki türevde de FP32 kullanır. Maskeler ve belirteç sayıları INT32 kullanır. [Hugging Face'teki SOMA kontrol noktası](https://huggingface.co/nvidia/MotionBricks/tree/main/soma), [dışa aktarıcının](ONNX-DISA-AKTARIM.md) kullandığı SafeTensors ağırlıklarını içerir.

## Eğitim, Test ve Değerlendirme Veri Setleri

### Eğitim Veri Seti

**Ad/Tanımlayıcı:** Bones RigPlay 1 (RP1).

**Veri kipi:** Diğer: 30 Hz'de tek tip 30 eklemli SOMA gösterimine işlenmiş insan iskelet hareket yakalama verisi. Eğitim girdileri metin, görüntü, ses ya da video değil, hareket öznitelikleridir.

**Eğitim verisi boyutu:** Seçilmiş eğitim alt kümesinde 313.526 insan hareketi klibi. Kaynak derlem yaklaşık 350.000 klip ve 700 saat hareket içerir. Kaynak derlemin süresi, seçilmiş eğitim alt kümesinin süresiyle aynı değildir.

**Veri toplama yöntemi:** Otomatik/Sensörler. Bones insan performanslarını bir hareket yakalama stüdyosunda kaydetti ve iskelet hareketi olarak sağladı. NVIDIA bu hareketi tek tip bir SOMA gösterimine işledi. Kaynak performanslar doğrudan SOMA iskeleti üzerinde yakalanmadı.

**Etiketleme yöntemi:** Yakalanan iskelet hareketi için Otomatik/Sensörler, ardından tek tip hareket özniteliklerine otomatik hazırlık. Öğrenme verisi poz ve hareket değerleridir. Kaydedilen çalıştırmalar metin gömmelerini (text embeddings) ya da metin açıklamalarını kullanmaz.

**Özellikler:** Eğitim, tek tip SOMA hareket özniteliklerini, rastgele ilk yönelimleri ve en çok 15 saniyelik rastgele kırpmaları kullanır. Hareket gösterimini kök hareketi ve eklem öznitelikleri sağlar. Metin gömmeleri ve metin çoğaltma (text augmentation) kapalıdır. Kontrol noktasının üç bileşeni de FP32'de eğitildi. Eğitim derlemi bu dışa aktarımlarla birlikte dağıtılmaz.

**Dilsel özellikler:** Hareket özniteliği girdileri için uygulanmaz. İlişkili metin açıklamaları model koşullandırma girdisi değil, veri seti üst verisidir.

### Test Veri Seti

**Ad, boyut ve rol:** Eğitim için 313.526 klip, eğitim sırasında doğrulama ve değerlendirme için ayrı ve seçilmiş 4.881 kliplik bir küme kullandık. Bu kümeler arasında ortak klip yoktur. 4.881 klip, ayrılmış havuzun tamamı değil, ondan yapılmış bir seçimdir. Kart, ayrı ve hiç dokunulmamış bir son test setini, oyunculara göre ayrılmış bir bölmeyi ve seçilen alt kümenin süresini belgelemez.

**Veri toplama yöntemi:** Otomatik/Sensörler, eğitim setiyle aynı Bones stüdyo hareket yakalama kaynağından ve aynı işlenmiş tek tip SOMA gösteriminden.

**Etiketleme yöntemi:** İskelet hareketi için Otomatik/Sensörler, ardından aynı hareket özniteliklerine otomatik hazırlık. Kaydedilen doğrulama yolu metin açıklamalarını kullanmaz.

**Özellikler:** Doğrulama girdileri, eğitim girdileriyle aynı tek tip SOMA gösteriminde 30 Hz iskelet hareketi öznitelikleridir. Dilsel özellikler bu yalnız harekete dayalı girdilere uygulanmaz. Kliplerin ayrık olması tek başına başka veri sızıntısı biçimlerini dışlamaz.

### Değerlendirme Veri Seti ve Sonuçlar

**Ad/Boyut:** Değerlendirme, [Test Veri Seti](#test-veri-seti) altında anlatılan aynı 4.881 kliplik RP1 alt kümesini kullanır. Ek ve bağımsız bir test seti değildir.

**Kıyaslama puanı:** Bu SOMA ONNX modelleri için baştan sona tam bir kıyaslama bu kartın dışında kalır.

**Veri toplama yöntemi:** Otomatik/Sensörler. Bones kaynak hareketi bir stüdyoda yakaladı.

**Etiketleme yöntemi:** Yakalanan iskelet hareketi için Otomatik/Sensörler, ardından tek tip SOMA hareket özniteliklerine otomatik hazırlık.

**Özellikler:** Metin koşullu etiketler içermeyen, 30 Hz SOMA iskelet hareketi öznitelikleri. Dilsel özellikler bu girdilere uygulanmaz. Alt kümenin süresi kartta yer almaz.

**Kaydedilen sonuçlar:** Eğitim sırasında belirteçleyici yeniden kurulumunu ve ayak kaymasını (foot skating), kök tahmincisi doğrulamasını ve poz tahmincisi eğitim kaybını ölçtü. Poz tahmincisi için tam doğrulama sonucu kartın dışında kalır. Bu ölçümler dışa aktarılan ONNX modellerini değerlendirmez.

ONNX modelleri [Yazılım Entegrasyonu](#yazılım-entegrasyonu) altında listelenen kurulumda test edildi. TensorRT planlarının farklı toplu iş boyutlarındaki GPU süresi ve iş hacmi [Çıkarım performansı](ONNX-CIKARIM.md#çıkarım-performansı) bölümündedir. Nicel hareket kalitesi ve FP32 ile karma FP16 arasındaki kalite karşılaştırması raporlanmaz. [MotionBricks makalesi](https://arxiv.org/abs/2604.24833), bu SOMA ONNX modellerinin bir kıyaslamasını değil, kendi modelleri ve veri setleri üzerindeki yöntem düzeyindeki deneyleri anlatır.

## Çıkarım

Varsayılan FP32'dir. Demoyu iki duyarlılıktan biriyle çalıştırın:

```sh
uv run python -m app.scene_preview --model-precision fp32
uv run python -m app.scene_preview --model-precision fp16
```

SOMA TensorRT yapılandırması, uygun FP32 işlemleri için TF32'ye izin verir.

Karma FP16 ONNX modeli, sinirsel tahminciler ve hareket kod çözücüsü için FP16 kullanır. Öznitelik normalleştirme, hareket geometrisi, LayerNormalization, Softmax ve Resize için FP32'yi korur. Demolar `N=1` için bir TensorRT planı derler.

Yuvarlama hareket belirteçlerinin seçimini etkilediği için karma FP16 ve FP32 farklı hareket üretebilir.

**Hızlandırma motoru:** NVIDIA TensorRT 10.8.

**Test donanımı kurulumu:** Windows üzerinde NVIDIA Ada Lovelace GeForce RTX 4090.

Dışa aktarıcı, kontrol noktası bütünlüğünü, ONNX grafiğinin geçerliliğini ve beklenen tensör arayüzünü denetler. Bu yapısal denetimler tam güvenlik kapsamını, güvenli fiziksel davranışı ya da duyarlılıklar ve arka uçlar arasında eşdeğer hareket kalitesini kanıtlamaz.

## Sınırlamalar

- Demolar, kullanıcılara sunulan aynı toplu ONNX modeliyle her seferinde bir istek çalıştırır. Bağlam ve hedef belirteçlerinin her biri en çok dört poz tutar, bu yüzden daha uzun hareket art arda üretim gerektirir. Bkz. [toplu işleme](ONNX-CIKARIM.md).
- Model SOMA eklem düzenini ve oranlarını kullanır. Başka iskeletler uygun bir gösterim ya da yeniden hedefleme (retargeting) adımı gerektirir.
- Koşullandırma pozlardan ve hareket kısıtlarından gelir. Metin istemleri ve sahne geometrisi bu modelin girdi arayüzünün dışındadır. Hedefleri uygulama yerleştirir ve engelleri uygulama yönetir.
- Model, eğitim dağılımı içindeki hedef pozlarla en iyi sonucu verir. Dağılım dışındaki pozlar daha düşük kaliteli hareket üretebilir.
- Üretilen hareket, duyarlılığa ve çıkarım çalışma zamanına göre değişebilir. Üretilen hareket bağlam olarak geri beslendiğinde farklar birikebilir.
- Kinematik hareket üretimi tek başına dinamik uygulanabilirliği, fiziksel robot izlemeyi, çarpışmadan kaçınmayı ya da güvenli teması sağlamaz. Fiziksel çalıştırma ayrı bir kontrolcü ve uygulamaya özgü doğrulama gerektirir.

## Etik Değerlendirmeler

NVIDIA, Güvenilir Yapay Zekanın (Trustworthy AI) ortak bir sorumluluk olduğuna inanır ve çok çeşitli yapay zeka uygulamalarında geliştirmeyi mümkün kılmak için politikalar ve uygulamalar oluşturmuştur. Geliştiriciler, bu modelin ilgili sektör ve kullanım durumunun gereksinimlerini karşıladığından ve öngörülmeyen ürün kötüye kullanımlarını ele aldığından emin olmak için kendi şirket içi model ekipleriyle çalışmalıdır.

Daha ayrıntılı bilgi için aşağıdaki [Yanlılık](#yanlılık-alt-kartı-bias-subcard), [Açıklanabilirlik](#açıklanabilirlik-alt-kartı-explainability-subcard), [Gizlilik](#gizlilik-alt-kartı-privacy-subcard) ve [Emniyet ve Güvenlik](#emniyet-ve-güvenlik-alt-kartı-safety-and-security-subcard) alt kartlarına bakın.

Model kalitesini, riskleri, güvenlik açıklarını ya da NVIDIA yapay zeka kaygılarını [buradan](https://www.nvidia.com/en-us/support/submit-security-vulnerability/) bildirin.

## Yanlılık Alt Kartı (Bias Subcard)

| Alan | Yanıt |
| --- | --- |
| Model tasarımında ve testinde olumsuz etkilenen grupların katılımına ilişkin değerlendirmeler | Raporlanmaz. |
| İstenmeyen yanlılığı azaltmak için alınan önlemler | Kart, ölçülmüş belirli bir azaltma yöntemi raporlamaz. Uygulamalar hareket kalitesini kendi kullanıcıları ve hareket koşulları için değerlendirmelidir. |
| Ölçüldüyse yanlılık ölçütü | Demografik, vücut ve hareket kapsamı karşılaştırmaları raporlanmaz. |

Sabit SOMA gösterimi, vücut oranları ya da hareket kategorileri arasında eşdeğer kalite sağlamaz. Duyarlılık dönüşümü, alt grup performansına dair yeni bir kanıt oluşturmaz.

## Açıklanabilirlik Alt Kartı (Explainability Subcard)

| Alan | Yanıt |
| --- | --- |
| Amaçlanan görev/alan | Pozlara koşullu insan iskelet hareketi üretimi ve ara kare üretimi. |
| Model türü | Dönüştürücü tahminciler ile öğrenilmiş poz belirteçleyici ve kod çözücü içeren modüler gizil üretken model. |
| Amaçlanan kullanıcılar | Animasyon, robotik ve simülasyon iş akışları kuran geliştiriciler ve araştırmacılar. |
| Çıktı | İki çıktılı ONNX arayüzü üzerinden toplu iskelet hareketi öznitelikleri ve seçilen belirteç sayısı. Motor öznitelikleri kök konumuna ve eklem dönüşlerine çözer. |
| Modelin nasıl çalıştığı | Son hareket ve hedef kısıtları, kök ve poz tahmincilerini koşullandırır. Bir belirteçleyici hareketi ayrık kodlar olarak gösterir, bir kod çözücü de iskelet hareketini yeniden kurar. Birleşik grafik, çıkarım öznitelik dönüşümlerini içerir. Uygulamalar çıktıyı bağlam olarak yeniden kullanabilir. |
| Karşılaştırılabilir sonuçlar için test edilen gruplar | Demografik alt grup karşılaştırmaları raporlanmaz. |
| Teknik sınırlamalar ve azaltma | Sabit iskelet, kısa bağlam, sahne geometrisinin girdi dışında kalması ve arka uçlar ile duyarlılıklar arasında değişkenlik. Uygulamaya özgü hareketi, hedef yerleşimini ve her türlü fiziksel çalıştırmayı ayrıca doğrulayın. |
| Belirlenmiş NVIDIA kalite standartlarını karşıladığı doğrulandı mı | Evet |
| Performans ölçütleri | Mevcut ölçümler ve sınırları için [Değerlendirme Veri Seti ve Sonuçlar](#değerlendirme-veri-seti-ve-sonuçlar) bölümüne bakın. |
| Bilinen olası riskler | Poz/temas bozulmaları, dağılım dışı kısıtlar altında istenmeyen hareket ve üretilen pozlar doğrulanmış robot komutları sayılırsa güvensiz fiziksel yorum. |
| Lisanslama | Bu modelin kullanımı [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) ile yönetilir. Ayrı yazılım ve örnek veri koşulları için [Lisans ve Kullanım Koşulları](#lisans-ve-kullanım-koşulları) bölümüne bakın. |

## Gizlilik Alt Kartı (Privacy Subcard)

Bu modeli eğitmek için kullanılan veri setleri yayımdan önce incelendi. Daha fazla bilgi için [NVIDIA Gizlilik Politikası'nı](https://www.nvidia.com/en-us/about-nvidia/privacy-policy/) inceleyin.

## Emniyet ve Güvenlik Alt Kartı (Safety and Security Subcard)

| Alan | Yanıt |
| --- | --- |
| Modelin uygulama alanları | Endüstri/Makine ve Robotik, Medya ve Eğlence. Üretilen hareket, fiziksel robot kontrolü için doğrulanmış bir çıktı değildir. |
| Yaşamsal etki | Üretilen iskelet hareketi, doğrulanmış bir fiziksel kontrolcü değildir. Robot komutu olarak kötüye kullanılması fiziksel risk yaratabilir. |
| Kullanım durumu kısıtları | Bu modelin kullanımı [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) ile yönetilir. Ayrı yazılım ve örnek veri koşulları için [Lisans ve Kullanım Koşulları](#lisans-ve-kullanım-koşulları) bölümüne bakın. |
| Model ve veri seti kısıtları | Bu ONNX dosyaları RP1 eğitim derlemini dağıtmaz. Veri setine erişim, model ağırlıklarından ayrı olarak yönetilir. |

Üretilen hareketi amaçlanan uygulamada değerlendirin. Fiziksel çalıştırma ayrı bir izleme ve güvenlik sistemi gerektirir. Yapıt (artifact) bütünlüğü ve grafik denetimleri tam güvenliği ya da sonraki aşamada güvenli davranışı kanıtlamaz.

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
