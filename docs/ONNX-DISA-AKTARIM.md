> Bu belge NVlabs/motionbricks deposundaki `tools/onnx/README.md` dosyasının Türkçe çevirisidir (CC-BY-4.0).

# MotionBricks kontrol noktalarını ONNX'e dışa aktarma

Hugging Face'teki SOMA ve Unitree G1 kontrol noktalarını (checkpoints), FP32 ya da karma FP16 duyarlılıkta toplu iş (batch) destekli ONNX modellerine dışa aktarın. Dışa aktarıcının kodu orijinal depodaki [tools/onnx](https://github.com/NVlabs/motionbricks/tree/main/tools/onnx) klasöründedir.

## Dışa aktarma

[uv](https://docs.astral.sh/uv/getting-started/installation/) ve
[Hugging Face CLI](https://huggingface.co/docs/huggingface_hub/en/guides/cli) kurun.
Kontrol noktaları, Hugging Face'teki tek bir
[MotionBricks deposunda](https://huggingface.co/nvidia/MotionBricks) klasörler olarak durur:

| Model | Kontrol noktası klasörü |
| --- | --- |
| SOMA | [soma/](https://huggingface.co/nvidia/MotionBricks/tree/main/soma) |
| Unitree G1 | [g1/](https://huggingface.co/nvidia/MotionBricks/tree/main/g1) |

Bu komutları orijinal deponun kök dizininden çalıştırın. İndirilen klasör yapısını olduğu gibi koruyun.
uv, dışa aktarıcıyı ve bağımlılıklarını ilk kullanımda kendiliğinden kurar.

```sh
hf download nvidia/MotionBricks --local-dir checkpoints/MotionBricks

# SOMA
uv run --locked --project tools/onnx motionbricks-onnx --model soma --checkpoints checkpoints/MotionBricks/soma

# Unitree G1
uv run --locked --project tools/onnx motionbricks-onnx --model unitree_g1 --checkpoints checkpoints/MotionBricks/g1
```

Tek bir model indirmek için `--include "soma/*"` ya da `--include "g1/*"` ekleyin. Kontrol noktalarını yalnız
dışa aktarma ya da başka bir çalışma zamanı için indirin. Demo uygulamaları yazılımla birlikte gelen ONNX modellerini kullanır.

Varsayılan FP32'dir. Komut, `mmm_v6_soma_fp32.onnx` ya da `mmm_v6_g1_fp32.onnx` dosyasını
geçerli dizine kaydeder. Karma FP16 için `--precision fp16` ekleyin. Bu durumda dosya adı `_fp16.onnx` ile biter.

Farklı bir hedef seçmek için `--output path/to/model.onnx`, var olan bir dosyanın yerine yazmak için
`--force` kullanın. Dışa aktarıcı geçici dosyalarını çıktı klasöründe hazırlar. Orada birkaç GB
boş disk alanı ayırın.

Ayrıntılı günlük mesajları için `--verbose` ekleyin.

## Çıktı ve denetimler

Modeller farklı toplu iş boyutlarını destekler, demolar bir boyutlu toplu iş (`N=1`) kullanır.
Kayan noktalı girdiler ve çıktılar iki duyarlılıkta da FP32 kullanır. Şekiller ve kullanım örnekleri için
[ONNX çıkarımı](ONNX-CIKARIM.md), [SOMA model kartı](SOMA-MODEL-KARTI.md) ve
[G1 model kartı](UNITREE-G1-MODEL-KARTI.md) belgelerine bakın.

Dışa aktarıcı kontrol noktası sağlama toplamlarını (checksums), dışa aktarılan ONNX grafiğini ve tensör arayüzünü denetler.
Geçersiz kontrol noktaları, hiçbir ağırlık yüklenmeden dışa aktarmayı durdurur. Var olan bir çıktının yerine
yalnız dışa aktarma başarıyla bittikten sonra yazılır.

Her kontrol noktası SafeTensors ağırlıklarını ve `.npy` dosyalarında normalleştirme istatistiklerini tutar.
Dışa aktarıcı yalnız bu biçimleri okur.

## Lisans

Kaynak kod, deponun [Apache 2.0 lisansını](https://github.com/NVlabs/motionbricks/blob/main/LICENSE) kullanır. Bağımlılık lisansları
[licenses/](https://github.com/NVlabs/motionbricks/tree/main/tools/onnx/licenses) içindedir.

İndirilen kontrol noktaları
[NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) ile lisanslanır.

NVIDIA Corporation tarafından NVIDIA Open Model Agreement kapsamında lisanslanır.

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
