# washer-shelf

洗面脱衣室の棚を検討するための FreeCAD モデル。`washer-shelf.FCStd` が本体。

FreeCAD へは MCP 経由で接続する。作業前に `get_rpc_status` で疎通を確認する。

## 座標系

実測の間取り図に合わせてあり、**上から見て原点が左上**。

| 軸 | 向き | 範囲 |
|---|---|---|
| X | 右 = 幅 | 0 〜 `room_width` |
| Y | **下（手前）= 奥行き。負の値** | 0 〜 −`room_depth` |
| Z | 上 = 高さ | 0 〜 `room_height` |

Y が負である点に注意。「奥から◯mm」は Y = −◯ を意味する。

## 部屋の形

L 型。右上に柱が室内へ張り出している。

```
(0,0)──910──┬──770──┐
  │         │ 柱     │ ← 200 手前に出る
  │         └────────┤
  │                  │
 1690                │
  │                  │
  └───────1680───────┘
```

- 奥の壁は X 0〜910 では Y=0、X 910〜1680 では Y=−200（柱の手前面）
- 壁は厚さ 20 の中空。上下は開口（天井・床なし）

## 寸法は必ずスプレッドシートのエイリアス経由で

**セル番地（`Spreadsheet.C42` など）で参照してはいけない。** 必ずエイリアス名を使う。

```python
obj.setExpression("Length", "Spreadsheet.room_width")   # ○
obj.setExpression("Length", "Spreadsheet.C1")           # × 行がずれると壊れる
```

### なぜか

このリポジトリでは番地参照が原因で 2 度事故を起こしている。

1. 洗濯機の配置値を末尾に追記したつもりが、窓の行を上書きし、窓枠幅と窓枠高さが消えた
2. 板の寸法を追記したつもりが、ドアの「枠出っ張り」を上書きし、ドア枠が 15mm → 2mm になった

いずれも「末尾だと思った行に既存データがあった」ことが原因。エイリアスなら参照は壊れないが、
**上書きそのものは防げない**ので、書き込む前に必ず空き行を確認すること。

```python
# 追記する前に必ず現在の中身を出力して確認する
for r in range(40, 60):
    cells = [sh.get("%s%d" % (c, r)) if sh.getContents("%s%d" % (c, r)) else "" for c in "ABC"]
    print(r, cells)
```

### 行の挿入は安全

FreeCAD は `insertRows` / `removeRows` のとき、他オブジェクトの式のセル参照を自動追従させる。
グループの途中に項目を足したいときは末尾に足さず `insertRows` を使うほうが安全。

### スプレッドシートの構成

A 列がグループ名、B 列が項目名、C 列が値（mm）。項目名は
**幅 / 高さ / 奥行き / 下端高さ / 奥から距離** で揃えている。

## エイリアス一覧

| エイリアス | グループ | 項目 | 値 |
|---|---|---|---|
| `room_width` | 部屋 | 幅 | 1680 |
| `room_depth` |  | 奥行き | 1690 |
| `room_height` |  | 天井までの高さ | 2400 |
| `washer_zone_width` | 洗濯機スペース | 幅 | 910 |
| `room_left_width` | 部屋(L型) | 左側の幅 | 910 |
| `pillar_depth` |  | 柱の出っ張り | 200 |
| `vanity_depth` | 洗面台 | 奥行き | 550 |
| `vanity_width` |  | 幅 | 750 |
| `vanity_height` |  | 高さ | 808 |
| `vanity_zone_width` |  | スペース幅 | 770 |
| `switch_width` | スイッチ | 幅 | 70 |
| `switch_height` |  | 高さ | 118 |
| `switch_bottom` |  | 下端高さ | 1140 |
| `switch_from_back` |  | 奥から距離 | 615 |
| `outlet_width` | コンセント | 幅 | 70 |
| `outlet_height` |  | 高さ | 119 |
| `outlet_bottom` |  | 下端高さ | 1140 |
| `outlet_from_back` |  | 奥から距離 | 408 |
| `faucet_width` | 水栓 | 幅 | 76 |
| `faucet_height` |  | 高さ | 120 |
| `faucet_bottom` |  | 下端高さ | 1140 |
| `faucet_from_back` |  | 奥から距離 | 303 |
| `dryer_width` | 浴室乾燥 | 幅 | 128 |
| `dryer_height` |  | 高さ | 118 |
| `dryer_bottom` |  | 下端高さ | 1393 |
| `dryer_from_back` |  | 奥から距離 | 587 |
| `washer_width` | 洗濯機 | 幅 | 590 |
| `washer_depth` |  | 奥行き | 750 |
| `washer_height` |  | 高さ | 1015 |
| `washer_lid_height` |  | 蓋の高さ | 273 |
| `washer_from_left` |  | 左から | 290 |
| `washer_from_back` |  | 奥から（柱の手前面が起点） | 130 |
| `window_width` | 窓 | 幅 | 300 |
| `window_height` |  | 高さ | 742 |
| `window_sill` |  | 下端高さ | 1274 |
| `window_proud` |  | 出っ張り | 17 |
| `window_from_left` |  | 左から | 484 |
| `window_casing_width` |  | 枠幅 | 18 |
| `baseboard_height` | 幅木 | 高さ | 55 |
| `baseboard_thickness` |  | 厚み | 3 |
| `wall_thickness` | 壁 | 壁厚 | 20 |
| `fixture_proud` | 壁付け機器 | 出っ張り | 5 |
| `door_from_back` | ドア | 奥から | 846 |
| `door_width` |  | 幅 | 803 |
| `door_height` |  | 高さ | 2000 |
| `door_casing_width` |  | 枠幅 | 20 |
| `door_casing_proud` |  | 枠出っ張り | 15 |
| `board_thickness` | 左壁の板1 | 厚み | 2 |
| `board_width` |  | 幅 | 50 |
| `board_from_back` |  | 奥から | 0 |
| `board2_from_faucet` | 左壁の板2 | 水栓の奥から | 20 |
| `wall_transparency` | 表示 | 壁の透明度 | 75 |

