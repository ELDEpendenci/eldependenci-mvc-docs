# 異常界面處理渲染

{% hint style="warning" %}
在閱讀本章節之前，請先參閱[快速開始](../quick-start.md)。
{% endhint %}

新增異常界面的目的是為了防止在使用界面時因為拋出異常而導致原有界面失效。因此，界面在拋出異常後將會跳轉到異常界面並以界面的形式顯示拋出的錯誤訊息。如果你有使用過 Web Server 的經驗的話，那就相當於在 500 錯誤 之後所拋出的錯誤頁面。

以下是本框架內置的默認異常錯誤界面:

```java
@ViewDescriptor(
        name = "Error Encountered",
        rows = 1,
        patterns = {"ZZZZAZZZZ"},
        cancelMove = {'A', 'Z'}
)
public class StaticErrorView implements View<Exception> { // 以 Exception 為數據模型

    @Override
    public void renderView(Exception ex, UIContext context) {
        ButtonFactory button = context.factory(ButtonFactory.class);

        context
                .pattern('Z')
                .fill(button.icon(Material.BLACK_STAINED_GLASS_PANE).create())
                .and() // 完成設置一個 pattern 後返回
                .pattern('A')
                .components(
                        button.icon(Material.BARRIER)
                                .title("&cError: " + ex.getClass().getSimpleName()) // 顯示異常類名稱
                                .lore("&c".concat(ex.getMessage())) // 顯示異常訊息
                                .create()
                );
    }
}
```

要使拋出錯誤後跳轉到這個界面，需要一個 異常界面處理器 控制跳轉。

```java
public final class ELDGExceptionViewHandler implements ExceptionViewHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(ELDGExceptionViewHandler.class);

    // 設置全局異常界面處理
    @Override
    public BukkitView<?, ?> createErrorView(Exception exception, String fromController, UISession session, Player player) {
        LOGGER.warn("Resolved Error: " + exception.getMessage(), exception);
        return new BukkitView<>(StaticErrorView.class, exception);
    }
    
}
```

如果要在處理指定的異常時跳轉到指定的界面，則可以新增自定義方法並標註 `@HandleException`

```java
public final class ELDGExceptionViewHandler implements ExceptionViewHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(ELDGExceptionViewHandler.class);

    // 全局處理異常，如果沒有指明其自定義方法，則使用此方法
    @Override
    public BukkitView<?, ?> createErrorView(Exception exception, String fromController, UISession session, Player player) {
        LOGGER.warn("Resolved Error: " + exception.getMessage(), exception);
        return new BukkitView<>(StaticErrorView.class, exception);
    }

    // 指定異常處理 (可多於一個，如果多於一個，參數應該為 Exception)
    @HandleException(MyCustomException.class)
    public BukkitView<?, ?> handleMyOwnException(MyCustomException ex, String from, UISession session, Player player) {
        LOGGER.warn("Resolved Error: " + ex.getMessage());
        session.setAttribute("exception", ex);
        session.setAttribute("from", from);
        return new BukkitRedirectView("error");
    }

}
```

{% hint style="danger" %}
新增的自定義處理方法的參數必須與全局處理方法的參數一致，否則會報錯。
{% endhint %}

## 設置為全局異常處理器

只需要調用 `MVCInstallation` 註冊即可。設置全局處理器將使所有界面插件所拋出的錯誤經由你的處理器去處理。

```java
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        MVCInstallation mvc = serviceCollection.getInstallation(MVCInstallation.class);
        mvc.setGlobalExceptionHandler(MyOwnExceptionViewHandler.class); // 設置全局處理器
    }
```

## 設置為插件內的異常處理器

要設置為 Plugin Scoped (插件內的異常處理器), 則使用 `addExceptionViewHandlers` 方法而非全局即可。

```java
    @Override
    public void bindServices(ServiceCollection serviceCollection) {
        MVCInstallation mvc = serviceCollection.getInstallation(MVCInstallation.class);
        mvc.addExceptionViewHandlers(List.of(MyOwnExceptionViewHandler.class)); // 新增多個異常處理器
    }
```

### 設置為指定 Controller 的異常處理器

其註冊方式與上述相同，只需要在 你的異常處理器上 標註 `@HandleForControllers` 即可。

```java
@HandleForControllers(MainController.class) // 指定 Controller (可多過一個)
public class MyOwnExceptionViewHandler implements ExceptionViewHandler {

    @Override
    public BukkitView<?, ?> createErrorView(Exception e, String s, UISession uiSession, Player player) {
        uiSession.setAttribute("error", e);
        uiSession.setAttribute("from", s);
        return new BukkitRedirectView("error");
    }
}
```
