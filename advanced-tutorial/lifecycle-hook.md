# 生命周期掛鉤

{% hint style="warning" %}
在閱讀本章節之前，請先參閱[快速開始](../quick-start.md)。
{% endhint %}

## 控制器的生命週期掛鉤

控制器 \(Controller\) 擁有生命周期挂鈎，只需要透過新增自定義方法並添加標注即可。

目前可挂鈎的狀態如下:

| 生命周期 | 使用標注 |
| :--- | :--- |
| 控制器初始化完成 \(index 界面渲染之前\) | `@PostConstruct` |
| 界面使用者即將離開當前控制器 | `@PreDestroy` |

範例如下

```java
@UIController("main")
public class MainController {

    public BukkitView<?, ?> index(Player player) {
        String greeting = "hello, " + player.getName() + "!";
        return new BukkitView<>(MainView.class, greeting);
    }
    
    @PostConstruct //新增標注以挂鈎控制器的生命周期
    public void onControllerCreated(Player player){ // 自定義方法也可按需定義填入參數
        player.sendMessage("controller has been created.");
    }

}
```

{% hint style="danger" %}
生命周期掛鉤在每個控制器只能每個狀態挂鈎一個方法，且必須使用 `void` 作為返回類型。
{% endhint %}

## 界面的生命週期掛鉤

除了控制器之外，界面也擁有生命周期挂鈎，他們分別如下:

| 生命周期 | 使用標註 |
| :--- | :--- |
| 更新界面之後 | `@PostUpdateView` |
| 銷毀界面之前 | `@PreDestroyView` |

範例如下

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


    @PreDestroyView(MainView.class)
    public void onDestroyView(Player player){
        player.sendMessage("pre destroy view for main view");
    }
    
    @PostUpdateView(MainView.class)
    public void postUpdateView(Player player){
        player.sendMessage("updated to main view");
    }
}
```

{% hint style="danger" %}
界面生命週期的自定義方法必須使用 `void` 作為返回類型。
{% endhint %}

除此之外，相同界面的生命週期狀態掛鉤方法只能使用一個進行處理，例如

```java
    @PostUpdateView(MainView.class)
    public void postUpdateView(Player player){
        player.sendMessage("updated to main view");
    }

    @PostUpdateView(MainView.class)
    public void postUpdateView2(Player player){
        player.sendMessage("updated to main view 2");
    }
```

則只會使用隨機一個而不是兩者都執行。

