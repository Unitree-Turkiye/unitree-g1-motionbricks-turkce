> Bu belge NVlabs/motionbricks deposundaki `docs/onnx_inference.md` dosyasının Türkçe çevirisidir (CC-BY-4.0).

# ONNX çıkarımı ve toplu işleme (batching)

Bu sayfa, MotionBricks ONNX modellerini birlikte gelen motorlar olmadan çalıştırmak içindir: örneğin başka bir çıkarım kütüphanesinde ya da tek bir toplu işte (batch) aynı anda birçok karakter için. [Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md) motorları, TensorRT ile tek bir karakteri karşılar.

MotionBricks her model ve duyarlılık (precision) için bir ONNX dosyası içerir. Dördü de bir boyutlu toplu iş dahil, değişken sayıda bağımsız isteği kabul eder.

| Model | FP32 | Karma FP16 | Arayüz |
| --- | --- | --- | --- |
| SOMA | [mmm_v6_soma_fp32.onnx](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/soma/mmm_v6_soma_fp32.onnx) | [mmm_v6_soma_fp16.onnx](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/soma/mmm_v6_soma_fp16.onnx) | [SOMA model kartı](SOMA-MODEL-KARTI.md) |
| Unitree G1 | [mmm_v6_g1_fp32.onnx](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/unitree_g1/mmm_v6_g1_fp32.onnx) | [mmm_v6_g1_fp16.onnx](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/unitree_g1/mmm_v6_g1_fp16.onnx) | [Unitree G1 model kartı](UNITREE-G1-MODEL-KARTI.md) |

Her klasör ayrıca bir model bildirimi (manifest) tutar: [soma_n1.json](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/soma/soma_n1.json) ve [unitree_g1_n1.json](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/unitree_g1/unitree_g1_n1.json). Bunlarda eklem adları ve bir boyutlu toplu iş için tensör tanımları bulunur.

Dosyalar Git LFS kullanır. İndirmek için klonladıktan sonra `git lfs pull` çalıştırın. Onları kontrol noktalarından yeniden oluşturmak için [dışa aktarıcıyı](ONNX-DISA-AKTARIM.md) kullanın. Modeller, [LICENSE.md](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/LICENSE.md) dosyasında yazdığı gibi NVIDIA Open Model Agreement ile lisanslanır.

## Tensörler

`N`, toplu işteki istek sayısıdır. Her tensörün ne tuttuğunu ve nasıl hazırlanacağını model kartları açıklar.

| Model | Girdiler | Çıktılar |
| --- | --- | --- |
| SOMA | <ul><li><code>global_root_values</code> [N, 8, 5]</li><li><code>local_root_values</code> [N, 8, 4]</li><li><code>local_poses</code> [N, 8, 267]</li><li><code>has_global_root_values</code> [N, 8]</li><li><code>has_local_root_values</code> [N, 8]</li><li><code>has_local_poses</code> [N, 8]</li><li><code>allowed_pred_num_tokens</code> [N, 11]</li></ul> | <ul><li><code>pred_global_motions</code> [N, 64, 366]</li><li><code>pred_num_tokens</code> [N]</li></ul> |
| Unitree G1 | <ul><li><code>context_qpos</code> [N, 4, 36]</li><li><code>target_qpos</code> [N, 4, 36]</li><li><code>has_global_root_values</code> [N, 8]</li><li><code>has_local_root_values</code> [N, 8]</li><li><code>has_local_poses</code> [N, 8]</li><li><code>allowed_pred_num_tokens</code> [N, 11]</li></ul> | <ul><li><code>pred_qpos</code> [N, 64, 36]</li><li><code>pred_foot_contacts</code> [N, 64, 4]</li><li><code>pred_num_tokens</code> [N]</li></ul> |

Kayan noktalı girdiler ve çıktılar iki duyarlılıkta da FP32 kullanır. Maskeler INT32 kullanır, kapalı için 0, açık için 1. `allowed_pred_num_tokens`, 6 ile 16 arasındaki her belirteç (token) sayısı için bir INT32 bayrağı tutar. Her istek en az bir belirteç sayısına izin vermelidir.

## Bir toplu iş hazırlayın

