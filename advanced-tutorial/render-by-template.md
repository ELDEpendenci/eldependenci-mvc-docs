# 使用文件預先渲染界面

{% hint style="warning" %}
讀本章節之前，請先參閱[快速開始](../quick-start.md)。
{% endhint %}

在創建界面的時候，你需要在 class 上標註 `@ViewDescriptor` 來定義界面設定和其樣式:

```java
// 定義界面
@ViewDescriptor(
        name = "Main View", // 界面標題
        rows = 1, // 界面行數
        patterns = "ZZZZAZZZZ", // 界面的樣式
        cancelMove = 'A' // 需要取消移動的 pattern
)
public class MainView implements View<String> { // 此界面裝載 String 作為數據

    @Override
    public void renderView(String s, UIContext context) {
        ButtonFactory button = context.factory(ButtonFactory.class); //獲取 按鈕組件工廠
        context.pattern('A') // 指定 Pattern A
                .components( // 放入組件
                        button.icon(Material.DIAMOND_BLOCK) // 設置鑽石方塊
                                .title(s) // 設置顯示
                                .create() // 創建組件
                );
    }
}
```

但除了直接在界面上直接創建以外，你也可以使用文件配置來進行設定，甚至預渲染界面物品。

### 創建文件

首先，創建一個 文件夾，然後根據以下的模版在文件夾內新增 YAML 文件:

```java
public abstract class InventoryTemplate extends GroupConfiguration {

    public String name; // 名稱

    public int rows; // 行數

    public List<String> pattern; // 樣式

    public Map<String, ItemDescriptor> items; // 物品

    public static class ItemDescriptor {

        public Material material = Material.AIR; // 物品類型

        public String name = ""; // 物品名稱

        public int amount = 1; // 物品數量

        public List<String> lore = new ArrayList<>(); // 物品敘述

        public boolean glowing = false; // 發光

        public boolean cancelMove = true; // 取消移動

    }
}

```

假設在 GUI 文件夾內新增 main.yml

{% code title="GUI/main.yml" %}
```yaml
name: "Main View"
rows: 1
pattern:
  - ZZZZAZZZZ
items:
  A:
    # 因為顯示數據來自 Controller, 因此在這裏不會實作物品
    # 但需要指明取消移動
    cancelMove: true
  Z:
    # 其餘位置填滿黑色玻璃
    material: BLACK_STAINED_GLASS_PANE
    name: "&c這裏什麼都沒有"
    lore:
      - "&e這個物品"
      - "&e只是裝飾"
    cancelMove: true
```
{% endcode %}

則創建一個代表 GUI 文件夾的 文件池組，但繼承的是 `InventoryTemplate`

```java
@GroupResource(
        folder = "GUI",
        preloads = {"main"}
)
public class GUITemplate extends InventoryTemplate {
}
```

{% hint style="success" %}
由於 `InventoryTemplate`  已經定義了界面文件所需要的屬性，因此你將無需定義任何屬性。
{% endhint %}

然後，就可以在 View 上直接使用 `@UseTemplate` 標註。

```java
@UseTemplate(
        template = "main", // 文件名稱
        groupResource = GUITemplate.class // 文件池類別
)
public class MainView implements View<String> { // 此界面裝載 String 作為數據

    @Override
    public void renderView(String s, UIContext context) {
        ButtonFactory button = context.factory(ButtonFactory.class); //獲取 按鈕組件工廠
        context.pattern('A') // 指定 Pattern A
                .components( // 放入組件
                        button.icon(Material.DIAMOND_BLOCK) // 設置鑽石方塊
                                .title(s) // 設置顯示
                                .create() // 創建組件
                );
    }
}
```

{% hint style="info" %}
當然，記得別忘了把 `GUITemplate` 註冊為文件池！
{% endhint %}
