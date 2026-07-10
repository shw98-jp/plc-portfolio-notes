---
layout: page
title: Conveyor Product Sorting System
permalink: /system-design/conveyor-sorting-system/
---

<p class="project-kicker">GX Works2 / PLC System Design</p>

# Conveyor Product Sorting System

GX Works2を使用して作成した、金属・非金属製品の仕分けコンベヤシステムです。

<div class="project-actions">
  <a href="https://github.com/shw98-jp/plc-portfolio-notes/tree/main/projects/gxworks2/01-conveyor-sorting-system">View Project Folder</a>
  <a href="{{ '/projects/gxworks2/01-conveyor-sorting-system/PLC_Conveyor_Sorting_System.zip' | relative_url }}">Download Project</a>
</div>

---

## 1. 目的

コンベヤで搬送される製品が金属か非金属かを判定し、製品が仕分け位置に到着するとコンベヤを停止させます。

その後、金属製品と非金属製品をそれぞれ異なる方向へ仕分けします。

```text
金属製品   → Y1動作 → Aボックス
非金属製品 → Y2動作 → Bボックス
```

### 設計条件

- 製品は一度に1個ずつ投入する。
- 製品の投入は作業者が手動で行うものとする。
- 製品が仕分け位置に到着すると、コンベヤを一時停止する。
- 仕分け装置が製品を排出すると、コンベヤを再運転する。

---

## 2. 設備構成

```text
製品の移動方向 →

[金属検出センサ X3] ─────── [仕分け位置センサ X4]
                                   │
                              コンベヤ停止
                                   │
                      ┌────────────┴────────────┐
                      │                         │
                  Y1 金属仕分け             Y2 非金属仕分け
                      │                         │
                   Aボックス                  Bボックス
```

---

## 3. I/Oリスト

### 入力デバイス

| アドレス | デバイス名 | 用途 |
| --- | --- | --- |
| X0 | START_PB | コンベヤ始動ボタン |
| X1 | STOP_PB | コンベヤ停止ボタン |
| X2 | EMG_STOP | 非常停止ボタン |
| X3 | METAL_SENSOR | 金属製品検出センサ |
| X4 | SORT_POSITION_SENSOR | 製品の仕分け位置到着検出 |

### 出力デバイス

| アドレス | デバイス名 | 用途 |
| --- | --- | --- |
| Y0 | CONVEYOR_MOTOR | コンベヤモータ運転 |
| Y1 | METAL_SORTER | 金属製品仕分け装置 |
| Y2 | NON_METAL_SORTER | 非金属製品仕分け装置 |

### 内部デバイス

| アドレス | デバイス名 | 用途 |
| --- | --- | --- |
| M0 | RUN_FLAG | コンベヤ運転状態の保持 |
| M10 | METAL_FLAG | 金属製品の判定結果を保持 |
| M20 | SORTING_ACTIVE | 仕分け作業中の状態を保持 |

---

## 4. 回路の動作手順

### 4.1 運転開始および自己保持

X0の始動ボタンを押すと、M0がONになります。

M0の接点をX0と並列に配置し、始動ボタンから手を離しても運転状態が維持されるように自己保持回路を構成しました。

X1の停止ボタン、またはX2の非常停止ボタンを押すと、自己保持が解除されます。

```text
X0 ON
   ↓
M0 ON
   ↓
M0 自己保持
```

<figure class="article-image">
  <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/01_run_self_hold.png' | relative_url }}" alt="運転開始および自己保持回路">
  <figcaption>01. 運転開始および自己保持回路</figcaption>
</figure>

---

### 4.2 コンベヤモータ制御

次の条件がすべて成立すると、Y0のコンベヤモータが動作します。

```text
M0 ON
+
X2 OFF
+
X4 OFF
   ↓
Y0 ON
```

製品が仕分け位置に到着してX4がONになると、`[/X4]`接点が開き、Y0がOFFになってコンベヤが停止します。

```text
X4 ON
   ↓
Y0 OFF
   ↓
コンベヤ停止
```

<figure class="article-image">
  <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/02_conveyor_motor_control.png' | relative_url }}" alt="コンベヤモータ制御回路">
  <figcaption>02. コンベヤモータ制御回路</figcaption>
</figure>

---

### 4.3 金属製品の判定結果保持

コンベヤ運転中にX3の金属検出センサがONになると、`SET M10`命令を実行します。

```text
M0 ON
+
X3 ON
   ↓
SET M10
```

製品がX3センサを通過してX3がOFFになっても、M10はON状態を維持します。

M10は、現在搬送中の製品が金属製品であるという判定結果を保持します。