Bir üretimin tüm girdileri aynı pozitif `N` değerini kullanmalıdır. Öteki boyutları sabittir. Her isteğin kendi pozları, maskeleri ve izin verilen belirteç sayıları vardır. İstekler birbirinden bağımsız işlenir.

ONNX Runtime, MotionBricks bağımlılıkları arasında yer almaz. Betiğinizi çalıştırırken ekleyin, örneğin `uv run --with onnxruntime python my_batch.py` ile.

ONNX Runtime ile her isteği bir boyutlu toplu iş olarak hazırlayın. Ardından istekleri 0 ekseni boyunca birleştirin:

```python
import numpy as np
import onnxruntime as ort

session = ort.InferenceSession(
    "motionbricks/assets/onnx/unitree_g1/mmm_v6_g1_fp32.onnx",
    providers=["CPUExecutionProvider"],
)

# requests, her biri bir boyutlu toplu iş olan tam girdi sözlüklerini içerir.
inputs = {
    name: np.concatenate([request[name] for request in requests], axis=0)
    for name in requests[0]
}
outputs = dict(zip(
    [output.name for output in session.get_outputs()],
    session.run(None, inputs),
))

# Her isteğin ham karelerini okuyun.
motions = [
    outputs["pred_qpos"][i, :4 * int(count)]
    for i, count in enumerate(outputs["pred_num_tokens"])
]
```

ONNX arayüzü `N` için sabit bir üst sınır koymaz. Pratikteki toplu iş boyutlarını kullanılabilir bellek ve ONNX Runtime ya da TensorRT gibi çıkarım kütüphanesi belirler.

## Çıktıları okuyun

`i` isteği, 30 Hz'de 24 ile 64 arasında `4 * pred_num_tokens[i]` ham kare üretir. Çıktı alanının geri kalanı kullanılmaz. İlk belirteç bağlam pozlarını (context poses), son belirteç hedef pozları (target poses) tutar.

G1 modeli ayak temaslarını `pred_qpos` ile aynı kareler için `pred_foot_contacts` içinde döndürür. SOMA modelinde ayrı bir ayak teması çıktısı bulunmaz. Onun `pred_global_motions` çıktısı hareketi ve ayak temaslarını birlikte tutar, SOMA motoru da ikisini birden çözer.

