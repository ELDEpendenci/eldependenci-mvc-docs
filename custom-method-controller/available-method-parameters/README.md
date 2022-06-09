# 目前可用的填入參數

填入參數是指自定義界面互動處理方法内的參數。

```java
@UIController("main")
public class MainController {

    public BukkitView<?, ?> index(Player player) {
        String greeting = "hello, " + player.getName() + "!";
        return new BukkitView<>(MainView.class, greeting);
    }

    @ClickMapping(pattern = 'A', view = MainView.class)
    public void onClickA(Player player) { // Player 是一個填入參數
        player.sendMessage("activated !!!!");
    }

}
```

目前可用的填入參數如下

| 參數類型                                | 解釋                                 | 用於 index | 用於 界面互動處理方法 | 用於控制器生命周期挂鈎      | 可用於界面生命週期掛鉤 |
| ----------------------------------- | ---------------------------------- | -------- | ----------- | ---------------- | ----------- |
| `Player`                            | 玩家 (界面使用者)                         | 可        | 可           | 可                | 可           |
| `UISession`                         | Session 數據                         | 可        | 可           | 可                | 可           |
| `@FromPattern List<ItemStack>`      | 獲取指定 Pattern 内的所有物品                | 不可       | 可           | 僅限 `@PreDestroy` | 可           |
| `ItemStack`                         | 界面互動事件的觸發物品                        | 不可       | 可           | 不可               | 不可          |
| `? extends InventoryInteractEvent`  | 原事件類 (必須根據觸發事件定義)                  | 不可       | 可           | 不可               | 不可          |
| `@ItemAttribute (自定義類型)`            | 獲取觸發物品的指定 key 的數值                  | 不可       | 可           | 不可               | 不可          |
| `@ModelAttribute (自定義類型)`           | 獲取指定 Pattern 内所有組件的綁定屬性數值並返回所屬類型實例 | 不可       | 可           | 僅限 `@PreDestroy` | 可           |
| `@MapAttribute Map<String, Object>` | 獲取指定 Pattern 内所有組件的綁定屬性數值並返回 Map   | 不可       | 可           | 僅限 `@PreDestroy` | 可           |

{% hint style="info" %}
解釋詳看後頁。
{% endhint %}
