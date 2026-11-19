> Bu belge NVlabs/motionbricks deposundaki `docs/unitree_g1_model.md` dosyasının Türkçe çevirisidir (CC-BY-4.0).

# MotionBricks Unitree G1 Model Kartı

## Model Açıklaması

MotionBricks Unitree G1, son pozlardan ve yerleştirilmiş hedef pozlardan robot hareketi üretir. Bir uygulamanın sürekli referans harekete birleştirebileceği kısa geçişler üretir.

[MotionBricks](https://nvlabs.github.io/motionbricks/) modeli, 34 eklemli bir Unitree G1 iskeletine yeniden hedeflenmiş (retargeted) hareket yakalama verisiyle eğitildi. [GR00T Whole-Body Control](https://github.com/NVlabs/GR00T-WholeBodyControl/tree/main/motionbricks) içinde yayımlanan özgün ağırlıkları yeniden kullanır.

Bu kart, MotionBricks v0.2.0'daki G1 ONNX modellerini kapsar. Her ONNX modeli, `MotionBricks-G1-RP-v1` içindeki poz belirteçleyici (tokenizer) ve kod çözücüyü, poz tahmincisini ve kök tahmincisini birleştirir. G1 eklem koordinatlarını alır ve döndürür, iskelet özniteliklerine dönüşümü model içinde yapar. FP32 ve karma FP16 türevleri aynı tensör arayüzünü paylaşır. FP32, 32 bit kayan nokta duyarlılığı kullanır. Karma FP16, seçili işlemler için 16 bit duyarlılık kullanır.

SafeTensors kontrol noktasının kendi model kartı ve bileşen arayüzleri vardır. [SOMA modelleri](SOMA-MODEL-KARTI.md) ayrıca belgelenir. Eğitim hattı (training pipeline) bu sürümün kapsamı dışındadır.

Bu model, ticari ve ticari olmayan hareket üretimi uygulamaları için tasarlandı.

### Nereden Başlanır

- G1 demosunu çalıştırmak için [Kurulum](../README.md#-kurulum) adımlarını, ardından [Çıkarım](#çıkarım) altındaki komutları izleyin.
- Bir uygulamaya üretim eklemek için [çalışma zamanı API'sini](CALISMA-ZAMANI-API.md#motorlar) kullanın.
- SafeTensors kontrol noktasından ONNX modelleri dışa aktarmak için [dışa aktarıcı rehberini](ONNX-DISA-AKTARIM.md) izleyin.

| Özellik | Değer |
| --- | --- |
| Model ailesi | MotionBricks |
| Görev | Pozlara koşullu robot hareketi üretimi ve ara kare üretimi (inbetweening) |
| Robot | Unitree G1, 29 eklem açısı ve yüzen kök (floating root) |
| İskelet | Unitree G1, 34 model eklemi |
| Parametreler | Kontrol noktasında yaklaşık 195 milyon |
| Açık poz gösterimi | Kök XYZ, XYZW dördey (quaternion) ve 29 eklem açısı |
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

**Kullanım durumu:** Animasyon, robotik simülasyonu ve sonraki aşama kontrol deneyleri için pozlara koşullu G1 referans hareketi üretmek.

- G1 pozları arasında ya da hareket kliplerinden örneklenen pozlar arasında geçişler üretin.
- Hedef pozlar yerleştirerek Newton içinde yürüyüşü/hareketi (lokomosyon) ve yön değiştirmeyi önizleyin.
- Robot kontrol deneyleri için kinematik referans hareket üretin.
- Önceki çıktıyı bir sonraki bağlam olarak kullanan art arda üretimi keşfedin.

Çıktı istenen pozları tanımlar. Bu pozları robot dinamiği ve fiziksel kısıtlar altında izlemek, sonraki aşamadaki bir kontrolcünün işidir.

Bu model, ticari ve ticari olmayan animasyon, robotik ve simülasyon iş akışları kuran geliştiriciler ve araştırmacılar için tasarlandı. Üretilen hareket, fiziksel robot kontrolü için doğrulanmış bir çıktı değildir.

## Kaynaklar

T. Wang, O. Dionne, M. De Ruyter, D. Minor, D. Rempe, K. Zhao, M. Petrovich, Y. Yuan, C. Li, Z. Luo, B. Robison, X. Blackwell, B. Antoniazzi, X. B. Peng, Y. Zhu, and S. Yuen. **MotionBricks: Scalable Real-Time Motions with Modular Latent Generative Model and Smart Primitives**. ACM Transactions on Graphics 45(4), 2026. [DOI 10.1145/3811334](https://doi.org/10.1145/3811334), [arXiv:2604.24833](https://arxiv.org/abs/2604.24833).

## Model Mimarisi

**Mimari türü:** Modüler gizil üretken model.

**Ağ mimarisi:** Kök hareketini ve hareket belirteçlerini tahmin eden dönüştürücüler (transformers), öğrenilmiş bir hareket kod çözücüsüyle birlikte.

| Bileşen | Görev | Parametreler |
| --- | --- | --- |
| Poz VQ-VAE (vektör nicemlemeli değişimsel otokodlayıcı) | Dört pozluk belirteçleri kodlar ve hareketi çözer. Kod kitabında 10 kodluk 8 başlık vardır. | 23,8 milyon |
| Poz modeli | Hareket belirteçlerini tahmin eden maskeli dönüştürücü, 1024 genişlikte 16 katman | 136,6 milyon |
| Kök modeli | Süreyi ve kök yolunu tahmin eder | 34,1 milyon |

ONNX modeli, G1 eklem koordinatları ile 34 eklemli iskeletinin dönüşümleri arasında çeviri yapar. Yöntem [MotionBricks makalesinde](https://arxiv.org/abs/2604.24833) anlatılır.

**Temel model:** Özgün G1 bileşenleri, G1'e yeniden hedeflenmiş RP1 hareketiyle sıfırdan eğitildi. Bu sürüm o ağırlıkları yeniden eğitmeden kullanır. ONNX dışa aktarımı bileşenleri birleştirir ve eklem koordinatı arayüzünü ekler.

## Girdiler

**Girdi kipi:** G1 eklem koordinatları, koşullandırma maskeleri ve izin verilen belirteç sayıları. Bağlam ve hedef pozlar G1 model uzayındadır ve 30 Hz'de örneklenir.

**Girdi biçimi:** Sayısal ONNX tensörleri. Eklem koordinatları FP32, maskeler ve belirteç sayıları INT32 kullanır. Girdi tensörü tablosu her parametrenin adını, şeklini ve anlamını tanımlar.

`N` bağımsız istek sayısıdır ve tüm girdiler aynı `N` değerini kullanmalıdır. `unitree_g1_n1.json` model bildirimi (manifest), `N=1` için tensörleri tanımlar.

### Poz Gösterimi

Her poz 36 FP32 eklem koordinatı içerir:

```text
[root_x, root_y, root_z, root_qx, root_qy, root_qz, root_qw, 29 joint angles]
```

G1 model uzayında Z yukarı, +X ileridir. Konumlar metre, eklem açıları radyan kullanır. Kök eklem leğen kemiğidir (pelvis). Dördey bileşenleri XYZW sırasını kullanır. Eklem açısı sırasını [unitree_g1_n1.json](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/unitree_g1/unitree_g1_n1.json) içindeki `joint_names` tanımlar.

### Girdi Tensörleri

| Girdi | Şekil | Tür | Anlam |
| --- | --- | --- | --- |
| `context_qpos` | `[N,4,36]` | FP32 | 30 Hz'de son dört bağlam pozu |
| `target_qpos` | `[N,4,36]` | FP32 | Yerleştirilmiş dört hedef poz |
| `has_global_root_values` | `[N,8]` | INT32 | Kök konumu ve yönelimi (heading) maskesi |
| `has_local_root_values` | `[N,8]` | INT32 | Kök hızı ve yüksekliği maskesi |
| `has_local_poses` | `[N,8]` | INT32 | Gövde pozu maskesi |
| `allowed_pred_num_tokens` | `[N,11]` | INT32 | İzin verilen belirteç sayıları, 6 ile 16 |

Altı girdinin hepsi zorunludur. Maskeler 0 ya da 1 içerir, önce bağlam yuvaları, sonra hedef yuvaları gelir. En az bir belirteç sayısına izin verilmelidir. `allowed_pred_num_tokens` dizisinin `i`. öğesi `6 + i` belirteç sayısına izin verir. Her hareket belirteci dört pozu temsil eder.

Poz değerleri sonlu, kök dördeyleri sıfırdan farklı olmalıdır. Bu kural maskelenmiş yuvalar için de geçerlidir.

## Çıktılar

**Çıktı kipi:** G1 eklem koordinatları, tahmin edilen ayak temasları ve seçilen belirteç sayısı.

**Çıktı biçimi:** Sayısal ONNX tensörleri. Eklem koordinatları ve ayak temasları FP32, belirteç sayısı INT32 kullanır. Çıktı tensörü tablosu her parametrenin adını, şeklini ve anlamını tanımlar.

| Çıktı | Şekil | Tür | Anlam |
| --- | --- | --- | --- |
| `pred_qpos` | `[N,64,36]` | FP32 | Girdilerle aynı uzayda çözülmüş eklem koordinatları |
| `pred_num_tokens` | `[N]` | INT32 | Modelin seçtiği belirteç sayısı |
| `pred_foot_contacts` | `[N,64,4]` | FP32 | `pred_qpos` içindeki her kare için tahmin edilen ayak temasları |

Baştaki `4 * pred_num_tokens` kare üretilen hareketi içerir. Ham çıktı 24 ile 64 kare arasıdır. Her iki uçtan üçer sınır karesi çıkarıldığında 30 Hz'de 18 ile 58 geçerli kare kalır.

`pred_foot_contacts` içindeki her kare sol topuk, sol parmak ucu, sağ topuk ve sağ parmak ucu için dört ayak teması tutar. Topuk temas noktaları ayak bileği yuvarlanma (ankle roll) eklemlerindedir. Her parmak ucu temas noktası, ayak bileği yuvarlanma bağlantısının (ankle roll link) yerel uzayında, kendi ayak bileği yuvarlanma ekleminin 0,14 m önünde ve 0,035 m altında durur. Anlamları için [foot_contacts.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/animation/foot_contacts.py) dosyasına bakın.

Ayak temasları model tahminleridir. Temas sensörü okuması değildir ve fiziksel robotta kararlı teması güvence altına almaz.

## Yazılım Entegrasyonu

| Bileşen | Entegrasyon |
| --- | --- |
| Test edilen çıkarım çalışma zamanı | NVIDIA TensorRT 10.8 |
| Uygulama çatısı | Python ve NVIDIA Warp |
| Demo ortamı | Newton |
| Test edilen işletim sistemi | Windows |
| Belgelenen dağıtım donanımı | NVIDIA Ada Lovelace GeForce RTX 4090 |

Uygulama, yerleştirilmiş bağlam ve hedef pozları sağlar. Motor onları çıkarım için hizalar ve üretilen hareketi dünya uzayında döndürür.

Tablo test edilen kurulumu gösterir, tam bir uyumluluk matrisi sunmaz. Süre ölçümleri yalnız Windows altındaki GeForce RTX 4090 için verilir. Kontrol noktasından ONNX'e dışa aktarıcı CPU üzerinde çalışır ve bu GPU çıkarım yolundan ayrıdır. Kart, CPU ile GPU çıkarımını karşılaştırmaz.

Temel modellerin ve ince ayarlı modellerin yapay zeka sistemlerine entegrasyonu, güvenli ve etkili dağıtım için kullanım durumuna özgü verilerle ek test gerektirir. V-model yöntemine göre birim ve sistem düzeyinde yinelemeli test ve doğrulama, riskleri azaltmak, teknik ve işlevsel gereksinimleri karşılamak ve dağıtımdan önce güvenlik ve etik standartlarına uyumu sağlamak için gereklidir.

### Çalışma Zamanında Üretim

`MotionBricksUnitreeG1Engine`, [Motorlar](CALISMA-ZAMANI-API.md#motorlar) bölümündeki üretim çağrılarını çalıştırır. `DOFToken` bağlamı ve hedefi, eklem koordinatlarını Warp FP32 dizileri olarak tutar. Yüklenen CSV kliplerinin kullandığı `DOFAnimationBuffer` türüne üretir.

Çağıran taraf, doğrudan bir robotun Newton `model.joint_q` görünümüne örnekleyebilir, ardından oynatma için `newton.eval_fk` çağırabilir. Hiçbir eklem adı yeniden eşlenmez. Dünya uzayı G1 model uzayından farklıysa `model_to_runtime` geçirin.

Motor, çıkarımdan önce son bağlam pozunun zemin konumunu ve yönelimini çıkarır. Kök yüksekliğini ve eklem açılarını korur. Çıkarımdan sonra dünya yerleşimini geri yükler ve her iki uçtan üçer sınır karesi çıkarır. Çıkarım varsayılan olarak son bağlam pozunu G1 model uzayında +X yönüne çevirir.

Motor, çözülen kök konumunu komşu kareler üzerinden yumuşatır. Bu işlem modelin çıktısındaki kareden kareye titremeyi giderir ve yürüyüş düzenini korur.

## Model Sürümleri

**Model sürümü:** `MotionBricks-G1-RP-v1`. Birlikte gelen yazılım sürümü v0.2.0'dır.

| Duyarlılık | ONNX dosyası |
| --- | --- |
| FP32 | `mmm_v6_g1_fp32.onnx` |
| Karma FP16 | `mmm_v6_g1_fp16.onnx` |

Dosyalar [motionbricks/assets/onnx/unitree_g1](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/assets/onnx/unitree_g1) içinde saklanır.

Hareket girdileri ve çıktıları iki türevde de FP32 kullanır. Maskeler ve belirteç sayıları INT32 kullanır. [Hugging Face'teki G1 kontrol noktası](https://huggingface.co/nvidia/MotionBricks/tree/main/g1), [dışa aktarıcının](ONNX-DISA-AKTARIM.md) kullandığı SafeTensors ağırlıklarını içerir.

## Eğitim, Test ve Değerlendirme Veri Setleri

### Eğitim Veri Seti

**Ad/Tanımlayıcı:** Bones RigPlay 1 (RP1).

**Veri kipi:** Diğer: 30 Hz'de 34 eklemli Unitree G1 gösterimine yeniden hedeflenmiş insan hareket yakalama verisi. Model koşullandırma için metin açıklamaları yerine iskelet hareket özniteliklerini kullanır.

**Eğitim verisi boyutu:** Seçilmiş 313.526 eğitim klibi. Kaynak derlem yaklaşık 350.000 RP1 klibi ve 700 saat hareket içerir. 700 saat yalnız kaynak derlemi tanımlar.

**Veri toplama yöntemi:** Otomatik/Sensörler. Bones insan performanslarını bir stüdyoda yakaladı ve iskelet hareketi olarak sağladı. NVIDIA bu hareketi [SOMA Retargeter](https://github.com/NVIDIA/soma-retargeter) ile G1'e yeniden hedefledi, ardından model özniteliklerine dönüştürdü. Veri bu yolla hazırlanır, robot üzerinde kaydedilmez.

**Etiketleme yöntemi:** Yakalanan iskelet hareketi için Otomatik/Sensörler, ardından otomatik yeniden hedefleme ve öznitelik hazırlığı. Bu yalnız harekete dayalı modelde metin açıklamaları koşullandırma girdisi olarak kullanılmaz.

**Özellikler:** Pozlara koşullu robot üretimi için zamanla değişen, yeniden hedeflenmiş iskelet hareketi.

**Dilsel özellikler:** Hareket özniteliği girdileri için uygulanmaz. İlişkili metin açıklamaları veri seti üst verisidir ve model koşullandırmasına girmez.

ONNX dönüşümü özgün G1 ağırlıklarını yeniden eğitmez ve yeni bir eğitim veri seti eklemez.

### Test Veri Seti

**Ad, boyut ve rol:** Eğitim için 313.526 klip, eğitim sırasında doğrulama ve değerlendirme için ayrı ve seçilmiş 4.881 kliplik bir küme kullandık. Bu kümeler arasında ortak klip yoktur. 4.881 klip, ayrılmış havuzdan yapılmış bir seçimdir. Kart, ayrı ve hiç dokunulmamış bir son test setini, oyunculara göre ayrılmış bir bölmeyi ve seçilen alt kümenin süresini belgelemez.

**Veri toplama yöntemi:** Otomatik/Sensörler, eğitim setiyle aynı Bones stüdyo hareket yakalama kaynağından. NVIDIA insan hareketini G1 gösterimine yeniden hedefledi.

**Etiketleme yöntemi:** Yakalanan iskelet hareketi için Otomatik/Sensörler, ardından otomatik yeniden hedefleme ve öznitelik hazırlığı.

**Özellikler:** Doğrulama girdileri, 34 eklemli G1 gösterimi üzerinde 30 Hz iskelet hareketi öznitelikleridir. Metin açıklamaları seçim üst verisi olarak kalır ve koşullandırmaya girmez. Dilsel özellikler bu yalnız harekete dayalı girdilere uygulanmaz. Kliplerin ayrık olması tek başına başka veri sızıntısı biçimlerini dışlamaz.

### Değerlendirme Veri Seti ve Sonuçlar

**Ad/Boyut:** Değerlendirme, [Test Veri Seti](#test-veri-seti) altında anlatılan aynı 4.881 kliplik RP1 alt kümesini kullanır. Ek ve bağımsız bir test seti değildir.

**Kıyaslama puanı:** Bu G1 ONNX modelleri için tam bir nicel değerlendirme bu kartın dışında kalır.

**Veri toplama yöntemi:** Otomatik/Sensörler, G1'e yeniden hedeflenmiş aynı stüdyo kaydı insan hareketiyle.

**Etiketleme yöntemi:** Yakalanan iskelet hareketi için Otomatik/Sensörler, ardından otomatik yeniden hedefleme ve öznitelik hazırlığı.

**Özellikler:** Değerlendirme, eğitimle aynı 30 Hz G1 iskelet hareketi gösterimini kullanır. Metin açıklamaları seçim üst verisi olarak kalır ve koşullandırmaya girmez. Dilsel özellikler yalnız harekete dayalı girdilere uygulanmaz. Alt kümenin süresi kartta yer almaz.

**Kaydedilen sonuçlar:** Özgün G1 bileşenlerinin eğitim ölçümleri kartta raporlanmaz.

ONNX modelleri [Yazılım Entegrasyonu](#yazılım-entegrasyonu) altında listelenen kurulumda test edildi. TensorRT planlarının farklı toplu iş boyutlarındaki GPU süresi ve iş hacmi [Çıkarım performansı](ONNX-CIKARIM.md#çıkarım-performansı) bölümündedir. Bellek kullanımı ve demografik karşılaştırmalar raporlanmaz. FP32 ile karma FP16 arasındaki kalite karşılaştırması da raporun dışındadır.

[MotionBricks makalesi](https://arxiv.org/abs/2604.24833), kendi modelleri ve veri setleri üzerindeki yöntem düzeyindeki deneyleri anlatır. Bu Newton entegrasyonunun fiziksel robotta izleme (tracking) değerlendirmesi bu sürümün kapsamı dışındadır.

## Çıkarım

Varsayılan FP32'dir. Sahne Önizleme'de G1 gym sahnesini iki duyarlılıktan biriyle çalıştırın:

```sh
uv run python -m app.scene_preview --scene assets/scenes/g1_gym.scene.json --model-precision fp32
uv run python -m app.scene_preview --scene assets/scenes/g1_gym.scene.json --model-precision fp16
```

G1 TensorRT yapılandırması iki duyarlılıkta da TensorFloat-32'yi (TF32) kapatır. TF32 açıkken bir boyutlu toplu iş yalnız yaklaşık yüzde 2 hızlanır, ama aynı girdilerde eklem açıları birkaç derece farklılaşabilir. Farklar zincirlenen üretimler boyunca büyür. [Çıkarım performansı](ONNX-CIKARIM.md#çıkarım-performansı) bölümünün gösterdiği gibi daha büyük toplu işler daha çok kazanır. Açmak için `TensorRTBuildOptions(enable_tf32=True)` geçirin.

Karma FP16 ONNX modeli, sinirsel tahminciler ve hareket kod çözücüsü için FP16 kullanır. Öznitelik normalleştirme, hareket geometrisi, LayerNormalization, Softmax ve Resize için FP32'yi korur. G1 ileri kinematiği, hareket entegrasyonu ve dördeyler ile eklem açıları arasındaki dönüşüm de FP32'de kalır. Demolar `N=1` için bir TensorRT planı derler.

Yuvarlama hareket belirteçlerinin seçimini etkilediği için karma FP16 ve FP32 farklı hareket üretebilir.

**Hızlandırma motoru:** NVIDIA TensorRT 10.8.

**Test donanımı kurulumu:** Windows üzerinde NVIDIA Ada Lovelace GeForce RTX 4090.

Dışa aktarıcı, kontrol noktası bütünlüğünü, ONNX grafiğinin geçerliliğini ve beklenen tensör arayüzünü denetler. Bu yapısal denetimler tam güvenlik kapsamını, güvenli fiziksel davranışı ya da duyarlılıklar ve arka uçlar arasında eşdeğer hareket kalitesini kanıtlamaz.

## Sınırlamalar

- Demolar, kullanıcılara sunulan aynı toplu ONNX modeliyle her seferinde bir istek çalıştırır. Bağlam ve hedef belirteçlerinin her biri en çok dört poz tutar, bu yüzden daha uzun hareket art arda üretim gerektirir. Bkz. [toplu işleme](ONNX-CIKARIM.md).
- Eklem koordinatı arayüzü, birlikte gelen G1 kinematiğine ve eklem sırasına özgüdür. Başka bir robot ya da eklem düzeni, eşleşen bir dönüşüm ve model gerektirir.
- Koşullandırma pozlardan ve hareket kısıtlarından gelir. Metin istemleri ve sahne geometrisi bu modelin girdi arayüzünün dışındadır. Hedefleri uygulama yerleştirir ve engelleri uygulama yönetir.
- Model, eğitim dağılımı içindeki hedef pozlarla en iyi sonucu verir. Dağılım dışındaki pozlar daha düşük kaliteli hareket üretebilir.
- Model kinematik pozlar üretir. Dinamik denge, eyleyici (actuator) sınırları, çarpışmadan kaçınma ve temas izleme sonraki aşamada bir kontrolcü gerektirir.
- Üretilen hareket, duyarlılığa ve çıkarım çalışma zamanına göre değişebilir. Üretilen hareket bağlam olarak geri beslendiğinde farklar birikebilir.
- Bu Newton entegrasyonunun fiziksel robotta izleme değerlendirmesi bu sürümün kapsamı dışındadır.

## Etik Değerlendirmeler

NVIDIA, Güvenilir Yapay Zekanın (Trustworthy AI) ortak bir sorumluluk olduğuna inanır ve çok çeşitli yapay zeka uygulamalarında geliştirmeyi mümkün kılmak için politikalar ve uygulamalar oluşturmuştur. Geliştiriciler, bu modelin ilgili sektör ve kullanım durumunun gereksinimlerini karşıladığından ve öngörülmeyen ürün kötüye kullanımlarını ele aldığından emin olmak için kendi şirket içi model ekipleriyle çalışmalıdır.

Daha fazla ayrıntı için aşağıdaki [Yanlılık](#yanlılık-alt-kartı-bias-subcard), [Açıklanabilirlik](#açıklanabilirlik-alt-kartı-explainability-subcard), [Gizlilik](#gizlilik-alt-kartı-privacy-subcard) ve [Emniyet ve Güvenlik](#emniyet-ve-güvenlik-alt-kartı-safety-and-security-subcard) alt kartlarına bakın.

Model kalitesini, riskleri, güvenlik açıklarını ya da NVIDIA yapay zeka kaygılarını [buradan](https://www.nvidia.com/en-us/support/submit-security-vulnerability/) bildirin.

## Yanlılık Alt Kartı (Bias Subcard)

| Alan | Yanıt |
| --- | --- |
| Model tasarımında ve testinde olumsuz etkilenen grupların katılımına ilişkin değerlendirmeler | Kart, özgün G1 kontrol noktaları ve bu dışa aktarımlar için ayrı katılım önlemleri raporlamaz. |
| İstenmeyen yanlılığı azaltmak için alınan önlemler | Kart, ölçülmüş bir yanlılık azaltma yöntemi raporlamaz. Hareket davranışını hedef uygulamanın kullanıcıları, hedefleri ve hareket koşulları üzerinde değerlendirin. |
| Ölçüldüyse yanlılık ölçütü | Bu dışa aktarımlar için demografik alt grup ya da hareket kapsamı yanlılık ölçütü raporlanmaz. |

Hareketi tek bir robot iskeletine yeniden hedeflemek, kaynak verideki insan davranışları ya da hareket kategorileri arasında eşit kalite sağlamaz.

## Açıklanabilirlik Alt Kartı (Explainability Subcard)

| Alan | Yanıt |
| --- | --- |
| Amaçlanan görev/alan | Pozlara koşullu Unitree G1 hareket üretimi ve ara kare üretimi. |
| Model türü | Dönüştürücü tahminciler ile öğrenilmiş bir belirteçleyici ve kod çözücü içeren modüler gizil üretken model. |
| Amaçlanan kullanıcılar | Robot hareketi önizlemeleri, etkileşimli poz geçişleri ve referans hareket iş akışları kuran geliştiriciler ve araştırmacılar. |
| Çıktı | Eklem koordinatları (kök konumu, XYZW dördeyi ve 29 eklem açısı), kare başına dört ayak teması ve belirteç sayısı. Bunlar kinematik referanslardır. |
| Modelin nasıl çalıştığı | ONNX modeli eklem koordinatlarını iskelet özniteliklerine dönüştürür. Kök ve poz tahmincileri, son ve hedef pozlara koşullu olarak hareket kodları ve kök hareketi üretir. Bir kod çözücü hareketi yeniden kurar ve ONNX modeli onu eklem koordinatlarına geri çevirir. |
| Karşılaştırılabilir sonuçlar için test edilen gruplar | Demografik alt grup karşılaştırmaları raporlanmaz. |
| Teknik sınırlamalar ve azaltma | Sabit G1 kinematiği, kısa bağlam, sahne geometrisinin girdi dışında kalması ve arka uçlar ile duyarlılıklar arasında değişkenlik. Hedefleri, hareket kalitesini, fiziksel sınırları ve sonraki aşamadaki izlemeyi hedef uygulama için doğrulayın. |
| Belirlenmiş NVIDIA kalite standartlarını karşıladığı doğrulandı mı | Evet |
| Performans ölçütleri | Mevcut ölçümler ve sınırları için [Değerlendirme Veri Seti ve Sonuçlar](#değerlendirme-veri-seti-ve-sonuçlar) bölümüne bakın. |
| Bilinen olası riskler | Poz/temas bozulmaları, dağılım dışı kısıtlardan gelen uygunsuz hareket ve üretilen bir referans güvenli bir kontrol komutu sayılırsa fiziksel risk. |
| Lisanslama | Bu modelin kullanımı [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) ile yönetilir. Ayrı yazılım ve örnek veri koşulları için [Lisans ve Kullanım Koşulları](#lisans-ve-kullanım-koşulları) bölümüne bakın. |

## Gizlilik Alt Kartı (Privacy Subcard)

Bu modeli eğitmek için kullanılan veri setleri yayımdan önce incelendi. Daha fazla bilgi için [NVIDIA Gizlilik Politikası'nı](https://www.nvidia.com/en-us/about-nvidia/privacy-policy/) inceleyin.

## Emniyet ve Güvenlik Alt Kartı (Safety and Security Subcard)

| Alan | Yanıt |
| --- | --- |
| Modelin uygulama alanları | Endüstri/Makine ve Robotik, Medya ve Eğlence. Üretilen hareket, fiziksel robot kontrolü için doğrulanmış bir çıktı değildir. |
| Yaşamsal etki | Üretilen referans hareket, doğrulanmış bir fiziksel kontrolcü değildir. Uygunsuz hareketi doğrudan bir robotta çalıştırmak fiziksel risk yaratabilir. |
| Kullanım durumu kısıtları | Bu modelin kullanımı [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) ile yönetilir. Ayrı yazılım ve örnek veri koşulları için [Lisans ve Kullanım Koşulları](#lisans-ve-kullanım-koşulları) bölümüne bakın. |
| Model ve veri seti kısıtları | ONNX dosyaları eğitim derleminin tamamını yeniden dağıtmaz. Veri setine erişim ve hareket örneklerinin kullanımı, model ağırlıklarından ayrı olarak yönetilir. |

Dışa aktarıcı yapıt (artifact) bütünlüğünü ve grafik yapısını denetler. Bu denetimler bir güvenlik onayı ya da güvenli davranış güvencesi değildir. Fiziksel kullanımdan önce modeli ve sonraki aşamadaki kontrol sistemini, amaçlanan çalışma koşulları altında birlikte değerlendirin.

Fiziksel çalıştırma, sonraki aşamada bir izleme kontrolcüsü, sınır uygulaması ve güvenlik doğrulaması gerektirir.

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
