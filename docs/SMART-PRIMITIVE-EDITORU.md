> Bu belge NVlabs/motionbricks deposundaki `docs/smart_primitive_editor.md` dosyasının Türkçe çevirisidir (CC-BY-4.0). Değişiklik: Türkçeleştirme.

# Smart Primitive Editor (Smart Primitive düzenleyicisi)

Bir Smart Primitive (akıllı ilkel), bir eylemi bir nesnenin çevresine yerleştirilmiş hedef pozlar (target pose) olarak anlatır. Karakter eylemi bir klip gibi oynatmaz. MotionBricks, karakter o an ne yapıyorsa oradan başlayarak hedef pozların içinden geçen hareketi üretir. Bu yüzden tek bir Smart Primitive pek çok yaklaşma yönünde ve hızda işler. Makale bunlara etkileşimleri anlattıklarında akıllı nesneler (smart objects), gezinmeyi yönettiklerinde akıllı lokomosyon (smart locomotion) adını verir.

Bu depoda Smart Primitive'ler, Scene Preview'daki atlama, eğilerek geçme ve oturma gibi etkileşimleri tanımlar. Her yürüyüş/hareket (lokomosyon) modunun karışım hedeflerini de tutarlar. Onları Smart Primitive Editor ile hazırlarsınız.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/smart_primitive_editor.png" alt="jump_0_5m Smart Primitive'i açıkken Smart Primitive Editor" width="100%">
</p>

```sh
uv run python -m app.smart_primitive_editor
```

Ayrıntılı günlük iletileri için `--verbose` ekleyin.

> **Not:** Smart Primitive Editor, model için davranışların ve etkileşimlerin nasıl hazırlanacağını gösteren bir örnektir, üretim aracı olarak tasarlanmamıştır. Bilinen sınırlaması: Geri alma ve yineleme komutu içermez, bu yüzden çalışmanızı sık kaydedin.

