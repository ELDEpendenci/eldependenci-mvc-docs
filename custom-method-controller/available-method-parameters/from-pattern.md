# @FromPattern 詳解

要獲取指定 pattern 内的所有物品，只需要填入方法參數 `@FromPattern List<ItemStack>` 並輸入指定 pattern:

```java
@UIController("main")
public class MainController {

    public BukkitView<?, ?> index(Player player) {
        String greeting = "hello, " + player.getName() + "!"; // 將顯示玩家的名稱
        return new BukkitView<>(MainView.class, greeting);
    }
    
    @ClickMapping(view = MainView.class, pattern = 'A')
    public void clicked(Player player, @FromPattern('Z') List<ItemStack> items, ItemStack clicked){
        player.sendMessage("you clicked "+clicked.getType());
        assert items.stream().allMatch(item -> item.getType() == Material.BLACK_STAINED_GLASS_PANE);
        assert clicked.getType() == Material.DIAMOND_BLOCK;
    }
}
```

從上述可以看見，填入參數中，`ItemStack` 只代表了被點擊的物品，而 `@FromPattern` 則代表了該 pattern 内的所有物品，因此必須使用 `List<ItemStack>` 作爲其類型。

{% hint style="info" %}
根據之前範例的物品，以上兩個 assert 理應為 true。
{% endhint %}



