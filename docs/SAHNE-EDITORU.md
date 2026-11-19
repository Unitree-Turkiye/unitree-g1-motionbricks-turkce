> Bu belge NVlabs/motionbricks deposundaki `docs/scene_editor.md` dosyasının Türkçe çevirisidir (CC-BY-4.0). Değişiklik: Türkçeleştirme.

# Scene Editor (sahne düzenleyicisi)

Bir sahne, Smart Primitive'leri (akıllı ilkeller) karakterin içinde dolaşabileceği bir alanda bir araya getirir. Smart Primitive örneklerini ve Solid Cube'ları yerleştirir, karakterin nereden başlayacağını belirler ve her örneği çalıştıran davranışı adlandırır. Kaydedilen `.scene.json` belgesini [Scene Preview](SAHNE-ONIZLEME.md) açar ve oynatır.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/scene_editor.png" alt="SOMA spor salonu sahnesi açıkken Scene Editor" width="100%">
</p>

```sh
uv run python -m app.scene_editor
uv run python -m app.scene_editor assets/scenes/soma_gym.scene.json
```

İsteğe bağlı bağımsız değişken, açılışta bir sahne açar. Ayrıntılı günlük iletileri için `--verbose` ekleyin.

> **Not:** Scene Editor, model için davranışların ve etkileşimlerin nasıl hazırlanacağını gösteren bir örnektir, üretim aracı olarak tasarlanmamıştır. Bilinen sınırlaması: Geri alma ve yineleme komutu içermez, bu yüzden çalışmanızı sık kaydedin.

