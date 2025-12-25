# aimsgb-matplotlib_python_notebook

Language: [English (英語)](README.md)

`aimsgb` ライブラリを使用して結晶粒界（Grain Boundary, GB）モデルを構築・解析するためのガイドです。

## 準備 (Preparation)

まず、`aimsgb` ライブラリをインストールします。

```bash
pip install aimsgb
```

## 基本的な使い方

### GrainBoundary クラス

構造を入力し、指定した回転軸、シグマ値、粒界面に基づいて粒界モデルを構築します。

#### 引数リスト
- **`axis`**: **回転軸**（例: `[1, 1, 0]`）
- **`sigma`**: **シグマ値 ($\Sigma$)**（例: `3`）
- **`face`**: **粒界面**（例: `[1, 1, 0]`）
- **`s_input`**: **結晶構造**（`Grain` オブジェクト）
- **`uc_a`**: **左側バルク倍率**（初期値: `1`）
- **`uc_b`**: **右側バルク倍率**（初期値: `1`）

> [!TIP]
> 粒界エネルギーの安定性のために、`uc_a` と `uc_b` を同じ値に設定することが推奨されます。

### 使用例 (Basic Example)

```python
from aimsgb import GrainBoundary, Grain
import os

# 結晶構造の読み込み
s_input = Grain.from_file("POSCAR_MASnI3")

axis = [1, 1, 0]
sigma = 3
face = [1, 1, 0]

# 粒界モデルの作成
gb = GrainBoundary(axis, sigma, face, s_input, uc_a=2, uc_b=1)

# 2つの結晶粒をスタック
structure = Grain.stack_grains(gb.grain_a, gb.grain_b, direction=gb.direction)

# 保存
folder_path = f"masn001/sigma{sigma}_{face}"
if not os.path.exists(folder_path):
    os.makedirs(folder_path)
structure.to(filename=os.path.join(folder_path, "POSCAR"))
```

## 便利なユーティリティ

### GBInformation
回転軸と最大シグマ値を指定して、利用可能なCSL情報を一覧表示します。

```python
from aimsgb import GBInformation

print(GBInformation([0, 0, 1], 20).__str__())
```

### 粒界モデルの一括自動生成 (`get_allgb`)

指定した回転軸に対して $\Sigma \le 13$ までの粒界モデルを自動生成し、それぞれの構造ファイル（POSCAR）と設定条件（Condition.txt）を保存する関数例です。

```python
def get_allgb(axis, s_input):
    from aimsgb import GrainBoundary, Grain, GBInformation
    import os

    strings = GBInformation(axis, 13).__str__()
    # ... (解析ロジック詳細は notebook 参照)
    # 各Σ値と面指数に基づいてフォルダを作成し、POSCARとCondition.txtを保存します。
```

## 各種ツールの使用例

- **Materials Project からの構造取得**: `Grain.from_mp_id("mp-2815")` を使用して、IDから直接構造を取得できます。
- **CSL Generator**: `gb_code.csl_generator` を使用して、シグマ値に対応する回転角や最小セルを計算できます。

## 注意事項

> [!WARNING]
> **ディレクトリセパレータ**:
> ノートブック内のコードに `\` が含まれている場合がありますが、Linux or MacOS 環境では `/` に変更してください。
> 例: `folder_path = "masn001/sigma" + str(sigma) + "_" + str(face)`

- **依存ライブラリ**: `aimsgb` および `pymatgen` が必要です。
- **非直交格子**: 構造が非直交格子の構成要素（Grain）である場合、強制的に直交化されることがあり、本来の対称性が崩れる可能性があるという警告が出ることがあります。
