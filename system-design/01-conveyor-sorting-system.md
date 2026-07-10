---
layout: page
title: Conveyor Product Sorting System
permalink: /system-design/conveyor-sorting-system/
---

<p class="project-kicker">GX Works2 / PLC System Design</p>

GX Works2で作成した、金属・非金属製品の仕分けコンベヤシステムです。  
搬送、判定、停止、仕分け、再運転までの一連の制御をPLCで構成しました。

<div class="summary-panel">
  <div>
    <span>Purpose</span>
    <p>製品を金属・非金属に判定し、Aボックス・Bボックスへ仕分けます。</p>
  </div>
  <div>
    <span>Control</span>
    <p>仕分け位置でコンベヤを停止し、排出後に再運転します。</p>
  </div>
  <div>
    <span>Tools</span>
    <p>GX Works2 / GX Simulator2</p>
  </div>
</div>

<div class="project-actions">
  <a href="https://github.com/shw98-jp/plc-portfolio-notes/tree/main/projects/gxworks2/01-conveyor-sorting-system">View Project Folder</a>
  <a href="{{ '/projects/gxworks2/01-conveyor-sorting-system/PLC_Conveyor_Sorting_System.zip' | relative_url }}">Download Project</a>
</div>

## System Flow

<div class="flow-strip">
  <span>START</span>
  <span>搬送</span>
  <span>金属判定</span>
  <span>位置検出</span>
  <span>仕分け</span>
  <span>初期化</span>
  <span>再運転</span>
</div>

## I/O Summary

<table class="io-table">
  <thead>
    <tr>
      <th>Type</th>
      <th>Device</th>
      <th>Role</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Input</td>
      <td>X0 / X1 / X2</td>
      <td>始動・停止・非常停止</td>
    </tr>
    <tr>
      <td>Input</td>
      <td>X3 / X4</td>
      <td>金属検出・仕分け位置検出</td>
    </tr>
    <tr>
      <td>Output</td>
      <td>Y0 / Y1 / Y2</td>
      <td>コンベヤ・金属仕分け・非金属仕分け</td>
    </tr>
    <tr>
      <td>Internal</td>
      <td>M0 / M10 / M20</td>
      <td>運転保持・判定保持・仕分け中保持</td>
    </tr>
  </tbody>
</table>

## Control Points

<ul class="compact-list">
  <li><strong>M0</strong>で始動状態を自己保持します。</li>
  <li><strong>M10</strong>で金属製品の判定結果を保持します。</li>
  <li><strong>X4 ON</strong>でコンベヤを停止し、仕分け動作を開始します。</li>
  <li><strong>M20</strong>で仕分け中状態を保持し、完了後に初期化します。</li>
</ul>

## Ladder Screens

<div class="image-grid">
  <figure>
    <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/01_run_self_hold.png' | relative_url }}" alt="Run self-hold ladder circuit">
    <figcaption>01. 自己保持回路</figcaption>
  </figure>
  <figure>
    <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/02_conveyor_motor_control.png' | relative_url }}" alt="Conveyor motor control ladder circuit">
    <figcaption>02. コンベヤモータ制御</figcaption>
  </figure>
  <figure>
    <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/03_metal_detection.png' | relative_url }}" alt="Metal detection ladder circuit">
    <figcaption>03. 金属検出</figcaption>
  </figure>
  <figure>
    <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/04_product_sorting.png' | relative_url }}" alt="Product sorting ladder circuit">
    <figcaption>04. 製品仕分け</figcaption>
  </figure>
  <figure>
    <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/05_sorting_active.png' | relative_url }}" alt="Sorting active ladder circuit">
    <figcaption>05. 仕分け中保持</figcaption>
  </figure>
  <figure>
    <img src="{{ '/assets/images/system-design/01-conveyor-sorting-system/06_sorting_reset.png' | relative_url }}" alt="Sorting reset ladder circuit">
    <figcaption>06. 状態初期化</figcaption>
  </figure>
</div>

## Test Result

<ul class="status-list">
  <li>始動・停止・非常停止制御を確認</li>
  <li>金属・非金属の仕分け動作を確認</li>
  <li>仕分け完了後の初期化と再運転を確認</li>
  <li>GX Simulator2で動作確認</li>
</ul>