ONNX çıktısı [sınır karelerini](CALISMA-ZAMANI-API.md#üretim-uzunluğu) içerir ve uç nokta düzeltmesi uygulamaz. SOMA ve G1 motorları sınır karelerini çıkarır ve uç nokta düzeltmesini uygular. Bu yüzden onların hareketi tam olarak son bağlam pozunda başlar ve ilk hedef pozda biter. Bir toplu işte her isteği ayrı ayrı kırpın, düzeltin ve yerleştirin.

## TensorRT ve demolar

Demolar TensorRT planlarını bir boyutlu toplu işle derler. Cihaz arabelleklerini ve CUDA grafiğini üretimler arasında yeniden kullanırlar.

Genel `InferenceEngine`, başka bir sabit toplu iş boyutu için TensorRT planı derlemek üzere `TensorRTBuildOptions(batch_size=N)` kabul eder. Plan önbelleği her boyut için ayrı bir plan saklar. SOMA ve G1 motor sınıfları planlarını bir boyutlu toplu işe göre denetler. Daha büyük bir toplu işte `InferenceEngine` sınıfını doğrudan çalıştırın. Bu durumda her isteğin girdilerini hazırlamak ve çıktılarını okumak uygulamanızın işidir.

Tek bir TensorRT planıyla üretimler arasında toplu iş boyutunu değiştirmek için en iyileme profilleri (optimization profiles) oluşturun ve tensör şekillerini ve arabellekleri uygulamanızda yönetin.

## Çıkarım performansı

Tablo, tek bir toplu iş için yalnız TensorRT planının GPU süresini gösterir, parantez içinde saniyedeki istek sayısı yer alır. Girdi hazırlama, çıktı çözme ve uç nokta düzeltmesi bu sürelere dahil değildir.

| Model | N = 1 | N = 8 | N = 16 | N = 32 |
| --- | --- | --- | --- | --- |
| SOMA, FP32 | 2,50 ms (400) | 2,96 ms (2.706) | 3,74 ms (4.280) | 5,28 ms (6.063) |
| SOMA, karma FP16 | 1,28 ms (778) | 1,54 ms (5.179) | 1,87 ms (8.535) | 3,93 ms (8.138) |
| Unitree G1, FP32 | 2,67 ms (375) | 3,78 ms (2.116) | 5,17 ms (3.096) | 7,81 ms (4.097) |
| Unitree G1, karma FP16 | 1,63 ms (613) | 1,91 ms (4.194) | 2,36 ms (6.783) | 3,38 ms (9.449) |

Planlar, çalışma zamanının kullandığı sürüm olan TensorRT 10.8.0.43 içindeki `trtexec` ile, Windows altında bir NVIDIA GeForce RTX 4090 üzerinde ölçüldü. Girdiler zaten GPU üzerindeydi ve demolardaki gibi her çıkarımı bir CUDA grafiği yeniden oynattı. Her toplu iş tek bir gerçek isteği tekrarlar. Karma FP16 çalıştırmaları, demolardaki gibi FP16 çekirdeklerini açar. Her değer altı plan derlemesinin en hızlısından gelir ve ölçülen her plan aynı istekte ONNX Runtime ile eşleşti. Süreler bir derlemeden ötekine değişir, en çok da karma FP16 modellerde.

G1 çalıştırmaları, çalışma zamanındaki gibi TF32'yi kapatır. Bu durumda FP32 evrişimleri (convolutions) ve matris çarpımları Tensor Core olmadan çalışır, bu yüzden G1 FP32 toplu iş büyüdükçe daha çok yavaşlar. Karşılaştırma için:

| Unitree G1, FP32 | N = 16 | N = 32 |
| --- | --- | --- |
| TF32 kapalı, çalışma zamanındaki gibi | 5,17 ms (3.096) | 7,81 ms (4.097) |
| TF32 açık | 4,06 ms (3.936) | 5,78 ms (5.537) |

TF32'nin neden kapalı kaldığını Unitree G1 model kartının [Çıkarım](UNITREE-G1-MODEL-KARTI.md#çıkarım) bölümü açıklar.

Kendi GPU'nuzu ölçmek için `trtexec` komutunu toplu iş boyutu `--shapes` içinde olacak şekilde çalıştırın. Bu komutlar FP32 modellerini 8 boyutlu toplu işle ölçer.

SOMA:

```sh
trtexec --onnx=motionbricks/assets/onnx/soma/mmm_v6_soma_fp32.onnx --shapes=global_root_values:8x8x5,has_global_root_values:8x8,local_root_values:8x8x4,has_local_root_values:8x8,local_poses:8x8x267,has_local_poses:8x8,allowed_pred_num_tokens:8x11 --useCudaGraph --noDataTransfers
```

Unitree G1, çalışma zamanındaki gibi TF32 kapalı:

```sh
trtexec --onnx=motionbricks/assets/onnx/unitree_g1/mmm_v6_g1_fp32.onnx --shapes=context_qpos:8x4x36,target_qpos:8x4x36,has_global_root_values:8x8,has_local_root_values:8x8,has_local_poses:8x8,allowed_pred_num_tokens:8x11 --useCudaGraph --noDataTransfers --noTF32
```

Karma FP16 modeller için `_fp16.onnx` dosyasını kullanın ve `--fp16` ekleyin. `--loadInputs` gerçek istekler vermedikçe `trtexec` girdileri rastgele değerlerle doldurur. `trtexec`, [TensorRT indirme sayfasındaki](https://developer.nvidia.com/tensorrt/download) TensorRT paketleriyle gelir.

## Ek kaynaklar

- [SOMA model kartı](SOMA-MODEL-KARTI.md) ve [Unitree G1 model kartı](UNITREE-G1-MODEL-KARTI.md): her tensörün ne tuttuğu, eklem sırası ve model uzayları.
- [Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md): bu tensörleri bir boyutlu toplu iş için hazırlayan motorlar.
- [ONNX dışa aktarıcı](ONNX-DISA-AKTARIM.md): ONNX dosyalarını kontrol noktalarından yeniden oluşturma.

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
