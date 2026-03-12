# Master Level Meter (OBS Plugin)

音声レベル表示 (RMS / Peak / Short LUFS) を行う OBS Studio 用フローティング・ウィンドウ・プラグインです。

Master の任意トラック (Track1..Track6) のメーター表示と、配信設定で使用される音声マスタートラックの音量が見れるようになります。

## ダウンロード
https://github.com/ShmKnd/MasterLevelMeter/releases/tag/v1.0.2a

---
## インストール方法

### macOS (.pkg インストーラー)
ダウンロードした `MasterLevelMeter.pkg` をダブルクリックすると、macOS 標準のインストーラーが起動し、自動的に以下のパスへインストールされます：
```
~/Library/Application Support/obs-studio/plugins/MasterLevelMeter.plugin/
```

### macOS (手動インストール)
`MasterLevelMeter.plugin` を以下のフォルダに配置してください：
```
~/Library/Application Support/obs-studio/plugins/
```

### Windows
```
C:\Program Files\obs-studio\obs-plugins\64bit\MasterLevelMeter.dll
```

未確認ですが、Streamlabs OBS でも同様の場所に配置すれば動作すると思われますが動作保証しません。
**インストール後、OBSを再起動してください。**

## Macユーザーへ
本プラグインは **Apple Developer ID で署名済み** です（Developer ID Application: Shoma Kondo）。
macOS の Gatekeeper による「確認できないため開けません」エラーは通常発生しません。

**対応アーキテクチャ:** Intel (x86_64) / Apple Silicon (arm64) のユニバーサルバイナリです。

もし万が一警告が表示された場合は、システム設定 → プライバシーとセキュリティ から「それでも許可」をクリックしてください。

---
## 使い方

- OBS メニュー: ドック → Master Level Meter
- Track ボタン: 対象トラック切替 (Track1..Track6)
- Streaming uses は 1 秒周期で設定から反映。設定 > 出力 >　配信タブ > 音声トラックで設定しているもの。出力モードが「基本」になっていると"Track1"になります
---


---
## 機能
- Master Track1..Track6 を流れる信号の表示
- Dual-channel (L/R) メータ:
- RMS
- Peak
- Short LUFS (3秒 ITU-R BS.1770 K-weighted 処理)
- K-weighting フィルタ:
- 二段ハイパス (60Hz) + High-shelf (+4 dB @ ~1.7 kHz) 実装
- -23 / -18 LUFS 強調目盛り
- Dock対応
- Streaming 設定から現在利用されるトラックの表示更新
  
---
### Audio Flow
1. OBS の `audio_output_connect` 経由で planar float オーディオフレーム取得
2. `audio_callback` で選択 Mix のバッファを `LevelCalc::process()` へ
3. K-weighting & サブブロック（100ms hop / 3000ms window）処理
4. Qt UI (約 60fps タイマ) が `updateLevelsLR()` を呼び内部状態を更新 → 再描画

  

### Loudness (Short)
- Short: 3000ms ウィンドウ平均エネルギー（ch 合算）→ -0.691 オフセット

