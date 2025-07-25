# MFCC34.py 詳細解説

## 概要
MFCC34.pyは、MFCC（メル周波数ケプストラム係数）の第3・第4係数（MFCC3, MFCC4）のみを使用した日本語5母音のリアルタイム認識システムです。フォルマント分析を組み合わせることで、特に「い」と「え」の判別精度を向上させています。

## 主要な特徴
- **MFCC3, MFCC4のみ使用**: 13次元MFCCのうち、母音識別に最も有効な2つの係数に絞り込み
- **フォルマント分析統合**: LPC分析によるフォルマント抽出で判別精度向上
- **2種類の可視化**: MFCC空間の散布図と伝統的な母音図（舌位置）
- **リアルタイム認識**: 1秒ごとに音声を録音・分析・表示

## 技術的詳細

### 1. 音声処理設定（20-32行目）
```python
RATE = 16000          # サンプリングレート 16kHz
DURATION = 1.0        # 録音時間 1秒
N_MFCC = 13          # MFCC次元数（ただし3,4のみ使用）
RECORDINGS_DIR = "recordings_formant"  # テンプレート保存先
```

### 2. MFCC3,4を選択した理由
- **MFCC3（第4係数）**: 中域スペクトル構造を表現（約1000-2000Hz）
- **MFCC4（第5係数）**: 中域スペクトル構造を表現（約1500-3000Hz）
- これらの周波数帯域は第1・第2フォルマント（F1, F2）と高い相関を持つ

### 3. フォルマント抽出（76-181行目）
```python
def extract_formants(y, sr, n_formants=3):
```
- **LPC（線形予測符号化）分析**を使用
- プリエンファシスで高周波成分を強調
- 25msフレーム、10msシフトで分析
- Levinson-Durbin再帰でLPC係数計算
- スペクトル包絡からピーク（フォルマント）検出
- 200-5000Hz範囲で有効なフォルマントを抽出

### 4. 特徴抽出プロセス（206-239行目）
```python
def extract_features():
```
各音声ファイルから：
1. **MFCC抽出**: librosaで13次元MFCC計算後、3,4のみ使用
2. **フォルマント抽出**: LPC分析で3つのフォルマント周波数取得
3. **テンプレート作成**: 各母音の平均値を計算

### 5. 分類アルゴリズム（297-445行目）
```python
def classify(user_mfcc, user_formants, templates, formant_templates, all_formants=None):
```

#### 距離計算の仕組み：
1. **MFCC距離**: ユークリッド距離で計算
2. **フォルマント距離**: 
   - F1, F2の差を最大周波数（4000Hz）で正規化
   - 「い」「え」はF2を70%重視（F1:30%, F2:70%）
   - その他の母音はF1,F2を均等重視（各50%）

3. **合成距離**:
   - 「い」「え」: MFCC 60% + フォルマント 40%
   - その他: MFCC 80% + フォルマント 20%

4. **特別な処理**:
   - **IE_BIAS**: 「い」を「え」より優先する度合い（デフォルト0.8）
   - **個別サンプル類似度**: 録音済みサンプルとの直接比較
   - **VERY_SIMILAR_THRESHOLD**: 0.85以上の類似度で特別扱い

### 6. 可視化機能

#### 6.1 MFCC3-4散布図（453-494行目）
```python
def init_3d_plot(X, y, pca, templates):
```
- X軸: MFCC4（第5係数）
- Y軸: MFCC3（第4係数）
- 各母音をカラーコードで表示
- 曖昧母音（ə）も理論値として表示

#### 6.2 伝統的な母音図（619-677行目）
```python
def create_vowel_chart():
```
- 舌の位置による母音配置
- X軸: 前舌←→後舌
- Y軸: 高←→低
- フォルマントから舌位置を推定して表示

#### 6.3 フォルマントチャート（589-617行目）
```python
def create_formant_chart(formant_templates):
```
- F1-F2平面での母音分布
- 音響音声学の慣例に従い軸を反転

### 7. リアルタイム処理フロー（722-816行目）

1. **録音** (1秒間)
2. **特徴抽出**:
   - MFCC3, MFCC4を計算
   - フォルマント（F1, F2, F3）を抽出
3. **分類**:
   - テンプレートとの距離計算
   - 「い」「え」の特別判定
   - 個別サンプルとの類似度チェック
4. **結果表示**:
   - 推定母音とスコア
   - フォルマント値
   - 類似度ランキング
   - 発音アドバイス
5. **可視化更新**:
   - MFCC散布図に赤い星マーク
   - 母音図に現在位置表示

### 8. 「い」と「え」の判別強化

これらの母音は音響的に近いため、特別な処理を実装：

1. **フォルマント重視**: 
   - 第2フォルマント（F2）の差を重点的に評価
   - 「い」は通常F2が2200Hz以上
   - 「え」は通常F2が1800-2200Hz

2. **バイアス調整**:
   - IE_BIAS=0.8で「い」を優先
   - 距離計算時に「い」の距離を20%短く調整

3. **個別サンプル比較**:
   - 録音済みサンプルとの直接比較で高精度判定

## パラメータ調整ガイド

### 判別精度の調整
- `IE_DISTINGUISH`: True/Falseで「い」「え」特別処理の有効/無効
- `IE_BIAS`: 0.5（中立）〜1.0（「い」最優先）
- `VERY_SIMILAR_THRESHOLD`: 個別サンプル類似度の閾値（0.7〜0.9推奨）

### 重み付けの調整
- MFCC/フォルマントの重み比率（376-389行目）
- フォルマント内のF1/F2重み比率（316-319行目）

## 出力ファイル
- `mfcc34_scatter.png`: MFCC3-4の分布図
- `formant_chart.png`: フォルマントチャート
- `user_input.wav`: 最新の録音データ（上書き）

## 必要な準備
1. `recordings_formant/`ディレクトリに各母音3サンプルずつ配置
   - 例: `a_1.wav`, `a_2.wav`, `a_3.wav`
2. 必要なライブラリのインストール（numpy, librosa, matplotlib等）

## 実行方法
```bash
python MFCC34.py
```

Ctrl+Cで終了