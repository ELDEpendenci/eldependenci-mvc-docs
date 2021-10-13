# 組件特性修飾

目前本框架為 UI 組件提供了五種特性修飾

| 特性修飾類型        | 解釋                                                          |
| ------------- | ----------------------------------------------------------- |
| `Clickable`   | 可點擊，點擊後能根據點擊事件修改屬性數值，此特性自身繼承了 `Disable`                     |
| `Disable`     | 可禁用，禁用後將成爲僅展示的組件                                            |
| `Listenable`  | 可監聽，用於接收界面使用者所屬的指定玩家事件，然後根據該事件修改屬性數值，此特性自身繼承了 `Activitable` |
| `Activitable` | 可啓動，為可監聽特性提供了條件過濾 (是否啓動監聽)                                  |
| `Animatable`  | 可執行動畫，不同於上述四項，此特性也可被展示類組件使用                                 |

以下將使用框架内置的組件進行範例。

## Clickable 可點擊 / Disable 可禁用

```java
public final class Checkbox extends AbstractComponent implements Clickable {

    private final Material checkedIcon, uncheckedIcon;
    private final String checkedShow, uncheckedShow;
    private final boolean disabled;

    private boolean currentValue;

    /**
    構造器略
    **/

    @Override
    public void onClick(InventoryClickEvent event) {
        this.currentValue = !this.currentValue;
        attributeController.setAttribute(getItem(), AttributeController.VALUE_TAG, this.currentValue);
        itemFactory.lore(List.of("-> "+(this.currentValue ? checkedShow : uncheckedShow)));
        itemFactory.material(this.currentValue ? checkedIcon : uncheckedIcon);
        this.updateInventory();
    }

    @Override
    public boolean isDisabled() {
        return disabled;
    }

}
```

以上是一個勾選框組件。當中可見實作 Clickable 之後，組件透過了 `onClick` 方法切換數值，並更新對組件的顯示。另外 Disable 特性方面，只需要返回是否禁用的狀態即可。

## Animatable 可執行動畫

```java
public final class AnimatedButton extends AbstractComponent implements Animatable {

    private final String[][] lores;
    private final Material[] icons;
    private final String[] displays;
    private final Integer[] numbers;
    private final int seconds;

    private BukkitTask task = null;

    /**
    構造器略
    **/

    @Override
    public void startAnimation() {
        if (seconds > 0 && this.task == null){
            this.task = new AnimatedRunnable().runTaskTimer(ELDGPlugin.getPlugin(ELDGPlugin.class), 0L, 20L);
        }
    }

    @Override
    public boolean isAnimating() {
        return task != null && !task.isCancelled();
    }

    @Override
    public void stopAnimation() {
        if (this.task == null || this.task.isCancelled()) return;
        this.task.cancel();
        this.task = null;
    }

    private class AnimatedRunnable extends BukkitRunnable {

        private long timer = 0;

        // 以下略...
    }

}
```

以上是一個動畫按鈕組件。在實作 Animatable 之後，組件透過 `BukkitRunnable` 實作執行動畫，結束動畫，和檢查動畫是否正在運行的動作。

## Listenable 可監聽 / Activitable 可啓動

```java
public final class TextInputField extends AbstractComponent implements Listenable<AsyncChatEvent> {

    private final boolean disabled;
    private final long maxWait;
    private final String inputMessage;

    /**
    構造器略
    **/

    @Override
    public void onListen(Player player) {
        player.sendMessage(inputMessage);
    }

    @Override
    public long getMaxWaitingTime() {
        return maxWait;
    }

    @Override
    public void callBack(AsyncChatEvent event) {
        final String message = ((TextComponent) event.message()).content();
        attributeController.setAttribute(getItem(), AttributeController.VALUE_TAG, message);
        itemFactory.lore(List.of("-> " + message));
        this.updateInventory();
    }

    @Override
    public Class<AsyncChatEvent> getEventClass() {
        return AsyncChatEvent.class;
    }

    @Override
    public boolean shouldActivate(InventoryClickEvent e) {
        return !disabled;
    }
}
```

以上為文字輸入組件。在實作 Listenable 之後，你需要實作監聽啓動條件 (`shouldActivate`)，監聽開始時的動作 (`onListen`)，監聽事件類型 (`getEventClass`)，監聽事件的最長等待時間 (`getMaxWaitingTime`)，以及事件輸入處理 (`callBack`)。

當中 shouldActiviate 繼承自 Activitable，而屬性數值修改部分則在事件輸入處理的部分。
