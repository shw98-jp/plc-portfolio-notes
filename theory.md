---
layout: page
title: Theory
permalink: /theory/
---

PLC・ラダー・FA制御に関する基礎理論を整理します。

<div class="project-list">
  <a class="project-card" href="{{ '/theory/plc-programming-basics/' | relative_url }}">
    <p class="project-kicker">PLC Basic / GX Works</p>
    <h2>PLC Programming Basics</h2>
    <p>PLCの基本動作、デバイス、I/Oリスト、ラダー図、インターロック、アラーム処理を整理します。</p>
    <ul>
      <li>PLCのスキャン動作</li>
      <li>X / Y / M / T / C / D デバイス</li>
      <li>安全・異常処理の考え方</li>
    </ul>
  </a>
  <a class="project-card" href="{{ '/theory/self-holding-circuit/' | relative_url }}">
    <p class="project-kicker">PLC Basic / Ladder Circuit</p>
    <h2>Self-Holding Circuit</h2>
    <p>スタートボタンを一度押した後も、出力のON状態を維持する自己保持回路を整理します。</p>
    <ul>
      <li>モーメンタリ動作とオルタネイト動作</li>
      <li>X0 / X1 / Y0を使った自己保持</li>
      <li>スタート・停止時の動作確認</li>
    </ul>
  </a>
  <a class="project-card" href="{{ '/theory/internal-devices/' | relative_url }}">
    <p class="project-kicker">PLC Basic / Internal Devices</p>
    <h2>PLC Internal Devices</h2>
    <p>PLC内部で使うMリレーとLリレーの役割、共通点、使い分けを整理します。</p>
    <ul>
      <li>M：補助リレーの使い方</li>
      <li>L：ラッチリレーの使い方</li>
      <li>自動ドアと看板照明の制御例</li>
    </ul>
  </a>
</div>
