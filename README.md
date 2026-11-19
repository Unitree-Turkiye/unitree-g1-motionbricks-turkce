<div align="center">

# NVIDIA MotionBricks Türkçe: Unitree G1 İnsansı Robot için Gerçek Zamanlı Hareket Üretimi

**MotionBricks: Scalable Real-Time Motions with Modular Latent Generative Model and Smart Primitives**

**Modüler gizil üretken model · Smart Primitive (akıllı ilkel) yapı taşları · SOMA ve Unitree G1 için ONNX modelleri**

**Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org · [www.robotlar.org](https://www.robotlar.org)**

👉 **Unitree G1 ürün sayfası: [robotlar.org/tr/insansi-robotlar/unitree-g1](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · G1 EDU için teklif: [robotlar.org/tr/fiyat/unitree-g1-edu-fiyat](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat)**

📖 **Türkçe rehber: [Unitree G1 MotionBricks rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-motionbricks-rehberi)**

[Kurulum](#-kurulum) · [Nasıl Çalışır](#-motionbricks-nasıl-çalışır) · [Ara Kare Üretici](#-ara-kare-üretici-inbetweener) · [Sahne Önizleme](#-sahne-önizleme-scene-preview) · [Belgeler](#-belgeler) · [G1 ile Başlangıç](#-unitree-g1-ile-motionbricks-türkiyede-nereden-başlanır) · [Unitree Ürünleri](#-unitree-ürün-ailesi-türkiye)

</div>

<p align="center">
  <a href="https://nvlabs.github.io/motionbricks/"><img src="https://img.shields.io/badge/Project-Page-blue" alt="Proje sayfası"></a>
  <a href="https://arxiv.org/abs/2604.24833"><img src="https://img.shields.io/badge/arXiv-2604.24833-b31b1b.svg" alt="Makale"></a>
  <a href="#-kontrol-noktaları-checkpoints"><img src="https://img.shields.io/badge/Hugging%20Face-Models-yellow" alt="Hugging Face modelleri"></a>
  <a href="https://github.com/NVlabs/motionbricks/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-Apache%202.0-76B900.svg" alt="Lisans"></a>
  <img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/teaser.jpg" alt="MotionBricks tek bir modelle karakterleri ve Unitree G1 insansı robotu canlandırır" width="100%">
</p>

> Bu depo, NVIDIA'nın (NVlabs) resmi [NVlabs/motionbricks](https://github.com/NVlabs/motionbricks) projesinin belgelerinin Türkçe çevirisidir. Kaynak kod, ONNX modelleri, örnek hareketler ve görseller orijinal depoda kalır. Burada yalnız Türkçe anlatım bulunur ve görseller orijinal depodan gösterilir.
>
> **Lisans dağılımı:** belgeler CC-BY-4.0 (bu çeviri aynı lisansla, atıf yapılarak yayımlanır), kod Apache-2.0, modeller NVIDIA Open Model Agreement, örnek hareketler NVIDIA Sample Data Evaluation License kapsamındadır. Ayrıntı [Lisans](#-lisans) bölümünde.
>
> **Kodu çalıştırmak için orijinal depoyu klonlayın:** [github.com/NVlabs/motionbricks](https://github.com/NVlabs/motionbricks)
>
> MotionBricks, NVIDIA'nın projesidir. Çeviri ve Türkçe teknik destek: Unitree Türkiye resmi distribütörü [Robotlar.org](https://www.robotlar.org).

MotionBricks, karakterleri ve robotları gerçek zamanlı canlandıran modüler bir gizil üretken modeldir (modular latent generative model). Son dört poz ile en çok dört hedef poz arasındaki hareketi doldurur. İşi öğrenilmiş üç modül paylaşır, [MotionBricks nasıl çalışır](#-motionbricks-nasıl-çalışır) bölümü bunu anlatır. Smart Primitive'ler (akıllı ilkeller) hedef pozları gezinme ve nesne etkileşimi için yapı taşlarına dönüştürür. Böylece engelin üstünden atlama ve oturma gibi davranışlar, yeniden eğitim gerekmeden tuğla gibi birbirine eklenir.

Orijinal depo, [SIGGRAPH 2026 makalesine](https://arxiv.org/abs/2604.24833) dayanan önceden eğitilmiş MotionBricks modellerini ve onları işe koşmak için gereken her şeyi sunar:

- **Önceden eğitilmiş modeller:** [SOMA vücut modeli](https://github.com/NVlabs/SOMA-X) ve Unitree G1 robotu için. ONNX dosyaları olarak gelirler, önceden eğitilmiş [kontrol noktaları](#-kontrol-noktaları-checkpoints) (checkpoints) Hugging Face'tedir.
- **Python çalışma zamanı (runtime):** [Warp](https://github.com/NVIDIA/warp), [Newton](https://github.com/newton-physics/newton) ve [TensorRT](https://github.com/NVIDIA/TensorRT) üzerine kuruludur ve üretimleri art arda bağlayarak sürekli hareket elde eder.
- **İki demo:** yerleştirdiğiniz pozlar arasında hareket üreten bir [Ara Kare Üretici (Inbetweener)](#-ara-kare-üretici-inbetweener) ve karakterin bir sahnede yürüyerek, koşarak, engelden atlayarak, eğilip altından geçerek ve oturarak ilerlediği bir [Sahne Önizleme (Scene Preview)](#-sahne-önizleme-scene-preview).
- **İki editör:** kendi [Smart Primitive'lerinizi](#-editörler) ve [sahnelerinizi](#-editörler) hazırlamak için.

Demolar modeli sürmenin bir yolunu gösterir. Hedeflerin nereden geleceğine uygulamanız karar verir. MotionBricks'e yeni başlıyorsanız [rehberli eğitim](docs/REHBERLI-EGITIM.md), tek bir etkileşim sahnesi kurarken dört uygulamanın hepsini adım adım gezdirir.

## 📰 Haberler

- MotionBricks'in artık kendi deposu ve SOMA insanları için önceden eğitilmiş bir modeli var. İlk sürüm, [GR00T Whole-Body Control](https://github.com/NVlabs/GR00T-WholeBodyControl/tree/main/motionbricks) içinde bir Unitree G1 önizlemesiydi.

## 📥 Kurulum

Çalışma zamanı TensorRT kullanır, bu yüzden NVIDIA GPU gerektirir. Demolar Windows ve Linux (x86-64) üzerinde NVIDIA GeForce RTX 4090 ile test edildi.

[Git LFS](https://git-lfs.com/) ve [uv](https://docs.astral.sh/uv/getting-started/installation/) kurun. Kod orijinal NVIDIA deposundadır, onu klonlayın:

```sh
git clone https://github.com/NVlabs/motionbricks.git
cd motionbricks
```

Ardından deponun kök dizininde şu komutları çalıştırın:

```sh
git lfs pull
uv sync
```

Her model ve duyarlılık (precision) ilk kez açıldığında bir TensorRT planı derlenir. Bu birkaç dakika sürer. Planlar `.trt-cache/` içinde önbelleğe alınır, sonraki açılışlar bu sayede hızlı başlar. Aşağıdaki her uygulama bölümü kendi komutunu verir.

## 🧩 MotionBricks nasıl çalışır

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/core_idea.gif" alt="Yeşil bağlam pozları, üretilen pozların art arda görüntüsü ve mavi hedef poz" width="100%">
</p>

Her üretim, iki poz kümesi arasındaki hareketi 30 Hz'de doldurur:

- **Bağlam (context):** karakterin en son dört pozudur (yukarıda yeşil).
- **Hedef (target):** hareketin ulaşması gereken bir ile dört arası pozdur (yukarıda mavi). Tek poz, karakterin nereye varıp duracağını belirler. Daha fazla poz, o noktada nasıl hareket edeceğini de belirler.

Geçişin ne kadar süreceğine model karar verir. Tek bir üretim, her biri dört pozdan oluşan 6 ile 16 arası belirteç (token), yani yaklaşık 0,6 ile 1,9 s hareket üretir. Kare sayılarını [Üretim uzunluğu](docs/CALISMA-ZAMANI-API.md#üretim-uzunluğu) açıklar.

İşi öğrenilmiş üç modül paylaşır:

- **Kök modülü (root module):** kökün süresini ve yolunu tahmin eder.
- **Poz modülü (pose module):** maskeli bir belirteç dönüştürücüsüdür (masked token transformer). O yol boyunca hareket belirteçlerini tahmin eder.
- **Kod çözücü (decoder):** belirteçleri hedeflere ulaşan sürekli harekete çevirir. Birkaç kod kitabı başlığı (codebook head) olan bir VQ-VAE kullanır.

Daha uzun hareket, üretimleri art arda bağlamaktan (chaining) gelir. Çalışma zamanı üretilen hareketi oynatır, oynattığını kaydeder ve kaydedilen son dört pozu bir sonraki üretimin bağlamı olarak kullanır. Hedefleri uygulamanız belirler. Tek bir ara kare üretimi (inbetweening) modelini anahtar kare animasyonuna, sürekli yürüyüşe/harekete (lokomosyon) ya da nesne etkileşimlerine dönüştüren de bu seçimdir. Yöntemi, eğitim verisini ve değerlendirmeyi [makale](https://arxiv.org/abs/2604.24833) anlatır.

### SOMA ve G1 için tek çalışma zamanı

İki model de aynı yolla üretir, yalnız poz düzeni farklıdır. Uygulamalar ve çalışma zamanı iki iskelette (rig) aynı çalışır. Böylece tek bir kod yolu hem bir insan iskeletini hem de bir robotu sürer. İki düzeni [Motorlar](docs/CALISMA-ZAMANI-API.md#motorlar) bölümü anlatır.

## 📁 Depo düzeni

`motionbricks` paketi kütüphanedir. `app` içindeki demolar ve editörler bunun üzerine kuruludur.

| Klasör | İçerik |
| --- | --- |
| [motionbricks](https://github.com/NVlabs/motionbricks/tree/main/motionbricks) | Kütüphane: motorlar (engines), hareket oynatıcılar, kaydedici, karışım uzayları (blend spaces), pivotlar ve ONNX modelleri |
| [app](https://github.com/NVlabs/motionbricks/tree/main/app) | Demolar, iki editör ve ortak kullandıkları karakter çalışma zamanı |
| [assets](https://github.com/NVlabs/motionbricks/tree/main/assets) | Örnek hareket klipleri, Smart Primitive'ler ve gym sahneleri |
| [docs](https://github.com/NVlabs/motionbricks/tree/main/docs) | Rehberler ve model kartları, [docs/examples](https://github.com/NVlabs/motionbricks/tree/main/docs/examples) içinde çalıştırılabilir örnekler (Türkçe çeviriler bu deponun [docs](docs) klasöründe) |
| [tools/onnx](https://github.com/NVlabs/motionbricks/tree/main/tools/onnx) | Bağımsız ONNX dışa aktarıcı (exporter) |

## 🎬 Ara Kare Üretici (Inbetweener)

Temel fikri görmek için buradan başlayın. Kliplerden pozlar seçin, onları görünüme yerleştirin ve sırayla bu pozlardan geçen hareket üretin.

<table>
  <tr>
    <th>SOMA</th>
    <th>Unitree G1</th>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/inbetweener_soma.gif" alt="SOMA ile Ara Kare Üretici" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/inbetweener_g1.gif" alt="Unitree G1 insansı robot ile Ara Kare Üretici" width="100%"></td>
  </tr>
</table>

Şu komutla çalıştırın:

```sh
uv run python -m app.inbetweener_demo
```

[Ara Kare Üretici](docs/ARA-KARE-URETICI.md) belgesi seçenekleri ve denenecek şeyleri listeler, kodu da adım adım açıklar.

## 🏃 Sahne Önizleme (Scene Preview)

Bir karakteri sürekli yürüyüş ve etkileşimlerle bir sahnede sürün. Karakter ileri yürüyüp koşabilir, kameraya bakarak yana kayabilir (strafe) ya da çömelmiş halde yana kayabilir. Skipping (sekerek yürüme) ve Zombie gibi altı stil modu yürüyüş biçimini değiştirir. Sahne Önizleme, [makale videolarının](https://nvlabs.github.io/motionbricks/) arkasındaki Unreal Engine demosunu küçük bir Python çalışma zamanına indirger. Fikirleri öne çıkarır, böylece onları her türlü entegrasyona taşıyabilirsiniz.

<table>
  <tr>
    <th>SOMA</th>
    <th>Unitree G1</th>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/scene_preview_soma.gif" alt="SOMA ile Sahne Önizleme" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/scene_preview_g1.gif" alt="Unitree G1 insansı robot ile Sahne Önizleme" width="100%"></td>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_targets_soma.gif" alt="SOMA bağlam, hedef ve karışım uzayı pozları gösterilirken yürür, koşar ve durur" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_targets_g1.gif" alt="Unitree G1 bağlam, hedef ve karışım uzayı pozları gösterilirken yürür, koşar ve durur" width="100%"></td>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_directional_soma.gif" alt="SOMA dört yöne yana kayar, çömelir ve yeniden ayağa kalkar, bağlam, hedef ve karışım uzayı pozları gösterilir" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_directional_g1.gif" alt="Unitree G1 dört yöne yana kayar, çömelir ve yeniden ayağa kalkar, bağlam, hedef ve karışım uzayı pozları gösterilir" width="100%"></td>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_styles_soma.gif" alt="SOMA, Skipping'den Combat'a her yürüyüş stilinde yürür" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_styles_g1.gif" alt="Unitree G1, Skipping'den Combat'a her yürüyüş stilinde yürür" width="100%"></td>
  </tr>
</table>

SOMA ya da Unitree G1 gym sahnesiyle çalıştırın:

```sh
uv run python -m app.scene_preview --scene assets/scenes/soma_gym.scene.json
uv run python -m app.scene_preview --scene assets/scenes/g1_gym.scene.json
```

[Sahne Önizleme](docs/SAHNE-ONIZLEME.md) belgesi seçenekleri ve kontrolleri listeler. Arkasındaki çalışma zamanını da açıklar: karakter, durum makinesi (state machine), yürüyüş ve etkileşimler.

## 🧱 Editörler

Bir Smart Primitive, bir eylemi bir nesnenin çevresine yerleştirilmiş hedef pozlarla tanımlar. Smart Primitive Editörü'nde bir tane hazırlayın. Sahne Editörü ile onu Sahne Önizleme için bir sahneye yerleştirin.

<table>
  <tr>
    <th>Smart Primitive Editörü</th>
    <th>Sahne Editörü</th>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/smart_primitive_editor.png" alt="Zıplama Smart Primitive'i açık Smart Primitive Editörü" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/scene_editor.png" alt="SOMA gym sahnesi açık Sahne Editörü" width="100%"></td>
  </tr>
</table>

Şu komutlarla çalıştırın:

```sh
uv run python -m app.smart_primitive_editor
uv run python -m app.scene_editor
```

Bu araçları öğrenmek için [rehberli eğitimle](docs/REHBERLI-EGITIM.md#bölüm-2-bir-smart-primitive-oluşturun) başlayın. Eğitim, iki editörle adım adım bir etkileşim kurar. [Smart Primitive Editörü](docs/SMART-PRIMITIVE-EDITORU.md) ve [Sahne Editörü](docs/SAHNE-EDITORU.md) belgeleri her editörü ve belgelerini anlatır.

Bu editörler dinamik diziler ve etkileşimler kurmanın bir yolunu gösterir. Uygulamanız aynı yaklaşımı ya da farklı bir yaklaşımı izleyebilir. Örneğin bu editörler kliplerden birkaç hedef poz örnekler. Bu pozları [Kimodo](https://github.com/nv-tlabs/kimodo) ya da [ARDY](https://github.com/nv-tlabs/ardy) gibi bir hareket üretim modeli de üretebilir.

## 💾 Kontrol Noktaları (Checkpoints)

MotionBricks kontrol noktaları SOMA vücut modeli ve Unitree G1 robotu için sunulur. Ağırlıkları SafeTensors biçimindedir.

> Not: uygulamalar [motionbricks/assets/onnx](https://github.com/NVlabs/motionbricks/tree/main/motionbricks/assets/onnx) içinde gelen ONNX modellerini kullanır, bu yüzden onları çalıştırmak için kontrol noktası indirmeniz gerekmez. Kendi ONNX modellerinizi dışa aktarmak ya da başka bir çalışma zamanı kurmak için bir kontrol noktası indirin. Orijinal depo [ONNX dışa aktarıcıyı](#-onnx-modellerini-dışa-aktarma) içerir.

| Model | İskelet | Eğitim Verisi | FPS | Hugging Face | Lisans |
|:-------|:-------------|:------:|:------:|:-------------:|:-------------:|
| **MotionBricks-SOMA-RP-v1** | [SOMA](https://github.com/NVlabs/SOMA-X) | [Bones RigPlay 1 (RP1)](https://bones.studio/datasets#rp01) | 30 | [Bağlantı](https://huggingface.co/nvidia/MotionBricks/tree/main/soma) | [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) |
| **MotionBricks-G1-RP-v1** | [Unitree G1](https://github.com/unitreerobotics/unitree_ros/tree/master/robots/g1_description) | [Bones RigPlay 1 (RP1)](https://bones.studio/datasets#rp01), [yeniden hedeflenmiş (retargeted)](https://github.com/NVIDIA/soma-retargeter) | 30 | [Bağlantı](https://huggingface.co/nvidia/MotionBricks/tree/main/g1) | [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/) |

[SOMA model kartı](docs/SOMA-MODEL-KARTI.md) ve [Unitree G1 model kartı](docs/UNITREE-G1-MODEL-KARTI.md) her modeli, girdilerini ve çıktılarını, sınırlamalarını anlatır.

## 📤 ONNX Modellerini Dışa Aktarma

Orijinal depodaki ONNX modelleri bu kontrol noktalarından, [tools/onnx](https://github.com/NVlabs/motionbricks/tree/main/tools/onnx) içindeki bağımsız dışa aktarıcıyla ([Türkçe rehber](docs/ONNX-DISA-AKTARIM.md)) dışa aktarıldı. Bunu kendiniz yapmak için [Hugging Face CLI](https://huggingface.co/docs/huggingface_hub/en/guides/cli) kurun, kontrol noktası klasörlerini indirin ve modelinizin komutunu deponun kök dizininden çalıştırın:

```sh
hf download nvidia/MotionBricks --local-dir checkpoints/MotionBricks

# SOMA
uv run --locked --project tools/onnx motionbricks-onnx --model soma --checkpoints checkpoints/MotionBricks/soma

# Unitree G1
uv run --locked --project tools/onnx motionbricks-onnx --model unitree_g1 --checkpoints checkpoints/MotionBricks/g1
```

Tek bir model indirmek için `--include "soma/*"` ya da `--include "g1/*"` ekleyin. İndirilen klasör yapısını olduğu gibi koruyun.

Dışa aktarıcının kendi ortamı vardır ve komutu ilk çalıştırdığınızda uv bu ortamı kurar. Varsayılan FP32'dir. Karma FP16 için `--precision fp16` ekleyin. Tüm seçenekleri [dışa aktarıcı rehberi](docs/ONNX-DISA-AKTARIM.md) listeler, tensör arayüzlerini ve toplu işlemeyi (batching) [ONNX çıkarımı](docs/ONNX-CIKARIM.md) açıklar.

## 📚 Belgeler

MotionBricks'e yeni misiniz? **[Rehberli eğitimle](docs/REHBERLI-EGITIM.md)** başlayın. Eğitim önce temel kavramları açıklar, sonra dört uygulamanın hepsiyle bir etkileşim sahnesi kurar.

**Uygulamalar**

- **[Ara Kare Üretici](docs/ARA-KARE-URETICI.md):** Ara Kare Üretici'nin nasıl çalıştırılacağını gösterir, ardından pozları örnekleme ve yerleştirmeden zincirleme oynatmaya kadar kodunu adım adım açıklar.
- **[Sahne Önizleme](docs/SAHNE-ONIZLEME.md):** Seçenekleri ve kontrolleri listeler, ardından karakter ve durum makinesinden yürüyüşe ve etkileşimlere kadar örnek çalışma zamanını anlatır.
- **[Smart Primitive Editörü](docs/SMART-PRIMITIVE-EDITORU.md):** Bir Smart Primitive'in neleri tuttuğunu ve nasıl hazırlandığını anlatır: düğüm hiyerarşisi, hedef pozlar, etkileşim kutuları, pivotlar ve diziler.
- **[Sahne Editörü](docs/SAHNE-EDITORU.md):** Bir sahnenin nasıl kurulacağını gösterir ve Sahne Önizleme'nin açtığı `.scene.json` belgesini anlatır.

**Entegrasyon**

- **[Çalışma Zamanı API'si](docs/CALISMA-ZAMANI-API.md):** MotionBricks'i uygulamanıza entegre etmek için buradan başlayın. Motorları, poz düzenlerini, klipleri ve üretimlerin zincirlenmesini anlatır. Görüntüleyici olmadan hareket üreten iki çalıştırılabilir betik içerir.
- **[ONNX çıkarımı](docs/ONNX-CIKARIM.md):** Hazır motorlar olmadan kullanım için tensör şekillerini, toplu işlemeyi ve öteki çıkarım kütüphanelerini anlatır.

**Modeller**

- **[ONNX dışa aktarıcı](docs/ONNX-DISA-AKTARIM.md):** Hazır gelen ONNX modellerinin Hugging Face'teki kontrol noktalarından nasıl dışa aktarılacağını anlatır.
- **[SOMA model kartı](docs/SOMA-MODEL-KARTI.md):** SOMA modelini, girdilerini ve çıktılarını, eğitim verisini ve sınırlamalarını anlatır.
- **[Unitree G1 model kartı](docs/UNITREE-G1-MODEL-KARTI.md):** Unitree G1 modelini, girdilerini ve çıktılarını, eğitim verisini ve sınırlamalarını anlatır.

| Türkçe belge | Kaynak dosya |
| --- | --- |
| [docs/REHBERLI-EGITIM.md](docs/REHBERLI-EGITIM.md) | `docs/guided_tutorial.md` |
| [docs/ARA-KARE-URETICI.md](docs/ARA-KARE-URETICI.md) | `docs/inbetweener.md` |
| [docs/SAHNE-ONIZLEME.md](docs/SAHNE-ONIZLEME.md) | `docs/scene_preview.md` |
| [docs/SMART-PRIMITIVE-EDITORU.md](docs/SMART-PRIMITIVE-EDITORU.md) | `docs/smart_primitive_editor.md` |
| [docs/SAHNE-EDITORU.md](docs/SAHNE-EDITORU.md) | `docs/scene_editor.md` |
| [docs/CALISMA-ZAMANI-API.md](docs/CALISMA-ZAMANI-API.md) | `docs/runtime_api.md` |
| [docs/ONNX-CIKARIM.md](docs/ONNX-CIKARIM.md) | `docs/onnx_inference.md` |
| [docs/ONNX-DISA-AKTARIM.md](docs/ONNX-DISA-AKTARIM.md) | `tools/onnx/README.md` |
| [docs/SOMA-MODEL-KARTI.md](docs/SOMA-MODEL-KARTI.md) | `docs/soma_model.md` |
| [docs/UNITREE-G1-MODEL-KARTI.md](docs/UNITREE-G1-MODEL-KARTI.md) | `docs/unitree_g1_model.md` |

## 🤝 NVIDIA'daki ilgili insansı çalışmaları

MotionBricks, animasyon, robotik ve fiziksel yapay zeka için insansı hareketi mümkün kılmaya yönelik daha geniş bir çalışmanın parçasıdır. İlgili projeler:

- [Kimodo](https://github.com/nv-tlabs/kimodo): denetlenebilir çevrim dışı hareket hazırlamada uzmanlaşmış insan hareketi üretim modeli. Türkçe: [repo](https://github.com/Unitree-Turkiye/nvidia-kimodo-turkce) · [rehber](https://www.robotlar.org/tr/rehber/nvidia-kimodo-hareket-uretimi-rehberi)
- [ARDY](https://github.com/nv-tlabs/ardy): Kimodo üzerine kurulur ve etkileşimli uygulamalar için gerçek zamanlı, denetlenebilir hareket üretimi sağlar. Türkçe: [repo](https://github.com/Unitree-Turkiye/nvidia-ardy-turkce) · [rehber](https://www.robotlar.org/tr/rehber/nvidia-ardy-gercek-zamanli-hareket-rehberi)
- [SOMA Body Model](https://github.com/NVlabs/SOMA-X): birleşik, parametrik bir insan vücudu modeli. Türkçe: [repo](https://github.com/Unitree-Turkiye/nvidia-soma-govde-modeli-turkce) · [rehber](https://www.robotlar.org/tr/rehber/nvidia-soma-govde-modeli-rehberi)
- [BONES-SEED Dataset](https://huggingface.co/datasets/bones-studio/seed): SOMA ve G1 biçiminde insanlar ve insansılar için büyük bir hareket yakalama veri seti. Türkçe: [repo](https://github.com/Unitree-Turkiye/bones-seed-hareket-veri-seti-turkce) · [rehber](https://www.robotlar.org/tr/rehber/bones-seed-hareket-veri-seti-rehberi)
- [ProtoMotions](https://github.com/NVlabs/ProtoMotions): fiziksel olarak benzetilen insanları ve insansıları eğitmek için simülasyon ve öğrenme çatısı. Türkçe: [repo](https://github.com/Unitree-Turkiye/nvidia-protomotions-turkce) · [rehber](https://www.robotlar.org/tr/rehber/nvidia-protomotions-insansi-simulasyon-rehberi)
- [SOMA Retargeter](https://github.com/NVIDIA/soma-retargeter): SOMA hareketini Unitree G1 gibi insansı robotlara yeniden hedefler. Türkçe: [repo](https://github.com/Unitree-Turkiye/unitree-g1-soma-retargeter-turkce) · [rehber](https://www.robotlar.org/tr/rehber/soma-retargeter-unitree-g1-hareket-aktarimi-rehberi)
- [GEM](https://github.com/NVlabs/GEM-X): videodan insan hareketi yeniden kurma. Türkçe: [repo](https://github.com/Unitree-Turkiye/nvidia-gem-x-turkce) · [rehber](https://www.robotlar.org/tr/rehber/nvidia-gem-x-videodan-hareket-rehberi)
- [GEAR SONIC](https://github.com/NVlabs/GR00T-WholeBodyControl): fiziksel robotlar için insansı davranış temel modeli. Türkçe: [repo](https://github.com/Unitree-Turkiye/unitree-g1-gear-sonic-turkce) · [rehber](https://www.robotlar.org/tr/rehber/gear-sonic-unitree-g1-tum-govde-kontrol-rehberi)

---

## 🚀 Unitree G1 ile MotionBricks: Türkiye'de nereden başlanır

MotionBricks'in G1 modeli (`MotionBricks-G1-RP-v1`), Unitree G1'in 29 eklem açısı ve yüzen kökten (floating root) oluşan düzeniyle çalışır. Model kartına göre çıktı, robot dinamiği altında bir kontrolcünün izleyeceği kinematik referans harekettir. Bu yüzden MotionBricks, G1 üzerinde animasyon, simülasyon ve kontrol deneyleri için bir hareket kaynağı olarak konumlanır. Fiziksel robota geçişte izleme kontrolcüsü, limit uygulaması ve güvenlik doğrulaması sizin iş akışınızın parçasıdır.

**Platform seçimi**

| Ürün | MotionBricks iş akışındaki yeri |
| --- | --- |
| [Unitree G1](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | Modelin hedeflediği insansı robot, 132 cm, 35 kg, 23-43 DoF |
| [Unitree G1 EDU teklif](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) | Geliştirici ve araştırma sürümü için teklif formu |
| [Unitree G1 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-plus) | 2 DoF boyun, yüksek tork, dokunsal kafa ve 6 mikrofonlu G1 |
| [Unitree G1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-d) | İnsansı robot veri toplama ve eğitim platformu, Jetson Orin |
| [G1 Ar-Ge platformu](https://www.robotlar.org/tr/g1-arge-platformu) | Üniversite ve Ar-Ge laboratuvarları için G1 çözüm sayfası |

**Türkçe rehberler (iş akışına göre)**

- Hareket üretimi ve bu deponun özeti: [Unitree G1 MotionBricks rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-motionbricks-rehberi)
- Tüm vücut kontrolü ve GR00T (MotionBricks'in ilk G1 önizlemesi GR00T Whole-Body Control içinde yayımlandı): [Unitree G1 Isaac GR00T rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-isaac-gr00t-rehberi)
- Referans hareketi izleyen politikaları eğitmek için pekiştirmeli öğrenme: [Unitree RL Gym](https://www.robotlar.org/unitree/rl-gym)
- Gerçek hareket verisi toplamak için XR teleoperasyon: [Unitree G1 teleoperasyon](https://www.robotlar.org/unitree/g1-teleoperasyon)
- Veri seti ve taklit öğrenme: [Unitree LeRobot rehberi](https://www.robotlar.org/tr/rehber/unitree-lerobot-rehberi)
- Robotla haberleşme ve düşük seviye kontrol: [Unitree SDK rehberi](https://www.robotlar.org/unitree/sdk-rehberi)
- Robot üstü hesaplama: [Jetson Orin rehberi](https://www.robotlar.org/tr/rehber/saha-robotu-jetson-orin-dayanikli-bilgisayar)

**Türkçe GitHub depoları:** [G1](https://github.com/Unitree-Turkiye/unitree-g1-turkce) · [SDK2](https://github.com/Unitree-Turkiye/unitree-sdk2-turkce) · [XR Teleoperasyon](https://github.com/Unitree-Turkiye/unitree-teleoperasyon-turkce) · [G1 dokunsal algılama giysisi](https://github.com/Unitree-Turkiye/unitree-g1-dokunsal-giysi-turkce)

Üniversite ve kurumlar için platform, eğitim ve Türkçe geliştirici desteği konusunda [kurumsal teklif formundan](https://www.robotlar.org/tr/fiyat/unitree-kurumsal-teklif-al) bize ulaşabilirsiniz.

## 🧬 NVIDIA İnsansı Hareket Ailesi (Türkçe)

NVIDIA'nın animasyon, robotik ve fiziksel yapay zeka için yayımladığı insansı hareket projeleri birbirini tamamlar: SOMA ortak gövde modelini, BONES-SEED veriyi, GEM videodan hareketi, Kimodo ve ARDY hareket üretimini, MotionBricks gerçek zamanlı hareket planlamayı, SOMA Retargeter insan hareketini Unitree G1'e aktarmayı, ProtoMotions fizik tabanlı eğitimi, GEAR SONIC de gerçek robotta tüm gövde kontrolünü üstlenir. Akışın bütününü [merkez rehberde](https://www.robotlar.org/tr/rehber/nvidia-insansi-hareket-ekosistemi-unitree-g1-rehberi) anlatıyoruz.

| Proje | Ne yapar | Türkçe repo | Türkçe rehber | Resmi depo |
|---|---|---|---|---|
| MotionBricks | Bağlam ve hedef pozlar arasında gerçek zamanlı hareket üretir, Unitree G1 modeliyle gelir | [unitree-g1-motionbricks-turkce](https://github.com/Unitree-Turkiye/unitree-g1-motionbricks-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/unitree-g1-motionbricks-rehberi) | [NVlabs/motionbricks](https://github.com/NVlabs/motionbricks) |
| Kimodo | Denetlenebilir çevrim dışı insan hareketi üretimi | [nvidia-kimodo-turkce](https://github.com/Unitree-Turkiye/nvidia-kimodo-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-kimodo-hareket-uretimi-rehberi) | [nv-tlabs/kimodo](https://github.com/nv-tlabs/kimodo) |
| ARDY | Kimodo üzerine gerçek zamanlı, etkileşimli hareket üretimi | [nvidia-ardy-turkce](https://github.com/Unitree-Turkiye/nvidia-ardy-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-ardy-gercek-zamanli-hareket-rehberi) | [nv-tlabs/ardy](https://github.com/nv-tlabs/ardy) |
| SOMA Body Model | Birleşik, parametrik insan gövde modeli | [nvidia-soma-govde-modeli-turkce](https://github.com/Unitree-Turkiye/nvidia-soma-govde-modeli-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-soma-govde-modeli-rehberi) | [NVlabs/SOMA-X](https://github.com/NVlabs/SOMA-X) |
| BONES-SEED | SOMA ve G1 biçiminde büyük hareket yakalama veri seti | [bones-seed-hareket-veri-seti-turkce](https://github.com/Unitree-Turkiye/bones-seed-hareket-veri-seti-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/bones-seed-hareket-veri-seti-rehberi) | [bones-studio/seed](https://huggingface.co/datasets/bones-studio/seed) |
| ProtoMotions | Benzetilen insan ve insansıları eğitmek için GPU hızlandırmalı simülasyon çatısı | [nvidia-protomotions-turkce](https://github.com/Unitree-Turkiye/nvidia-protomotions-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-protomotions-insansi-simulasyon-rehberi) | [NVlabs/ProtoMotions](https://github.com/NVlabs/ProtoMotions) |
| SOMA Retargeter | SOMA hareketini Unitree G1 gibi insansı robotlara aktarır | [unitree-g1-soma-retargeter-turkce](https://github.com/Unitree-Turkiye/unitree-g1-soma-retargeter-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/soma-retargeter-unitree-g1-hareket-aktarimi-rehberi) | [NVIDIA/soma-retargeter](https://github.com/NVIDIA/soma-retargeter) |
| GEM | Görüntü ve videodan tüm gövde ve el hareketini kestirir | [nvidia-gem-x-turkce](https://github.com/Unitree-Turkiye/nvidia-gem-x-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-gem-x-videodan-hareket-rehberi) | [NVlabs/GEM-X](https://github.com/NVlabs/GEM-X) |
| GEAR SONIC | Fiziksel insansı robotlar için davranış temel modeli ve tüm gövde kontrolü | [unitree-g1-gear-sonic-turkce](https://github.com/Unitree-Turkiye/unitree-g1-gear-sonic-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/gear-sonic-unitree-g1-tum-govde-kontrol-rehberi) | [NVlabs/GR00T-WholeBodyControl](https://github.com/NVlabs/GR00T-WholeBodyControl) |

İlgili Türkçe kaynaklar: [Unitree G1 + Isaac GR00T](https://github.com/Unitree-Turkiye/unitree-g1-isaac-groot-turkce) · [XR teleoperasyon](https://github.com/Unitree-Turkiye/unitree-teleoperasyon-turkce) · [LeRobot](https://github.com/Unitree-Turkiye/unitree-lerobot-turkce) · [RL Gym](https://github.com/Unitree-Turkiye/unitree-rl-gym-turkce) · [MuJoCo](https://github.com/Unitree-Turkiye/unitree-mujoco-turkce)

## 🤖 Unitree Ürün Ailesi (Türkiye)

Robotlar.org, Unitree Robotics'in Türkiye resmi distribütörüdür. Aşağıdaki ürünlerin Türkçe tanıtımı, teknik özellikleri ve kurumsal teklif formu sitede yer alır. Fiyatlar konfigürasyona göre değiştiği için [teklif üzerinden](https://www.robotlar.org/tr/fiyat/unitree-kurumsal-teklif-al) paylaşılır.

### İnsansı robotlar

| Ürün | Ne işe yarar |
|---|---|
| [Unitree G1](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | 132 cm, 35 kg, 23-43 DoF insansı robot |
| [Unitree G1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-d) | İnsansı robot veri toplama ve eğitim platformu, Jetson Orin |
| [Unitree G1 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-plus) | 2 DoF boyun, yüksek tork, dokunsal kafa ve 6 mikrofonlu G1 |
| [Unitree H2](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2) | 182 cm, 70 kg, 31 DoF tam boy insansı robot |
| [Unitree H2-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-d) | 7 DoF kollu, tekerlekli çift kollu insansı robot |
| [Unitree H2 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-plus) | NVIDIA Isaac GR00T referans insansı robotu, 75 DoF, Jetson Thor |
| [Unitree H2-X](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-x) | Hizmet ve koruma görevleri, 4 m platformdan iniş |
| [Unitree R1](https://www.robotlar.org/tr/insansi-robotlar/unitree-r1) | 123 cm, 29 kg, 26 DoF insansı robot (AIR, R1, EDU sürümleri) |
| [Unitree R1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-r1-d) | R1-A5 ya da R1-A7 (7-DOF) kollu çift kollu insansı platform |
| [Tüm insansı robotlar →](https://www.robotlar.org/tr/insansi-robotlar) | |

### Robot elleri

| Ürün | Ne işe yarar |
|---|---|
| [Unitree Dex3-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex3-1) | G1 için 7 serbestlik dereceli el, opsiyonel 33 dokunsal sensör |
| [Unitree Dex5-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex5-1) | 20 serbestlik dereceli (16 aktif) el, Dex5-1P sürümünde 94 dokunsal sensör |
| [Unitree Dex5-S](https://www.robotlar.org/tr/robot-elleri/unitree-dex5-s) | 22 aktif serbestlik, 620 g, çift T1s Ethernet, 1000 Hz |
| [Unitree Dex1-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex1-1) | İki çeneli tutucu: 90 mm strok, 5-120 N ayarlı kavrama, RS-485 |
| [Tüm robot elleri →](https://www.robotlar.org/tr/robot-elleri) | |

### Dört ayaklı robotlar ve bileşenler

[Go2](https://www.robotlar.org/tr/kopek-robotlar/unitree-go2) · [Go2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-go2-w) · [B2](https://www.robotlar.org/tr/kopek-robotlar/unitree-b2) · [B2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-b2-w) · [A2](https://www.robotlar.org/tr/kopek-robotlar/unitree-a2) · [AS2](https://www.robotlar.org/tr/kopek-robotlar/unitree-as2) · [Tüm dört ayaklı robotlar](https://www.robotlar.org/tr/kopek-robotlar) · [4D LiDAR L2](https://www.robotlar.org/tr/unitree-bilesenler/4d-lidar-l2) · [Tüm Unitree bileşenleri](https://www.robotlar.org/tr/unitree-bilesenler)

## 📖 Türkçe Rehberler

- [NVIDIA insansı hareket ekosistemi ve Unitree G1 (merkez rehber)](https://www.robotlar.org/tr/rehber/nvidia-insansi-hareket-ekosistemi-unitree-g1-rehberi)
- [Unitree G1 MotionBricks rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-motionbricks-rehberi)
- [Unitree G1 Isaac GR00T rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-isaac-gr00t-rehberi)
- [Unitree G1 dokunsal algılama giysisi rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-dokunsal-giysi-rehberi)
- [Unitree LeRobot: veri toplama ve taklit öğrenme](https://www.robotlar.org/tr/rehber/unitree-lerobot-rehberi)
- [Unitree G1 XR teleoperasyon](https://www.robotlar.org/unitree/g1-teleoperasyon)
- [Unitree RL Gym](https://www.robotlar.org/unitree/rl-gym)
- [Unitree SDK rehberi](https://www.robotlar.org/unitree/sdk-rehberi)
- [Unitree eğitim çözümleri (üniversite ve laboratuvar)](https://www.robotlar.org/tr/unitree-egitim-cozumleri)
- [Tüm açık kaynak Unitree projeleri (Türkçe)](https://www.robotlar.org/tr/acik-kaynak)

## 🔗 Unitree Türkiye GitHub Ağı

**Modeller:** [Go2](https://github.com/Unitree-Turkiye/unitree-go2-turkce) · [Go2-W](https://github.com/Unitree-Turkiye/unitree-go2-w-turkce) · [B2](https://github.com/Unitree-Turkiye/unitree-b2-turkce) · [A2](https://github.com/Unitree-Turkiye/unitree-a2-turkce) · [G1](https://github.com/Unitree-Turkiye/unitree-g1-turkce) · [H2](https://github.com/Unitree-Turkiye/unitree-h2)
**Geliştirici:** [SDK2](https://github.com/Unitree-Turkiye/unitree-sdk2-turkce) · [XR Teleoperasyon](https://github.com/Unitree-Turkiye/unitree-teleoperasyon-turkce) · [LeRobot](https://github.com/Unitree-Turkiye/unitree-lerobot-turkce) · [Becerikli El](https://github.com/Unitree-Turkiye/unitree-hunerli-el-turkce) · [MuJoCo](https://github.com/Unitree-Turkiye/unitree-mujoco-turkce) · [ROS2](https://github.com/Unitree-Turkiye/unitree-ros2-turkce) · [VLA](https://github.com/Unitree-Turkiye/unitree-vla-turkce)
**G1 araştırma:** [G1 MotionBricks](https://github.com/Unitree-Turkiye/unitree-g1-motionbricks-turkce) · [G1 dokunsal algılama giysisi](https://github.com/Unitree-Turkiye/unitree-g1-dokunsal-giysi-turkce) · [🏠 Unitree Türkiye](https://github.com/Unitree-Turkiye)

## 🙏 Teşekkür

Orijinal proje şu açık kaynak projelerin üzerine kuruludur (orijinal depodan):

- Görüntüleyici ve sahne için [Newton](https://github.com/newton-physics/newton)
- GPU çekirdekleri için [NVIDIA Warp](https://github.com/NVIDIA/warp)
- Çıkarım için [NVIDIA TensorRT](https://github.com/NVIDIA/TensorRT)
- Kullanıcı arayüzü için [imgui_bundle](https://github.com/pthom/imgui_bundle) üzerinden [Dear ImGui](https://github.com/ocornut/imgui)
- SOMA karakteri için [OpenUSD](https://github.com/PixarAnimationStudios/OpenUSD)
- Unitree G1 robot modeli ve ağları (mesh) için [Unitree Robotics](https://github.com/unitreerobotics/unitree_ros/tree/master/robots/g1_description)

## 📄 Lisans

Orijinal depoda kod, belgeler, modeller ve örnek hareketler farklı lisanslar taşır:

- **Kod:** [Apache 2.0](https://github.com/NVlabs/motionbricks/blob/main/LICENSE), belgelerdeki kod örnekleri ve parçacıkları dahil.
- **Belgeler ve medya:** NVIDIA'nın yazdığı belge metni ve medya (`docs/media/` içindeki dosyalar dahil görseller, ekran görüntüleri ve videolar) [Creative Commons Attribution 4.0 International (CC-BY-4.0)](https://creativecommons.org/licenses/by/4.0/) lisansını kullanır.
- **Modeller:** [NVIDIA Open Model Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-agreement/). Hugging Face'teki SOMA ve G1 kontrol noktalarını ve [ONNX modellerini](https://github.com/NVlabs/motionbricks/blob/main/motionbricks/assets/onnx/LICENSE.md) kapsar. Anlaşmanın koşullarına bağlı olarak ticari kullanıma ve değiştirilmiş modellerin yeniden dağıtımına izin verir. NVIDIA Corporation tarafından NVIDIA Open Model Agreement kapsamında lisanslanır.
- **Örnek hareketler:** [NVIDIA Sample Data Evaluation License](https://github.com/NVlabs/motionbricks/blob/main/assets/motions/LICENSE.txt). Kapsanan insan BVH klipleri ve yeniden hedeflenmiş G1 CSV klipleri, NVIDIA teknolojilerini değerlendirmek ve test etmek içindir. [Örnek veri bildirimi](https://github.com/NVlabs/motionbricks/blob/main/assets/motions/README.md) kapsanan dosyaları listeler ve Bones'a atıf yapar. Bu örnek koşulları model koşullarından ayrıdır.

Bağımlılıkların ve birlikte gelen robot varlıklarının lisansları [licenses](https://github.com/NVlabs/motionbricks/tree/main/licenses) klasöründedir. Dışa aktarıcı bağımlılıklarının lisansları [tools/onnx/licenses](https://github.com/NVlabs/motionbricks/tree/main/tools/onnx/licenses) klasöründedir.

Proje, kendi lisans koşulları olan ek bileşenleri indirir ve kurar. Kullanmadan önce bu koşulları inceleyin.

[TensorRT OSS bildirimi](https://github.com/NVlabs/motionbricks/blob/main/licenses/tensorrt-LICENSE.txt) TensorRT OSS bileşenlerine uygulanır. Ayrıca kurulan `tensorrt-cu12`, `tensorrt-cu12-bindings` ve `tensorrt-cu12-libs` çalışma zamanı paketleri ise paketlerle birlikte dağıtılan NVIDIA TensorRT Software License Agreement'a tabidir.

**Bu çevirinin lisansı:** Bu depodaki metinler, NVIDIA belgelerinin Türkçe çevirisidir ve [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/) ile yayımlanır. Özgün eser: NVIDIA, [NVlabs/motionbricks](https://github.com/NVlabs/motionbricks) belgeleri. Değişiklik: çeviri ve Türkiye bağlantıları. Bu depo kaynak kod, model ya da medya dosyası barındırmaz. Görseller orijinal depodan gösterilir.

## 📝 Atıf

Bu kodu araştırmanızda kullanırsanız lütfen şu atfı verin:

```bibtex
@article{wang2026motionbricks,
  title   = {MotionBricks: Scalable Real-Time Motions with Modular Latent Generative Model and Smart Primitives},
  author  = {Wang, Tingwu and Dionne, Olivier and De Ruyter, Michael and Minor, David and Rempe, Davis and Zhao, Kaifeng and Petrovich, Mathis and Yuan, Ye and Li, Chenran and Luo, Zhengyi and Robison, Brian and Blackwell, Xavier and Antoniazzi, Bernardo and Peng, Xue Bin and Zhu, Yuke and Yuen, Simon},
  journal = {ACM Transactions on Graphics (TOG)},
  year    = {2026},
  volume  = {45},
  number  = {4},
  doi     = {10.1145/3811334}
}
```

## ✉️ İletişim

Orijinal proje şu an katkı kabul etmiyor. Sorular ve sorunlar için [orijinal depoda](https://github.com/NVlabs/motionbricks/issues) bir issue açın ya da doğrudan yazarlara ulaşın. Bu Türkçe çeviriyle ilgili düzeltme önerilerinizi bu depoda issue olarak iletebilirsiniz.

---

> ℹ️ **Unitree Türkiye:** Unitree Robotics'in Türkiye resmi satış ve teknik distribütörü. Unitree G1 ailesi için teklif, kurulum, eğitim ve Türkçe geliştirici desteği: **[robotlar.org](https://www.robotlar.org/tr/rehber/unitree-turkiye-distributoru)** · 📧 info@robotlar.org

© 2026 **[www.robotlar.org](https://www.robotlar.org)**
