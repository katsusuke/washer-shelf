# washer-shelf

洗面脱衣室の棚を検討するための FreeCAD モデル。`washer-shelf.FCStd` が本体。

FreeCAD へは MCP 経由で接続する。作業前に `get_rpc_status` で疎通を確認する。

## 座標系

実測の間取り図に合わせてあり、**上から見て原点が左上**。

| 軸 | 向き | 範囲 |
|---|---|---|
| X | 右 = 幅 | 0 〜 `room_width`(1680) |
| Y | **下（手前）= 奥行き。負の値** | 0 〜 −`room_depth`(1690) |
| Z | 上 = 高さ | 0 〜 `room_height`(2400) |

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
- 開口は 3 つ: 左壁のドア（Y −846〜−1649）、手前壁のドア（X 980〜1680）、奥壁の窓

## いまの棚の構成

**南海プライウッド アームハング棚柱SS** による壁付け方式。左の壁だけに付く。

```
Z=1838 ── 棚柱の上端
Z=1558 ── カゴの上端
Z=1308 ┬─ 棚板の上面
Z=1288 ┴─ 棚板の下面 ＝ 棚受の上面
Z=1238 ── 棚柱の下端（蓋の上端 1288 の 50 下）
Z=1218 ── 棚受の下端
```

| 部材 | 型番 | 寸法 | 位置 |
|---|---|---|---|
| 棚柱 ×2 | SS-H06W | 600 長 / 8.1 × 11 | Y −151.5、−546.5（芯々 395） |
| 棚受 ×2 | SS-MD40W | 三角 388 × 70 × 10 | 棚柱と同じ Y |
| 棚板 | G20-14R630-4WV | 630 × 450 × 20 | X 11〜461、Y −64〜−694 |
| カゴ ×2 | — | 上面 510×360 / 下面 390×250 / 高 250 | X 30〜540 |

棚柱の位置は機器の**すき間の中央**に置いてある。プレートの真上に置くと下端が
プレートに食い込む。

- 棚柱1 = コンセントの手前端とスイッチの奥端の中央
- 棚柱2 = 奥の壁と水栓の奥端の中央

棚柱の可動ピッチは **19mm**（1238 + 19n の位置にしか棚受が付かない）。棚板の下面
1288 はその倍数に乗っていないので、実際は 1276 か 1295 になる。

カゴは上が広がった錐台。棚板からリムがはみ出すのは承知のうえ（前に 30、手前に 56）。

## 寸法は必ずスプレッドシートのエイリアス経由で

**セル番地（`Spreadsheet.C42` など）で参照してはいけない。** 必ずエイリアス名を使う。

```python
obj.setExpression("Length", "Spreadsheet.room_width")   # ○
obj.setExpression("Length", "Spreadsheet.C1")           # × 行がずれると壊れる
```

### 書き込む前に空き行を確認する

このリポジトリでは既存セルの上書きを 3 回やっている。窓枠の寸法、ドアの枠出っ張り、
棚板の透明度。いずれも「末尾だと思った行に既存データがあった」ため。

```python
# 追記する前に必ず現在の中身を出力して確認する
for r in range(100, 115):
    cells = [sh.get("%s%d" % (c, r)) if sh.getContents("%s%d" % (c, r)) else "" for c in "ABC"]
    print(r, cells)
```

行の挿入（`insertRows` / `removeRows`）は安全。FreeCAD が他オブジェクトの式のセル参照を
自動追従させる。グループの途中に項目を足すならこちらを使う。

### 式の括弧に注意

文字列で式を組み立てるとき、部分式を括弧で囲み忘れると意味が変わる。

```python
TOP = "Spreadsheet.a + Spreadsheet.b"
"(Spreadsheet.room_height - %s) / 3" % TOP     # × room_height - a + b になる
TOP = "(Spreadsheet.a + Spreadsheet.b)"        # ○
```

## 使用中のエイリアス

A 列がグループ名、B 列が項目名、C 列が値（mm）。

