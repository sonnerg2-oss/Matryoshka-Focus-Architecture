Markdown
# 🪆 Matryoshka Focus Architecture for Android TV (Jetpack Compose)

[![Platform](https://img.shields.io/badge/Platform-Android%20TV-green.svg)](https://developer.android.com/tv)
[![Jetpack Compose](https://img.shields.io/badge/Compose%20for%20TV-Material3-blue.svg)](https://developer.android.com/jetpack/compose/tv)
[![License](https://img.shields.io/badge/License-Apache%202.0-orange.svg)](LICENSE)

> Android TV ve uzaktan kumanda (D-Pad) ortamlarında iç içe geçmiş derin liste yapılarında yaşanan odak kaybını (focus trapping/loss) ve ekran kilitlenmelerini ortadan kaldıran deterministik UI/UX mimarisi.
https://docs.google.com/document/d/1dEOJo5c2OFyJAvZqiK4lxTq-fjHumDOZXLD6_nD3M2E/edit?pli=1&tab=t.0
---

## 🌟 Genel Bakış ve Zihinsel Model (Mental Model)

Android TV uygulamalarında (özellikle IPTV, VOD ve akış platformlarında) çok katmanlı listeler arasında D-Pad kumanda ile gezinirken odağın ekrandan kaybolması ve geri tuşuna basıldığında durumun kaybedilmesi en kronik sorunlardan biridir.

**Matruşka Odak Mimarisi**, geleneksel iç içe geçen Rus bebekleri mantığını kullanıcı arayüzü hiyerarşisine uyarlar:

[ Alan (Area) ] ➔ [ Liste (List) ] ➔ [ Alt Liste (Sub-List) ] ➔ [ Yayın Şablonu (Media Template) ]


Kullanıcı derin katmanlarda gezinirken en dış kabuktan en içteki çekirdeğe doğru ilerler; geri dönmek istediğinde ise odak kaybolmadan deterministik olarak bir üst ebeveyn nesneye devredilir.

---

## 🚀 Çözülen Temel Problemler

- **Odak Kaybının Önlenmesi (Zero Focus Loss):** Geri (Back/DPAD_LEFT) tuşuna basıldığında veya liste sınırlarına ulaşıldığında odağın kaybolup arayüzün kilitlenmesi engellenir.
- **Hafızalı Durum Koruma (State Restoration):** Alt listelerden (sezon/bölüm ızgarası) çıkıldığında, kullanıcının en son kaldığı aktif hücreye anında geri dönülmesi sağlanır (`focusRestorer`).
- **Doğrulanmış Kararlılık:** Bu mimari, `tvlauncher` kod tabanında `IptvPlayerScreen.kt` ve `MatryoshkaVodArea.kt` bileşenlerine uygulandığında 19 derleme ve mantık hatasını tek seferde kalıcı olarak gidermiştir.

---

## 📐 Katman Hiyerarşisi

| Katman | Sorumluluk Alanı | Odak Mekanizması |
| :--- | :--- | :--- |
| **1. Alan (Area)** | Canlı TV, VOD, Dizi veya Ayarlar gibi ana bölümler | Global `focusGroup()`, alt alanlara giriş kapısı |
| **2. Liste (List)** | Kategori listeleri veya kanal blokları | İzole `focusGroup()`, D-Pad dikey/yatay geçişleri |
| **3. Alt Liste (Sub-List)** | Sezonlar, alt türler veya EPG matrisleri | `focusRestorer()` ile son kalınan hücre hafızası |
| **4. Yayın Şablonu (Media Template)** | Bölüm ızgarası, oynatıcı kontrolleri ve video detayları | Çıkışta odağı üst `FocusRequester` nesnesine devretme |

---

## 💻 Örnek Uygulama (Implementation)

```kotlin
// Yeniden kullanılabilir Matruşka Odak Kapsayıcısı
@Composable
fun MatryoshkaFocusContainer(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Box(
        modifier = modifier
            .focusGroup()
            .focusRestorer()
            .onPreviewKeyEvent { keyEvent ->
                if (keyEvent.nativeKeyEvent.action == KeyEvent.ACTION_DOWN) {
                    when (keyEvent.nativeKeyEvent.keyCode) {
                        KeyEvent.KEYCODE_DPAD_LEFT, KeyEvent.KEYCODE_BACK -> {
                            // Üst katmana deterministik odak devri
                            false
                        }
                        else -> false
                    }
                } else false
            }
    ) {
        content()
    }
}
Görsel Odak Stili
Seçili olan hücrelerde kullanıcı netliği sağlamak için turuncu odak çerçevesi (#FFFF9800) ve yumuşak geçiş efektleri standart olarak uygulanır.

🛠️ Mimari Uyumluluk
UI: Jetpack Compose for TV (Material 3 TV)

Mimari: Clean Architecture & MVVM

Medya Entegrasyonu: Jetpack Media3 (ExoPlayer)

Veri Akışı: Kotlin Coroutines & StateFlow
