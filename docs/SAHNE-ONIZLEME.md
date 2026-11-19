> Bu belge NVlabs/motionbricks deposundaki `docs/scene_preview.md` dosyasının Türkçe çevirisidir (CC-BY-4.0). Değişiklik: Türkçeleştirme.

# Scene Preview (sahne önizleme)

[Scene Preview](https://github.com/NVlabs/motionbricks/blob/main/app/scene_preview.py), tek bir karakteri sürekli yürüyüş/hareket (lokomosyon) ve etkileşimlerle bir sahnenin içinde yönetir. MotionBricks üzerine kurulmuş küçük bir karakter çalışma zamanıdır. Hedef pozlara (target pose) durumlar karar verir. Motor onlara doğru üretir. Karakter sonucu oynatır ve bir sonraki üretim için kaydeder.

<table>
  <tr>
    <th>SOMA</th>
    <th>Unitree G1</th>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/scene_preview_soma.gif" alt="SOMA ile Scene Preview" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/scene_preview_g1.gif" alt="Unitree G1 ile Scene Preview" width="100%"></td>
  </tr>
</table>

Bu sayfa önce uygulamanın nasıl çalıştırılacağını gösterir, sonra bir karakterin sahnede kesintisiz hareket etmesini ve nesnelerle etkileşmesini isteyen geliştiriciler için çalışma zamanını açıklar. Her parça düz Python'dur. SOMA ve Unitree G1 karakterlerini aynı kod çalıştırır.

[Rehberli eğitimin 4. bölümü](REHBERLI-EGITIM.md#bölüm-4-sahneyi-oynatın), eğitimde kurulan etkileşim sahnesini oynatır. Aşağıdaki bölümler orada gördüklerinizi kodla ilişkilendirir.

## Scene Preview'u çalıştırın

```sh
uv run python -m app.scene_preview --scene assets/scenes/soma_gym.scene.json
uv run python -m app.scene_preview --scene assets/scenes/g1_gym.scene.json
```

- `--scene` bir sahne belgesi açar. Belgenin rig'i SOMA'yı ya da Unitree G1'i seçer. Varsayılan SOMA spor salonudur (gym).
- `--model-precision fp16` karma FP16 modelini kullanır. Varsayılan FP32'dir. FP16 ve FP32 hareketinin nasıl farklılaşabileceğini [TensorRT derlemeleri](CALISMA-ZAMANI-API.md#tensorrt-derlemeleri) açıklar.
- `--verbose` ayrıntılı günlük iletilerini gösterir.

Spor salonu sahnelerinde küçük bir atlama, bir engel aşma (vault), altından eğilerek geçilecek bir çubuk ve üzerine oturulacak bir kutu bulunur. Kliplerden oynayan bekleme ve oturma döngüleri dışında her hareketi model üretir.

Deneyebilecekleriniz:

- Her nesneye doğru yürüyün. **Auto Interact** açıkken karakter vardığında atlar, engeli aşar, çubuğun altından eğilerek geçer ya da kutuya oturur.
- Aynı nesneye başka açılardan ve başka hızlarla yaklaşın. Pivotlar her diziyi yaklaşmaya doğru döndürür ve kaydırır. Atlamalar ve çubuk, varyasyonlarını hıza göre seçer.
- Otururken ayağa kalkmak için **I**'ya (kumandada **X**) basın.
- **Scene** bölümünde **Auto Interact**'i kapatın, sonra bir nesnenin yakınında **I**'ya (kumandada **X**) basarak etkileşimini elle başlatın.
- **P** (kumandada **B**) ile Crouching moduna geçin ve çubuğun altından yürüyün. Çubuğun yükseltilmiş etkileşim kutusu yalnız ayakta duran bir karakteri yakalar, bu yüzden etkileşim başlamaz.
- **O** (kumandada **Y**) ile Strafing moduna geçin ya da **[** (kumandada **A**) ile stil modları arasında sırayla dolaşın.
- Her üretimin arkasındaki pozları ve kutuları görmek için **Visibility** altındaki **Show Context Poses** ve **Show Probe Box** gibi seçenekleri açın.
- **Scene**'in yanındaki **...** ile öteki spor salonunu açın. Sahnenin rig'i karakteri SOMA ile Unitree G1 arasında değiştirir.

### Kontroller

| Eylem | Klavye ve fare | Oyun kumandası |
| --- | --- | --- |
| İleri, sola, geri ya da sağa hareket | **W**, **A**, **S** ve **D** ya da **ok tuşları** | Sol çubuk ya da D-pad |
| Koşma | Hareket ederken **Space**'i basılı tutun | Sol çubuğu daha fazla itin |
| Elle etkileşim ya da oturduktan sonra kalkma | **I** | **X** |
| Forward ve Strafing modları arasında geçiş | **O** | **Y** |
| Crouching ve Strafing modları arasında geçiş | **P** | **B** |
| Skipping ve Zombie gibi stil modları arasında sırayla geçiş | **[** | **A** |
| Kamerayı karakterin çevresinde döndürme | **Orta fare tuşuyla** sürükleyin | Sağ çubuk |

## Kod haritası

Çalışma zamanının modelle konuşan akışını şu semboller taşır:

| Sembol | Sorumluluk |
| --- | --- |
| [scene_preview.py](https://github.com/NVlabs/motionbricks/blob/main/app/scene_preview.py) içindeki `ScenePreview.run()` | Bekleyen bir sahneyi kareler arasında yükler, sonra her kareyi motor akışında (stream) ilerletip çizer. |
| `ScenePreview.step()` | Girdiyi okur, karakteri ilerletir, kamerayı taşır, sonra etkileşimleri günceller. |
| `ScenePreview.load_scene()` | Bir sahneyi okuyup denetler, rig'ini ve Smart Primitive'lerini yükler, sonra karakteri oluşturur. Pencereden açılan bir sahne, motoru yüklenmeden önce hata verirse o anki sahne çalışmayı sürdürür ve pencere hatayı gösterir. |
| `ScenePreview._update_smart_primitive_interaction()` | Etkileşim kutusu yoklama kutusuyla kesişen en yakın Smart Primitive'i bulur. |
| `ScenePreview._dispatch_interaction_input()` | Bir etkileşimi elle ya da **Auto Interact** ile başlatır veya bir tuş basışını çalışan davranışa iletir. |
| [character.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/character.py) içindeki `BaseCharacter.step()` | Durum makinesini günceller, o anki pozu oynatır ve bir sonraki üretim için kaydeder. |
| `BaseCharacter.generate_to_target_poses()` | Kaydedilen pozlardan yerleştirilmiş hedef pozlara doğru üretir, sonra oynatmayı yeniden başlatır. |
| [state_machine.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/state_machine.py) içindeki `StateMachine.update()` | Modları değiştirir, hareketi yumuşatır, sonra o anki durumu günceller. |
| [states.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/states.py) içindeki `IdleState`, `LocomotionState` ve `InteractionState` | Ne zaman, hangi hedeflere doğru üretileceğine ve başka bir duruma ne zaman geçileceğine karar verir. |
| [target_pose_blender.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/target_pose_blender.py) içindeki `BaseTargetPoseBlender.evaluate()` | Lokomosyon hedeflerini istenen hıza göre karıştırır ve karakterin önüne yerleştirir. |
| [motion.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/motion.py) içindeki `LocomotionMode` ve `LocomotionSettings` | Her modun hedeflerini ve hızlarını, hareketin girdiye ne kadar çabuk yanıt verdiğini ve lokomosyonun ne zaman yeniden ürettiğini tutar. |
| [smart_primitive_behaviors.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/smart_primitive_behaviors.py) içindeki `SmartPrimitiveBehavior` | Sahnelerin seçtiği davranışların temel sınıfı, örneğin `VelocityMatchingSequenceBehavior` ve `SittingBehavior`. |
| [app/runtime/soma](https://github.com/NVlabs/motionbricks/tree/main/app/runtime/soma), [app/runtime/unitree_g1](https://github.com/NVlabs/motionbricks/tree/main/app/runtime/unitree_g1) | Her rig'in karakteri, hedef poz karıştırıcısı ve hazır ayarları. |

`scene_preview.py` dosyasının geri kalanı kamerayı, çizimi ve **Scene Preview** penceresini yönetir.

## Ana güncelleme döngüsü

`ScenePreview.run()` bekleyen bir sahneyi kareler arasında yükler. Yükleme başka bir motor derleyebilir, bu yüzden etkin motorun akışının dışında çalışır. `run()` sonra motor akışına girer ve `step()` ile `render()`'ı çağırır.

Her `step()`:

1. Son kareden bu yana geçen süreyi ölçer.
2. Tuşları ve kumandayı okur, sonra kamerayı döndürür.
3. Kameraya göreli hareketten, koşma girdisinden ve istenen moddan bir `LocomotionCommand` kurar.
4. `BaseCharacter.step()`'i çağırır.
5. Kamerayı karakterle birlikte taşır.
6. Yoklama kutusunu her etkileşim kutusuna karşı sınar.
7. Bir etkileşimi başlatır ya da bir tuş basışını çalışan davranışa iletir.

Karakter, etkileşim sınamasından önce ilerler. Bu karede başlayan bir etkileşim ilk segmentini bu yüzden önceden kaydedilmiş pozlardan üretir.

### Karakter güncellemesi

`BaseCharacter.step()` karakteri kare süresi kadar ilerletir:

```python
self.state_machine.update(self, delta_time, command)
previous_ground_position = self.character_ground_position.copy()
self._sample_playback(delta_time)
self._update_character_velocity(previous_ground_position, delta_time)
self.player.record_pose(delta_time, self.current_pose)
self._update_probe_box()
```

1. Durum makinesi hareketi ve o anki durumu günceller. Bu durum yeni hareket üretebilir ya da bir klip başlatabilir.
2. `_sample_playback()` gösterilecek pozu `current_pose` içine yazar. Üretilen hareketi ya da bir klibi örnekler.
3. `_update_character_velocity()` karakterin ne kadar yol aldığını ölçer.
4. `record_pose()` gösterilen pozu saklar.
5. `_update_probe_box()` yoklama kutusunu karakterin önüne taşır.

Karakter üç kaynaktan birinden oynar:

- `GENERATED`: Hedef pozlara doğru üretilen hareket.
- `CLIP_TRANSITION`: Bir klibin ilk karesine doğru üretilen hareket.
- `CLIP`: Bir kez ya da döngüde oynatılan klibin kendisi.

### Kaydedilen hareketten üretin

Her durum `generate_to_target_poses()` üzerinden üretir. Karakterin hareket oynatıcısı, kaydedilen geçmişten en son pozda biten dört bağlam pozunu (context pose) 30 Hz'de örnekler. Sonra motoru çağırır ve oynatmayı yeniden başlatır. Bunlar [Inbetweener](ARA-KARE-URETICI.md#üretim-akışı) ile aynı çağrılardır.

Kaydedilen geçmiş, klip kareleri ve [IK Pins](SMART-PRIMITIVE-EDITORU.md#ik-pins) düzeltmeleri dahil gerçekten gösterilmiş pozları tutar. Bir üretimin bir klipten ya da başka bir üretimin ortasından başlayabilmesinin nedeni budur. Bir durum üretimi her an başlatabilir. Başlatmazsa o anki segment geçiş noktasına kadar oynar.

## Durum makinesi ve durumlar

![Idle, Locomotion ve Interaction durumlarıyla Scene Preview durum makinesi](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/state_machine.png)

`StateMachine` beş durum tutar:

| Durum | Girişte | Her güncellemede |
| --- | --- | --- |
| Idle, Crouching Idle | Bir bekleme klibine geçiş üretir | Hareket girdisi geldiğinde lokomosyona geçer. Geçiş bitince klibi döngüye alır. |
| Locomotion, Crouching Locomotion | Karıştırılmış hedefe doğru üretir | Girdi durunca beklemeye geçer. Gerektiğinde yeniden üretir. |
| Interaction | Smart Primitive'in davranışını başlatır | Her geçiş noktasında sıradaki hedefe doğru üretir ya da klibin bitmesini bekler. |

Bekleme ve lokomosyon, ayakta ve çömelik olmak üzere iki çift halinde gelir. Ayakta lokomosyonun Forward, Strafing ve stil modları var. Çömelik lokomosyonun Crouching modu var ve yoklama kutusu yüzde 60 yüksekliğindedir. Bir mod tuşu öteki çiftten bir mod seçtiğinde karakter, girdiye göre o çiftin bekleme ya da lokomosyon durumuna geçer.

`StateMachine.update()` her karede komut bir mod istiyorsa modu değiştirir. Sonra hareketi yumuşatır ve o anki durumu günceller. Her geçiş yeni duruma girmeden önce eski durumdan çıkar. Hareket her durumda yumuşatılmaya devam eder, böylece bir sonraki duruma taşınır.

### Idle (bekleme)

Bekleme durumu döngüdeki klipleri oynatır, klibe girişi de model üretir:

1. Rig'in bekleme kliplerinden o ankinden farklı birini rastgele seçin.
2. Klibi karakterin gerçek hızı doğrultusunda bir saniye ileriye yerleştirin. Klibi son hareket girdisinin yönelimine çevirin.
3. Klibin ilk dört pozunu 30 Hz'de hedef belirtecine örnekleyin. Bu pozlara doğru bir geçiş üretin.
4. Geçiş bitince klibi döngüde oynatın.

Bir sahne başladığında karakter doğrudan bir bekleme döngüsünde başlar ve geçmişi o ilk pozla doldurulur.

### Interaction (etkileşim)

`start_interaction()`, bir Smart Primitive'in [davranışıyla](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/smart_primitive_behaviors.py) etkileşim durumuna girer. Davranış işi bitince `finish()`'i çağırır. `resume_movement()` sonra hareket girdisi varsa lokomosyonu, yoksa beklemeyi seçer. `on_complete` geri çağrısı olmadan oynatılan bir dizi ya da klip, bittiğinde `finish()`'i çağırır.

[Rehberli eğitimde](REHBERLI-EGITIM.md#çalışma-zamanını-inceleyin) kaçınma ve atlama etkileşimleri dizileri biter bitmez lokomosyona döndü. Oturma etkileşimi ise klibi döngüdeyken etkileşim durumunda kaldı.

## Sürekli lokomosyon

<table>
  <tr>
    <th>SOMA</th>
    <th>Unitree G1</th>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_targets_soma.gif" alt="SOMA bağlam, hedef ve karışım uzayı pozları görünürken yürüyor, koşuyor ve duruyor" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_targets_g1.gif" alt="Unitree G1 bağlam, hedef ve karışım uzayı pozları görünürken yürüyor, koşuyor ve duruyor" width="100%"></td>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_directional_soma.gif" alt="SOMA bağlam, hedef ve karışım uzayı pozları görünürken dört yöne yan adımla ilerliyor, çömeliyor ve yeniden ayağa kalkıyor" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_directional_g1.gif" alt="Unitree G1 bağlam, hedef ve karışım uzayı pozları görünürken dört yöne yan adımla ilerliyor, çömeliyor ve yeniden ayağa kalkıyor" width="100%"></td>
  </tr>
</table>

*Üstte: Forward modunda her karakter tam koşuya kadar hızlanır, sonra durup beklemeye geçer. Altta: Strafing modunda her karakter tek bir yöne bakarken ileri, sağa, geri ve sola hareket eder. Sonra hareket halindeyken Crouching moduna, durmadan önce de yeniden Strafing moduna geçer. Bağlam pozları mor, o anki hedefler mavi, canlı karışım uzayı pozu yeşildir.*

Lokomosyon, karakterin önüne yerleştirilmiş bir hedefe doğru üretmeyi sürdürür. Her üretim az önce gösterilen pozlardan başlar, böylece hareket kesintisiz devam eder. Hareket girdisi kameraya görelidir. `MovementControl` bu girdiyi yumuşatıp istenen bir hıza ve bakış yönüne çevirir. Strafing ve Crouching modlarında karakter kameranın yönüne bakar.

### Lokomosyon modları

Her rig'in `presets.py` dosyası kendi lokomosyon modlarını tanımlar. Bir `LocomotionMode`, karışım hedeflerinden oluşan bir Smart Primitive belgesini, her hedefin karışım değerini ve hız sınırlarını adlandırır. Karışım hedefleri, lokomosyon kliplerinden örneklenmiş dört poz tutar:

| Mod | Karışım uzayı (blend space) | Hedefler |
| --- | --- | --- |
| Forward | 1B, hıza göre | Idle, Walk, Jog, Run |
| Strafing | 2B, bakış yönüne göreli hıza göre | Idle, Forward, Backward, Left, Right |
| Crouching | 2B, bakış yönüne göreli hıza göre | Idle, Forward, Backward, Left, Right |
| Styles | 1B, hıza göre | Idle, ardından Skipping, Vogue, Injured Torso, Injured Leg, Zombie ya da Combat |

Altı stil modu tek bir belgeyi, `locomotion_style_variations_bs1_linear` belgesini paylaşır. Her biri 0'daki Idle hedefini 1'deki kendi stil hedefine karıştırır. Bir stil modunun varsayılan hızı en yüksek hızıdır, böylece stil pozu tam olarak görünür. Hedefleri de daha ileride durur ve mod, klibinin hızına yakın bir hızla yürür. Geçiş noktası her geçişin sonuna daha yakındır, böylece hareket stil pozuna daha çok yaklaşır. Başka bir moddayken **[** (kumandada **A**) en son kullanılan stil moduna döner. Bir stil modundayken **O** (kumandada **Y**) Forward moduna döner, **P** (kumandada **B**) de Crouching moduna geçer.

<table>
  <tr>
    <th>SOMA</th>
    <th>Unitree G1</th>
  </tr>
  <tr>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_styles_soma.gif" alt="SOMA, Skipping'den Combat'a her lokomosyon stilinde yürüyor" width="100%"></td>
    <td><img src="https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/locomotion_styles_g1.gif" alt="Unitree G1, Skipping'den Combat'a her lokomosyon stilinde yürüyor" width="100%"></td>
  </tr>
</table>

*SOMA ve Unitree G1, adları sol üstte yazan altı stil modunun her birinde yürüyor*

### Hedefi karıştırın ve yerleştirin

Lokomosyon her ürettiğinde şunları yapar:

1. Karışım uzayını istenen hızda değerlendirir. Hedefleri karıştırır.
2. Karıştırılmış hedefin zemin hızını, istenen hız ile modun `velocity_scale` değerinin çarpımına ölçekler.
3. Karıştırılmış hedefi, ölçeklenmiş istenen hız doğrultusunda karakterin `position_scale` saniyelik yol önüne yerleştirir. Hedefi bakış yönüne çevirir. Yalnız kök (root) hareket eder.
4. Modun **Transition Early By** değeri ve belirteç sayılarıyla dört hedef pozun hepsine doğru üretir.

İlk üç adımı `BaseTargetPoseBlender.evaluate()` çalıştırır. Her rig, karıştırma, hız ve yerleştirme adımlarını kendi poz düzeni için geçersiz kılar.

Bunlar klasik animasyon karışım uzayları değildir. Oyun motorlarındakiler gibi klasik bir karışım uzayı, oynarken birkaç klibi birbirine karıştırır. Kliplerin adım adıma uyması gerekir, bu yüzden oyun motoru ayak evrelerini eşzamanlar ve klipleri zamanda uzatır. Burada ise karışım değeri, her klipten bir kez örneklenmiş hedef pozları karıştırır. Karıştırılmış hedefe doğru hareketi model üretir, böylece eşzamanlanacak ya da uzatılacak bir şey kalmaz. Kaba bir karışım yeterlidir, çünkü model kusurlu hedeflere karşı dayanıklıdır ve lokomosyon her hedefe ulaşılmadan önce yeniden üretir. [Makalenin](https://arxiv.org/abs/2604.24833) D ekine bakın.

### Yürüyüş ne zaman üretir

`LocomotionState.update()` her karede karar verir:

```python
if character.movement.input_speed <= 0.0:
    state_machine.transition_to_idle(character)
    return

if (
    self._force_regeneration
    or character.time_to_transition() <= delta_time
    or self._needs_regeneration(character.movement)
):
    self._generate(character)
```

Hareket girdisi yoksa karakter beklemeye geçer. Girdi varsa lokomosyon üç durumda üretir:

- Mod değişmiştir ve bu `_force_regeneration`'ı ayarlar.
- Oynatma o anki segmentin geçiş noktasına ulaşmıştır.
- Girdi yeterince değişmiştir. `_needs_regeneration()`, karakterin harekete başlayıp başlamadığını, hızın kalan hız aralığının belli bir kesrinden fazla değişip değişmediğini ya da yönün bir eşiği aşacak kadar dönüp dönmediğini denetler. Bu eşikleri [motion.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/motion.py) içindeki `LocomotionSettings` tutar.

Bu kurallar ne zaman üretileceğine karar vermenin yollarından biridir. Sabit bir zamanlamayla ya da yalnız geçiş noktalarında üretmek gibi başka kurallar da işe yarar. Kendi kurallarınızı `LocomotionState` içinde deneyin ve hareketin nasıl yanıt verdiğini karşılaştırın.

### Görüntüleyicide izleyin

**Scene Preview** penceresinde **Visibility** altındaki **Show Target Poses**, **Show Context Poses** ve **Show Blend Space Poses** seçeneklerini açın. Bunlar o anki üretimin yerleştirilmiş hedeflerini, bir sonraki üretimin okuyacağı bağlam pozlarını ve o anki karenin karışım uzayı hedefini gösterir. Karışım uzayı hedefi yalnız lokomosyon sırasında görünür. Rehberli eğitimin [Çalışma zamanını inceleyin](REHBERLI-EGITIM.md#çalışma-zamanını-inceleyin) adımında açtığınız seçenekler bunlardır. **Show Foot Contacts**, modelin [ayak temaslarını](CALISMA-ZAMANI-API.md#ayak-temasları) karakterin tabanlarında çizer.

## Smart Primitive davranışları

### Etkileşim bulma

Karakter, istenen hız doğrultusunda önüne uzanan bir yoklama kutusu (probe box) taşır. Scene Preview her karede bu kutuyu Smart Primitive'lerin her etkileşim kutusuna karşı sınar ve kesişen en yakın örneği tutar. Etkileşim kutuları yalnız davranışları tetikler. Model onları hiçbir zaman çarpışma geometrisi olarak görmez.

- **Auto Interact** varsayılan olarak açıktır. Her yaklaşmada bir etkileşimi bir kez başlatır.
- **I** (kumandada **X**) bir etkileşimi elle başlatır. Etkileşim sırasında aynı basış çalışan davranışa gider. `SittingBehavior` karakteri bu yolla ayağa kaldırır.

### Davranış çalıştırma

`StateMachine.start_interaction()` etkileşim durumuna girer ve bu durum davranışın `enter()` yöntemini çağırır. Davranış sonra dizileri ve klipleri etkileşim durumu üzerinden oynatır:

- `play_target_pose_sequence()` bir dizinin hedefleri boyunca sırayla üretir. Her adım kendi **Poses to Use**, **Transition Early By** ve **Tokens to Generate** ayarlarını korur. Bir SOMA adımı ayrıca [IK Pins](SMART-PRIMITIVE-EDITORU.md#ik-pins) ile elleri ya da ayakları sabitleyebilir.
- `play_clip()` bir klibi bir kez ya da döngüde oynatır. Varsayılan olarak klibin ilk kök pozu karakterin o anki köküyle hizalanır.
- `finish()` denetimi lokomosyona ya da beklemeye geri verir.

Her geçiş noktasında `InteractionState.update()` dizinin sıradaki hedefine doğru üretir. Son hedeften sonra dizinin `on_complete` geri çağrısını çalıştırır. Her adımın geçiş noktasını o adımın **Transition Early By** değeri belirler.

### Hazır davranışlar

Bir sahnedeki her Smart Primitive örneği kendi davranış sınıfını adlandırır. [smart_primitive_behaviors.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/smart_primitive_behaviors.py) içindeki `behavior_types()`, her `SmartPrimitiveBehavior` alt sınıfını sınıf adıyla bulur:

| Sınıf | Ne yapar |
| --- | --- |
| `VelocityMatchingSequenceBehavior` | Hazırlanmış başlangıç hızı karakterin hızına en yakın diziyi seçer, pivotunu hizalar ve oynatır |
| `SittingBehavior` | Sit dizisini oynatır, oturma klibini döngüye alır ve bir sonraki basışta Stand dizisini oynatır |

[Rehberli eğitimin 3. bölümünde](REHBERLI-EGITIM.md#bölüm-3-kaçınma-atlama-ve-oturma-içeren-bir-etkileşim-sahnesi-kurun) kaçınma ve atlama için `VelocityMatchingSequenceBehavior`'ı seçtiniz. Oturma için `SittingBehavior`'ı seçtiniz.

Spor salonu sahnelerindeki hazır kaçınma belgesinin bir Walk ve bir Jog dizisi var. `VelocityMatchingSequenceBehavior` aralarından hıza göre seçim yapar:

![VelocityMatchingSequenceBehavior, Walk ya da Jog dizisini hıza göre seçer](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/velocity_matching_flow.png)

`SittingBehavior` Sit dizisini oynatır, sonra bir sonraki basışa kadar bir oturma klibini döngüde tutar. Ardından Stand dizisi ayağa kalkma hareketini üretir:

![SOMA oturuyor, bekliyor, ayağa kalkıyor ve yürüyüp gidiyor](https://media.githubusercontent.com/media/NVlabs/motionbricks/main/docs/media/sit_hold_soma.gif)
*SOMA oturuyor, bekliyor, ayağa kalkıyor ve yürüyüp gidiyor*

### Davranış yazın

Bir davranış eklemek için `SmartPrimitiveBehavior`'dan bir alt sınıf türetin, `enter()` ve `interact()` yöntemlerini geçersiz kılın ve gereken dizileri ve klipleri bildirin:

```python
class LeaningBehavior(SmartPrimitiveBehavior):
    """Lean on a wall, stay there, and step away when the user interacts again."""

    required_sequences = ("Lean", "StepAway")
    clip_paths = (LEAN_LOOP_CLIP,)

    def enter(self) -> None:
        self.interaction_state.play_target_pose_sequence("Lean", on_complete=self._hold)

    def _hold(self) -> None:
        self.interaction_state.play_clip(LEAN_LOOP_CLIP, loop=True, align_to_current_pose=True)

    def interact(self) -> None:
        if self.interaction_state.is_looping:
            self.interaction_state.play_target_pose_sequence("StepAway")
```

Sınıf bundan sonra Scene Editor'ın **Behavior** listesinde görünür. Onu `smart_primitive_behaviors.py` içinde tanımlayın ya da modülünü sahneler yüklenmeden önce içe aktarın. Bir davranışın bildirdiği klipler sahneyle birlikte yüklenir. Bildirilen diziler örnek oluşturulurken denetlenir. Bir bekleme pozu için dizilerin nasıl hazırlanacağını [Smart Primitive Editor](SMART-PRIMITIVE-EDITORU.md#örnek-oturup-bekleme) sayfası gösterir.

Hazır davranışlar ve bu örnek, pek çok kurulumdan yalnız biridir. Model yalnız bağlam ve hedef pozlara ihtiyaç duyar, dolayısıyla bir davranışın hedeflerini nasıl seçip zamanlayacağı size kalmıştır. Bunları başlangıç noktası olarak kullanın ve kendi fikirlerinizi keşfedin.

## Sahneler

Bir `.scene.json` belgesi rig'ini, CharacterStart'ı, Smart Primitive örneklerini ve Solid Cube'ları kaydeder. Her örnek kendi yerleşimi, davranışı ve görünürlük ayarlarıyla bir Smart Primitive belgesine başvurur. Aynı belgenin örnekleri tek bir Smart Primitive varlığını paylaşır. Sahneleri [Scene Editor](SAHNE-EDITORU.md) oluşturur. Dosyalar düz JSON'dur ve onları elle de düzenleyebilirsiniz.

`load_scene()` öğe hiyerarşisini dünya dönüşümlerine düzleştirir, çünkü üst öğelerin tek işi düzenlemeyi kolaylaştırmaktır. Bir motor derlemeden önce her örneği denetler. Yeni bir Smart Primitive öğesinin **Behavior** alanı boştur, bu yüzden sahneyi kaydetmeden önce bir davranış seçin.

## Çalışma zamanını keşfedin ve uyarlayın

Scene Preview, üzerine kurmanız gereken bir çatı değil, öğrenmek için bir başvuru örneğidir. Hareketin nasıl yanıt verdiğini görmek için parçalarını değiştirin, sonra fikirleri kendi motorunuza ya da araçlarınıza taşıyın. Her parçanın kendi yeri var:

| Amaç | Nereye bakılır |
| --- | --- |
| Etkileşim ekleme | Bir Smart Primitive belgesi hazırlayın ve mevcut sınıflar uymuyorsa bir davranış sınıfı yazın. Bkz. [Davranış yazın](#davranış-yazın). |
| Lokomosyon pozlarını değiştirme | Her rig'in `presets.py` dosyasının adlandırdığı karışım hedefi belgeleri, örneğin `locomotion_bs1_linear`. |
| Lokomosyonun hızlarını ya da geçiş noktasını değiştirme | Rig'in `presets.py` dosyasındaki `LocomotionMode`. |
| Lokomosyonun ne zaman ürettiğini değiştirme | [motion.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/motion.py) içindeki `LocomotionSettings` ya da `LocomotionState` içindeki kurallar. |
| Etkileşimlerin ne kadar önceden başladığını değiştirme | Her rig'in hazır ayarlarındaki `INTERACTION_PROBE`: [soma/presets.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/soma/presets.py) ve [unitree_g1/presets.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/unitree_g1/presets.py). Her değeri [interaction.py](https://github.com/NVlabs/motionbricks/blob/main/app/runtime/common/interaction.py) açıklar. |
| Aynı kalıbı başka bir motorda kurma | Gösterdiğiniz her pozu kaydedin ve her bağlamı 30 Hz'de en son dört pozdan kurun. Hedefleri kendi durumlarınız seçip yerleştirsin. Her geçiş noktasında ya da durumlarınız istediğinde yeniden üretin. Parçaları [Üretimleri zincirleme](CALISMA-ZAMANI-API.md#üretimleri-zincirleme) anlatır. |
| Bu çalışma zamanını başka bir Python uygulamasında yeniden kullanma | Rig'in karakterini, örneğin `SOMACharacter`'ı, bir motor ve bir `SmartPrimitiveAssetPool` ile oluşturun. Her karede onun `step()` yöntemini bir `LocomotionCommand` ile bir kez çağırın, sonra `current_pose`'u çizin. Etkileşimleri `ScenePreview`'un yaptığı gibi `state_machine.start_interaction()` ile başlatın. |

## Ek kaynaklar

- [Rehberli eğitim](REHBERLI-EGITIM.md): Bu uygulamayla biten, kullanıcıya dönük iş akışı.
- [Çalışma Zamanı API'si](CALISMA-ZAMANI-API.md): Motorlar, hareket oynatıcıları, üretim ayarları ve zincirleme.
- [Smart Primitive Editor](SMART-PRIMITIVE-EDITORU.md): Davranışların oynattığı hedef pozları, dizileri ve pivotları hazırlama.
- [Scene Editor](SAHNE-EDITORU.md): Smart Primitive örneklerini yerleştirme ve davranışlarını seçme.
- [Inbetweener](ARA-KARE-URETICI.md): Durum makinesi olmadan aynı üretim akışı.

---

**Unitree G1 Türkiye:** Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org. [Unitree G1 ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat)
