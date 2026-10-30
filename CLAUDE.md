# Basit Çizim Uygulaması

macOS ekranının üstünde açılan şeffaf çizim katmanı. Canlı yayında / anlatımda
mevcut ekranın üstüne çizerek anlatmak için. Tarayıcı / beyaz tahta modu ikincil.

## Mimari

| Dosya | Ne |
|--|--|
| `index.html` | Tüm çizim: canvas, hız-bazlı değişken kalınlık. Inline CSS/JS, bağımlılık yok. |
| `main.js` | Electron: her bağlı ekrana bir tam ekran şeffaf `alwaysOnTop` pencere + global hotkey + toggle'da temizle + menü çubuğu ikonu. |
| `preload.js` | `window.electronAPI` köprüsü (hide, onClear, sync/onSync, focusMe). |
| `kisayol.html` + `kisayol-preload.js` | Menü çubuğundan açılan kısayol penceresi. Kısayol `userData/ayarlar.json`'da (güncellemede korunur). Pencere açıkken genel kısayol kapalı, kapanınca geri gelir. |
| `tray/trayTemplate*.png` | Menü çubuğu ikonu (template: macOS açık/koyu boyar). Menü yalnız: Kısayolu değiştir… (yanında geçerli kısayol) ve Çık. Menüden çizim açılmaz, bilerek (Mert): çizim yalnız kısayolla. |

- Ekranda arayüz yok, bilerek: araç çubuğu çizim alanını kapatıyor, tıklamaları yiyordu. Geri ekleme.
- Kalem tek renk, her zeminde kırmızı (`index.html` → `const color`). Renk paleti bilerek yok.
- Silme yok, bilerek (Mert): silgi ve `C` temizle yok; yalnız `Cmd+Z` (son çizgi). Geri ekleme.
- `index.html` `window.electronAPI` varsa overlay modu (şeffaf), yoksa beyaz tahta modu.

### Çoklu ekran

- `wins[]` = ekran başına bir pencere, `type: "panel"` (NSNonactivatingPanel). "Displays have separate Spaces" açıkken dock'suz uygulamanın ikinci ekran penceresi ancak böyle key olup fare olayı alır.
- Key-pencere takibi main sürecinde: `startCursorFollow` 150 ms'de bir imleç konumunu okur, imleç ekran değiştirince o pencereyi `focus()` eder.
- Renderer'da focus/enter sonrası imleç `crosshair → dot` diye zorla yeniden dayatılır: Chromium focus değişince özel CSS imlecini kendiliğinden yeniden çizmiyor.
- Araç durumu (kalınlık/zemin, `clearAll`) `sync` IPC ile pencereler arası paylaşılır; main yalnız relay eder, `applying` bayrağı eko döngüsünü keser. Çizim geçmişi (`strokes`, `Cmd+Z`) ekrana özel kalır.
- Ekran sayısı değişince `rebuild`, yalnız yerleşim/çözünürlük değişince `syncBounds` (pencereleri yok etmeden konumlar).

## Kullanım: mouse asıl yol

Logitech mouse yan tuşları Logi Options+ ile atanır (kodda değil):

- Yan tuş → Keystroke `Ctrl+Alt+Cmd+Shift+9` (varsayılan) = aç / kapat. Kapatınca ekrandaki her şey silinir, her açılış temiz.

`main.js`'te `globalShortcut` ile kayıtlı. Varsayılan 4 modifierlı ("hyper")
kombinasyon bilerek: hiçbir uygulamayla çakışmaz, klavyeden yanlışlıkla basılmaz.
Kullanıcı menü çubuğundan değiştirebilir (en az ⌘/⌥/⌃ biri zorunlu).

## Klavye (ekranda düğme yok, tek görsel olmayan yol)

`[` `]` kalınlık · `Cmd+Z` geri al · `B` zemin döngüsü (şeffaf → beyaz → siyah) ·
`Esc` kapat

## İmleç

Küçük nokta (artı değil). Koyu/şeffaf zeminde beyaz, beyaz zeminde siyah.

## Çizgi hissi

"Çizgi çok kalın / çok ince / geç tepki veriyor" → yalnız `index.html` en üstteki
`TUNING` objesini oyna; alanların açıklaması kodda.

## Build ve test

```
npm install          # ilk sefer
npm start            # geliştirirken (paketlemeden)
npm run dist         # dist/mac-arm64/BasitCizim.app
```

- `Basit Çizim Uygulaması.app` → `/Applications`, çift tık. Terminal, dock ikonu, pencere yok; arka planda hotkey dinler. `openAtLogin` yalnız paketli sürümde açık.
- Değişikliği paketli `.app` üstünden dene: `npm run dist` + `.app`'i `/Applications`'a kopyala.

- İç ad (`productName`, çalıştırılabilir dosya ve yardımcı süreçler) ASCII kalmalı: `BasitCizim`. Türkçe harfli ad (Ç, ı) Electron yardımcılarını açılışta çökertiyor (SIGTRAP, çıkış 133). Görünen ad `CFBundleDisplayName` ve Release zip'indeki klasör adı: "Basit Çizim Uygulaması.app".
- Release: `ditto dist/mac-arm64/BasitCizim.app "<geçici>/Basit Çizim Uygulaması.app"` → o klasörden `ditto -c -k --keepParent` ile `Basit-Cizim-Uygulamasi-mac-arm64.zip`. Yayından önce zip'ten çıkan uygulamayı çalıştırıp açıldığını gör.

- Kullanıcı kurulumu `kur.sh` (README'deki tek satır): `releases/latest/download/Basit-Cizim-Uygulamasi-mac-arm64.zip` adına bağlı; zip adı ve içindeki "Basit Çizim Uygulaması.app" adı değişirse betik bozulur.

## Arama dışı

`dist/`
