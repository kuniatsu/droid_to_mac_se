# Android External Soundboard for Mac/Zoom

Android端末を物理的な「音声ボタン（サウンダー）」として機能させ、USB接続されたMacを通じてZoom等のWeb会議システムに特定の音声をデジタル入力するためのプロジェクトです。

## 1. プロジェクトの概要
リモート会議中、特定の定型音声（例：体育祭の笛、拍手、定型メッセージ等）を、マイクを通さずデジタル信号としてクリアに相手に届ける仕組みを構築します。

### 主なユースケース
- 会議の進行合図（ホイッスル音）
- クイックなリアクション
- Text-to-Speech (TTS) を利用した音声代行

---

## 2. システム構成
Android側で再生されたデジタルオーディオデータを、USB経由でMacの「入力デバイス（マイク）」としてルーティングします。



1.  **Source:** Android App (MediaPlayer / SoundPool)
2.  **Transport:** USB Cable (UAC: USB Audio Class Protocol)
3.  **Routing (Mac):** Virtual Audio Driver (BlackHole / AudioRelay)
4.  **Destination:** Zoom / Microsoft Teams / Google Meet (Input Source)

---

## 3. 技術仕様 (Requirements)

### ハードウェア
- **Android Device:** Android 8.0以上 (Pixel 7 Pro 等)
- **Host Machine:** MacBook Air 2020 (macOS Sonoma以上推奨)
- **Connection:** USB-C to USB-C ケーブル (データ転送対応)

### ソフトウェア・ドライバ
- **Mac側仮想ドライバ:** [BlackHole 2ch](https://github.com/ExistentialAudio/BlackHole)
- **転送ブリッジ:** [AudioRelay](https://audiorelay.net/) (USB経由の音声転送を簡略化するため推奨)
- **開発環境:** Android Studio (Kotlin 1.9+)

---

## 4. 実装コード例 (Android)

`res/raw/whistle.mp3` に音声ファイルを配置し、ボタン押下で再生する基本実装です。

```kotlin
// MainActivity.kt
class MainActivity : AppCompatActivity() {
    private var mediaPlayer: MediaPlayer? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val btnWhistle = findViewById<Button>(R.id.btn_whistle)
        
        btnWhistle.setOnClickListener {
            // 音声の重複再生を避けるためリセット処理
            mediaPlayer?.stop()
            mediaPlayer?.release()
            
            mediaPlayer = MediaPlayer.create(this, R.raw.whistle)
            mediaPlayer?.start()
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        mediaPlayer?.release()
    }
}
```

---

## 5. セットアップ手順 (Setup Guide)

1.  **Mac側の準備:**
    - `BlackHole 2ch` をインストールします。
    - `AudioRelay` をMacとAndroidの両方にインストールします。
2.  **接続設定:**
    - AndroidとMacをUSBで接続します。
    - Androidの「USB設定」を「MIDI」または「データ転送なし（オーディオソース）」に設定（※機種・OSバージョンにより挙動が異なります）。
3.  **音声のルーティング:**
    - Android側 `AudioRelay`: `Server` モードを起動。
    - Mac側 `AudioRelay`: Androidを認識し、出力先に `BlackHole 2ch` を指定。
4.  **Zoomの設定:**
    - Zoomの `設定 > オーディオ > マイク` にて `BlackHole 2ch` を選択。

---

## 6. 今後の展望 (Roadmap)
- [ ] 独自TTS（Text-to-Speech）機能の実装（自分の声をクローン）
- [ ] 複数ボタンへの音源割り当て機能
- [ ] ネットワーク経由（Wi-Fi）でのワイヤレス音声送信対応
- [ ] 物理的な音量調整スライダーのUI実装