<figure class="article-image">
  <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/03_metal_detection.png' | relative_url }}" alt="金属製品の判定結果保持回路">
  <figcaption>03. 金属製品の判定結果保持回路</figcaption>
</figure>

---

### 4.4 金属・非金属製品の仕分け

製品が仕分け位置に到着すると、X4がONになります。

PLCはX4とM10の状態を確認し、Y1またはY2を動作させます。

#### 金属製品

```text
M0 ON
+
X2 OFF
+
X4 ON
+
M10 ON
   ↓
Y1 ON
   ↓
Aボックスへ仕分け
```

#### 非金属製品

```text
M0 ON
+
X2 OFF
+
X4 ON
+
M10 OFF
   ↓
Y2 ON
   ↓
Bボックスへ仕分け
```

<figure class="article-image">
  <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/04_product_sorting.png' | relative_url }}" alt="金属・非金属製品仕分け回路">
  <figcaption>04. 金属・非金属製品仕分け回路</figcaption>
</figure>

---

### 4.5 仕分け作業中状態の保持

製品がX4に到着すると、`SET M20`命令を実行します。

```text
M0 ON
+
X4 ON
   ↓
SET M20
```

M20は、製品が仕分け位置に到着し、仕分け作業が開始されたことを保持する内部デバイスです。

<figure class="article-image">
  <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/05_sorting_active.png' | relative_url }}" alt="仕分け作業中状態保持回路">
  <figcaption>05. 仕分け作業中状態保持回路</figcaption>
</figure>

---

### 4.6 仕分け完了および状態の初期化

仕分け装置が製品を押し出すと、製品がX4センサから離れ、X4がOFFになります。

このとき、M20はまだON状態であるため、次の条件が成立します。

```text
M20 ON
+
X4 OFF
   ↓
RST M10
RST M20
```

初期化される情報は次のとおりです。

- `M10`：金属製品の判定結果
- `M20`：仕分け作業中の状態

初期化が完了すると、X4がOFF状態であるためY0が再びONになり、コンベヤが再運転します。

<figure class="article-image">
  <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/06_sorting_reset.png' | relative_url }}" alt="仕分け完了および状態初期化回路">
  <figcaption>06. 仕分け完了および状態初期化回路</figcaption>
</figure>

---

## 5. 全体の動作順序

```text
始動ボタンX0を入力
        ↓
M0自己保持
        ↓
Y0コンベヤ運転
        ↓
製品がX3を通過
        ↓
金属製品の場合はSET M10
        ↓
製品がX4に到着
        ↓
Y0 OFFおよびSET M20
        ↓
M10 ON  → Y1動作 → Aボックス
M10 OFF → Y2動作 → Bボックス
        ↓
製品がX4から離れる
        ↓
RST M10およびRST M20
        ↓
Y0再運転
```

---

## 6. ラダープログラム全体

GX Works2で作成したラダープログラム全体を確認し、各回路が設計した順序どおりに動作することを確認しました。

---

## 7. テスト手順

### 7.1 基本運転テスト

- [x] X0を押すと、M0とY0がONになることを確認
- [x] X0を離しても、自己保持が維持されることを確認
- [x] X1を押すと、M0とY0がOFFになることを確認
- [x] X2を押すと、M0とY0がOFFになることを確認
- [x] 停止解除後に、自動で再始動しないことを確認

### 7.2 金属製品テスト

```text
X0 ON/OFF
→ X3 ON/OFF
→ M10 ON状態を維持
→ X4 ON
→ Y0 OFF
→ Y1 ON
→ X4 OFF
→ M10・M20初期化
→ Y0再運転
```

### 7.3 非金属製品テスト

```text
X0 ON/OFF
→ X3入力なし
→ M10 OFF
→ X4 ON
→ Y0 OFF
→ Y2 ON
→ X4 OFF
→ M20初期化
→ Y0再運転
```

---

## 8. 実装結果

- [x] 始動・停止・非常停止制御
- [x] 自己保持回路
- [x] コンベヤモータ制御
- [x] 金属製品の判定結果保持
- [x] 金属・非金属製品の仕分け
- [x] 仕分け位置到着時のコンベヤ停止
- [x] 仕分け完了後の状態初期化
- [x] GX Simulator2による動作テスト

---

## 画像管理

画像は、リポジトリ内で次のように管理します。

```text
assets/images/system-design/01-conveyor-sorting-system/
├─ 01_run_self_hold.png
├─ 02_conveyor_motor_control.png
├─ 03_metal_detection.png
├─ 04_product_sorting.png
├─ 05_sorting_active.png
└─ 06_sorting_reset.png
```