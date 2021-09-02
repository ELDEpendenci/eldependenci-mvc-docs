# 異步加載界面

{% hint style="warning" %}
在閱讀本章節之前，請先參閱[快速開始](../quick-start.md)。
{% endhint %}

在快速開始中提到，自定義方法的回傳類型是支援以 `BukkitPromise<T>` 回傳 `BukkitView` 的，那麼在異步加載中尚未跳轉到下一個界面的時候，就需要一個異步加載界面來向界面使用者表示該界面正在加載中。

{% hint style="info" %}
如果沒有異步加載界面，除了會讓界面使用者失去反饋感之外，他們也會誤認為界面失去了回應，便打算再次點擊，這將會造成不必要的效能問題。
{% endhint %}

本框架有內置的默認異步加載界面，但你亦可以套用自己的異步加載界面，本章節將講述如何新增自定義異步加載界面。

要新增一個異步加載界面，你需要創建一個 class 並繼承 `LoadingView`:

```java
@ViewDescriptor(
        name = "Loading...",
        rows = 1,
        patterns = {"ZZZZZZZZZ"},
        cancelMove = {'Z'}
)
public final class MyLoadingView implements LoadingView {

    @Override
    public void renderView(Void model, UIContext context) {
        AnimatedButtonFactory animatedButton = context.factory(AnimatedButtonFactory.class); // 動畫按鈕組件工廠
        context.pattern('Z') // 指定 pattern Z
                .fill( // 填滿組件
                        animatedButton.interval(1) // 動畫間隔
                                .icons( // 圖案動畫列表
                                        Material.GREEN_STAINED_GLASS_PANE,
                                        Material.RED_STAINED_GLASS_PANE,
                                        Material.BLUE_STAINED_GLASS_PANE,
                                        Material.BLACK_STAINED_GLASS_PANE,
                                        Material.WHITE_STAINED_GLASS_PANE
                                ).create()
                );
    }

}
```

## 設置為全局異步加載界面

如果你希望覆蓋本框架內置默認的異步加載界面設定，讓所有沒有指定異步加載界面的界面插件都使用你的異步加載界面的話，那麼你可以調用 `MVCInstallation` 設置如下:

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {


    @Override
    protected void bindServices(ServiceCollection serviceCollection) {

        MVCInstallation mvc = serviceCollection.getInstallation(MVCInstallation.class);
        mvc.setGlobalLoadingView(MyLoadingView.class); // 註冊全局默認異步加載界面
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {

    }
}
```

## 設置為指定 Controller 的異步加載界面

如果你希望設置為指定的 Controller 所使用的異步記載界面的話，你只需要在 Controller class 上標註 `@AsyncLoadingView` 即可。

```java
@AsyncLoadingView(MyLoadingView.class) // 設置為 AsyncController 所使用的異步加載界面
@UIController("async")
public final class AsyncController {

    @Inject
    private ScheduleService scheduleService;

    @Inject
    private ELDGPlugin plugin;

    public ScheduleService.BukkitPromise<BukkitView<?, ?>> index(){
        return scheduleService.runAsync(plugin, () -> {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }).thenApplySync(v -> new BukkitView<>(AsyncView.class));
    }

    @ClickMapping(view = AsyncView.class, pattern = 'A')
    public ScheduleService.BukkitPromise<BukkitView<?, ?>> onClick(Player player){
        player.sendMessage("3 seconds to go to the user view.");
        return scheduleService.runAsync(plugin, () -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).thenApplySync(v -> new BukkitRedirectView("user"));
    }
}
```

## 設置為指定的界面互動處理的異步記載界面

如果你希望設置為指定的界面互動處理\(自定義 method\)所使用的異步加載界面的話，只需要在 該界面互動處理的 method 上標註 `@AsyncLoadingView` 即可。

```java
@UIController("async")
public final class AsyncController {

    @Inject
    private ScheduleService scheduleService;

    @Inject
    private ELDGPlugin plugin;
    
    // 這裏則使用回默認的異步加載界面
    public ScheduleService.BukkitPromise<BukkitView<?, ?>> index(){
        return scheduleService.runAsync(plugin, () -> {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }).thenApplySync(v -> new BukkitView<>(AsyncView.class));
    }


    @AsyncLoadingView(MyLoadingView.class) // 設置為指定界面互動處理的異步加載界面
    @ClickMapping(view = AsyncView.class, pattern = 'A')
    public ScheduleService.BukkitPromise<BukkitView<?, ?>> onClick(Player player){
        player.sendMessage("3 seconds to go to the user view.");
        return scheduleService.runAsync(plugin, () -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).thenApplySync(v -> new BukkitRedirectView("user"));
    }
}
```