---
## Qt 6 Usage
- モジュール: Core / Gui / Widgets
- CMake: `find_package(Qt6 COMPONENTS Core Widgets REQUIRED)`
- AUTOMOC 有効化
- 動的リンク前提（LGPLv3 への準拠）
静的リンクはライセンス義務が増すため非推奨。
> NOTE: obs-deps 付属の Qt (ヘッダ/Config が省略される場合) を直接利用するより、開発環境では公式 Qt SDK または Homebrew (macOS: `brew install qt`) を推奨。
---
## 【開発者向け】ビルド方法 
CMake関連ファイルは[obs-plugintemplate](https://github.com/obsproject/obs-plugintemplate) をベースにしています。

初回ビルド時に OBS Studio ソース・依存ライブラリ (Qt6 含む) が自動ダウンロードされます。

### macOS Build (Universal Binary: Intel + Apple Silicon)
```bash
# CMake プリセットで構成 (Xcode ジェネレータ + Universal)
cmake --preset macos

# Release ビルド
cmake --build build_macos --config Release

# .pkg インストーラー生成
cmake --install build_macos --config Release
```

コード署名を有効にするには環境変数を設定してから configure してください:
```bash
export CODESIGN_IDENT="Developer ID Application: Your Name (TEAMID)"
export CODESIGN_TEAM="TEAMID"
cmake --preset macos
```

### Windows (PowerShell)
```powershell
cmake --preset windows-x64
cmake --build build_x64 --config Release
```
> `OBS::libobs` は自動でダウンロード・ビルドされます。
---

## 備考
- 音声コールバックは最小限計算 (K-weighting + accumulations)
- 3000ms window / 100ms hop
- Atomic 変数で UI スレッドとのロックレス共有（OBS main/Qt GUI thread）
---

## License
This plugin: GPL-2.0-or-later
Links dynamically to Qt6 (LGPLv3). See `LICENSE`.
OBS Studio (libobs) is GPLv2 (or later – confirm upstream license text).
If you bundle Qt frameworks, include LGPLv3 text and allow replacement.

---
## Third-Party Notices (Summary)
| Component | License | Notes |
|-----------|---------|-------|
| OBS (libobs) | GPLv2 | Core streaming / audio API |
| Qt 6 (Core/Gui/Widgets) | LGPLv3 | Dynamic link only |
| (Transitive: FFmpeg, x264, etc.) | Mixed (LGPL/GPL) | Via OBS, not directly redistributed unless you package them |
---
## セキュリティ / プライバシー
- このプラグインはネットワークにアクセスしません。
- ウィンドウサイズと表示トラックを、OBSを閉じても保存できるように`QSettings` (ローカルのOSに依存した保存領域)のみを使用します。
---
## ロードマップ (やりたいこと・アイディア)
- トゥルーピーク対応
- スキン・カラーテーマの対応
- ドック対応（これが難しいんだ）
- ストリーミングではなく録画時のマスタートラック表示（録画はトラックが一つだけとは限らないからUIどうするか）
---
## Q&A
- OBSのバージョンは？
    - OBS 29.0 以降を想定しています (Qt6 / C++17 必須)。
    - Windows OBS 29.1.3/31.1.2で動作確認済み
    - macOS 14.6 & OBS 31.1.2(Apple Silicon)で動作確認済み
    - macOS 15.3 & OBS 31.1.2(Intel)で動作確認済み *ただしIntel Build

- macOSのバージョンは？
    - macOS 12.0 (Monterey) 以降を想定しています (Qt6 / C++17 必須)。
    - Intel / Apple Silicon 両対応（ユニバーサルバイナリ）。

- windowsのバージョンは？
    - Windows 10 以降を想定しています (Qt6 / C++17 必須)。

- OBS Studio以外でも動作しますか？
    - Streamlabs OBS など Qt6 ベースの OBS fork でも動作すると思われますが、未確認です。

- 他のプラグインと競合しませんか？

    - OBSのオーディオコールバックは複数登録可能なので、基本的に競合しませんが、競合するものがあれば連絡ください。

- CPU負荷はどのくらいですか？
    - macOS (M1) で 0.5% 以下、Windows (i7-9700K) で 1% 以下です。OBS全体の負荷に対しては微小です。

- **マスターエフェクト対応する予定ありますか？**
    - audio_output_connectのAPI仕様上、そこで加工しても元のパイプラインへ戻す仕組みがなく対応予定はありません。OBSの将来的なAPI拡張次第です。

## 謝辞
  
- OBS Project & contributors
- Qt Project
- ITU-R BS.1770 / EBU R128 specification references
---
## 免責
本ドキュメントおよびソフトウェアは現状有姿 (“AS IS”) で提供され、いかなる保証も行いません。
バイナリを広範に配布する場合などの法的適合性 (GPL / LGPL 等) については、専門の弁護士に相談してください。
本ソフトウェアのインストールまたは使用に起因して発生した損害、データ消失、動作不良その他一切の不利益について、責任を負いません。

---

  

---

# -English README-

  

# Master Level Meter (OBS Plugin)

  

A floating window plugin for OBS Studio that displays audio levels: RMS / Peak / Momentary LUFS for any Master track (Track1..Track6).

It also visualizes which audio tracks are currently selected in the streaming (Output) settings.

---
## Downloads
https://github.com/ShmKnd/MasterLevelMeter/releases/tag/2025-10-04_v1.0.2

## Installation

### macOS (.pkg Installer)
Double-click the downloaded `MasterLevelMeter.pkg` to launch the macOS standard installer. It will automatically install to:
```
~/Library/Application Support/obs-studio/plugins/MasterLevelMeter.plugin/
```

### macOS (Manual)
Place `MasterLevelMeter.plugin` in:
```
~/Library/Application Support/obs-studio/plugins/
```

### Windows
```
C:\Program Files\obs-studio\obs-plugins\64bit\MasterLevelMeter.dll
```
Restart OBS after installing.
(Other Qt6-based forks such as Streamlabs OBS may work, unverified.)

## for Mac User IMPORTANT note
Troubleshooting on macOS: 
"Plugin cannot be opened because Apple cannot check it for malicious software"

When you install MasterLevelMeter.plugin on macOS, you may see a warning such as:

“MasterLevelMeter.plugin” can’t be opened because Apple cannot check it for malicious software.

This happens because the plugin is not signed with the Apple Developer Program.
It is safe to use if you trust the source. Follow these steps to allow the plugin:

1.Install the plugin

2.Launch OBS
*The plugin will be blocked by macOS and you will see a warning dialog.
Close the dialog with OK.

3.Allow the plugin in System Settings
Open System Settings → Privacy & Security.

Scroll down until you see a message like:
“MasterLevelMeter.plugin was blocked because it is not from an identified developer.”

Click Allow Anyway.

4.Restart OBS
The next time you launch OBS, you will be asked if you really want to open the plugin.
Click Open. From now on, the plugin will load automatically.

---
## Usage
- OBS menu: Docks →  Master Level Meter
- Track buttons: choose track (Track1..Track6)
- “Streaming uses”: updated every second from Output → Streaming settings
- Shows "Track1" if Output Mode is “Simple”
---

---  

## Features
- Visualize Master Track1..Track6 signal levels
- Dual-channel (L/R) meters:
- RMS
- Peak
- Short LUFS (3000 ms ITU-R BS.1770 K-weighted)
- K-weighting filter:
- Two-stage high-pass (60 Hz) + high-shelf (+4 dB @ ~1.7 kHz)
- Emphasis ticks at -23 / -18 LUFS
- Dock compatible
- Detects and shows which tracks are used for streaming (updates every 1 s)
---
## Audio Flow
1. Obtain planar float audio frames via `audio_output_connect`
2. The selected mix buffer is pushed to `LevelCalc::process()` inside `audio_callback`
3. K-weighting & sub-block processing (100 ms hop / 3000 ms window)
4. A ~60 fps Qt timer calls `updateLevelsLR()` → triggers repaint

### Loudness (Short)
- Short: 3000 ms sliding window energy (summed channels) with -0.691 offset

---
## Qt 6 Usage
- Modules: Core / Gui / Widgets
- CMake: `find_package(Qt6 COMPONENTS Core Widgets REQUIRED)`
- AUTOMOC enabled
- Dynamic linking (LGPLv3 compliance). Static linking discouraged due to extra obligations
- Prefer official Qt SDK (or Homebrew `brew install qt` on macOS) over ad‑hoc stripped deps
---

## Build (Developer Notes)
CMake logic based on [obs-plugintemplate](https://github.com/obsproject/obs-plugintemplate)

OBS Studio sources and dependencies (including Qt6) are automatically downloaded on first build.

### macOS (Universal Binary: Intel + Apple Silicon)
```bash
# Configure with CMake preset (Xcode generator + Universal)
cmake --preset macos

# Release build
cmake --build build_macos --config Release

# Generate .pkg installer
cmake --install build_macos --config Release
```

To enable code signing, set environment variables before configuring:
```bash
export CODESIGN_IDENT="Developer ID Application: Your Name (TEAMID)"
export CODESIGN_TEAM="TEAMID"
cmake --preset macos
```

### Windows (PowerShell)
```powershell
cmake --preset windows-x64
cmake --build build_x64 --config Release
```
> `OBS::libobs` is automatically downloaded and built.
---


## Implementation Notes
- Audio callback: minimal K-weight + accumulations only
- 3000 ms window / 100 ms hop keeps cost low
- Lock-free sharing between audio and UI threads via atomic variables
---
## License
Plugin: GPL-2.0-or-later
OBS Studio (libobs) is GPLv2 (only).
Links dynamically to Qt6 (LGPLv3). If you bundle Qt libraries, include the LGPLv3 text and allow user relinking.
See `LICENSE` for full texts.

---
## Third-Party Summary
| Component | License | Notes |
|-----------|---------|-------|
| OBS (libobs) | GPLv2 | Core streaming / audio API |
| Qt 6 (Core/Gui/Widgets) | LGPLv3 | Dynamic linkage |
| (Transitive: FFmpeg, x264, etc.) | Mixed (LGPL/GPL) | Via OBS; not directly redistributed here |

---

## Security / Privacy
- No network access.
- Persists only window geometry & selected track via QSettings (local OS-dependent storage).

---
## Roadmap (Ideas)
- EBU R128 Short-Term (3 s)
- True Peak
- Skins / color themes
- Dock integration
- Enhanced recording (non-stream) master track visualization
  
---
## Q&A
- Required OBS?
    - OBS 29.0+ (Qt6 / C++17)
    - Tested on Windows OBS 29.1.3/31.1.2
    - Tested on macOS 14.6 & OBS 31.1.2 (Apple Silicon)
    - Tested on macOS 15.3 & OBS 31.1.2 (Intel) *Intel Build only

- macOS version?
    - 12.0+ (Monterey or later)
    - Universal Binary: Intel (x86_64) and Apple Silicon (arm64) supported.

- Windows version?
    - Windows 10+

- Other forks?
    - Likely works with Qt6-based forks; untested

- Conflicts with other plugins?
    - Multiple audio callbacks are allowed; report issues if found

- CPU usage?
    - Approx: macOS (M1) < 0.5%, Windows (i7-9700K) < 1%

- **Master effects planned?**
    - Current API (`audio_output_connect`) cannot feed processed data back; depends on future OBS API changes.
---

## Acknowledgements
- OBS Project & contributors
- Qt Project
- ITU-R BS.1770 / EBU R128 specifications
---
## Disclaimer
This documentation and software are provided “AS IS” without any warranty (express or implied).
Consult an attorney for compliance questions (GPL / LGPL) if distributing binaries broadly.
I assume no liability for any damage, data loss, malfunction, loss of profit, or other issues arising from installing or using this software. 

---
(End)
