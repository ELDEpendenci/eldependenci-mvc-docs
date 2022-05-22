# 自定義界面互動過濾

{% hint style="warning" %}
在閱讀本章節之前，請先參閱[快速開始](../quick-start.md)。
{% endhint %}

在快速開始中，我們講述過界面互動處理的需要標註： `@RequestMapping`， `@ClickMapping` 和 `@DragMapping` ，但當中的參數也就只有基本的指定界面和 Pattern。

有時候我們需要新增一些條件過濾，去過濾不必要的事件處理，這個時候就需要新增自定義界面互動過濾。

### 範例演示

首先，創建一個標註，你可以透過定義標註參數來為你的過濾元素提供可變性。

```java
@Target(ElementType.METHOD) // 標註位置必須為 method
@Retention(RetentionPolicy.RUNTIME)
public @interface MyOwnFilter {

    ClickType type(); // 新增要過濾的點擊類型作為參數

}
```

然後，到 `MVCInstallation` 那邊新增你的標註並定義其過濾方式。

```java
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        MVCInstallation mvc = serviceCollection.getInstallation(MVCInstallation.class);
        // 註冊你的自定義過濾標註
        mvc.registerQualifier(MyOwnFilter.class, (interactEvent, pattern, myOwnFilter) -> {
            if (!(interactEvent instanceof InventoryClickEvent)) return false; // 非點擊事件一律不處理
            var clickEvent = (InventoryClickEvent) interactEvent;
            return clickEvent.getClick() == myOwnFilter.type(); // 檢查點擊類型是否符合
        });
    }
```

最後，便可以到任何的 Controller 的界面互動處理方法中使用:

```java
@UIController("main")
public class MainController {

    public BukkitView<?, ?> index(Player player) {
        String greeting = "hello, " + player.getName() + "!"; // 將顯示玩家的名稱
        return new BukkitView<>(MainView.class, greeting);
    }

    @MyOwnFilter(type = ClickType.MIDDLE) // 中鍵點擊才會被觸發
    @ClickMapping(pattern = 'A', view = MainView.class)
    public void onClickA(Player player) {
        player.sendMessage("activated !!!!");
    }

}
```

{% hint style="danger" %}
有一點要注意的是，自定義界面互動過濾標註是建基於基本標註 ( @XXXXMapping ) 的，也就是說若果你的自定義處理方法中沒有 基本標註，標註任何的自定義界面互動過濾標註都不會有作用。
{% endhint %}