板1 と板2 は厚み・幅・高さを共有している（`board_thickness` / `board_width`）。
板2 の位置だけ水栓を起点にした式で、水栓を動かせば追従する。

## オブジェクト構成

- `Body`（PartDesign）— 部屋の壁。`RoomOuter` でパッド → `RoomOutline` でポケット →
  `DoorOpening` / `WindowOpening` でポケット
- それ以外（洗面台・洗濯機・壁付け機器・幅木・ドア枠・窓枠・板）は `Part::Box` を
  `Part` コンテナ直下に置き、寸法と `Placement` を式でリンク

## PartDesign のポケットは向きに注意

`Type = 1`（ThroughAll）でも、既定では材料と逆方向を切ってしまうことがある。
実際にドアの開口が反対側（右の壁）に開いた。`Reversed` を切り替えて、
**必ず切れた位置を検証すること**。

```python
s.isInside(FreeCAD.Vector(-10, -1200, 1000), 0.1, True)   # True なら壁がある
```

## スケッチ平面のローカル座標

| 平面 | ローカル u | ローカル v |
|---|---|---|
| XY | X | Y |
| XZ | X | Z |
| YZ | Y | Z |

新しい平面に描く前は、短い線分を 2 本置いて `Shape.Edges` の座標を見て確かめる。

## 変更したら必ず検証する

寸法を変えたり物を足したりしたら、以下を毎回確認する。

```python
# 干渉チェック
a.Shape.common(b.Shape).Volume        # 0 でなければ食い込んでいる

# スケッチが完全拘束のままか
sk.FullyConstrained

# 形状が壊れていないか
body.Shape.isValid()
len(body.Shape.Solids)
```

実測値どうしが矛盾していることが何度もあった（合計が部屋の幅に合わない等）。
干渉が出たら黙って位置をずらさず、**どの寸法が疑わしいかを数字で示して確認する**。

## ユーザーに図を見せる

リモートセッションなので、FreeCAD の画面はそのままでは相手に見えない。
画像に書き出して送る。

```python
v = FreeCADGui.getDocument("washer_shelf").ActiveView
v.viewIsometric(); v.fitAll()
v.saveImage("/path/out.png", 1600, 1200, "White")
```

- 奥の壁を見るのは `viewRear`（`viewFront` は手前の壁）
- 左の壁は `viewLeft`
- 壁が邪魔なときは `Body.ViewObject.Transparency`（現在 75）

## 壁の透明度が勝手に戻る

Body の表示は **Tip（先端フィーチャー）の ViewObject** が担っている。ポケットを追加すると
新しいフィーチャーが Tip になり、その Transparency は既定の 0 なので**不透明に戻ったように見える**。
実際に窓を足した直後にこれが起きた。

ViewObject のプロパティは式リンクできない（`setExpression` を持たない）ので、
値は `wall_transparency` に置いてあり、変更時・フィーチャー追加時にスクリプトで適用する。

```python
t = int(sh.get("C53"))                       # wall_transparency
body.ViewObject.Transparency = t
for o in body.Group:                         # Tip を含む全フィーチャーに効かせる
    if hasattr(getattr(o, "ViewObject", None), "Transparency"):
        o.ViewObject.Transparency = t
```

補助スケッチ（`RoomOutline` / `RoomOuter` / `DoorOpening` / `WindowOpening`）は
非表示にしておく。これらもフィーチャー追加時に表示が復活することがある。

## コミット

`*.FCBak` と `*.FCStd1` は `.gitignore` 済み。コミット前に FreeCAD 側で `doc.save()` すること。
