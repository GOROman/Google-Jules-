# JUCE VSTプラグイン開発 要件定義：ADSRエンベロープ対応シンセ

## 🎯 プラグイン仕様

| 項目           | 内容                                       |
|----------------|--------------------------------------------|
| プラグイン形式 | VST3（AudioProcessor + Editor構成）        |
| 種類           | ソフトシンセ（ADSR制御付き）               |
| 入力           | MIDI（Note On / Off）                      |
| 出力           | オーディオ（モノラル or ステレオ）         |
| GUI            | Attack / Decay / Sustain / Release スライダー（4本） |
| 音源           | サイン波（`juce::dsp::Oscillator`使用）    |
| エンベロープ   | `juce::ADSR` / `ADSR::Parameters`           |
| パラメータ管理 | `AudioProcessorValueTreeState`（GUI連携用） |

---

## 🎛 パラメータ設計

| パラメータID | 表示名 | 範囲           | 初期値 | 単位 |
|--------------|--------|----------------|--------|------|
| `attack`     | Attack | 0.01 ～ 5.0 秒  | 0.1    | 秒   |
| `decay`      | Decay  | 0.01 ～ 5.0 秒  | 0.1    | 秒   |
| `sustain`    | Sustain| 0.0 ～ 1.0      | 0.8    | 係数 |
| `release`    | Release| 0.01 ～ 5.0 秒  | 0.2    | 秒   |

- すべて `AudioProcessorValueTreeState` で定義
- GUIと `SliderAttachment` で双方向バインド

---

## 🔊 音声処理フロー（`processBlock()`）

1. MIDIイベントからNote On/Off検出
2. `ADSR.noteOn()` or `noteOff()` を呼び出し
3. MIDIノートに応じた周波数でサイン波生成
4. 出力バッファにADSRエンベロープを適用（`applyEnvelopeToBuffer()`）

---

## 🧠 使用クラス構成

- `juce::ADSR`：エンベロープ処理
- `juce::ADSR::Parameters`：A, D, S, R構造体
- `juce::dsp::Oscillator<float>`：波形生成（初期はサイン波）
- `juce::AudioProcessorValueTreeState`：パラメータ一元管理
- `juce::SliderAttachment`：GUIと音声処理の接続

---

## 💻 GUI構成（`PluginEditor.cpp`）

- `Slider` × 4（A, D, S, R）
- `Label` 付き（スライダー名）
- 縦並びレイアウト or グリッドレイアウト
- 色指定：黒背景 + 明るいアクセント（緑・ピンクなど）

---

## 🔐 リアルタイムセーフ設計

- `processBlock()`内でnew/delete禁止
- `ADSR.noteOn()`などはlock-freeで使用可能
- DSP処理はシンプルなfloat演算のみ

---

## 🚀 拡張の余地（今後のステップ）

- 複数同時発音（Polyphonic対応）
- 他の波形追加（矩形波・ノコギリ波）
- 可視化UI（エンベロープカーブのリアルタイム表示）
- モジュレーション（LFO、Filterなど）

---

## ✅ 開発スタック例

```txt
JUCE 7.x
CMake構成（Projucer非依存）
CLion / VSCode
GitHub Actions（CI/CD用）
Reaper / Bitwig（テストDAW）
Pluginval（プラグイン検証ツール）
```
