---
marp: true
theme: academic
paginate: true
math: katex
style: |
  .cjk_fallback {
    font-family: 'Noto Sans JP';
    font-size: 0.8em
  }
---
<!-- _header:  -->

## Self-supervised network distillation an effective approach to exploration in sparse reward environments

---
<!-- _header: Abstract -->

<div style="font-size: 0.6em">

## 概要
- 強化学習は意思決定問題を解くために用いられるが、報酬が希薄な環境では探索が困難となる。
- **自己教師型ネットワーク蒸留（SND）** は、予測モデルとターゲットモデル間の蒸留誤差を新規性指標として活用する内発的動機付けアルゴリズムである。
- 実験により、SNDは10種類の難解な環境において、学習速度を向上させ、従来モデルよりも高い外部報酬を達成。

## 手法
### ランダムネットワーク蒸留（RND）
- ランダムなターゲットモデルとその予測を試みる予測モデルを使用。
- 課題:
  1. 適切な初期化が必要。
  2. 内発的報酬の変動が小さい。
  3. モデル適応による動機付け信号の消失。

### 自己教師型ネットワーク蒸留（SND）
- 自己教師型学習アルゴリズムを用いてターゲットモデルを更新し、蒸留誤差を内発的報酬として利用。
- ターゲットモデルの特徴量空間を拡大し、新規状態の区別能力を向上。

</div>

---
<!-- _header: RNDの問題点はなにか -->

<div style="font-size: 0.6em">
RNDは、エージェントが未知の状態を探索するための「新規性検出」手法

**ターゲットモデル**: ランダムに初期化されたモデルで、状態を入力として特徴量を生成します。
**予測モデル**: ターゲットモデルの出力を模倣するように学習します。
2つのモデルの出力の差（予測誤差）が、エージェントの内発的動機となる

#### RNDの問題点
**ターゲットモデルの初期化依存**
ランダムに初期化されたネットワークがその出力（特徴量）が状態空間の構造を適切に反映しない可能性がある
状態空間内の似た状態を似ているように、異なる状態を異なるように表現することができないと、新規性の検出ができない

**内発的報酬のばらつきが小さい**
ランダムであるがゆえに、多くの状態が同じ程度の報酬を生む。これにより興味深い状態への誘導がなされない

</div>

---
<!-- _header: ネットワーク蒸留 -->

<div class="flex" >
<div style="">

<img src=images/image.png height=500 >

</div>
<div style="font-size: 0.7em">

本論文の手法（RND、SND）は
蒸留誤差（self supervised loss）を新規性検出に
活用することが主眼

ターゲットモデルは、自己教師あり学習にて学習する

ターゲットモデルの出力に合致するよう、
予測モデルを学習する（蒸留）

本論文では３パターン


</div>
</div>

---
<!-- _header: Contrastive Learning（コントラスト学習, SND-V） -->

<div style="font-size: 0.6em">

### **コントラスト学習の基本的な考え方**

- **似ているデータ（Positive Pair）**:
  - 同じ状態の異なるバリエーション（例：画像の一部をズーム、回転、明るさ変更など）
- **異なるデータ（Negative Pair）**:
  - 異なる状態を表すデータ（異なるタイミングや場所での状態）
  
### **本論文でのContrastive Learningの動作**
#### 1. **データペアの生成**
- 各状態 $(s_t)$ に対して、以下のようなデータペアを生成します：
  - **Positive Pair**: $(s_t)$ とそのデータ拡張（Data Augmentation）版（例：画像を回転、ズームなど）。
