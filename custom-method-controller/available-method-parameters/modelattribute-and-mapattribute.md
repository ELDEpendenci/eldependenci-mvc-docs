# @ModelAttribute 與 @MapAttribute

## @ModelAttribute 詳解

在某些輸入類組件中，擁有綁定一個 Model 的屬性和數值的功能。與 `bind(key, value)` 方法不同的是，他們需要根據該 Model 的 屬性類型去綁定數值，而非像 `bind` 一樣可任意綁定自定義的鍵和任意數值。

以下為 `TextInputField` 工廠的綁定屬性方法源碼:

```java
    @Override
    public TextInputFactory bindInput(String field, String initValue) {
        bind(AttributeController.FIELD_TAG, field);
        bind(AttributeController.VALUE_TAG, initValue);
        return this;
    }
```

從上述代碼可以看到，所綁定的方式並不是以 屬性作爲 key，數值作爲 value，而是把他們透過特定的 key 來分開綁定。因此，**一個組件只能綁定一個屬性數值**。

### 使用範例

我們以框架内的 demo 作爲教學:

{% tabs %}
{% tab title="數據模型" %}
```java
public class TestModel {

    public Color testColor;

    public LocalDate testDate;

    public LocalTime testTime;

    @Override
    public String toString() {
        return "TestModel{" +
                "testColor=" + testColor.toString() +
                ", testDate=" + testDate.toString() +
                ", testTime=" + testTime.toString() +
                '}';
    }
}
```
{% endtab %}

{% tab title="界面" %}
```java
@ViewDescriptor(
        name = "Test GUI",
        rows = 2,
        patterns = {
                "ZZZZZZZZZ",
                "ZZZZZZZZA"
        },
        cancelMove = {'Z', 'A'}
)
public class TestView implements View<Void> {

    @Override
    public void renderView(Void model, UIContext context) {
        ButtonFactory button = context.factory(ButtonFactory.class);

        RGBSelectorFactory rgbSelector = context.factory(RGBSelectorFactory.class);
        DateSelectorFactory dateSelector = context.factory(DateSelectorFactory.class);
        TimeSelectorFactory timeSelector = context.factory(TimeSelectorFactory.class);

        context.pattern('Z')
                .components(
                        rgbSelector
                                .bindInput("testColor", Color.WHITE)
                                .label("&aColor Select: (shift to move color, click to +/-, middle to input)")
                                .create(),
                        dateSelector
                                .bindInput("testDate", LocalDate.now())
                                .label("&aDate Select: (shift move unit, click to +/-, middle to input)")
                                .icon(Material.BEACON)
                                .create(),
                        timeSelector
                                .bindInput("testTime", LocalTime.now())
                                .label("&aTime Select: (shift move unit, click to +/-, middle to input)")
                                .icon(Material.CLOCK)
                                .create()
                )
                .and()
                .pattern('A')
                .components(
                        button.title("&aTest").icon(Material.DIAMOND_BLOCK).create()
                );
    }

}
```
{% endtab %}
{% endtabs %}

從上述代碼中你可以發現，`bindInput` 所綁定的 屬性名稱 與 數據模型内的屬性名稱是一致的。

控制器代碼如下

```java
@UIController("test")
public class TestController {

    @PostConstruct
    public void beforeCreate(Player player){
        player.sendMessage("life cycle: before create for test controller");
    }

    public BukkitView<?, ?> index(){
        return new BukkitView<>(TestView.class);
    }


    @ClickMapping(view = TestView.class, pattern = 'A')
    public void onClick(@ModelAttribute('Z') TestModel test, Player player){
        player.sendMessage(test.toString());
    }


    @PreDestroy
    public void beforeDestroy(Player player){
        player.sendMessage("life cycle: before destroy for test controller");
    }
}
```

Pattern Z 内有三個組件，他們分別是 RGB顔色選擇器，日期選擇器和時間選擇器。於是，本框架會根據該組件所綁定的屬性和數值，為指定的 POJO 類型創建一個新實例並為該實例賦值。

{% hint style="danger" %}
該 POJO 類型不能爲泛型且必須擁有無參數構造器，否則會報錯。
{% endhint %}

## @MapAttribute 詳解

`@MapAttribute` 與 `@ModelAttribue` 性質大致相同。唯一與 `@ModelAttribute` 不一樣的是，`@MapAttribute` 必須以 `Map<String, Object>` 作爲其返回類型。

```java
@UIController("test")
public class TestController {

    @PostConstruct
    public void beforeCreate(Player player){
        player.sendMessage("life cycle: before create for test controller");
    }

    public BukkitView<?, ?> index(){
        return new BukkitView<>(TestView.class);
    }


    @ClickMapping(view = TestView.class, pattern = 'A')
    public void onClick(Player player, @MapAttribute('Z') Map<String, Object> map){
        player.sendMessage(map.toString());
    }


    @PreDestroy
    public void beforeDestroy(Player player){
        player.sendMessage("life cycle: before destroy for test controller");
    }
}
```

返回的屬性數值與剛才使用 `@ModelAttribute` 所返回的基本相同，只是改用了 Map 的形式裝載著。

## 選填屬性

`@ModelAttribute` 和 `@MapAttribute` 可以接受數值為 null 的屬性，你可以用以檢測界面使用者有什麼數值尚未填寫。例如:

```java
    @ClickMapping(view = UserUpdateView.class, pattern = 'B')
    public BukkitView<?, ?> onSave(@ModelAttribute('A') User user, Player player) {
        if (user.username == null){ // 若果是 MapAttribute, 則 map.get("username") == null 
            player.sendMessage("username is required."); // 玩家尚未填寫 username 欄位
            return null; // 返回 null 以不跳轉任何界面
        }
        player.sendMessage("Pre Saving: "+user.toString());
        userService.save(user);
        player.sendMessage("Save Success");
        return index();
    }
```