| エイリアス | 値 | 意味 |
|---|---|---|
| `room_width` / `room_depth` / `room_height` | 1680 / 1690 / 2400 | 部屋の内寸 |
| `room_left_width` / `pillar_depth` | 910 / 200 | L 型の左側の幅 / 柱の出っ張り |
| `wall_thickness` | 20 | 壁厚 |
| `vanity_width` / `vanity_depth` / `vanity_height` | 750 / 550 / 808 | 洗面台 |
| `washer_width` / `washer_depth` / `washer_height` | 590 / 515 / 1015 | 洗濯機 |
| `washer_from_left` / `washer_from_back` | 290 / 130 | 洗濯機の位置（奥は柱の手前面基準） |
| `washer_lid_height` | 273 | 蓋を開けたときの立ち上がり |
| `lid_width` / `lid_depth` | 195 / 280 | 蓋の実寸 |
| `lid_from_left` / `lid_from_front` | 50 / 180 | 蓋の位置（洗濯機の左手前基準） |
| `switch_*` / `outlet_*` / `faucet_*` / `dryer_*` | | 壁付け機器の 幅/高さ/下端高さ/奥から距離 |
| `fixture_proud` | 5 | 壁付け機器の出っ張り |
| `window_width` / `window_height` / `window_sill` | 300 / 742 / 1277 | 窓 |
| `window_from_left` / `window_proud` / `window_casing_width` | 485 / 17 / 18 | 窓の位置・枠 |
| `door_from_back` / `door_width` / `door_height` | 846 / 803 / 2000 | 左壁のドア |
| `door_casing_width` / `door_casing_proud` | 20 / 15 | ドア枠 |
| `door2_width` / `door2_height` | 700 / 2000 | 手前壁のドア |
| `baseboard_height` / `baseboard_thickness` | 55 / 3 | 幅木 |
| `board_thickness` | 30 | 木の断面（カゴの逃げに流用中） |
| `shelf1_thickness` | 7 | 棚板の下面 1288 を出すのに使っている |
| `post_ss_width` / `post_ss_depth` / `post_ss_length` | 8.1 / 11 / 600 | 棚柱 SS-H06W |
| `post_ss_below_lid` | 50 | 棚柱の下端（蓋の上端からの下がり） |
| `shelf_w` / `shelf_d` / `shelf_t` | 630 / 450 / 20 | 棚板 G20-14R630-4WV |
| `shelf_gap_back` / `shelf_gap_wall` | 30 / 11 | 棚板の位置（奥の壁・壁からの逃げ） |
| `basket_width` / `basket_depth` / `basket_height` | 510 / 360 / 250 | カゴの上面と高さ |
| `basket_bottom_width` / `basket_bottom_depth` | 390 / 250 | カゴの下面 |

### 未使用のエイリアス（旧構成の名残）

削除した部材のもので、いまはどの式からも参照されていない。行を消すと番地がずれるので
**残してある**。同じ名前を再利用するときは値を確認すること。

`washer_zone_width` `vanity_zone_width` `board_width` `board_from_back`
`board2_from_faucet` `shelf1_depth` `basket_from_left` `shelf2_depth`
`shelf2_thickness` `foot_*` `angle_*` `strip_*` `post_width`
`rail_bottom_thickness` `arm_depth` `arm_width` `arm_height` `arm_overhang`
`*_transparency`（透明度は式リンクできないので値の記録用）

棚受の寸法（`arm_*`）が未使用なのは、棚受を `Part::Feature` で直接生成しているため。

## オブジェクト構成

- `Body`（PartDesign）— 部屋の壁。`RoomOuter` パッド → `RoomOutline` ポケット →
  `DoorOpening` / `WindowOpening` / `Door2Opening` ポケット
- それ以外は `Part` コンテナ直下。ほとんどは `Part::Box` で寸法と `Placement` を式リンク
- カゴは `Part::Wedge`（錐台）を X 軸まわり 90 度回転。ローカル (x,y,z) → グローバル (x, −z, y)
- **棚受だけ `Part::Feature`**。三角形を拘束付きスケッチで作ると形が崩れたため、
  `Part.Face(...).extrude(...)` で直接生成している。**式リンクがないので寸法変更時は
  作り直しが必要**

## PartDesign のポケットは向きと範囲に注意

`Type = 1`（ThroughAll）は既定と逆方向を切ることがある。実際にドアの開口が反対側の壁に
開いた。`Reversed` を切り替えて、**必ず切れた位置を検証する**。

```python
s.isInside(FreeCAD.Vector(-10, -1200, 1000), 0.1, True)   # True なら壁がある
```

貫通させると意図しない壁まで切る場合がある。手前壁のドアは柱と X 範囲が重なるため、
`Type = 0`（Length）で壁厚 20 ぶんだけ切り、スケッチを壁の内面に
`AttachmentOffset` で寄せている。

## スケッチ平面のローカル座標

| 平面 | ローカル u | ローカル v | 法線 |
|---|---|---|---|
| XY | X | Y | +Z |
| XZ | X | Z | −Y |
| YZ | Y | Z | +X |

XZ 平面の `AttachmentOffset.Base.z` は **グローバル −Y** に効く。

## 壁の透明度が勝手に戻る

