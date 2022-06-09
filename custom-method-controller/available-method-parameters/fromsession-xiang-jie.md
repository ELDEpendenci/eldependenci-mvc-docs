# @FromSession 詳解

顧名思義，則是從 Session 中提取數據，避免了直接對 `UISession` 的填入呼叫。

```java
@ClickMapping(view = MainView.class, pattern = 'B')
public void onClick(@FromSession("word") @Nullable String word, Player player){
   // 相當於填入 UISession session 並使用 session.getAttribute("word")
   // 因此是可為 null 的數值。
   player.sendMessage(word);
}
```

`@FromSession` 也可透過填入是否為 poll 而使用 `pollAttribute`

```java
@ClickMapping(view = MainView.class, pattern = 'B')
public void onClick(@FromSession("word", poll = true) @Nullable String word, Player player){
   // 相當於填入 UISession session 並使用 session.pollAttribute("word")
   // 因此是可為 null 的數值。
   player.sendMessage(word);
}
```
