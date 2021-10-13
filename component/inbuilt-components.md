# 框架内置組件工廠一覽



|          組件工廠名稱         | 組件名稱   | 是否為輸入類 | 綁定的屬性類型             | 特性                    |
| :---------------------: | ------ | ------ | ------------------- | --------------------- |
|    `NumInputFactory`    | 數字輸入組件 | 是      | 任何繼承 `Number` 的數字類型 | Clickable, Listenable |
|    `TextInputFactory`   | 文字輸入組件 | 是      | `String`            | Listenable            |
| `AnimatedButtonFactory` | 動畫按鈕組件 | 否      | /                   | Animatable            |
|   `BukkitItemFactory`   | 物品組件   | 否      | /                   | /                     |
|    `CheckboxFactory`    | 勾選框組件  | 是      | `Boolean`           | Clickable             |
|  `DateSelectorFactory`  | 日期輸入組件 | 是      | `LocalDate`         | Clickable, Listenable |
|   `RGBSelectorFactory`  | 顔色輸入組件 | 是      | `org.bukkit.Color`  | Clickable, Listenable |
|    `SelectionFactory`   | 選擇組件   | 是      | 任何                  | Clickable             |
|  `TimeSelectorFactory`  | 時間輸入組件 | 是      | `LocalTime`         | Clickable, Listenable |

{% hint style="info" %}
關於各個組件工廠的使用方法解釋，請自行參閲 javadoc 文件。
{% endhint %}
