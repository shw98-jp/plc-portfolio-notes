---
layout: page
title: Conveyor Product Sorting System
permalink: /system-design/conveyor-sorting-system/
---

{%- if site.github.build_revision -%}
  {%- assign page_cache_bust = site.github.build_revision -%}
{%- else -%}
  {%- assign page_cache_bust = site.time | date: "%s" -%}
{%- endif -%}

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

## 2. 設備構成および動作の流れ

<div class="operation-flow" aria-label="設備構成および動作の流れ">
  <div class="flow-item flow-start">
    <span class="flow-badge">START</span>
    <h3>製品投入</h3>
    <p>製品をコンベヤへ投入し、搬送を開始します。</p>
  </div>

  <div class="flow-arrow" aria-hidden="true">↓</div>

  <div class="flow-item">
    <span class="flow-badge">X3</span>
    <h3>金属検出センサ</h3>
    <p>金属を検出した場合、<code>SET M10</code>で判定結果を保持します。</p>
  </div>

  <div class="flow-arrow" aria-hidden="true">↓</div>

  <div class="flow-item">
    <span class="flow-badge">X4</span>
    <h3>仕分け位置センサ</h3>
    <p><code>X4 ON</code>で<code>Y0 OFF</code>となり、コンベヤを停止します。</p>
  </div>

  <div class="flow-arrow" aria-hidden="true">↓</div>

  <div class="flow-split">
    <div class="flow-branch-card">
      <span class="flow-badge">M10 ON</span>
      <h3>金属製品</h3>
      <p><code>Y1 ON</code>で金属製品をAボックスへ仕分けます。</p>
    </div>
    <div class="flow-branch-card">
      <span class="flow-badge">M10 OFF</span>
      <h3>非金属製品</h3>
      <p><code>Y2 ON</code>で非金属製品をBボックスへ仕分けます。</p>
    </div>
  </div>

  <div class="flow-arrow" aria-hidden="true">↓</div>

  <div class="flow-item flow-end">
    <span class="flow-badge">RESET</span>
    <h3>仕分け完了</h3>
    <p>製品がX4から離れると、<code>RST M10</code> / <code>RST M20</code>を実行し、<code>Y0 ON</code>でコンベヤを再運転します。</p>
  </div>
</div>

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

各回路の詳細は、項目を開いて確認できます。

<div class="process-steps">
  <details class="process-step">
    <summary><span class="step-label">4.1</span><span class="step-title">運転開始および自己保持</span></summary>
    <div class="step-body">
      <p>X0の始動ボタンを押すと、M0がONになります。</p>
      <p>M0の接点をX0と並列に配置し、始動ボタンから手を離しても運転状態が維持されるように自己保持回路を構成しました。</p>
      <p>X1の停止ボタン、またはX2の非常停止ボタンを押すと、自己保持が解除されます。</p>
      <pre><code>X0 ON
   ↓
M0 ON
   ↓
M0 自己保持</code></pre>
      <figure class="article-image">
        <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/01_run_self_hold.png' | relative_url }}?v={{ page_cache_bust }}" alt="運転開始および自己保持回路">
        <figcaption>01. 運転開始および自己保持回路</figcaption>
      </figure>
    </div>
  </details>

  <details class="process-step">
    <summary><span class="step-label">4.2</span><span class="step-title">コンベヤモータ制御</span></summary>
    <div class="step-body">
      <p>次の条件がすべて成立すると、Y0のコンベヤモータが動作します。</p>
      <pre><code>M0 ON
+
X2 OFF
+
X4 OFF
   ↓
Y0 ON</code></pre>
      <p>製品が仕分け位置に到着してX4がONになると、<code>[/X4]</code>接点が開き、Y0がOFFになってコンベヤが停止します。</p>
      <pre><code>X4 ON
   ↓
Y0 OFF
   ↓