<img src=images/image-1.png height=200>
  - **Negative Pair**: $(s_t)$ と別の状態 $(s_t')$（例：別の時間ステップの状態）。

</div>

---
<!-- _header: Contrastive Learning（コントラスト学習, SND-V） -->

<div style="font-size: 0.6em">

#### 2. **損失関数（Contrastive Loss）の最適化**
- Contrastive Loss を使い、以下のような学習目標を設定します：
  1. Positive Pairの特徴量間の距離を縮める。
  2. Negative Pairの特徴量間の距離を広げる。
  
- 一般的なコントラスト学習の損失関数は次のように定義されます：
  $$
  L = -\log \frac{\exp(\text{sim}(z_i, z_j) / \tau)}{\sum_{k=1}^{N} \exp(\text{sim}(z_i, z_k) / \tau)}
  $$
  - $z_i, z_j$: Positive Pairの特徴量。
  - $z_k$: Negative Pairを含む特徴量。
  - $\text{sim}(\cdot)$: 特徴量間の類似度（例：コサイン類似度）。
  - $\tau$: 温度パラメータ（分布をスムーズにするためのスケール）。
  
## **Contrastive Learningのメリット**
  
  1. 似た状態を近づけ、異なる状態を遠ざける。
  2. 特徴空間を拡張して、新規性検出能力を向上。
  3. 蒸留誤差を大きく保つことで、内発的報酬のばらつきを増やし、探索行動を促進。

> 異なる状態の生成方法によって結構違いそうな気がしてきた。
> オーグメンテーションのやり方の工夫。例えば、時間が近いのを正例にするとか

---
<!-- _header: Spatio-Temporal DeepInfoMax  SND-STD -->

<div style="font-size: 0.6em">


### **Spatio-Temporal DeepInfoMaxの基本的な考え方**
DeepInfoMax（DIM）は、入力データの高次元表現間で、情報量を最大化することを目指す手法です。
これを拡張した**Spatio-Temporal（時空間）版**では、空間的および時間的な特徴を考慮します。
1. **空間的特徴**（Spatio-）:
   - 同じ状態の内部（例：画像内のピクセルやパッチ間）の特徴間での情報量を最大化。
2. **時間的特徴**（-Temporal）:
   - 時間的に隣接する状態（例：連続するフレームや動きの変化）の情報量を最大化。

## **動作の流れ**
1. **状態間の特徴量を計算**:
   - 現在の状態 $s_t$ のグローバルおよびローカル特徴量を計算。
   - 次の状態 $s_{t+1}$ と比較。
2. **関連性の強化**:
   - $s_t$ と $s_{t+1}$（連続する状態）を近づけ、無関係な状態から遠ざける。
3. **特徴空間の安定化**:
   - 正則化項により、特徴量の分散を広げつつ、爆発的な成長を防ぐ。


---
<!-- _header: Spatio-Temporal DeepInfoMax  SND-STD -->

<div style="font-size: 0.6em">


### **1. 損失関数の構成**
SND-STDでは、特徴空間で以下の2種類の関連性を最大化します：
- **Global-Local Objective（GL）**: グローバルな特徴とローカルな特徴の関連性。
- **Local-Local Objective（LL）**: 畳み込み層のローカルな特徴同士の関連性。

これらを組み合わせた損失関数は次のように定義されています：
$$
L = \frac{1}{M N} (L_{\text{GL}} + L_{\text{LL}})
$$
ここで、$M$ と $N$ はミニバッチのサイズと入力数です。

---
<!-- _header: Spatio-Temporal DeepInfoMax  SND-STD -->

<div style="font-size: 0.6em">

### **2. Global-Local Objective（$L_{\text{GL}}$）**
この項は、グローバルな特徴量（入力全体を表す特徴）と、局所的な特徴量（畳み込み層の出力の一部）間の相互情報量を最大化します。

$$
L_{\text{GL}} = -\sum_{h=1}^{H} \sum_{w=1}^{W} \log \frac{\exp(g_{h,w}(s_t, s_{t+1}))}{\sum_{s_t^* \in S_{\text{next}}} \exp(g_{h,w}(s_t, s_t^*))}
$$

#### **記号の意味**:
- $g_{h,w}(s_t, s_{t+1})$: $s_t$ のグローバル特徴量と $s_{t+1}$ のローカル特徴量間のスコア。
  - 計算式は次のように定義：
    $$
    g_{h,w}(s_t, s_{t+1}) = \Phi^\top(s_t) W_g \Phi(l,h,w)(s_{t+1})
    $$
  - $\Phi(s_t)$: $s_t$ のグローバル特徴ベクトル。
  - $\Phi(l,h,w)(s_{t+1})$: $s_{t+1}$ のローカル特徴ベクトル。
  - $W_g$: 学習可能な変換行列。
- $S_{\text{next}}$: ネガティブサンプル（非連続な状態の集合）。

この項は、連続する状態 $s_t$ と $s_{t+1}$ を近づけ（Positive Sample）、無関係な状態 $s_t^*$ から遠ざけます（Negative Sample）。


---
<!-- _header: Spatio-Temporal DeepInfoMax  SND-STD -->

<div style="font-size: 0.6em">

$$
L_{\text{LL}} = -\sum_{h=1}^{H} \sum_{w=1}^{W} \log \frac{\exp(f_{h,w}(s_t, s_{t+1}))}{\sum_{s_t^* \in S_{\text{next}}} \exp(f_{h,w}(s_t, s_t^*))}
$$

#### **記号の意味**:
- $f_{h,w}(s_t, s_{t+1})$: 畳み込み層内の局所特徴量同士のスコア。
  - 計算式は次のように定義：
    $$
    f_{h,w}(s_t, s_{t+1}) = \Phi^\top(l,h,w)(s_t) W_l \Phi(l,h,w)(s_{t+1})
    $$
  - $\Phi(l,h,w)(s_t)$: $s_t$ のローカル特徴ベクトル。
  - $W_l$: 学習可能な変換行列。

#### **動作**:
- この項は、畳み込み層の局所的な特徴量同士で、連続する状態の関連性を強化します。

---
<!-- _header: Spatio-Temporal DeepInfoMax  SND-STD -->

<div style="font-size: 0.6em">

#### **4. 正則化項**
ターゲットモデルの特徴空間が「爆発的に広がる問題」を防ぐため、以下の正則化が追加されています。

##### **L2正則化**:
$$
L_2 = p_{\text{GL}} + p_{\text{LL}} = \sum_{h=1}^{H} \sum_{w=1}^{W} \left( \|f_{h,w}\| + \|g_{h,w}\| \right)
$$

##### **分散正則化**:
$$
L_\sigma = -\sigma(\Phi^\top(s_t))
$$
- 特徴量ベクトルの分散を最大化し、特徴空間のすべての次元が均等に使われるようにします。
- 分散が高くなるほど、情報が広く分布し、新規性検出能力が向上します。

##### **5. 全体の損失関数**
$$
L_T = \frac{1}{H W} \left( L_{\text{GL}} + L_{\text{LL}} + \beta_1 L_2 + \beta_2 L_\sigma \right)
$$
- $\beta_1$ と $\beta_2$: 正則化項の重み。

---
<!-- _header: SND-VIC（Variance-Invariance-Covariance Regularization）-->

<div style="font-size: 0.6em">

ターゲットモデルの特徴空間を適切に広げつつ、情報の重複や崩壊を防ぐために設計された正則化手法です。この手法は、特徴ベクトルの分散、類似性、不必要な相関を管理することで、効果的な新規性検出を可能にします。

### **SND-VICの目的**
1. **分散（Variance Term）**:
   - 特徴ベクトルが均等に広がり、情報が次元全体にわたって分散するように調整。
2. **不変性（Invariant Term）**:
   - 同じ状態（または連続する状態）の特徴ベクトルが似た特徴を持つように制約。
3. **共分散（Covariance Term）**:
   - 特徴ベクトル間の不要な相関を抑え、各次元がユニークな情報を学習できるようにする。

---
<!-- _header: SND-VIC（Variance-Invariance-Covariance Regularization）-->

<div style="font-size: 0.6em">


### **損失関数の構成**
SND-VICでは、3つの正則化項とそれらを組み合わせた最終的な損失関数を使用します。

### **1. 分散正則化（Variance Term）**
分散を管理するための正則化項は以下のように定義されています：
$$
L_v(Z) = \frac{1}{D} \sum_{d=1}^{D} \max(0; \tau - \sigma(Z_d))
$$
#### **記号の意味**:
- $Z_d$: バッチ内の特徴ベクトルの $d$-次元。
- $\sigma(Z_d)$: $Z_d$ の標準偏差。
- $\tau = 1$: 目標となる標準偏差。

#### **目的**:
- 各次元の標準偏差を $\tau$ に近づけ、次元全体に情報が均等に分布するようにします。
- **効果**: 特徴空間の「崩壊」を防ぎ、多様性を維持。


---
<!-- _header: SND-VIC（Variance-Invariance-Covariance Regularization）-->

<div style="font-size: 0.6em">


### **2. 共分散正則化（Covariance Term）**
共分散を制御するための正則化項は次のように定義されています：
$$
L_c(Z) = \frac{1}{(N-1)D} \sum_{i \neq j} [C(Z)]_{i,j}^2
$$
#### **記号の意味**:
- $C(Z)$: バッチ $Z$ の特徴ベクトルの共分散行列。
- $[C(Z)]_{i,j}$: 共分散行列の $i,j$ 要素。

#### **目的**:
- 共分散行列のオフダイアゴナル成分（二つの特徴ベクトル間の相関）を最小化。
- **効果**: 特徴ベクトルの次元が互いに独立した情報を持つようにする。

---
<!-- _header: SND-VIC（Variance-Invariance-Covariance Regularization）-->

<div style="font-size: 0.6em">


### **3. 不変性正則化（Invariant Term）**
不変性を維持するための正則化項は以下のように定義されています：
$$
L_s(Z, Z') = \frac{1}{N} \sum_{b=1}^{N} \|Z_n - Z'_n\|^2
$$
#### **記号の意味**:
- $Z, Z'$: 2つの状態（または連続状態）から得られる特徴ベクトルのバッチ。
- $\|Z_n - Z'_n\|$: 特徴ベクトル間のユークリッド距離。

#### **目的**:
- 同じ状態（または連続状態）の特徴ベクトルが似た特徴を持つようにする。
- **効果**: ターゲットモデルが入力状態の構造を一貫して捉えられるようになる。

---
<!-- _header: SND-VIC（Variance-Invariance-Covariance Regularization）-->

<div style="font-size: 0.6em">


### **4. 全体の損失関数**
最終的な損失関数は、上記の正則化項を組み合わせて次のように定義されています：
$$
L_T = \lambda L_s(Z, Z') + \mu [L_v(Z) + L_v(Z')] + \nu [L_c(Z) + L_c(Z')]
$$
#### **記号の意味**:
- $\lambda, \mu, \nu$: 各正則化項の重み。
  - 実験的に $\lambda = 1$, $\mu = 1$, $\nu = 1/25$ に設定。

---
<!-- _header: SND-VIC（Variance-Invariance-Covariance Regularization）-->

<div style="font-size: 0.6em">


## **動作の流れ**
1. **状態間の特徴ベクトルを計算**:
   - 現在の状態 $s_t$ と次の状態 $s_{t+1}$ から特徴ベクトル $Z, Z'$ を抽出。
2. **分散を管理**:
   - 各次元の標準偏差を目標値に近づけ、情報が均等に分布するようにする。
3. **共分散を制御**:
   - 特徴ベクトル間の不要な相関を抑え、各次元が独立した情報を持つようにする。
4. **不変性を維持**:
   - 同じ状態（または連続状態）間で特徴ベクトルが似るようにする。

---
<!-- _header: SND-VIC（Variance-Invariance-Covariance Regularization）-->

<div style="font-size: 0.6em">


## **SND-VICの効果**
1. **特徴空間の多様性の確保**:
   - 分散を制御することで、特徴ベクトルが次元全体にわたって広がり、情報の欠損や崩壊を防ぎます。
2. **不要な相関の除去**:
   - 共分散を最小化することで、各次元がユニークな情報を持つようにします。
3. **状態間の類似性を管理**:
   - 不変性を維持することで、入力データの構造を正確に捉えます。
