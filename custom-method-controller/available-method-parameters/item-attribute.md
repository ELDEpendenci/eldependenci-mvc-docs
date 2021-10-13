# @ItemAttribute 詳解

在界面渲染中，你可以為任意組件綁定 key 和數值。以之前的演示 `MainView` 爲例:

```java
@UseTemplate(
        template = "main",
        groupResource = GUITemplate.class
)
public class MainView implements View<String> {

    @Override
    public void renderView(String s, UIContext context) {
        ButtonFactory button = context.factory(ButtonFactory.class);
        context.pattern('A')
                .components(
                        button.icon(Material.DIAMOND_BLOCK)
                                .title(s)
                                .bind("say", "Have a nice day!") // 綁定鍵 say 為數值 "Have a nice day!"
                                .create()
                );
    }
}

```

從上述的例子中，該組件了類型為 String 的數值，那麽則在 控制器 使用如下:

```java
@UIController("main")
public class MainController {

    public BukkitView<?, ?> index(Player player) {
        String greeting = "hello, " + player.getName() + "!"; // 將顯示玩家的名稱
        return new BukkitView<>(MainView.class, greeting);
    }

    @ClickMapping(view = MainView.class, pattern = 'A')
    public void clicked(Player player, @ItemAttribute("say") String say){
        player.sendMessage(say); // 發送 "Have a nice day!"
    }
}
```

標注内填入的參數為該綁定的 key ，然後其填入的參數類型則爲該數值的類型。

{% hint style="danger" %}
參數類型必須符合數值内的類型，否則會報錯。
{% endhint %}