コンベヤ停止</code></pre>
      <figure class="article-image">
        <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/02_conveyor_motor_control.png' | relative_url }}?v={{ page_cache_bust }}" alt="コンベヤモータ制御回路">
        <figcaption>02. コンベヤモータ制御回路</figcaption>
      </figure>
    </div>
  </details>

  <details class="process-step">
    <summary><span class="step-label">4.3</span><span class="step-title">金属製品の判定結果保持</span></summary>
    <div class="step-body">
      <p>コンベヤ運転中にX3の金属検出センサがONになると、<code>SET M10</code>命令を実行します。</p>
      <pre><code>M0 ON
+
X3 ON
   ↓
SET M10</code></pre>
      <p>製品がX3センサを通過してX3がOFFになっても、M10はON状態を維持します。</p>
      <p>M10は、現在搬送中の製品が金属製品であるという判定結果を保持します。</p>
      <figure class="article-image">
        <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/03_metal_detection.png' | relative_url }}?v={{ page_cache_bust }}" alt="金属製品の判定結果保持回路">
        <figcaption>03. 金属製品の判定結果保持回路</figcaption>
      </figure>
    </div>
  </details>

  <details class="process-step">
    <summary><span class="step-label">4.4</span><span class="step-title">金属・非金属製品の仕分け</span></summary>
    <div class="step-body">
      <p>製品が仕分け位置に到着すると、X4がONになります。</p>
      <p>PLCはX4とM10の状態を確認し、Y1またはY2を動作させます。</p>
      <h4>金属製品</h4>
      <pre><code>M0 ON
+
X2 OFF
+
X4 ON
+
M10 ON
   ↓
Y1 ON
   ↓
Aボックスへ仕分け</code></pre>
      <h4>非金属製品</h4>
      <pre><code>M0 ON
+
X2 OFF
+
X4 ON
+
M10 OFF
   ↓
Y2 ON
   ↓
Bボックスへ仕分け</code></pre>
      <figure class="article-image">
        <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/04_product_sorting.png' | relative_url }}?v={{ page_cache_bust }}" alt="金属・非金属製品仕分け回路">
        <figcaption>04. 金属・非金属製品仕分け回路</figcaption>
      </figure>
    </div>
  </details>

  <details class="process-step">
    <summary><span class="step-label">4.5</span><span class="step-title">仕分け作業中状態の保持</span></summary>
    <div class="step-body">
      <p>製品がX4に到着すると、<code>SET M20</code>命令を実行します。</p>
      <pre><code>M0 ON
+
X4 ON
   ↓
SET M20</code></pre>
      <p>M20は、製品が仕分け位置に到着し、仕分け作業が開始されたことを保持する内部デバイスです。</p>
      <figure class="article-image">
        <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/05_sorting_active.png' | relative_url }}?v={{ page_cache_bust }}" alt="仕分け作業中状態保持回路">
        <figcaption>05. 仕分け作業中状態保持回路</figcaption>
      </figure>
    </div>
  </details>

  <details class="process-step">
    <summary><span class="step-label">4.6</span><span class="step-title">仕分け完了および状態の初期化</span></summary>
    <div class="step-body">
      <p>仕分け装置が製品を押し出すと、製品がX4センサから離れ、X4がOFFになります。</p>
      <p>このとき、M20はまだON状態であるため、次の条件が成立します。</p>
      <pre><code>M20 ON
+
X4 OFF
   ↓
RST M10
RST M20</code></pre>
      <p>初期化される情報は次のとおりです。</p>
      <ul>
        <li><code>M10</code>：金属製品の判定結果</li>
        <li><code>M20</code>：仕分け作業中の状態</li>
      </ul>
      <p>初期化が完了すると、X4がOFF状態であるためY0が再びONになり、コンベヤが再運転します。</p>
      <figure class="article-image">
        <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/06_sorting_reset.png' | relative_url }}?v={{ page_cache_bust }}" alt="仕分け完了および状態初期化回路">
        <figcaption>06. 仕分け完了および状態初期化回路</figcaption>
      </figure>
    </div>
  </details>
</div>

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