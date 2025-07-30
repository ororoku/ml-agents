# 目的地観測のノイズ追加について

## 概要

Unity ML-AgentsのWalker環境を用いて、エージェントが目的地に到達するタスクを行いました。本プロジェクトでは、「現実的なセンサー誤差に対する頑健性向上」を目的に、**目的地の観測値にノイズを加える工夫**を行いました。

## 観測ノイズと行動

現実のセンサーは必ずしも状態 $s_t$ を完全には観測できず，観測 $o_t$ は真の状態にノイズを重ねた確率モデルで表されます。これを部分観測マルコフ決定過程（POMDP）として定式化すると，

$$
\begin{aligned}
&\text{状態遷移: } s_{t+1}\sim P(s_{t+1}\mid s_t, a_t),\\
&\text{観測: } o_t\sim O(o_t\mid s_t)\,,
\end{aligned}
$$

となる。ここで $O$ は観測モデルであり，例えば一様分布ノイズ $o_t = s_t + \eta_t, \quad \eta_t \sim \mathcal{U}([-a, a])$ として定義される[^1]。

## 実装した工夫

- **ノイズの付加方法**
`WalkerAgent.cs` で `CollectObservations()` 内、目的地情報を計算・観測する直前に、下記のようなホワイトノイズを追加しました。

```csharp
// 目的地座標の観測結果にノイズを加える
m_NoisyTargetPosition = target.transform.position + new Vector3(
    Random.Range(-1, 1),
    0,
    Random.Range(-1, 1)
);
```
変更点としては
- ノイズ付きターゲット位置の保存: m_NoisyTargetPositionフィールドを追加
- エピソード開始時にノイズ生成: OnEpisodeBegin()でエピソードごとにノイズ付きターゲット位置を生成
- 観測: CollectObservations()でノイズ付き位置を観測して保存
- 行動: UpdateOrientationObjects()でノイズ付きターゲット位置を行動に反映



## 結果・考察

- ノイズ量が大きすぎるとタスク達成が困難になるため、現実の用途やタスク難易度を考慮して調整が重要と感じた。
- ホワイトノイズやガウスノイズなど、ノイズの種類を変えても試してみたい。

