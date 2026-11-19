> Bu belge NVlabs/motionbricks deposundaki `docs/runtime_api.md` dosyasının Türkçe çevirisidir (CC-BY-4.0).

# Çalışma Zamanı API'si

Bu sayfa, `motionbricks` kütüphanesini kendi uygulamanızda nasıl kullanacağınızı gösterir. Bölümler bir entegrasyonun sırasını izler: bir motor (engine) kurun, pozları hazırlayın, üretin ve üretimleri zincirleyin. `app` paketi kütüphane üzerine kurulu demoları tutar. Bunlardan ikisi kütüphaneyi iş başında gösterir. [Ara Kare Üretici (Inbetweener)](ARA-KARE-URETICI.md) üretimleri elle zincirler, böylece her adım görünür kalır. [Sahne Önizleme (Scene Preview)](SAHNE-ONIZLEME.md), sürekli yürüyüş/hareket (lokomosyon) ve etkileşimler için bir hareket oynatıcı ve bir kaydedici kullanır.

## Genel Bakış

### Üretim akışı

MotionBricks sürekli hareketi her seferinde bir üretimle kurar. Her üretim dört bağlam pozu (context pose) ve bir ile dört arası hedef poz (target pose) alır. Uygulamanız üretilen hareketi geçiş noktasına kadar oynatır ve gösterdiği her pozu kaydeder. Sonraki üretim bağlamını bu kayıtlardan okur ve yeni bir hedefe doğru üretir.

![MotionBricks üretim akışı: motor bağlam ve hedef pozlardan üretir, oynatılan son dört poz bir sonraki bağlam olur](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/generation_flow.png)

