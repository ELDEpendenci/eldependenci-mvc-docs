# 開始前的準備

這是本 UI 框架的架構流程圖:

![](.gitbook/assets/eldependenci-mvc%20%281%29.jpg)

當中你可以到 數據模型 \(Model\)，控制器 \(Controller\) 和界面 \(View\) 三個區塊，它是 MVC 模式的主要架構。而當中，Controller 負責處理業務邏輯，Model 負責裝載數據，View 負責渲染界面。

{% hint style="success" %}
Controller 在執行一輪後台操作後取得 Model，然後選擇指定的 View，再連帶 Model 透過 View 去渲染界面。
{% endhint %}

除了 MVC 模式架構外，相信還有一點你應該注意到了，它霸佔了流程圖的一半空間 —— UI 組件系統。

如果你有了解過 HTML5 Form 的話，本框架的 UI 組件就相當於 form 內的 input 組件，他們可以綁定 Model 一個屬性的數值，在 遞交 的時候可連帶這些屬性數值一同傳回給 Controller 。

在本框架的 UI 組件系統中，目前共提供了五種特性修飾。這些特性能新增更多對組件的使用方式，也使組件能輸入和綁定更多不同類型的數值。

