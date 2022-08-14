---
title: window.navigator.battery
slug: Web/API/Navigator/getBattery
tags:
  - API
  - Battery
  - Battery API
  - Deprecated
  - Device API
  - Navigator
  - Non-standard
  - Property
  - Reference
translation_of: Web/API/Navigator/battery
original_slug: Web/API/Navigator/battery
browser-compat: api.Navigator.battery
---
<p>{{ Apiref() }}</p>
<p><code>battery 객체는 시스템의 배터리 충전 상태에 대한 정보를 제공합니다. 배터리의 충전 상태가 변화할때 발생하는 이벤트에 대한 처리도 가능 합니다. 이 객체는 </code><a href="/en-US/docs/Web/API/Battery_Status_API" title="/en-US/docs/WebAPI/Battery_Status">Battery Status API</a> 의 구현입니다. 보다 자세한 내용, API, 샘플 코드 등은 문서를 참고 해 주세요.</p>
<h2 id="Syntax" name="Syntax">문법</h2>
<pre class="syntaxbox">var battery = window.navigator.battery;</pre>
<h2 id="값">값</h2>
<p><code>navigator.battery</code> 는 {{domxref("BatteryManager")}} 객체 입니다.</p>
<h2>브라우저 호환성</h2>
<p>{{Compat}}</p>
<h2>같이 보기</h2>
<ul>
 <li>{{domxref("BatteryManager")}}</li>
 <li><a href="/ko/docs/Web/API/Battery_Status_API">Battery Status API</a> 문서</li>
 <li><a class="external" href="https://hacks.mozilla.org/2012/02/using-the-battery-api-part-of-webapi/">블로그 - Using the Battery API</a></li>
 <li><a class="external" href="https://davidwalsh.name/battery-api">David Walsh 가 쓴 the JavaScript Battery Api</a></li>
 <li><a href="https://github.com/pstadler/battery.js">battery.js - 경량의 크로스 브라우저 랩퍼(wrapper)</a></li>
</ul>