| Adım | API | Bölüm |
| --- | --- | --- |
| Pozları hazırlayın | `ClipRegistry`, `root_offset` ile `sample()` ya da uygulamanızdan gelen pozlar | [Pozları sahneye yerleştirin](#pozları-sahneye-yerleştirin), [Klipler](#klipler) |
| Belirteçleri doldurun | `create_input()`, `update_from_poses()`, `update_from_recorder()` | [Motorlar](#motorlar), [Gösterdiğinizi kaydedin](#gösterdiğinizi-kaydedin) |
| Üretin | `generate_animation()`, `MotionBricksInferenceConfig`, `TargetPoseInferenceParams` | [Üretim ayarları](#üretim-ayarları) |
| Oynatın | `sample()`, `output.num_frames`, `sample_foot_contacts()` | [Motorlar](#motorlar), [Ayak temasları](#ayak-temasları) |
| Kaydedin ve zincirleyin | `MotionRecorder`, `GeneratedMotionPlayer`, `time_to_transition` | [Üretimleri zincirleme](#üretimleri-zincirleme) |

Ara Kare Üretici bu döngüyü listesindeki her hedef için bir kez çalıştırır. Sahne Önizleme ise döngüyü yürüyüş ve Smart Primitive (akıllı ilkel) davranışlarından gelen hedeflerle durmadan çalıştırır.

### Çalıştırılabilir örnekler

[examples](https://github.com/NVlabs/motionbricks/tree/main/docs/examples) içindeki iki kısa betik, kütüphaneyi bir görüntüleyici ya da uygulamalar olmadan çağırır. İki iskelet (rig) için de bunları deponun kök dizininden çalıştırın:

```sh
uv run python docs/examples/generate_once.py
uv run python docs/examples/locomotion_loop.py --model unitree_g1
```

- [generate_once.py](https://github.com/NVlabs/motionbricks/blob/main/docs/examples/generate_once.py), klip pozlarından üretir, ardından oynattığı son dört pozdan ikinci bir üretim zincirler. [Üretim akışını](#üretim-akışı) en yalın biçimiyle gösterir.
- [locomotion_loop.py](https://github.com/NVlabs/motionbricks/blob/main/docs/examples/locomotion_loop.py), [sürekli yürüyüş döngüsünü](#sürekli-bir-yürüyüş-döngüsü) bir hareket oynatıcıyla çalıştırır. Sola dönüşte bir hafif koşuyu yönlendirir, hedef pozları istenen hıza göre ölçekler ve onları karakterin önüne yerleştirir.

[_rig.py](https://github.com/NVlabs/motionbricks/blob/main/docs/examples/_rig.py), iki betiğin paylaştığı kurulumu tutar. Yalnız `motionbricks` paketini kullanır, bu yüzden kopyalayabilirsiniz. Her model ilk açılışta bir TensorRT planı derler ve bu birkaç dakika sürer.

### Paketler

| Paket | İçerik |
| --- | --- |
| [motionbricks/model](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/model) | Model sözleşmeleri, motorlar, bağlam belirteçleri, hedef belirteçleri ve üretim ayarları |
| [motionbricks/inference](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/inference) | TensorRT planlarını derleme, önbelleğe alma ve çalıştırma |
| [motionbricks/animation](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/animation), [motionbricks/robotics](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/robotics) | `AnimationBuffer`, `DOFAnimationBuffer`, iskeletler ve ağlar (mesh) |
| [motionbricks/io](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/io) | BVH, CSV ve USD yükleme |
| [motionbricks/runtime](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/runtime) | Hareket oynatıcılar ve kaydediciler, hedef poz yardımcıları, karışım uzayları (blend spaces), çalışma zamanı Smart Primitive'leri ve pivotlar |
| [motionbricks/renderers](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/renderers) | Ağlar, iskeletler ve hata ayıklama şekilleri için Newton görüntüleyici çizicileri |
| [app/runtime](https://github.com/NVlabs/motionbricks/tree/main/app/runtime) | Demo çalışma zamanı: karakterler, durumlar, davranışlar, belgeler ve hazır ayarlar |

Her sınıf ya da fonksiyon kendi `device` argümanını, aldığı bir nesnenin cihazını ya da geçerli Warp cihazını kullanır. Örneğin `create_input()` motorun cihazını kullanır. Bir kaydedici, referans pozunun cihazını kullanır. Bir çizici (renderer) ise iskeletinin ya da Newton modelinin cihazını kullanır.

## Motorlar

Her motor, birlikte gelen bir ONNX modelinden bir TensorRT planı derler ya da önbellekteki planı yükler. Arabelleklerini ve CUDA grafiğini üretimler arasında korur. Motorlar bir CUDA cihazı gerektirir ve işlerini `engine.stream` üzerinde sıralar. Bir motoru bağlam yöneticisi (context manager) olarak kullanın ya da işiniz bittiğinde `close()` çağırın.

```python
from motionbricks.model.soma.contract import SOMA_N1_FP32_ONNX_PATH
from motionbricks.model.soma.engine import MotionBricksSOMAEngine

with MotionBricksSOMAEngine(SOMA_N1_FP32_ONNX_PATH, model_to_runtime=model_to_runtime) as engine:
    context = engine.create_input(skeleton)
    target = engine.create_input(skeleton)
    output = engine.create_output(skeleton)

    context.update_from_poses(context_poses, stream=engine.stream)  # dört poz
    target.update_from_poses(target_poses, stream=engine.stream)    # bir ile dört poz
    engine.generate_animation(context, target, output)

    # output.num_frames, yeniden kullanılan arabelleğin geçerli aralığını işaretler.
    output.sample(time, current_pose, stream=engine.stream)
    engine.synchronize()
```

```python
from motionbricks.model.unitree_g1.contract import UNITREE_G1_N1_FP32_ONNX_PATH
from motionbricks.model.unitree_g1.engine import MotionBricksUnitreeG1Engine

with MotionBricksUnitreeG1Engine(UNITREE_G1_N1_FP32_ONNX_PATH, model_to_runtime=model_to_runtime) as engine:
    context = engine.create_input()
    target = engine.create_input()
    output = engine.create_output()

    context.update_from_poses(context_joint_q, stream=engine.stream)  # dört poz
    target.update_from_poses(target_joint_q, stream=engine.stream)    # bir ile dört poz
    engine.generate_animation(context, target, output)

    # output.num_frames, yeniden kullanılan arabelleğin geçerli aralığını işaretler.
    output.sample(time, robot_joint_q, stream=engine.stream)
    engine.synchronize()
```

| | `MotionBricksSOMAEngine` | `MotionBricksUnitreeG1Engine` |
| --- | --- | --- |
| Bağlam ve hedef belirteçleri | `PoseToken`, yerel eklem dönüşümleri | `DOFToken`, eklem koordinatları |
| Çıktı arabelleği | `AnimationBuffer`, yüklenen BVH klipleriyle aynı tür | `DOFAnimationBuffer`, yüklenen CSV klipleriyle aynı tür |
| Poz | `wp.transform` olarak yerel eklem dönüşümleri, kök önce: eklem başına bir XYZ konumu ve bir XYZW dördeyi (quaternion) | 36 eklem koordinatı: kökün XYZ konumu ve XYZW dördeyi, ardından 29 eklem açısı |
| Model uzayı | Y yukarı, XZ zemin düzlemi | Z yukarı, +X ileri |
| Dünya uzayı | `model_to_runtime` ile belirlenir | `model_to_runtime` ile belirlenir |

Uygulamalar model öznitelikleriyle hiçbir zaman doğrudan çalışmaz. Kliplerden pozlar örnekler, her pozu sahneye yerleştirmek için kökü (root) taşır ve pozları motora geçirir. Eklem değerleri örneklendiği gibi kullanılır. Klipler, Smart Primitive belgeleri, durumlar ve sahneler iki düzenle de çalışır, böylece aynı uygulamalar iki karakteri de sürer. Her iskelete özgü kod küçüktür ve [app/runtime/soma](https://github.com/NVlabs/motionbricks/tree/main/app/runtime/soma) ile [app/runtime/unitree_g1](https://github.com/NVlabs/motionbricks/tree/main/app/runtime/unitree_g1) içinde durur. Her klasör iskeletin karakterini, hedef poz karıştırıcısını ve hazır ayarlarını tutar.

Bir motor bağlamı ve hedefi çıkarım için hizalar, ardından dünya yerleşimini geri yükler. Üretilen hareketin her iki ucundaki üç [sınır karesini](#üretim-uzunluğu) de çıkarır. Bir SOMA motoru, modelin 30 eklemini bir iskelette adlarıyla bulur. Kök sıfır indeksinde olmalıdır. İskelet, SOMA parmakları gibi modelin tahmin etmediği ek eklemler tutabilir. Model SOMA eklem düzenini ve oranlarını kullandığı için başka iskeletli bir karakter yeniden hedefleme (retargeting) adımı ister. G1 motoru doğrudan robotun eklem koordinatları üzerinde çalışır, bu sayede doğrudan bir robotun Newton `model.joint_q` dilimine örnekleyebilirsiniz.

İki motor da [motionbricks/model/engine.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/model/engine.py) içindeki `MotionBricksEngine` sınıfından türer. Bu sınıf kurucuyu ve üretim akışını tutar. Her iskelet motoru yalnız poz düzenine bağlı adımları sağlar: arabellekleri denetleme, bağlamı ve hedefi kodlama, tahmini çözme ve uç nokta düzeltmesi.

Uygulamalar motorlarını [app/utils/model_loading.py](https://github.com/NVlabs/motionbricks/blob/main/app/utils/model_loading.py) içindeki `create_engine()` ile oluşturur. Bu fonksiyon birlikte gelen FP32 ya da karma FP16 modeli seçer ve TensorRT derleme ilerlemesini görüntüleyicide bildirir. Uygulamalar aynı anda tek motor tutar. Öteki iskelete geçince yeni motor yüklenmeden önce geçerli motor kapanır, TensorRT plan önbelleği de yeniden yüklemeyi kısa tutar.

### TensorRT derlemeleri

Her model, duyarlılık (precision) ve toplu iş boyutu ilk açılışta bir TensorRT planı derler ve onu `.trt-cache/` içinde önbelleğe alır. Derlemeyi `TensorRTBuildOptions` denetler:

| Seçenek | Varsayılan | Anlam |
| --- | --- | --- |
| `enable_fp16` | `False` | TensorRT'nin FP16 kullanmasına izin verir. Uygulamalar karma FP16 modeliyle bunu açar. |
| `enable_tf32` | `None` | `None` modelin varsayılanını kullanır. SOMA TF32'ye izin verir. G1 ise kapatır, çünkü [model kartının](UNITREE-G1-MODEL-KARTI.md#çıkarım) açıkladığı gibi TF32, bir boyutlu toplu işte az bir hızlanma karşılığında hareketini değiştirir. Açıkça verilen değer her zaman önceliklidir. |
| `batch_size` | 1 | Sabit bir toplu iş boyutu için TensorRT planı derler |

Bir motorun önbellekteki planını yok saymak için `force_rebuild=True` geçirin. Her seçenek kümesinin kendi önbellek girdisi vardır. Karma FP16 modeller normalleştirme, hareket geometrisi ve öteki hassas işlemler için FP32'yi korur. Girdileri ve çıktıları FP32 kalır. Karma FP16 ve FP32 farklı hareket belirteçleri seçebilir. Üretilen hareket bağlam olarak geri beslendiğinde farklar büyüyebilir.

Öteki toplu iş boyutları ve çıkarım kütüphaneleri için [ONNX çıkarımı](ONNX-CIKARIM.md) belgesine bakın.

### Günlük kaydı (logging)

Kütüphane Python'un standart `logging` modülünü, her modül için bir günlükçüyle (logger) kullanır. TensorRT mesajları `motionbricks.inference.tensorrt` günlükçüsüne gider. Günlük kaydını uygulamanızda her zamanki gibi yapılandırın:

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.getLogger("motionbricks").setLevel(logging.DEBUG)
```

Uygulamalar varsayılan olarak bilgi (info) mesajlarını kaydeder. Ayrıntılı günlük mesajları için herhangi bir uygulama komutuna `--verbose` ekleyin.

## Poz düzenleri ve model uzayı

Uygulamalar Z yukarı olan dünya uzayında çalışır. SOMA model uzayı ve BVH klipleri Y yukarıdır. G1 model uzayı ve CSV klipleri zaten Z yukarıdır.

[app/runtime/common/rig.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/rig.py) içindeki `model_to_runtime(rig)`, her iskeletin uzayını dünya uzayına çevirir. SOMA için Y yukarıyı Z yukarıya döndürür. MuJoCo kuralını izlediği için SOMA, G1 ile aynı yöne bakar. G1 için birim dönüşümdür (identity).

Uygulamalar bu dönüşümü motora ve klip örneklemeye geçirir, böylece üretilen hareket ve klipler aynı uzaya düşer. İki motor da varsayılan olarak birim dönüşümü kullanır. Bu, zaten model uzayında çalışan bir uygulamaya uyar.

### Pozları sahneye yerleştirin

Uygulamanız her bağlam ve hedef pozu dünya uzayına yerleştirir. Bir pozu yerleştirmek için kökünü tek bir dünya dönüşümüyle taşıyın ve öteki eklem değerlerini koruyun. Kök, pozun nerede durduğunu ve hangi yöne baktığını belirler. SOMA için kök ilk yerel eklem dönüşümüdür. G1 için ilk yedi eklem koordinatıdır: kökün XYZ konumu ve XYZW dördeyi.

Bir klibi örneklerken yerleşimi `root_offset` olarak geçirin. Kaynak koordinatlarındaki bir klip ayrıca, önce uygulanan `model_to_runtime` ister:

```python
clip.sample(sample_time, pose, root_offset=wp.mul(placement, model_to_runtime), stream=engine.stream)
```

`root_offset` ve `stream` değerlerini anahtar sözcükle geçirin. Elinizde zaten olan bir poz için kökünü yerleşimle çarpın. [target_pose.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/runtime/target_pose.py) içindeki `place_target_poses()` ve `place_target_dof()` bunu her poz düzeni için yapar.

Uygulamalar pozları dört yerde yerleştirir:

| Nerede | Neyi yerleştirir | Kod | Ayrıntı |
| --- | --- | --- | --- |
| Ara Kare Üretici | Başlangıcı ve her hedefi, gizmodan ve **Transform** alanlarından | [inbetweener_demo.py](https://github.com/NVlabs/motionbricks/blob/main/app/inbetweener_demo.py) içindeki `sample_pose_window()` | [Bir pozu sahneye yerleştirin](ARA-KARE-URETICI.md#bir-pozu-sahneye-yerleştirin) |
| Yürüyüş | Karıştırılmış hedef pozları, karakterin önüne ve baktığı yöne çevrilmiş olarak | `place_target_poses()` ve `place_target_dof()` | [Hedefi karıştırın ve yerleştirin](SAHNE-ONIZLEME.md#hedefi-karıştırın-ve-yerleştirin) |
| Smart Primitive'ler | Hazırlanmış hedef pozları, örneğin yerleşimi ve düğüm hiyerarşisi üzerinden | [smart_primitive.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/runtime/smart_primitive.py) içindeki `get_world_target_poses()` | [Sahne pozları nasıl yerleştirir](SMART-PRIMITIVE-EDITORU.md#sahne-pozları-nasıl-yerleştirir) |
| Klipler | Bekleme ve etkileşim kliplerini, karakterin önüne ya da geçerli kökünün yerine | [character.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/character.py) içindeki `generate_clip_transition()` ve `start_clip()` | [Klipler ve üretilen hareket](#klipler-ve-üretilen-hareket) |

Yalnız kök hareket ettiği için tek bir örneklenmiş poz kümesi her yerleşime hizmet eder. Rehberli eğitimin Entry ve Duck-under hedeflerini gövde pozlarını değiştirmeden gizmoyla taşıyabilmesinin nedeni budur.

### Klipler

[ClipRegistry](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/clip_registry.py), tek bir iskelet için klipleri yükler ve önbelleğe alır. SOMA için BVH, Unitree G1 için CSV kliplerini okur:

```python
from app.runtime.common.rig import load_rig_assets
from motionbricks.model.contract import ModelRig

assets = load_rig_assets(ModelRig.SOMA)
clip = assets.clip_registry.load("assets/motions/soma/locomotion/idle/idle_loop_001__A268.bvh").animation
```

`load_rig_assets()`, bir `RigAssets` döndürür. Bunun içinde iskeletin iskelet ağı (skeletal mesh) ya da Newton modeli, klip kaydı, arabellek düzenini belirleyen bir referans poz ve `model_to_runtime` dönüşümü bulunur. Klipler pozların yalnız bir kaynağıdır. Doğru düzendeki her poz bağlam ya da hedef olarak çalışır.

G1 CSV klipleri kökü santimetre, açıları derece olarak 120 Hz'de saklar. Yükleyici bunları metreye, XYZW dördeylerine ve radyana çevirir.

## Üretim

`generate_animation()` bağlam ve hedef belirteçlerini okur ve çıktı arabelleğini üretilen hareket ve ayak temaslarıyla doldurur. İki tür ayar onu ince ayarlar: tüm üretim için ayarlar ve her hedef için ayarlar.

### Üretim ayarları

`generate_animation()` isteğe bağlı bir `MotionBricksInferenceConfig` kabul eder:

| Alan | Varsayılan | Anlam |
| --- | --- | --- |
| `allowed_token_counts` | 8 ile 14 | Modelin seçebileceği uzunluklar, dört pozluk 6 ile 16 belirteç arası |
| `correct_generated_diff` | `True` | Küçük düzeltmeleri hareket boyunca yayar, böylece hareket son bağlam pozunda başlar ve hedefte biter |
| `heading_alignment` | (1, 0) | Çıkarımın son bağlam pozunu çevirdiği, model uzayındaki zemin yönü: SOMA için (x, z), G1 için (x, y). (1, 0) +X yönüne bakar. Her model uzayını [SOMA](SOMA-MODEL-KARTI.md) ve [Unitree G1](UNITREE-G1-MODEL-KARTI.md) model kartları anlatır. |

### Her hedef için ayarlar

Her hedef, [target_pose.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/runtime/target_pose.py) içindeki `TargetPoseInferenceParams` ile kendi ayarlarını da taşır. Ara Kare Üretici ve Smart Primitive Editörü'nde her hedef için düzenlediğiniz ayarlar bunlardır. Hareket oynatıcılar bunları `generate()` içinde okur.

| Ayar | Alan | Etki |
| --- | --- | --- |
| **Poses to Use** | `num_poses_to_use` | Hedefin dört pozundan kaçının modeli bilgilendireceği, 1 ile 4 arası. Tek poz varış noktasını, karakterin durduğu duruşu ve yönü belirler. İkinci poz hız ekler. Üçüncü ve dördüncü poz ivme ve momentum ekler. |
| **Transition Early By** | `transition_early_by_frames` | Geçiş noktasını üretilen hareketin sonundan bu kadar kare önceye taşır. Daha büyük bir değer bu hedefi bir kılavuza çevirir. Karakter ona doğru hizalanır, sonra varmadan önce daha kesin bir sonraki hedefe geçer. |
| **Tokens to Generate** | `allowed_token_counts` | Bu hedef için üretim ayarlarındaki izin verilen belirteç sayılarının yerini alır. `None` onları korur. [Üretim uzunluğu](#üretim-uzunluğu) bölümünün açıkladığı gibi tek bir sayı, hedefe kaç kare üretileceği üzerinde tam denetim verir. |
| **IK Pins** | `ik_pin_mask` | Yalnız SOMA. Üretilen hareketin oynatımı hedefe yaklaşırken `TargetPoseIKCorrector`, sabitlenen (pinned) elleri ve ayakları iki kemikli IK ile yumuşakça hedefe oturtur. Bkz. [IK Pins](SMART-PRIMITIVE-EDITORU.md#ik-pins). |

### Üretim uzunluğu

Her geçişin ne kadar süreceğine model karar verir. Her belirteç 30 Hz'de dört poz tutar. Tek bir üretim 6 ile 16 belirteç, yani 24 ile 64 ham kare üretir. İlk belirteç bağlamı, son belirteç hedefi tutar. Motor her iki uçtan üçer sınır karesi çıkarır. Üretilen hareket bundan sonra son bağlam pozunda başlar ve ilk hedef pozda biter. Geriye 18 ile 58 geçerli kare, yani yaklaşık 0,6 ile 1,9 s hareket kalır. `output.num_frames`, son üretimin geçerli kare sayısını tutar.

### Ayak temasları

İki model de hareketle birlikte ayak temaslarını tahmin eder. Her kare dört değer tutar. Her değer, bir topuğun ya da parmak ucunun zemine bastığına dair 0 ile 1 arasında bir güven değeridir. Motorlar ham model değerlerini bu aralığa sıkıştırır (clamp). Sütunlar şunlardır:

| Sütun | Temas | SOMA eklemi | G1 noktası |
| --- | --- | --- | --- |
| 0 | Sol topuk | `LeftFoot` | Sol ayak bileği yuvarlanma (ankle roll) eklemi |
| 1 | Sol parmak ucu | `LeftToeBase` | Sol ayak bileği yuvarlanma ekleminin 0,14 m önü ve 0,035 m altı |
| 2 | Sağ topuk | `RightFoot` | Sağ ayak bileği yuvarlanma eklemi |
| 3 | Sağ parmak ucu | `RightToeBase` | Sağ ayak bileği yuvarlanma ekleminin 0,14 m önü ve 0,035 m altı |

G1 parmak ucu noktaları her ayak bileği yuvarlanma bağlantısının yerel uzayındadır. [soma/contract.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/model/soma/contract.py) içindeki `SOMA_N1_FOOT_CONTACT_JOINTS`, SOMA eklemlerini listeler. [unitree_g1/contract.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/model/unitree_g1/contract.py) içindeki `UNITREE_G1_N1_FOOT_CONTACT_BODIES` ve `UNITREE_G1_N1_FOOT_CONTACT_OFFSETS`, G1 gövdelerini ve noktalarını listeler.

Üretilen hareket ayak temaslarını `output.foot_contacts` içinde tutar. Bu, `[capacity, 4]` şeklinde bir FP32 dizisidir. İlk `output.num_frames` karesi geçerlidir. `generate_animation()` bu diziyi gerektirir, `create_output()` da onu ayırır. Dosyadan yüklenen kliplerde ayak teması bulunmaz, `foot_contacts` değerleri `None` olur.

`sample_foot_contacts()`, `sample()` pozları nasıl karıştırıyorsa dört değeri bir oynatma zamanında öyle karıştırır. Ara Kare Üretici ve Sahne Önizleme'deki **Show Foot Contacts** seçeneği örneklenen değerleri çizer.

[foot_contact_renderer.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/renderers/foot_contact_renderer.py) içindeki `skeletal_mesh_sole_outlines()` ve `body_sole_outlines()`, bu noktaları topuk ve parmak ucu ana hatlarına çevirir. `SkeletonFootContactRenderer` ana hatları bir SOMA pozu üzerine, `DOFFootContactRenderer` ise bir G1 pozu üzerine çizer. Uygulamalar her iskeletin ağ, iskelet ve ayak teması çizicilerini [rig.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/rig.py) içindeki `create_mesh_renderer()`, `create_skeleton_renderer()` ve `create_foot_contact_renderer()` ile oluşturur.

## Üretimleri zincirleme

Tek bir üretim kısadır. [Rehberli eğitimin](REHBERLI-EGITIM.md#üretimleri-zincirleme) açıkladığı gibi sürekli hareket üretimleri zincirlemekten gelir. Her yeni bağlam, gerçekten gösterilen pozları tutmalıdır. Aksi halde sonraki üretim başka bir pozdan başlar ve karakter sıçrar. Ara Kare Üretici elle zincirler, bu da her çağrıyı gösterir.

[motionbricks/runtime](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/runtime) içindeki iki parça, bu kalıbı gerçek zamanlı kullanım için uygular. `MotionRecorder` gösterilen pozları saklar. Hareket oynatıcılar bu pozlardan üretir ve sonucu oynatır. Sahne Önizleme [her durumu](SAHNE-ONIZLEME.md#kaydedilen-hareketten-üretin) bunların üzerine kurar. Onları bir referans olarak okuyun, sonra aynı kalıbı kendi uygulamanızda yeniden kurun.

### Ara Kare Üretici'nin yaklaşımı

Ara Kare Üretici, [üretimleri](ARA-KARE-URETICI.md#oynatma-ve-zincirleme-üretim) bir oynatıcı olmadan zincirler. Her geçiş noktasında, üretilen hareketin oynatılmış son dört karesini bağlama kopyalar. Bu yöntem işe yarar, çünkü her parça kendi geçiş noktasına kadar oynar ve yol boyunca pozu hiçbir şey değiştirmez.

### Gösterdiğinizi kaydedin

[MotionRecorder](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/runtime/motion_recorder.py), sabit sayıda son pozu en eskisi önde olacak şekilde saklar. Her pozdan bir sonrakine geçen süreyi de saklar. Uygulamanızın her güncellemesinde bir poz kaydedin. Güncelleme hızının modelin 30 Hz'ine eşit olması gerekmez.

```python
from motionbricks.runtime.motion_recorder import MotionRecorder

history = MotionRecorder(skeleton.reference_local_transforms, capacity=32, stream=engine.stream)
history.fill(start_pose, stream=engine.stream)

# Her güncellemede, poza yapılan her değişiklikten sonra:
history.record(delta_time, current_pose, stream=engine.stream)

# Her üretimden önce:
context.update_from_recorder(history, stream=engine.stream)
```

G1 için iskeletin referans yerel dönüşümleri yerine robotun referans eklem koordinatlarını geçirin. `fill()` her yuvayı tek bir poza ayarlar, sanki karakter o pozu tutmuş gibi. İlk üretimden önce çağırın.

`update_from_recorder()`, en son kaydedilen pozda biten dört bağlam pozunu 30 Hz'de örnekler. Her örnek zamanının çevresindeki iki kayıtlı pozu karıştırır. Dört poz 0,1 s'yi kapsar. Varsayılan 32 poz, bu aralığı yaklaşık 300 Hz'e kadarki güncelleme hızlarında karşılar.

Uygulamanızın poza yaptığı her değişiklikten sonra pozu kaydedin. Klip kareleri, hedef poz IK'si ve fizik bunlara dahildir. Kaydedici bir pozun nereden geldiğini izlemez. Bir üretimin bir klipten ya da bir parçanın ortasından başlayabilmesini sağlayan da budur.

### Üretilen hareketi oynatın

[motion_player.py](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/runtime/motion_player.py), bir motoru bir kaydediciyle eşler. `GeneratedMotionPlayer` SOMA'ya, `GeneratedDOFMotionPlayer` G1'e hizmet eder. Her oynatıcı kendi bağlam belirtecini, hedef belirtecini, çıktı arabelleğini ve kaydedicisini oluşturur.

| Üye | Kullanım |
| --- | --- |
| `seed_history_from_pose(pose)` | Geçmişi ve bağlamı tek bir pozla doldurur. İlk üretimden önce çağırın. |
| `generate(target_poses, inference_params)` | Bağlam pozlarını geçmişten okur, hedefe doğru üretir ve oynatmayı yeniden başlatır. İsteğe bağlı bir `config`, öteki üretim ayarlarını tutar. |
| `sample(delta_time, pose, foot_contacts)` | Oynatma zamanını ilerletir, sonra pozu ve ayak temaslarını yazar. |
| `record_pose(delta_time, pose)` | Gösterdiğiniz pozu kaydeder. |
| `time_to_transition` | Geçiş noktasına kalan saniye. |

Ne zaman üretileceğine oynatıcı karar vermez. `generate()` çağrısını kodunuz yapar, genellikle `time_to_transition` bir sonraki güncellemenin süresinin altına düştüğünde. Örnekleme geçiş noktasından sonra da sürebilir. Geçerli karelerin sonunda poz sabit kalır.

### Sürekli bir yürüyüş döngüsü

Bu taslak bir SOMA karakterini bir oynatıcıyla sürer. `input_changed()` ve `place_target()` sizin kendi kodunuzun yerini tutar.

```python
import warp as wp

from motionbricks.runtime.motion_player import GeneratedMotionPlayer
from motionbricks.runtime.target_pose import TargetPoseInferenceParams

player = GeneratedMotionPlayer(engine, skeleton)
player.seed_history_from_pose(start_pose)
current_pose = wp.empty_like(start_pose)
foot_contacts = wp.zeros(4, dtype=wp.float32, device=engine.device)
target_poses = wp.empty((4, skeleton.num_joints), dtype=wp.transform, device=engine.device)
settings = TargetPoseInferenceParams(transition_early_by_frames=10, num_poses_to_use=4)


def update(delta_time, desired_velocity):
    if player.time_to_transition <= delta_time or input_changed(desired_velocity):
        place_target(target_poses, desired_velocity)
        player.generate(target_poses, settings)
    player.sample(delta_time, current_pose, foot_contacts)
    # Hedef poz IK'sini, fiziği ya da current_pose üzerindeki öteki değişiklikleri burada uygulayın.
    player.record_pose(delta_time, current_pose)
```

Her `update()` çağrısı [üretim akışının](#üretim-akışı) bir turunu çalıştırır. [locomotion_loop.py](https://github.com/NVlabs/motionbricks/blob/main/docs/examples/locomotion_loop.py), bu döngüyü `input_changed()` ve `place_target()` doldurulmuş olarak çalıştırır. Uygulamanız döngüde iki karar verir:

- **Ne zaman üretileceği.** En geç oynatma geçiş noktasına vardığında üretin. Hareket girdisi yeterince değiştiğinde daha erken yeniden üretin, böylece karakter hızlı tepki verir. Sahne Önizleme'nin kuralları [Yürüyüş ne zaman üretir](SAHNE-ONIZLEME.md#yürüyüş-ne-zaman-üretir) bölümündedir.
- **Hedefin nereye yerleştirileceği.** İstenen hıza uyan hedef pozları örnekleyin. Onları karakterin önüne, istenen hız yönünde yerleştirin ve yalnız kökü taşıyın. Dört hedef pozun hepsini kullanın, böylece model hedefteki hıza da uyar. Sahne Önizleme, [Hedefi karıştırın ve yerleştirin](SAHNE-ONIZLEME.md#hedefi-karıştırın-ve-yerleştirin) bölümünde gösterildiği gibi hedef pozlarını bir karışım uzayında karıştırır.

### Klipler ve üretilen hareket

Kaydedici gösterilen her pozu sakladığı için klipler ve üretilen hareket serbestçe karışır. [character.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/character.py) içindeki `BaseCharacter` bunu iki yönde kullanır:

- **Klipten çıkış.** `start_clip()` bir klibi oynatır, `step()` de karelerini kaydetmeyi sürdürür. Sonraki üretim, klibin hareketinden herhangi bir karede başlar.
- **Klibe giriş.** `generate_clip_transition()` klibi karakterin önüne yerleştirir. Klibin ilk dört pozunu hedef olarak örnekler ve sıfır **Transition Early By** değeriyle onlara doğru üretir. Klip geçişi bitince durum klibi başlatır.

Bekleme (idle) durumu ikisini de kullanır. Döngüsel bir bekleme klibine doğru üretir, yürüyüş de daha sonra ondan çıkarak üretir.

### Bir yaklaşım seçin

| | Ara Kare Üretici | Hareket oynatıcı |
| --- | --- | --- |
| Bağlam pozları | Üretilen hareketin oynatılmış son dört karesi | Kayıtlı geçmişten 30 Hz'de örneklenen dört poz |
| Ne zaman üretilir | Her geçiş noktasında | Herhangi bir güncellemede |
| Kliplerden, IK'den ya da fizikten gelen pozlar | Bağlama hiç ulaşmaz | Gösterilen her poz gibi bağlama ulaşır |

Her çağrıyı görmek için Ara Kare Üretici ile başlayın. Uygulamanız erken yeniden ürettiği, klip oynattığı ya da örneklemeden sonra pozu değiştirdiği anda bir kaydediciye geçin.

## Ek kaynaklar

- [Ara Kare Üretici](ARA-KARE-URETICI.md): bu API'nin adım adım bir entegrasyonu.
- [Sahne Önizleme](SAHNE-ONIZLEME.md): hareket oynatıcılar üzerine kurulu; durumlar, yürüyüş ve davranışlar içeren bir karakter çalışma zamanı.
- [Rehberli eğitim](REHBERLI-EGITIM.md): bu API'yi uygulamalar üzerinden kullanan, kullanıcıya dönük iş akışı.
- [ONNX çıkarımı](ONNX-CIKARIM.md): doğrudan ONNX arayüzleri, toplu işleme ve öteki çıkarım kütüphaneleri.
- [SOMA model kartı](SOMA-MODEL-KARTI.md): SOMA girdileri, çıktıları, koordinatları ve sınırlamaları.
- [Unitree G1 model kartı](UNITREE-G1-MODEL-KARTI.md): G1 eklem düzeni, girdileri, çıktıları ve sınırlamaları.

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
