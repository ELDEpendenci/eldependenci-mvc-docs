---
description: 此為 v0.1.5 之後的功能
---

# 傳到 Controller 前的攔截

{% hint style="warning" %}
在閱讀本章節之前，請先參閱[快速開始](../quick-start.md)。
{% endhint %}

在界面事件把資料傳送到你所定義的 Controller 方法之前，可透過 中間件 (MiddleWare) 在兩者直接進行攔截修改。

假設你需要定義一個檢查權限的中間件，首先創建一個用於該中間件的標註:&#x20;

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface HasPermission {
    String value();
}
```

完成後，便可以實作所屬該標註的中間件:

```java
public class PermissionMiddleWare implements MiddleWare<HasPermission> {
    
    @Override
    public void intercept(InterceptContext interceptContext, HasPermission hasPermission) throws Exception {
        var permission = hasPermission.value();
        var player = interceptContext.getPlayer();
        if (!player.hasPermission(permission)) {
            // 沒有權限時重導向至 NoPermissionView
            player.sendMessage("you dont have permission!");
            interceptContext.setRedirect(new BukkitView<>(NoPermissionView.class, permission));
        }
        // 否則繼續通過
    }
    
    @ViewDescriptor(
            name = "沒有權限！",
            rows = 1,
            patterns = "ZZZZAZZZZ",
            cancelMove = {'Z', 'A'}
    )
    public static class NoPermissionView implements View<String> {

        @Override
        public void renderView(String s, UIContext uiContext) {
            uiContext.pattern('A').components(
                    uiContext.factory(ButtonFactory.class)
                            .icon(Material.IRON_AXE)
                            .title("&a你沒有以下權限: " + s)
                            .create()
            );
        }
    }
}
```

除了導向到其他 `View` 之外，你也可以透過拋出 `Exception` 來進行攔截。

```java
public class PermissionMiddleWare implements MiddleWare<HasPermission> {
    
    @Override
    public void intercept(InterceptContext interceptContext, HasPermission hasPermission) throws Exception {
        var permission = hasPermission.value();
        var player = interceptContext.getPlayer();
        if (!player.hasPermission(permission)) {
            // 沒有權限時拋出自定義錯誤
            player.sendMessage("you dont have permission!");
            throw new NoPermissionException(permission); // 使用 ViewExceptionHandler 來處理錯誤
        }
        // 否則繼續通過
    }
}
```

完成後，可透過 `MVCInstallation` 進行註冊。

```java
@Override
    public void bindServices(ServiceCollection serviceCollection) {
        MVCInstallation mvc = serviceCollection.getInstallation(MVCInstallation.class);
        mvc.registerMiddleWare(HasPermission.class, PermissionMiddleWare.class);
    }
```

### 使用在 Controller 內所有 Mapping 方法

要套用於一整個 Controller, 只需要在 Controller class 上進行標註即可。

```java
@UIController("main")
@HasPermission("gui.main") // 標註在 class 上
public class MainController {

    public BukkitView<?, ?> index(Player player) {
        String greeting = "hello, " + player.getName() + "!"; // 將顯示玩家的名稱
        return new BukkitView<>(MainView.class, greeting);
    }

    @ClickMapping(pattern = 'B', view = MainView.class)
    public void onSubmit(Player player, @MapAttribute('A') Map<String, Object> map) {
        var passwordHashed = (String) map.get("password");
        player.sendMessage(passwordHashed == null ? "null" : passwordHashed);
    }

}
```

### 使用在特定的 Mapping 方法上

直接在該方法上標註即可。

```java
@UIController("main")
public class MainController {

    @HasPermission("main.index")
    public BukkitView<?, ?> index(Player player) {
        String greeting = "hello, " + player.getName() + "!"; // 將顯示玩家的名稱
        return new BukkitView<>(MainView.class, greeting);
    }

    @HasPermission("main.submit")
    @ClickMapping(pattern = 'B', view = MainView.class)
    public void onSubmit(Player player, @MapAttribute('A') Map<String, Object> map) {
        var passwordHashed = (String) map.get("password");
        player.sendMessage(passwordHashed == null ? "null" : passwordHashed);
    }

}
```

{% hint style="success" %}
你可以透過標註多個中間件所屬標註來執行多個中間件的攔截修改。
{% endhint %}
