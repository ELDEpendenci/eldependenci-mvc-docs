# 組件簡介

本框架的 UI 組件大致分成兩種，展示類和輸入類。

展示類並無輸入性功能，也就是只用作裝飾或按鈕使用。

```java
@UseTemplate(
        template = "main",
        groupResource = GUITemplate.class
)
public class MainView implements View<String> {

    @Override
    public void renderView(String s, UIContext context) {
        ButtonFactory button = context.factory(ButtonFactory.class); // Button 是展示類組件
        context.pattern('A')
                .components(
                        button.icon(Material.DIAMOND_BLOCK)
                                .title(s)
                                .create()
                );
    }
}
```

輸入類則相反，擁有數值輸入功能，主要用於綁定 Model 的屬性數值。

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
        // 以下的組件工廠都是輸入類
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

### 組件工廠

組件工廠是構建組件的工具，可在界面透過 `UIContext#factory` 取得。組件工廠采用鏈式建造模式作爲架構，使構造過程更簡潔和可讀。

**所有的組件工廠都會提供 `create()` 方法以創建並返回組件。**

