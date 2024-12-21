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

## Discovering and Achieving Goals via World Models

[NeurIPS 2021](https://proceedings.neurips.cc/paper/2021/hash/cc4af25fa9d2d5c953496579b75f6f6c-Abstract.html)

---
<!-- _header: Abstract  -->

<div style="font-size: 0.6em">

### 研究背景
- 複雑な視覚環境で多様なタスクを解くエージェントを監督なしで学習させることは困難。
- 従来の強化学習手法では、新しいタスクごとに多大な人的労力が必要。
- 未探索の状態を効率的に発見し、それを目標として活用する戦略の必要性が認識されている。

### 研究目的
- 未探索状態の効率的な発見と、その状態を目標とした行動の学習を統合的に実現する。
- 無監督での目標条件付き強化学習を改善し、複雑なタスク達成を可能にする。

</div>

---
<!-- _header: Abstract  -->

<div style="font-size: 0.6em">

### 提案手法
#### **Latent Explorer Achiever (LEXA)**
- **環境モデルの学習**:
  - 視覚的入力から環境モデルを学習し、探査ポリシー（Explorer）と達成ポリシー（Achiever）を構築。
- **探査ポリシー**:
  - 「未探索の興味深い状態」をモデルの想像内で計画的に発見し、その情報ゲインを最大化。
- **達成ポリシー**:
  - 発見された状態を目標として学習し、ゴール画像を基に行動を計画。
- **ゼロショット適応**:
  - 学習後、新しいタスク（目標画像）に追加の訓練なしで対応可能。

### 結果
- **効率的な探索**:
  - 従来手法と比較して目標探索の効率性が大幅に向上。
- **タスク達成性能**:
  - ロボット操作や物体配置など、複雑なタスク環境での成功率が向上。
- **新規ベンチマークでの成功**:
  - 複数の物体操作や連続的なタスク達成を実証し、他の手法を上回る性能を確認。


</div>