Body の表示は **Tip（先端フィーチャー）の ViewObject** が担っている。ポケットを追加すると
新しいフィーチャーが Tip になり、その Transparency は既定の 0 なので**不透明に戻る**。

ViewObject のプロパティは式リンクできないので、値は `wall_transparency` に置いてあり、
変更時・フィーチャー追加時にスクリプトで適用する。

```python
t = int(sh.get("C53"))                       # wall_transparency
body.ViewObject.Transparency = t
for o in body.Group:                         # Tip を含む全フィーチャーに効かせる
    if hasattr(getattr(o, "ViewObject", None), "Transparency"):
        o.ViewObject.Transparency = t
```

補助スケッチ（`RoomOutline` / `RoomOuter` / `DoorOpening` / `WindowOpening` /
`Door2Opening`）は非表示にしておく。フィーチャー追加時に表示が復活することがある。

## 変更したら必ず検証する

```python
a.Shape.common(b.Shape).Volume        # 0 でなければ食い込んでいる
sk.FullyConstrained                    # スケッチが完全拘束のままか
body.Shape.isValid()
len(body.Shape.Solids)
```

**実測値どうしが矛盾していることが何度もあった**（合計が部屋の幅に合わない、洗濯機が柱に
めり込む等）。干渉が出たら黙って位置をずらさず、**どの寸法が疑わしいかを数字で示して確認する**。
ユーザーが指示した位置を勝手に変えない。

## 別案は既存を壊さずに作る

構成を変えて試したいときは、**いまの構成を書き換えない**。新しいグループを作り、そこに
別オブジェクトとして作る。見比べるときは片方の `Visibility` を落とす。

```
Part
├── 棚柱                    案で共通のもの（棚柱は動かさない）
├── 棚一式                  案A: MD40 + 奥行450 + カゴ2個/段
└── 案B (MD30 + 奥行300)    案B: MD30 + 奥行300 + カゴ1個/段（縦置き）
```

スプレッドシートも同じで、既存のエイリアスを書き換えず `b_` を付けた別セルを足す
（`b_arm_depth` `b_arm_overhang` `b_shelf_d`）。案が確定したら不要な方を消す。

採用しなかった案をすぐ消さないこと。寸法を戻すのは手間で、`Part::Feature` の部材は
作り直しになる。

## 寸法の報告は、指示とは限らない

「実測が◯◯から△△になりました」のような**報告を、値の更新指示と受け取って反映しない**。
「△△に変更しますか」と一度聞いてから直す。

このセッションでは「845 でした」「590 です」「450 です」といった報告がそのまま更新指示
だったやり取りが数十回続き、そのパターンに乗って蓋の高さを勝手に 273 → 203 に変えて
コミットまでした。頻度に引きずられて確認を省いた判断ミスで、文脈が足りなかったわけではない。

**他の部材との干渉判定が変わる値ほど確認する**。蓋の高さは棚板と当たるかどうかを決めていた。

体積で形を検証できる。三角柱なら等価な直方体の半分、扇形なら `r²(1−π/4)×厚み`。

## GUI dispatch がタイムアウトする

`execute_code` が「GUI dispatch timed out after 90s」で失敗することがある。**その場合
コードは実行されていない**ので、状態を確認してからやり直す。長いコードや大量のオブジェクト
削除で起きやすいので、削除は数個ずつに分ける。

## ユーザーに図を見せる

リモートセッションなので、FreeCAD の画面はそのままでは相手に見えない。画像に書き出して送る。

```python
v = FreeCADGui.getDocument("washer_shelf").ActiveView
cam = v.getCameraNode()
cam.orientation.setValue(coin.SbRotation(*q))   # 下の look() で作る
FreeCADGui.updateGui(); v.fitAll(); FreeCADGui.updateGui()
v.saveImage("/path/out.png", 3200, 2400, "White")
```

- **`viewIsometric()` などの標準ビュー関数は使わない**。切り替えアニメーション中に
  `saveImage` すると傾いた絵が撮れる。カメラのクォータニオンを直接設定する
- 方位角から向きを作る `look(az, el)` を使う。標準アイソメは方位 −45°、仰角 35.264°。
  −135° が左手前、−45° が右手前
- カメラの `position` を直接動かすとクリッピングで真っ白になりやすい。ズームより
  **高解像度（3200×2400）で出して相手に拡大してもらう**方が確実
- 部材が重なって見づらいときは、関係ないオブジェクトを一時的に `Visibility = False` に

## コミット

`*.FCBak` と `*.FCStd1` は `.gitignore` 済み。コミット前に FreeCAD 側で `doc.save()` すること。