[Rehberli eğitimin 2. bölümü](REHBERLI-EGITIM.md#bölüm-2-bir-smart-primitive-oluşturun) adım adım bir kaçınma Smart Primitive'i kurar. Bu sayfa bir Smart Primitive'in her parçasının ne tuttuğunu ve Scene Preview'un onu nasıl kullandığını anlatır.

## Kontroller

Belge oluşturmak, açmak ve kaydetmek için **File** menüsünü kullanın. Düzenleyicinin üç paneli var: **Hierarchy**, **Sequences** ve **Properties**. Panellerin kontrolleri, düzenledikleri parçalarla birlikte anlatılır.

| Eylem | Kontrol |
| --- | --- |
| Kamerayı taşıma | **W**, **A**, **S** ve **D** ya da **ok tuşları**. **Q** ve **E** kamerayı aşağı ve yukarı taşır. |
| Kamera pivotunun çevresinde döndürme | **Orta fare tuşuyla** sürükleyin |
| Çevreye bakma | **Sol fare tuşuyla** sürükleyin |
| Kaydırma | **Shift** + **orta fare tuşuyla** sürükleme |
| Kamera pivotuna yaklaşma ya da uzaklaşma | **Ctrl** + **orta fare tuşuyla** sürükleme ya da **fare tekerleği** |
| Görüş alanını değiştirme | **Ctrl** + **fare tekerleği** |
| Düğüm seçme | Görünümde pozuna ya da etkileşim kutusuna tıklayın veya **Hierarchy** içinde ona tıklayın |
| Seçimi taşıma ya da döndürme | Tutamacı (gizmo) sürükleyin ya da **Transform** alanlarını düzenleyin |
| Dönüşümü sıfırlama ya da kilitleme | Sıfırlamak için **Translation** ya da **Rotation**'a sağ tıklayın. Bir ekseni kilitlemek için eksen etiketine sağ tıklayın. |

**Transform** değerleri ve tutamaç kolları üst düğümün eksenlerini kullanır.

## Bir Smart Primitive neleri içerir

Bir Smart Primitive, SOMA ya da Unitree G1 rig'lerinden biri için hazırlanmış bir `.smart_primitive.json` belgesidir. Şunları tutar:

- Kök (root) düğümün altındaki bir dönüşüm hiyerarşisinde **düğümler** (Nodes).
- Düğümlerin üzerinde **bileşenler** (Components): **Target Pose**, **Interaction Box** ya da **Pivot**. Bir düğüm her türden en çok bir tane tutar.
- Hedef poz düğümlerini karakterin onlara ulaştığı sırayla listeleyen **diziler** (Sequences).

Eğitimdeki kaçınma Smart Primitive'i bunların nasıl bir araya geldiğini gösterir:

```text
Root
├── Interaction box       Interaction Box: etkileşimin başlayabileceği yer
└── Pivot                 Pivot: diziyi yaklaşmaya doğru döndürür ve kaydırır
    ├── Entry pose        Target Pose: çubuktan önceki bir yürüyüş pozu
    └── Duck-under pose   Target Pose: çubuğun altındaki poz

Sequences
  Sequence 1   Entry pose, Duck-under pose   Pivot: Pivot
```

Hazır belgeler [assets/smart_primitives](https://github.com/NVlabs/motionbricks/tree/main/assets/smart_primitives) klasöründedir:

| Belge | Ne gösterir |
| --- | --- |
| `jump_0_5m` | İki varyasyonlu küçük bir atlama |
| `jump_0_75m` | Üç varyasyonlu bir engel aşma, ikisinde el küpün üzerinde |
| `avoidance` | Yürüyüş ve hafif koşu varyasyonlarıyla bir çubuğun altından eğilerek geçme |
| `sit` | Oturma, oturarak bekleme ve ayağa kalkma |
| `locomotion_bs1_linear` | Forward için Idle, Walk, Jog ve Run hedefleri |
| `locomotion_bs2_directional` | Strafing için Idle, Forward, Backward, Left ve Right hedefleri |
| `locomotion_bs2_directional_crouch` | Crouching için Idle, Forward, Backward, Left ve Right hedefleri |
| `locomotion_style_variations_bs1_linear` | Stil modları için Idle ve Skipping, Zombie gibi altı stil hedefi |

## Düğüm hiyerarşisi

Her düğümün bir adı ve üst düğümüne göre bir yerel dönüşümü vardır. Bir düğümü taşımak altındaki her şeyi taşır. Kök düğüm Smart Primitive'in kendisini temsil eder. Bir sahne her örneği kökünü taşıyarak yerleştirir, öteki düğümler de köke göre hazırlandıkları yerleşimi korur.

Hangi parçaların birlikte hareket edeceğine de hiyerarşi karar verir. Bir pivotun taşıması gereken hedef pozları pivot düğümünün altına koyun. Etkileşim kutularını ve hedef pozları, eylemin geçtiği nesnenin çevresine yerleştirin.

**Hierarchy** içinde:

- **+** seçili düğüme bir alt düğüm ekler. **-** seçili düğümü alt düğümleriyle birlikte siler ve diziler o adımları düşürür. Aynı işlemler için bir düğüme sağ tıklayın.
- Adını değiştirmek için bir düğüme çift tıklayın. Üst düğümünü değiştirmek için onu başka bir düğümün üzerine sürükleyin.
- Onay kutusu bir düğümü ve alt düğümlerini görünümde gizler.
- **Character** seçicisi rig'i belirler. Bir hedef pozun klibi olduğu anda kilitlenir.

Eğitimdeki [Düğüm hiyerarşisini oluşturun](REHBERLI-EGITIM.md#düğüm-hiyerarşisini-oluşturun) adımı kaçınma hiyerarşisini kurar.

## Bileşenler

Bir bileşen eklemek için bir düğüm seçin, **Properties** içinde boş bir alana sağ tıklayın ve **Add** > **Components** yolunu izleyin. **Referenced by**, düğümü kullanan dizileri listeler.

### Target Pose (hedef poz)

Bir Target Pose, seçilen zamandan başlayarak bir klipten 30 Hz'de dört poz örnekler. Bir dizi hedefi kullandığında bu pozların yalnız kökü düğümün dünya yerleşimine taşınır. Eklem değerleri örneklendiği gibi kalır, böylece örneklenmiş tek bir hedef her yerleşimde işe yarar.

| Ayar | Anlamı |
| --- | --- |
| Clip | SOMA için BVH klibi, Unitree G1 için CSV klibi |
| Seconds or Frames | Örnekleme zamanı. |
| Constrain Root | Klibin kökünü zemin boyunca düğümün orijinine taşır ve yüksekliğini korur. Bu seçenek kapalıyken poz, klibin orijinine göre klipte durduğu yerde kalır. |
| IK Pins | Yalnız SOMA. Seçilen elleri ya da ayakları, adımın ikinci yarısı boyunca hedef pozun üzerine çeker. Bkz. [IK Pins](#ik-pins). |
| Poses to Use | Dört pozdan kaçının modeli bilgilendireceği. Bkz. [Poses to Use ayarı](ARA-KARE-URETICI.md#poses-to-use-ayarı). |
| Transition Early By (frames) | Üretilen hareket bitmeden bu kadar kare önce sıradaki adıma geçer. |
| Tokens to Generate | Modelin adım için seçebileceği uzunluğu sınırlar. Bir Target Pose 6 ile 14 belirteçle başlar, model de 6 ile 16 arasına izin verir. Sabit bir uzunluk istemek için iki değeri aynı sayıya ayarlayın. **Raw frames:** göstergesi karşılık gelen kare aralığını gösterir. |

**Poses to Use**, **Transition Early By** ve **Tokens to Generate**, varsayılan belirteç sayıları dışında [Inbetweener hedeflerindeki](ARA-KARE-URETICI.md#poses-to-use-ayarı) gibi çalışır. **Speed**, örneklenen pozların ne kadar hızlı hareket ettiğini gösterir. Bir dizinin ilk hedefi dizinin başlangıç hızını belirler. **Show Mesh**, **Show Skeleton(s)**, **Show Velocity** ve **Show Trajectory** yalnız görünümü değiştirir.

Eğitimdeki [Hedef pozları yapılandırın](REHBERLI-EGITIM.md#hedef-pozları-yapılandırın) adımı Entry ve Duck-under hedeflerini kurar.

#### IK Pins

**IK Pins**, sol ve sağ elleri ve ayakları listeler. Hedef pozun koyduğu yere tam olarak inmesi gereken eli ya da ayağı sabitleyin. Örneğin SOMA engel aşma belgesi `jump_0_75m` içindeki iki hedef sol eli sabitler.

Oynatma sırasında iki kemikli IK, sabitlenmiş her eli ya da ayağı adımın ilk hedef pozundaki yerine çeker. Çekme, üretilen adımın yarısında başlar ve geçiş noktasında tam güce ulaşır. Her üretim sabitlemeleri yeniler, yani sabitleme içermeyen bir adım onları temizler. Dizi bittiğinde de temizlenirler.

Karakter düzeltilmiş pozu kaydeder, böylece bir sonraki üretim elin ya da ayağın gerçekte durduğu yerden başlar. Hedef poz IK'si yalnız SOMA'da bulunur. Onu [target_pose.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/runtime/target_pose.py) içindeki `TargetPoseIKCorrector` uygular.

### Interaction Box (etkileşim kutusu)

**Width**, **Depth** ve **Height**, düğümün çevresindeki kutuyu boyutlandırır. Scene Preview'da karakterin yoklama kutusu (probe box) Smart Primitive'in etkileşim kutularından herhangi biriyle kesiştiğinde bir etkileşim başlayabilir. Kutuların tek işi etkileşim başlatmaktır. Model için çarpışma geometrisi işlevi görmezler.

Bir Smart Primitive'in her biri ayrı bir düğümde olmak üzere birkaç etkileşim kutusu olabilir. Çömelmiş bir karakterin yoklama kutusu daha kısadır, böylece yükseltilmiş bir etkileşim kutusu yalnız ayakta duran bir karakteri yakalar. Hazır `avoidance` belgesi kutusunu bu nedenle yükseltir.

Eğitimdeki [Etkileşim kutusunu (Interaction Box) yapılandırın](REHBERLI-EGITIM.md#etkileşim-kutusunu-interaction-box-yapılandırın) adımı kaçınma kutusunu boyutlandırıp yerleştirir.

### Pivot

Pivot, tek bir dizinin pek çok yaklaşmadan işlemesini sağlar. Bir dizi başladığında pivot düğümü karakterin yaklaşmasına doğru hareket eder ve döner, altındaki her hedef de onu izler. Yalnız hedef pozların kökleri hareket eder.

![Yandan gelen bir karakter, pivotsuz ve eğitimdeki Pivot ile](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_pivot_00_problem.gif)

*Karakter, hazırlanan diziye 90 derecelik açıyla gelir. Soldaki pivotsuz durumda hazırlanan Entry pozuna yürür ve döner. Sağdaki eğitim Pivot'uyla hedef pozlar döner ve karakterin rotasına kayar.*

| Ayar | Anlamı |
| --- | --- |
| Motion Direction | Pivotun yaklaşmayla aynı hizaya gelmesi gereken yerel yönü |
| Translation | Pivotu tek bir yerel eksen boyunca kaydırır. **Line Projection**, pivotu hazırlandığı yerden ölçülen **Offsets** sınırları içinde tutar. |
| Rotation | Pivotu tek bir yerel eksen çevresinde döndürür. **Cone**, dönüşü koninin merkezinin iki yanında **Half Angle** kadarla sınırlar. **Cone Side** bu merkezi belirler: **Motion Direction**, ters yön için **Mirrored** ya da ikisi için **Both**. |
| Operation Order | **Translation -> Rotation** önce kaydırır. **Rotation -> Translation** önce döndürür ve bu dönüş, pivotun boyunca kaydığı çizgiyi de döndürür. |

**Show Motion Direction**, **Show Translation Constraint** ve **Show Rotation Constraint** bu ayarları görünümde çizer:

![Eğitimdeki Pivot'un hazırlanan yerleşimi, üstten görünüm](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/tut_pivot_01_layout.png)

*Eğitimdeki Pivot, üstten. Sarı ok Motion Direction'dır, kırmızı kesikli çizgi de Line Projection aralığıdır.*

Pivot, dizi başladığında bir kez hizalanır. Sınırları hazırlanan yerleşimden ölçülür, bir önceki hizalamadan ölçülmez. Böylece bir Smart Primitive tekrarlanan etkileşimlerde kaymaz. Pivotu olmayan bir dizi o anki yerleşimini korur.

Eğitimdeki [Pivotu yapılandırın](REHBERLI-EGITIM.md#pivotu-yapılandırın) adımı bir pivot kurar. [Pivotu anlamak](REHBERLI-EGITIM.md#pivotu-anlamak) bölümü her ayarın nasıl davrandığını gösterir, altı yaklaşmayı karşılaştırır ve hizalanmış pozların gelen hareketi neden sürdürdüğünü açıklar.

## Diziler

Bir dizi, hedef poz düğümlerini karakterin onlara ulaştığı sırayla listeler. Her adım, o hedefin ayarlarıyla tek bir hedefe doğru üretir. Bir dizi bir pivot düğümünü adlandırabilir. Bu pivot, dizi başlamadan önce dizinin tamamını yaklaşmayla hizalar.

Birkaç dizi aynı düğümleri paylaşabilir. Tek bir belge bir eylemin varyasyonlarını bu yolla tutar, örneğin hazır `avoidance` belgesinin Walk ve Jog dizileri. Diziyi, sahnenin o örnek için seçtiği davranış seçer:

- `VelocityMatchingSequenceBehavior`, başlangıç hızı karakterin hızına en yakın olan diziyi seçer.
- `SittingBehavior`, Sit ve Stand adlı dizileri oynatır.

Davranışları ve nasıl yazılacaklarını [Scene Preview](SAHNE-ONIZLEME.md#smart-primitive-davranışları) anlatır.

**Sequences** içinde:

- **+** bir dizi ekler ve adını değiştirmeye başlar. **-** seçili diziyi siler. Bir dizinin adını değiştirmek ya da onu silmek için ona sağ tıklayın.
- **Pivot**, dizinin pivot düğümünü ya da **None** değerini ayarlar.
- **+ Add Target Pose** bir adım ekler. Bir düğümü **Hierarchy**'den dizinin başlığına sürükleyerek de ekleyebilirsiniz. Yalnız **Target Pose** içeren düğümler adım olur.
- Adımları yeniden sıralamak için sürükleyin, birini kaldırmak için **x**'e tıklayın.

Eğitimdeki [Diziyi tanımlayın](REHBERLI-EGITIM.md#diziyi-tanımlayın) adımı Entry ve Duck-under adımlarını sıralar. [Varyasyonları keşfedin](REHBERLI-EGITIM.md#varyasyonları-keşfedin) aynı belgeye yeni diziler ekler.

## Sahne pozları nasıl yerleştirir

Scene Preview her adım için şunları yapar:

1. Davranışın bir dizi seçmesine ve pivotunu karakterin yaklaşmasıyla hizalamasına izin verir.
2. Her hedef düğümün dünya dönüşümünü, o örneğin sahnedeki yerleşiminden ve düğüm hiyerarşisinden hesaplar.
3. Düğümün örneklenmiş dört pozunun kökünü bu dönüşümle taşır. Eklem değerleri örneklendiği gibi kalır.
4. Karakterin son hareketinden bu pozlara doğru üretir.

Yalnız kök hareket ettiği için örneklenmiş tek bir poz takımı her yerleşimde işe yarar. Aynı belge her sahnedeki her örnekte ve aynı küpün iki atlama yönünde de çalışır. Her örneğin yerleşimini [Scene Editor](SAHNE-EDITORU.md) saklar.

## Örnek: oturup bekleme

Bekleme (hold), karakteri bir şey olana kadar bir pozda tutar. [sit.smart_primitive.json](https://github.com/NVlabs/motionbricks/blob/main/assets/smart_primitives/soma/sit.smart_primitive.json) içindeki oturup bekleme, iki diziyi döngüdeki bir kliple birleştirir:

```text
Root
├── InteractionBox        Interaction Box
└── Pivot                 Pivot, yaklaşmaya bakmak için Z çevresinde döner
    └── Sit               Target Pose: sit_on_chair_loop, frame 0, 1 pose, Constrain Root
        ├── ToStand       Target Pose: sit_on_chair_stop at 1.99 s, 4 poses
        └── ToSit         Target Pose: sit_on_chair_start at 1.85 s, 4 poses

Sequences
  Sit     ToSit, Sit      Pivot: Pivot
  Stand   ToStand         Pivot: None
```

1. **Sit** önce, oturma klibinden karakter oturmadan hemen önceki bir poz olan ToSit'e ulaşır. Sonra döngüdeki klibin ilk karesi olan Sit'e ulaşır. Pivot iki hedefi de yaklaşmaya doğru döndürür, böylece karakter her yandan dönüp oturur.
2. Sit tamamlandığında `SittingBehavior`, [sit_on_chair_loop](https://github.com/NVlabs/motionbricks/blob/main/assets/motions/soma/sitting/sit_on_chair_loop.bvh) klibini ulaşılan poza hizalanmış bir döngü olarak oynatır. Sit hedefi döngünün ilk karesi olduğundan üretilen hareketten klibe geçiş pürüzsüz olur.
3. **I**'ya (kumandada **X**) bastığınızda davranış **Stand** dizisini oynatır. Bu dizi, ayağa kalkma klibinden alınmış bir ayakta durma pozuna doğru üretir. Stand'in pivotu yoktur, bu yüzden Sit'in seçtiği yerleşimi korur.
4. Stand tamamlandığında karakter lokomosyona ya da beklemeye döner.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/sit_hold_soma.gif" alt="SOMA oturuyor, bekliyor, ayağa kalkıyor ve yürüyüp gidiyor" width="80%">
</p>

Duvara yaslanma gibi başka bir bekleme kurmak için:

1. Beklenen pozun döngüdeki bir klibini hazırlayın.
2. Döngüdeki klibin ilk karesinde **Constrain Root** ile biten bir giriş dizisi ve ayakta durma pozuna dönen bir çıkış dizisi hazırlayın.
3. Beklemenin yaklaşmaya uyması için giriş dizisine bir pivot ekleyin.
4. `SittingBehavior` benzeri bir davranış sınıfı yazın. Scene Preview sayfasındaki [Davranış yazın](SAHNE-ONIZLEME.md#davranış-yazın) bölümü bir örnek gösterir.
5. Bu sınıfı [Scene Editor](SAHNE-EDITORU.md) içinde sahne örneğinin **Behavior** değeri olarak seçin.

## Belge biçimi

Belgeler düz JSON'dur:

| Alan | İçerik |
| --- | --- |
| `format`, `schema_version` | `motionbricks.smart_primitive.authoring` ve 1 |
| `name`, `rig` | Belge adı ve `soma` ya da `unitree_g1` |
| `root` | Düğüm ağacı. Her düğümün bir `id`, bir `name`, bir `local_transform`, kendi `components` ve `children` alanları vardır. |
| `sequences` | Her dizinin bir `id`, bir `name`, sıralı `node_ids` alanı ve bir `pivot_node_id` ya da `null` değeri vardır |
| `editor` | Seçili düğüm ve kamera. Düzenleyici bunları geri yükler. |

Klip yolları belgeye göre görelidir. Bir belge açılırken rig'i kliplere karşı denetlenir. Scene Preview aynı dosyaları okur, bu yüzden ayrıca dışa aktarma adımı gerekmez.

**File** > **Save** ve **Save As...** belgeyi yazar. Her bileşen, `target_pose`, `interaction_box` ya da `pivot` gibi `type` değeri ve ayarlarıyla kaydedilir. Bir Target Pose örneklenmiş pozları değil, klibini ve örnekleme zamanını kaydeder. Yükleme pozları klipten yeniden örnekler.

## Ek kaynaklar

- [Rehberli eğitim](REHBERLI-EGITIM.md#bölüm-2-bir-smart-primitive-oluşturun): Kaçınma Smart Primitive'ini adım adım kurma.
- [Scene Editor](SAHNE-EDITORU.md): Smart Primitive örneklerini bir sahneye yerleştirme.
- [Scene Preview](SAHNE-ONIZLEME.md): Smart Primitive dizilerini oynatan davranışlar ve durumlar.
- [Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md): Hedef ayarları, hareket oynatıcıları ve zincirleme.

---

**Unitree G1 Türkiye:** Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org. [Unitree G1 ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat)
