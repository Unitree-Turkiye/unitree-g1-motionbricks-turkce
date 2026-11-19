> Bu belge NVlabs/motionbricks deposundaki `docs/inbetweener.md` dosyasının Türkçe çevirisidir (CC-BY-4.0). Değişiklik: Türkçeleştirme.

# Inbetweener (ara kare üretici)

[Inbetweener](https://github.com/NVlabs/motionbricks/blob/main/app/inbetweener_demo.py), bu depodaki MotionBricks üretiminin en küçük eksiksiz örneğidir. Bağlam pozlarını (context pose) ve hedef pozları (target pose) hazırlar, bir geçiş üretir ve onu geçiş noktasına kadar oynatır. Sonra oynatılan hareketi bir sonraki üretime geri besler. Uygulamanın yaptığı iş ara kare üretimidir (inbetweening).

<table>
  <tr>
    <th>SOMA</th>
    <th>Unitree G1</th>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/inbetweener_soma.gif" alt="SOMA ile Inbetweener" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/inbetweener_g1.gif" alt="Unitree G1 ile Inbetweener" width="100%"></td>
  </tr>
</table>

Bu sayfa önce uygulamanın nasıl çalıştırılacağını gösterir, sonra MotionBricks'i başka bir uygulamaya entegre eden geliştiriciler için kodunu açıklar. Newton görüntüleyicisi, ImGui kontrolleri ve görüntüleyicinin çizicileri veri akışını görünür kılar. Model bunlara ihtiyaç duymaz.

[Rehberli eğitim](REHBERLI-EGITIM.md#bölüm-1-inbetweener-ile-animasyon-üretin) bu uygulamayla Start, Entry, Duck-under ve End pozlarından bir kaçınma dizisi kurar. Aşağıdaki bölümler bu hazırlık adımlarını, model girdilerini hazırlayan ve her üretimi çalıştıran kodla ilişkilendirir.

## Inbetweener'ı çalıştırın

```sh
uv run python -m app.inbetweener_demo
```

- `--model soma` ya da `--model unitree_g1` başlangıç karakterini seçer. Varsayılan SOMA'dır.
- `--model-precision fp16` karma FP16 modelini kullanır. Varsayılan FP32'dir.
- `--verbose` ayrıntılı günlük iletilerini gösterir.

Uygulama, aynı yürüyüş klibinden 1 s ve 7 s anlarında örneklenmiş Start ve **Target 1** ile açılır. Target 1, Start'tan -Y yönünde 2 m uzakta durur. Start yeşil, hedefler mavidir.

Deneyebilecekleriniz:

- **Character** seçicisiyle SOMA ve Unitree G1 arasında geçiş yapın. Her geçiş Start, tek bir hedef ve varsayılan görünümle baştan başlar.
- Bir pozu taşıyın ve döndürün. Yükseklik ve eğim varsayılan olarak kilitlidir, böylece poz zeminde kalır.
- **...** ile başka bir klip seçin ve ondan bir poz seçmek için zaman kaydırıcısını sürükleyin.
- **+** ile hedef ekleyin, sıralarını değiştirmek için listede sürükleyin, sonra **Generate**'e basın.
- Her hedef için **Poses to Use**, **Transition Early By (frames)** ve **Tokens to Generate** değerlerini ayarlayın. Her ayarı [Eğitim adımları model girdilerini nasıl günceller](#eğitim-adımları-model-girdilerini-nasıl-günceller) bölümü açıklar.
- Üretilen hareketin ayak temaslarını çizmek için **Generate**'in yanındaki **Show Foot Contacts**'u açın.

### Kontroller

| Eylem | Kontrol |
| --- | --- |
| Kamerayı taşıma | **W**, **A**, **S** ve **D** ya da **ok tuşları**. **Q** ve **E** kamerayı aşağı ve yukarı taşır. |
| Kamera pivotunun çevresinde döndürme | **Orta fare tuşuyla** sürükleyin |
| Çevreye bakma | **Sol fare tuşuyla** sürükleyin |
| Kaydırma | **Shift** + **orta fare tuşuyla** sürükleme |
| Kamera pivotuna yaklaşma ya da uzaklaşma | **Ctrl** + **orta fare tuşuyla** sürükleme ya da **fare tekerleği** |
| Görüş alanını değiştirme | **Ctrl** + **fare tekerleği** |
| Poz seçme | Görünümde ağına ya da zemin halkasına tıklayın veya **Target Poses** içinde ona tıklayın |
| Seçimi taşıma ya da döndürme | Tutamacı (gizmo) sürükleyin ya da **Transform** alanlarını düzenleyin |
| Dönüşümü sıfırlama ya da kilitleme | Sıfırlamak için **Translation** ya da **Rotation**'a sağ tıklayın. Bir ekseni kilitlemek için eksen etiketine sağ tıklayın. |

## Üretim akışı

Demo, çalışma zamanının dört temel işlemini kullanır:

```python
context.update_from_poses(recent_poses)
target.update_from_poses(target_poses)
engine.generate_animation(context=context, target=target, output=output)
output.sample(time, current_pose)
```

Aynı kalıp SOMA ve G1 için de geçerli. Motor sınıfları ve poz düzenleri farklıdır ama çağrılar birbirinin karşılığıdır.

Her üretim kısa bir geçiş verir. Tekrarlanan üretimlerin nasıl sürekli harekete dönüştüğünü [Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md#üretim-akışı) açıklar.

## Kod haritası

Modelle konuşan kod, [`app/inbetweener_demo.py`](https://github.com/NVlabs/motionbricks/blob/main/app/inbetweener_demo.py) içindeki şu sembollerde toplanır:

| Sembol | Sorumluluk |
| --- | --- |
| `PoseEntry` | Demo için tek bir kaynak klibi, örnekleme zamanını, dünya yerleşimini, hedef ayarlarını ve dört pozluk örnekleme penceresini saklar. |
| `sample_pose_window()` | 30 Hz'de dört pozu çağıranın dizisine örnekler ve köklerini dünya uzayına yerleştirir. |
| `InbetweenerDemo.use_rig()` | Önceki rig'in motorunu kapatır, SOMA ya da G1 için bir motor oluşturur ve uygulamayı o rig'e sıfırlar. |
| `InbetweenerDemo._prepare_rig()` | Yeniden kullanılabilir bağlam belirtecini, hedef belirtecini ve çıktı arabelleğini oluşturur, sonra Start'ı ve ilk hedefi kurar. |
| `InbetweenerDemo.refresh_poses()` | Start'ı ve her hedefi yeniden örnekler, önizlemelerini ve hızlarını günceller, sonra Start'a döner. |
| `InbetweenerDemo.reset()` | Start'ın dört pozunu bağlam belirtecine yükler. |
| `InbetweenerDemo.generate_next_segment()` | Sıradaki hedefi yükler ve hedefin izin verilen belirteç sayılarıyla `engine.generate_animation()`'ı çağırır. |
| `InbetweenerDemo.step()` | O anki segmenti geçiş noktasına kadar oynatır. Orada oynatılan son dört pozu bir sonraki bağlama kopyalar ve sıradaki hedefe geçer. Her hedef için ilk çağrı bunun yerine `generate_next_segment()`'i çalıştırır. |
| `InbetweenerDemo._draw_foot_contacts()` | Üretilen hareketin ayak temaslarını örnekler ve karakterin tabanlarına çizer. |

Kalan işlevler arayüzü, çizimi, seçmeyi ve dosya iletişim kutularını uygular. Bir hazırlık ortamını gösterirler. Bir entegrasyon bunların yerine oyun durumu, yordamsal hedefler, ağ girdisi ya da başka bir animasyon aracı koyabilir.

## Rig'i ve motoru hazırlayın

`use_rig()`, seçilen rig için motor oluşturmadan önce önceki rig'in motorunu kapatır. Böylece GPU belleğini tek bir motor tutar:

```python
if self.engine is not None:
    self.engine.synchronize()
    ...
    self.engine.close()
    self.engine = None
self.rig, self.engine = rig, create_engine(self.viewer, rig, self.fp16)
```

Ardından `_prepare_rig()`'i çağırır. Bu işlev `load_rig_assets()` ile şunları içeren bir `RigAssets` alır:

- SOMA karakteri ya da G1 robotu.
- Rig için bir `ClipRegistry`.
- Doğru arabellek düzenine sahip bir başvuru pozu.
- Kaynak koordinatlarından dünya uzayına dönüşüm.

`_prepare_rig()` bir rig her seçildiğinde bağlam belirtecini, hedef belirtecini ve çıktı arabelleğini oluşturur. O rig'le yapılan her üretim bu arabellekleri yeniden kullanır:

```python
if rig is ModelRig.SOMA:
    skeleton = assets.model.skeleton
    self.context = engine.create_input(skeleton)
    self.target = engine.create_input(skeleton)
    self.output = engine.create_output(skeleton)
else:
    self.context = engine.create_input()
    self.target = engine.create_input()
    self.output = engine.create_output()
```

SOMA'da her poz, iskeletin her eklemi için bir yerel eklem dönüşümü tutar. G1'de her poz 36 eklem koordinatı tutar: kök konumu, kökün XYZW dördeyi (quaternion) ve 29 eklem açısı. Sözleşmelerin tamamı için [SOMA](SOMA-MODEL-KARTI.md) ve [Unitree G1](UNITREE-G1-MODEL-KARTI.md) model kartlarına bakın. [Poz düzenleri ve model uzayı](CALISMA-ZAMANI-API.md#poz-düzenleri-ve-model-uzayı) konusunu Çalışma Zamanı API'si anlatır.

## Demoda pozların temsili

`PoseEntry` bu demonun uygulama durumudur. MotionBricks kütüphanesi ona ihtiyaç duymaz. Her `PoseEntry` şunları kaydeder:

- Bir klip ve bir örnekleme zamanı.
- Dünya uzayında bir kök yerleşimi.
- Dört pozluk bir örnekleme penceresi.
- **Poses to Use** ile belirlenen, modele gönderilecek hedef poz sayısı.
- **Transition Early By (frames)** ile belirlenen, geçiş noktasının kaç kare öne alınacağı.
- **Tokens to Generate** ile belirlenen, modelin seçebileceği en küçük ve en büyük belirteç sayısı.
- Görüntüleyici için görselleştirme durumu.

Liste her zaman Start ile başlar. Hedefler onu izler.

### Dört pozluk pencereyi örnekleyin

`sample_pose_window()`, bir klip örneğini modelin beklediği biçime dönüştürür. Çağıranın verdiği `poses` dizisini doldurur ve bir değer döndürmez. Yalnız anahtar sözcükle verilen `anchor` bağımsız değişkeni, örnekleme zamanına denk gelen pozu seçer:

```python
def sample_pose_window(
    entry: PoseEntry, model_to_runtime: wp.transform, poses: wp.array, *, anchor: int,
) -> None:
    ...
    for frame in range(MODEL_POSES_PER_TOKEN):
        sample_time = entry.sample_time + (frame - anchor) / MODEL_SAMPLE_RATE
        entry.clip.sample(sample_time, poses[frame], root_offset=root_offset)
```

`MODEL_POSES_PER_TOKEN` dörttür, `MODEL_SAMPLE_RATE` de 30 Hz'dir.

- Start için `refresh_poses()` `anchor=3` geçirir, böylece seçilen örnek pencerenin son pozu olur. Önceki üç poz Start'a giren hareketi anlatır.
- Bir hedef için `anchor=0` geçirir, böylece seçilen örnek pencerenin ilk pozu olur. Ardından gelen en çok üç poz, o hedeften sonra istenen hareketi anlatır.

[Rehberli eğitim](REHBERLI-EGITIM.md#hedef-ve-bağlam-pozlarını-inceleyin) bu farkı gösterir. **Show Skeleton(s)** açıkken görünüm, Start'ın örnekleme zamanında biten dört pozunu gösterir. End hedefi için örnekleme zamanında başlayan **Poses to Use** kadar pozu gösterir. Bu farkı `anchor` bağımsız değişkeni uygular.

### Bir pozu sahneye yerleştirin

Demo, örneklenen kökün zemin kaymasını kaldırır ve pozun yerleşimini uygular. Bu yerleşimi tutamaç ve **Transform** alanları düzenler. Demo ayrıca kaynak koordinatlarını dünya uzayına dönüştürür:

```python
ground_offset = wp.transform(
    wp.vec3(-float(root[0]), -float(root[1]), 0.0),
    wp.quat_identity(),
)
root_offset = wp.mul(entry.placement, wp.mul(ground_offset, model_to_runtime))
```

Yalnız kök yerleşimi değişir. SOMA'nın yerel eklem dönüşümleri ve G1'in eklem açıları örneklendiği gibi kalır. [Eğitimde](REHBERLI-EGITIM.md#ara-hedefleri-ekleyin) Entry ve Duck-under'ın gövde pozları yeniden hazırlanmadan tutamaçla taşınıp döndürülebilmesinin nedeni budur.

Kilitli eksenler değerlerini korur. `apply_transform_locks()` kilitleri tutamaç düzenlemelerine uygular, **Transform** alanları da kilitli eksenleri atlar.

`model_to_runtime`, SOMA'nın Y yukarı kaynak koordinatlarını Z yukarı dünya uzayına dönüştürür. G1 dönüşümü birim dönüşümdür. Motorun ve klip örneklemenin neden aynı dönüşümü aldığını [Poz düzenleri ve model uzayı](CALISMA-ZAMANI-API.md#poz-düzenleri-ve-model-uzayı) açıklar.

## Eğitim adımları model girdilerini nasıl günceller

### Klip ve kare seçin

Bir klip ve bir kare seçmek, örneğin [eğitimde](REHBERLI-EGITIM.md#klibi-ve-kareleri-atayın) Entry için kaçınma BVH klibinin 199. karesini seçmek, `_choose_pending_clip()` ve `_draw_properties()` içinde `PoseEntry.path`, `PoseEntry.clip` ve `PoseEntry.sample_time` değerlerini ayarlar. Her değişiklik `refresh_poses()`'u çağırır:

```python
for index, entry in enumerate(self.pose_entries):
    anchor = MODEL_POSES_PER_TOKEN - 1 if index == 0 else 0
    entry.sampled_poses = wp.empty_like(self.context.poses)
    sample_pose_window(entry, self.model_to_runtime, entry.sampled_poses, anchor=anchor)
```

Bir entegrasyonun klibe ihtiyacı yoktur. Doğru SOMA ya da G1 düzenindeki her sonlu pozu bağlam ve hedef belirteçlerine yükleyebilir. Klipler bu demoda yalnız elverişli bir hedef poz kaynağıdır. Örneğin pozları bunun yerine [Kimodo](https://github.com/nv-tlabs/kimodo) ya da [ARDY](https://github.com/nv-tlabs/ardy) üretebilir.

### Hedefleri ekleyin ve sıralayın

[Eğitimde](REHBERLI-EGITIM.md#ara-hedefleri-ekleyin) olduğu gibi **+**, `add_target()`'ı çağırır. Bu işlev seçimin klibini ve örnekleme zamanını kopyalar, hedefi aynı dönüşle son hedeften -Y yönünde 1.5 m uzağa yerleştirir ve ona kimliğine göre bir ad verir, örneğin **Target 2**. Bir hedefi listede sürüklemek `move_target()`'ı çağırır.

Oynatmayı liste sırası belirler. `next_target_index`, `pose_entries` içinde ilerler. Her hedef bir üretim verir. Start sıfırıncı sırada kalır, çünkü hedef değil ilk bağlamı sağlar.

### Poses to Use ayarı

**Poses to Use**, bir hedefin dört pozundan kaçının modeli bilgilendireceğini belirler. Bir hedefin yalnız bir duruş mu belirleyeceğine, yoksa hareketi de mi anlatacağına bu ayar karar verir:

```python
self.target.update_from_poses(entry.sampled_poses[:entry.num_poses_to_use], stream=self.engine.stream)
```

Kabul edilen aralık 1 ile 4 arasıdır:

- Tek poz varış noktasını belirler. Model onu hızı sıfır olan bir duruş olarak ele alır.
- İkinci poz hız ekler.
- Üçüncü ve dördüncü poz ivme ve momentum ekler.

Start her zaman dört bağlam pozunun hepsini sağlar. Hiçbir üretim Start'ta bitmez. Bu yüzden Start için **Poses to Use**, **Transition Early By (frames)** ve **Tokens to Generate** devre dışıdır.

### Transition Early By ayarı

**Transition Early By (frames)**, [eğitimde](REHBERLI-EGITIM.md#klibi-ve-kareleri-atayın) olduğu gibi, o anki segment son karesine ulaşmadan dizinin sıradaki hedefe doğru ilerlemesini sağlar:

```python
self.segment_end_frame = max(
    MODEL_POSES_PER_TOKEN - 1, self.output.num_frames - 1 - entry.transition_early_by_frames,
)
```

Bu ayar modele gönderilen hedefi değiştirmez. Geçiş noktasını öne alır, böylece bir sonraki üretimden önce üretilen hareketin daha azı oynar. O üretimin bağlamı da bu yüzden geçiş noktasında biter. Her zaman en az dört kare oynar, çünkü bir sonraki bağlam oynatılan son dört karedir.

Bu, bir hedefi çalışma zamanında kılavuz olarak kullanışlı kılar. Karakter ona doğru yönelir ve varmadan yoluna devam eder. Böyle bir hedefi, karakteri daha hassas bir etkileşim pozuna doğru üretmeden önce yönlendirmek ve yerleştirmek için kullanın. [Eğitimde](REHBERLI-EGITIM.md#klibi-ve-kareleri-atayın) Entry, Duck-under'dan önce bu işi görür.

### Tokens to Generate ayarı

**Tokens to Generate**, modelin bu hedefe doğru yapılan üretim için seçebileceği en küçük ve en büyük belirteç (token) sayısını belirler. Her hareket belirteci dört ham kare tutar. **Raw frames:** göstergesi karşılık gelen aralığı gösterir. `generate_next_segment()` bu aralığı motora izin verilen belirteç sayıları olarak geçirir:

```python
config=MotionBricksInferenceConfig(
    allowed_token_counts=tuple(range(entry.min_tokens, entry.max_tokens + 1)),
),
```

Her hedef [`DEFAULT_TOKEN_COUNTS`](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/model/config.py) içindeki 8 ile 14 belirteçlik aralıkla başlar. `MotionBricksInferenceConfig` de aynı varsayılanı kullanır. Model sözleşmesi 6 ile 16 arasına izin verir. **Min** ve **Max** eşit olduğunda geçişin uzunluğu sabitlenir. Geniş bir aralık, modelin mesafeye ve harekete uyan bir süre seçmesine olanak verir.

### Generate düğmesine tıklayın

**Generate** düğmesi `play()`'i çağırır. Liste en az bir hedef içerene kadar düğme devre dışıdır. `play()` önce `reset()`'i çağırır, sonra oynatmayı başlatır:

```python
def play(self) -> None:
    self.reset()
    if len(self.pose_entries) > 1:
        self.playing = True
```

Her önizleme bu yüzden Start bağlamından başlar ve o anki hedef listesinde sırayla ilerler.

## Tek bir üretim

`generate_next_segment()` tek bir üretim yapar:

```python
entry = self.pose_entries[self.next_target_index]
self.target.update_from_poses(entry.sampled_poses[:entry.num_poses_to_use], stream=self.engine.stream)
self.engine.generate_animation(
    context=self.context, target=self.target, output=self.output,
    config=MotionBricksInferenceConfig(
        allowed_token_counts=tuple(range(entry.min_tokens, entry.max_tokens + 1)),
    ),
)
```

Bağlamı ya `reset()` ya da önceki segmentin `step()`'i yükledi. Hedef, listedeki bir sonraki hedeftir. `generate_animation()` yeniden kullanılabilir çıktı arabelleğini doldurur ve `output.num_frames` değerini geçerli kare sayısına ayarlar.

Çıktı arabelleği desteklenen en büyük sonuç için ayrılır. O anki üretim için yalnız ilk `output.num_frames` kare geçerlidir. Demo, segmentin geçiş noktasını bu değerden ve hedefin **Transition Early By** ayarından türetir.

Bütün GPU işlemleri `engine.stream` üzerinde sıralanır. Ana döngü `wp.ScopedStream(self.engine.stream)` içine girer, tek tek arabellek işlemleri de `stream=self.engine.stream` alır. Bir entegrasyon bu sıralamayı korumalı. GPU sonuçlarını CPU'da okumadan önce motoru eşzamanlamalı.

### Ayak temaslarını gösterin

**Generate**'in yanındaki **Show Foot Contacts** onay kutusu, üretilen hareket için modelin ayak temaslarını çizer. Yanındaki kaydırıcı eşiği belirler. `_draw_foot_contacts()`, temasları oynatma zamanında `output.sample_foot_contacts()` ile örnekler. Sonra teması eşiğin üzerinde olan her tabanın çevresini çizer. Dört değerin sırasını ve anlamını [foot_contacts.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/animation/foot_contacts.py) tanımlar.

## Oynatma ve zincirleme üretim

Dizinin tamamı üç işlevi izler.

### 1. `reset()`: ilk bağlamı oluşturun

```python
start = self.pose_entries[0]
self.context.update_from_poses(start.sampled_poses, stream=self.engine.stream)
wp.copy(self.current_pose, start.preview_pose, stream=self.engine.stream)
```

Start'ın dört pozu bağlam olur. Ekranda gösterilen o anki poz, bu pencerenin son pozuna döner.

### 2. `generate_next_segment()`: tek bir hedefe doğru üretin

Demo, sıradaki hedefin pozlarını hedef belirtecine kopyalar ve `generate_animation()`'ı çağırır. Segmentin geçiş noktasını kaydeder ve ilk pozunu örnekler.

### 3. `step()`: segmenti oynatın ve sürekliliği koruyun

```python
end_time = self.segment_end_frame / MODEL_SAMPLE_RATE
self.playback_time = min(self.playback_time + delta_time, end_time)
self.output.sample(self.playback_time, self.current_pose, stream=self.engine.stream)
```

`step()`, üretilen hareketi geçiş noktasına ulaşana kadar uygulamanın güncelleme hızında örnekler. `run()`, oynatmayı son kareden bu yana geçen süre kadar ilerletir ve bu süreyi 0.1 s ile sınırlar. Daha yavaş donanımda oynatma ileri atlamak yerine yavaşlar.

`step()` sıradaki hedefe geçmeden önce gerçekten oynatılmış son dört pozu kopyalar:

```python
played_frames = self.segment_end_frame + 1
recent_poses = self.output.poses[played_frames - MODEL_POSES_PER_TOKEN:played_frames]
self.context.update_from_poses(recent_poses, stream=self.engine.stream)
```

[Rehberli eğitimde](REHBERLI-EGITIM.md#son-animasyonu-üretin) üretilen dizi Start'tan Entry'ye, Duck-under'a ve End'e ilerledi. Her segmentten sonra `step()`, oynatılan son dört pozu sıradaki hedefin bağlamı yaptı. Bu geri besleme döngüsü birbirinden bağımsız üretimleri sürekli bir harekete bağlar.

## SOMA ve Unitree G1

**Character** seçicisi, entegrasyon akışının rig'e bağlı olmadığını gösterir:

| | SOMA | Unitree G1 |
| --- | --- | --- |
| Kaynak klipler | BVH | CSV |
| Motor girdisi | Yerel eklem dönüşümlerini tutan `PoseToken` | Eklem koordinatlarını tutan `DOFToken` |
| Üretilen çıktı | Yerel eklem dönüşümlerini tutan `AnimationBuffer` | Eklem koordinatlarını tutan `DOFAnimationBuffer` |
| Dünya uzayına dönüşüm | `model_to_runtime`, Y yukarı kaynak koordinatlarını Z yukarı dünya uzayına dönüştürür | Birim dönüşüm |
| Bu demodaki çizim | `SkeletalMeshRenderer` | `DOFMeshRenderer` |

Başka bir rig seçtiğinizde demo geçişi bir sonraki karenin başına erteler. `use_rig()` sonra o anki motoru kapatır ve seçilen rig için bir motor oluşturur. Rig'in arabelleklerini oluşturan ve listeyi Start ile tek bir hedefe sıfırlayan `_prepare_rig()`'i çağırır.

TensorRT plan önbelleği, önceki rig'e geri dönüşü kısa tutar: Planı yeniden derlemek yerine yükler. Birkaç motoru bir arada tutan daha büyük bir uygulama, motorları rig'e, hassasiyete ve TensorRT planını etkileyen öteki derleme seçeneklerine göre anahtarlamalı.

## Bu kalıbı bir entegrasyonda kullanın

Inbetweener üretim döngüsünü elle çalıştırır, böylece her çağrı görünür kalır. Çalışma Zamanı API'si sayfasındaki [üretim akışı](CALISMA-ZAMANI-API.md#üretim-akışı) döngünün adımlarını sıralar. Daha büyük, gerçek zamanlı bir uygulama için [motion_player.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/runtime/motion_player.py) içindeki `GeneratedMotionPlayer` ya da `GeneratedDOFMotionPlayer` sınıflarını örnek alın. Bunları [Üretimleri zincirleme](CALISMA-ZAMANI-API.md#üretimleri-zincirleme) anlatır.

## Ek kaynaklar

- [Rehberli eğitim](REHBERLI-EGITIM.md): Bu uygulamayı kullanan, kullanıcıya dönük iş akışı.
- [Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md): Motorlar, poz düzenleri, klipler, üretim ayarları, zincirleme, TensorRT derlemeleri ve günlük kaydı.
- [ONNX çıkarımı ve toplu işleme](ONNX-CIKARIM.md): Doğrudan ONNX arayüzleri, toplu işleme ve alternatif çalışma zamanları.
- [SOMA model kartı](SOMA-MODEL-KARTI.md): SOMA girdileri, çıktıları, koordinatları ve sınırlamaları.
- [Unitree G1 model kartı](UNITREE-G1-MODEL-KARTI.md): G1 eklem düzeni, girdileri, çıktıları ve sınırlamaları.

---

**Unitree G1 Türkiye:** Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org. [Unitree G1 ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat)