[Rehberli eğitimin 3. bölümü](REHBERLI-EGITIM.md#bölüm-3-kaçınma-atlama-ve-oturma-içeren-bir-etkileşim-sahnesi-kurun) adım adım bir etkileşim parkuru kurar. Bu sayfa bir sahnenin neleri tuttuğunu ve Scene Preview'un onu nasıl kullandığını anlatır.

## Kontroller

Sahne oluşturmak, açmak ve kaydetmek için **File** menüsünü kullanın. Düzenleyicinin iki paneli var: **Scene** ve **Properties**. Panellerin kontrolleri, düzenledikleri parçalarla birlikte anlatılır.

| Eylem | Kontrol |
| --- | --- |
| Kamerayı taşıma | **W**, **A**, **S** ve **D** ya da **ok tuşları**. **Q** ve **E** kamerayı aşağı ve yukarı taşır. |
| Kamera pivotunun çevresinde döndürme | **Orta fare tuşuyla** sürükleyin |
| Çevreye bakma | **Sol fare tuşuyla** sürükleyin |
| Kaydırma | **Shift** + **orta fare tuşuyla** sürükleme |
| Kamera pivotuna yaklaşma ya da uzaklaşma | **Ctrl** + **orta fare tuşuyla** sürükleme ya da **fare tekerleği** |
| Görüş alanını değiştirme | **Ctrl** + **fare tekerleği** |
| Öğe seçme | Görünümde ona tıklayın ya da **Scene** ağacında ona tıklayın |
| Seçimi taşıma ya da döndürme | Tutamacı (gizmo) sürükleyin ya da **Transform** alanlarını düzenleyin |
| Dönüşümü sıfırlama ya da kilitleme | Sıfırlamak için **Translation** ya da **Rotation**'a sağ tıklayın. Bir ekseni kilitlemek için eksen etiketine sağ tıklayın. |

**Transform** değerleri ve tutamaç kolları üst öğenin eksenlerini kullanır.

## Bir sahne neleri içerir

Bir sahne, SOMA ya da Unitree G1 rig'lerinden biri için hazırlanmış bir `.scene.json` belgesidir. Şunları tutar:

- Tek bir CharacterStart: Karakterin nerede belireceği ve hangi yöne bakacağı.
- Smart Primitive örnekleri: Her biri sahneye bir Smart Primitive belgesi yerleştirir ve onu çalıştıran davranışı adlandırır.
- Solid Cube'lar: Bir atlamanın altındaki kutu gibi sahne dekoru.

SOMA spor salonu bunların nasıl bir araya geldiğini gösterir:

```text
CharacterStart
SmallJumpSP     jump_0_5m, VelocityMatchingSequenceBehavior
└── Box         Üzerine atlanacak Solid Cube
MidJumpSP       jump_0_75m, VelocityMatchingSequenceBehavior
└── Wall        Üzerinden aşılacak Solid Cube
SitSP           sit, SittingBehavior
└── Seat        Üzerine oturulacak Solid Cube
AvoidanceSP     avoidance, VelocityMatchingSequenceBehavior
└── Bar         Altından eğilerek geçilecek Solid Cube
```

Hazır sahneler [assets/scenes](https://github.com/NVlabs/motionbricks/tree/main/assets/scenes) klasöründedir:

| Sahne | Ne gösterir |
| --- | --- |
| `soma_gym` | SOMA için küçük bir atlama, bir engel aşma, üzerine oturulacak bir kutu ve altından eğilerek geçilecek bir çubuk |
| `g1_gym` | Unitree G1 için, robotun boyutuna ölçeklenmiş aynı yerleşim |

## Sahne ağacı

**Scene** paneli öğeleri listeler. Üst öğelerin tek işi düzenlemeyi kolaylaştırmaktır: Bir Smart Primitive'i taşımak altındaki küpleri de taşır. Scene Preview her öğeyi dünya dönüşümüne göre yerleştirir.

**Scene** içinde:

- **+** bir Smart Primitive ya da bir Solid Cube ekler. **-** seçili öğeyi alt öğeleriyle birlikte siler. Bir öğeyi silmek için ona sağ tıklayın.
- Adını değiştirmek için bir öğeye çift tıklayın. Bir öğeyi alt öğe yapmak için onu bir Smart Primitive'in ya da bir Solid Cube'un üzerine sürükleyin. Öğeyi en üst düzeye geri taşımak için ağacın altına bırakın ya da sağ tık menüsünden **Move to Root**'u seçin.
- CharacterStart en üst düzeyde kalır. Adı değiştirilemez, silinemez ve başka bir öğenin altına taşınamaz.
- **Character** seçicisi sahnenin rig'ini belirler. Sahnede hiç başvuru yokken SOMA ile Unitree G1 arasında geçiş yapar. Rig'i sonradan değiştirmek için başvuru tutan Smart Primitive'leri silin ya da yeni bir sahne başlatın.

## Öğeler

Bir öğeyi **Properties** içinde düzenlemek için onu seçin.

### CharacterStart

CharacterStart, Scene Preview sahneyi açtığında karakterin nerede belireceğini belirler. Oku, karakterin baktığı yönü gösterir. Her sahnede tam olarak bir tane bulunur.

### Smart Primitive

| Ayar | Anlamı |
| --- | --- |
| Reference | Yerleştirilecek `.smart_primitive.json` belgesi. İlk başvuru sahnenin rig'ini belirler. Sonraki başvurular aynı rig'i kullanmalı. |
| Show Target Poses | Belgenin hedef pozlarını (target pose) sahnede, seçilen renkte çizer: bütün dizileri ya da yanında seçilen tek diziyi |
| Show Interaction Boxes | Belgenin etkileşim kutularını çizer |
| Behavior | **Runtime** altında, örneği Scene Preview'da çalıştıran sınıf, örneğin `VelocityMatchingSequenceBehavior` ya da `SittingBehavior`. Liste, içe aktarılmış her `SmartPrimitiveBehavior` alt sınıfını gösterir. |

Birkaç örnek aynı belgeye başvurabilir. Her örneğin kendi yerleşimi ve davranışı olur. Bir Smart Primitive öğesi başvurusuz ve **Behavior** alanı boş olarak başlar. Scene Preview, her örnekte ikisi de dolu olana kadar sahneyi kabul etmez. Listeye bir sınıfın nasıl ekleneceğini Scene Preview sayfasındaki [Davranış yazın](SAHNE-ONIZLEME.md#davranış-yazın) bölümü gösterir.

Eğitimdeki [Kaçınma Smart Primitive'ini ekleyin](REHBERLI-EGITIM.md#kaçınma-smart-primitiveini-ekleyin) adımı bir örnek yerleştirir ve davranışını seçer. [Parkuru tamamlayın](REHBERLI-EGITIM.md#parkuru-tamamlayın) adımı atlamayı ve oturmayı ekler.

### Solid Cube

**Width**, **Depth**, **Height** ve **Color** küpü ayarlar. Küpler, bir atlamanın altındaki kutu gibi sahne dekorudur. Model için çarpışma geometrisi işlevi görmezler. İkisinin birlikte hareket etmesi için küpü kendi Smart Primitive'inin altına koyun.

## Scene Preview sahneyi nasıl kullanır

Sahneyi **File** > **Save** ile kaydedin, sonra Scene Preview'da açın:

```sh
uv run python -m app.scene_preview --scene path/to/your.scene.json
```

Scene Preview öğe hiyerarşisini dünya dönüşümlerine düzleştirir ve bir motor derlemeden önce her örneği denetler. Ardından her örneğin Smart Primitive'inin kök düğümünü örneğin yerleşimine taşır ve karakteri CharacterStart'ta başlatır. Yüklemeyi Scene Preview sayfasındaki [Sahneler](SAHNE-ONIZLEME.md#sahneler) bölümü anlatır.

## Belge biçimi

Sahne belgeleri düz JSON'dur:

| Alan | İçerik |
| --- | --- |
| `format`, `schema_version` | `motionbricks.scene` ve 1 |
| `name`, `rig` | Dosya adından alınan sahne adı ve rig, `soma` ya da `unitree_g1` |
| `character_start` | Kimlik 1, `transform` ve `euler_degrees` değerleri |
| `smart_primitives` | Her örneğin bir `id`, bir `name`, bir `parent_id` ya da `null`, bir `transform`, `euler_degrees`, bir `reference`, `visibility`, bir `target_pose_color` ve bir `behavior` sınıf adı vardır |
| `solid_cubes` | Her küpün bir `id`, bir `name`, bir `parent_id` ya da `null`, bir `transform`, `euler_degrees`, `dimensions` ve bir `color` değeri vardır |
| `editor` | Seçili öğe ve kamera. Düzenleyici bunları geri yükler. |

Bir `transform`, bir `translation` ve bir `rotation_xyzw` dördeyi (quaternion) tutar. Üst öğeye, en üst düzeyde ise dünya uzayına göredir. Bir `reference` sahne dosyasına göre görelidir, dosyalar farklı sürücülerdeyse mutlak yol olur. Bu dosyaları başka bir program da yazabilir. Başvurulan her Smart Primitive için tek bir rig kullanması gerekir.

## Ek kaynaklar

- [Rehberli eğitim](REHBERLI-EGITIM.md#bölüm-3-kaçınma-atlama-ve-oturma-içeren-bir-etkileşim-sahnesi-kurun): Eğitim sahnesini adım adım kurma.
- [Smart Primitive Editor](SMART-PRIMITIVE-EDITORU.md): Bir sahnenin başvurduğu Smart Primitive belgelerini hazırlama.
- [Scene Preview](SAHNE-ONIZLEME.md): Bir sahnenin nasıl yüklendiği ve davranışlarının nasıl çalıştığı.

---

**Unitree G1 Türkiye:** Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org. [Unitree G1 ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat)
