# wormsim_rs

Rust製の高速シミュレーションエンジンによって、線虫（*Caenorhabditis elegans*）の塩濃度依存的な走性行動を再現します。Pythonバインディングを通して、Pythonから簡単に利用できます。

本シミュレータは以下の論文に基づいています：

> Hironaka, M., & Sumi, T. (2025). *A neural network model that generates salt concentration memory-dependent chemotaxis in Caenorhabditis elegans*. eLife, 14, RP104456. https://doi.org/10.7554/eLife.104456.1

---

## 🚀 特徴（Features）

- Rustによる高速なシミュレーション
- [PyO3](https://github.com/PyO3/pyo3) と [maturin](https://github.com/PyO3/maturin) によるPythonバインディング
- `Gene`および`Const`構造体による柔軟なパラメータ指定
- 濃度場の切り替え（線形・ガウス分布-1ピーク・ガウス分布-2ピーク）

---

## 📦 インストール方法（Installation）

### Python（PyPI）
```bash
pip install wormsim-rs
```

---

## 🧪 クイックスタート（Python）

```python
from wormsim_rs import klinotaxis

# パラメータを定義
gene = {
    "gene": [
        -0.8094022576319283,
        -0.6771492613425638,
        0.05892807075993428,
        -0.4894407617977082,
        0.1593721867510597,
        0.3576592038271041,
        -0.5664294232926526,
        -0.7853343958692636,
        0.6552003805912084,
        -0.6492992485125678,
        -0.5482223848375227,
        -0.956542705465967,
        -1.0,
        -0.7386107983898611,
        0.02074396537515929,
        0.7150315462816783,
        -0.9243504880454858,
        0.1353396882729762,
        0.9494528443702027,
        0.7727883271643218,
        -0.6046043758402895,
        0.7969062294208619,
    ]
}
const = {
    "alpha": -0.01,  # 線形濃度の勾配
    "c_0": 1.0,  # ガウス濃度の設定
    "lambda": 1.61,  # ガウス濃度の設定
    "x_peak": 4.5,  # 勾配のピークのx座標 /cm
    "y_peak": 0.0,  # 勾配のピークのy座標 /cm
    "dt": 0.01,  # 時間刻みの幅 /s
    "periodic_time": 4.2,  # 移動の1サイクルの継続時間 /s
    "frequency": 0.033,  # 方向の平均周波数 /Hz
    "mu_0": 0.0,  # 初期角度 /rad
    "velocity": 0.022,  # 線虫の速度 /cm/s
    "simulation_time": 300.0,  # シミュレーション時間 /s
    "time_constant": 0.1,  # 時定数 /s
}


# シミュレーション実行(mode=1: ガウス分布-1ピークの濃度場)
x, y = klinotaxis(gene, const, mode=1)

# 結果を可視化（matplotlib使用）
import matplotlib.pyplot as plt

plt.plot(x, y, label="Trajectory")
plt.scatter(0, 0, label="Starting Point")
plt.scatter(const["x_peak"], const["y_peak"], label="Gradient Peak")
plt.axis("equal")
plt.legend()
plt.show()
```

---

## 🧬 API仕様

### `klinotaxis(gene: Gene, constant: Const, mode: int) -> tuple[list[float], list[float]]`

線虫1個体の移動軌跡をシミュレーションします。

- `gene`: ニューロンモデルの重みなどを格納する `Gene` 構造体
- `constant`: シミュレーションに関する定数を格納する `Const` 構造体
- `mode`: 濃度マップのモードを指定（下記参照）

戻り値は `(x: list[float], y: list[float])` のタプルで、移動経路を表します。

### `Gene`
神経モデルに関連するパラメータ（ニューロン間の接続強度、閾値など）をまとめた構造体。

#### フィールド:
- `gene`: `Vec<f64>` 型で、ニューロンのシナプス重みなどを格納する22次元のベクタ。範囲は-1.0〜1.0。

##### Pythonでの例:
```python
gene = {
    "gene": [
        -0.8094022576319283,
        -0.6771492613425638,
        0.05892807075993428,
        -0.4894407617977082,
        0.1593721867510597,
        0.3576592038271041,
        -0.5664294232926526,
        -0.7853343958692636,
        0.6552003805912084,
        -0.6492992485125678,
        -0.5482223848375227,
        -0.956542705465967,
        -1.0,
        -0.7386107983898611,
        0.02074396537515929,
        0.7150315462816783,
        -0.9243504880454858,
        0.1353396882729762,
        0.9494528443702027,
        0.7727883271643218,
        -0.6046043758402895,
        0.7969062294208619,
    ]
}
```

### `Const`
シミュレーションにおける環境定数（時間刻み、速度、初期角度など）をまとめた構造体。

#### フィールド:
- **`alpha`**: `f64` 型で、線形濃度の勾配を指定します。
- **`c_0`**: `f64` 型で、ガウス濃度の設定値を指定します。
- **`lambda`**: `f64` 型で、ガウス濃度の設定値を指定します。
- **`x_peak`**: `f64` 型で、勾配のピークの x 座標を指定します（単位: cm）。
- **`y_peak`**: `f64` 型で、勾配のピークの y 座標を指定します（単位: cm）。
- **`dt`**: `f64` 型で、シミュレーションの時間刻みの幅を指定します（単位: s）。
- **`periodic_time`**: `f64` 型で、移動の1サイクルの継続時間を指定します（単位: s）。
- **`frequency`**: `f64` 型で、方向の平均周波数を指定します（単位: Hz）。
- **`mu_0`**: `f64` 型で、初期角度を指定します（単位: rad）。
- **`velocity`**: `f64` 型で、線虫の速度を指定します（単位: cm/s）。
- **`simulation_time`**: `f64` 型で、シミュレーションの総時間を指定します（単位: s）。
- **`time_constant`**: `f64` 型で、シミュレーションの時間定数を指定します（単位: s）。

##### Pythonでの例:
```python
const = {
    "alpha": -0.01,  # 線形濃度の勾配
    "c_0": 1.0,  # ガウス濃度の設定
    "lambda": 1.61,  # ガウス濃度の設定
    "x_peak": 4.5,  # 勾配のピークのx座標 /cm
    "y_peak": 0.0,  # 勾配のピークのy座標 /cm
    "dt": 0.01,  # 時間刻みの幅 /s
    "periodic_time": 4.2,  # 移動の1サイクルの継続時間 /s
    "frequency": 0.033,  # 方向の平均周波数 /Hz
    "mu_0": 0.0,  # 初期角度 /rad
    "velocity": 0.022,  # 線虫の速度 /cm/s
    "simulation_time": 300.0,  # シミュレーション時間 /s
    "time_constant": 0.1,  # 時定数 /s
}
```

---

## 🧭 濃度マップモード（Concentration Modes）

- `mode = 0`: 線形勾配（遺伝的アルゴリズムでの学習用）
- `mode = 1`: ガウス分布-1ピーク（基本形）
- `mode = 2`: ガウス分布-2ピーク

---

## 📚 参考文献（References）

Hironaka, M., & Sumi, T. (2025). *A neural network model that generates salt concentration memory-dependent chemotaxis in Caenorhabditis elegans*. eLife, 14, RP104456. https://doi.org/10.7554/eLife.104456.1

---
