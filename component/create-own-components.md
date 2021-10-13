# 創建自定義組件

有時候你希望創建獨特的物品展示，又或者創建特殊的輸入組件以綁定特定的數值類型，僅限框架內置的組件可能無法滿足你的需要。考慮到這點，本框架提供了組件接口，讓你可以創建你自己的組件，供給自己甚至他人使用。

要創建組件，必須要先創建生成組件的對外接口——組件工廠。

架設你欲創建一個密碼輸入組件，你需要創建一個 接口 繼承 `ComponentFactory<T>` 並自定義其建造方法。

```java
// 密碼輸入組件
public interface PasswordFieldFactory extends ComponentFactory<PasswordFieldFactory> { // 繼承 ComponentFactory

    // 綁定屬性。由於是密碼，所以沒有初始數值。
    PasswordFieldFactory bindInput(String field);

    // 顯示密碼文字
    PasswordFieldFactory showPasswordTxt(String show);

    // 隱藏密碼文字
    PasswordFieldFactory hidePasswordTxt(String hide);

    // 設置密碼混淆類型
    PasswordFieldFactory hashType(HashType type);

    // 設置標題顯示
    PasswordFieldFactory label(String label);
    
    // 設置輸入提示訊息
    PasswordFieldFactory inputMessage(String input);

    // 設置無效提示訊息
    PasswordFieldFactory invalidMessage(String invalid);
    
    // 設置 regex 來規限密碼格式
    PasswordFieldFactory regex(String regex);
    
    // 設置等待最大輸入時間
    PasswordFieldFactory maxWait(long maxWait);
    
    // 設置禁用組件
    PasswordFieldFactory disabled();
    
    // hash類型
    enum HashType {
        SHA_256, MD5
    }

}
```

{% hint style="warning" %}
被 `UIContext` 取出的 組件工廠類別 必須 為`Interface`，因此你有必要創建接口類別。
{% endhint %}

接口創建完成後，就可以開始創建實作類別。

為了使創建組件工廠更簡單，框架內置了抽象類別 `AbstractComponentFactory<T>`。透過繼承它，創建組件工廠將會更快捷簡單。

```java
public class PasswordFieldFactoryImpl extends AbstractComponentFactory<PasswordFieldFactory> implements PasswordFieldFactory {

    private static final Map<HashType, String> hashConvert = Map.of(
            HashType.MD5, "MD5",
            HashType.SHA_256, "SHA-256"
    );

    // 設置可變屬性
    private String showPasswordTxt;
    private String hidePasswordTxt;
    private String inputMessage;
    private String invalidMessage;
    private long maxWait;
    private Pattern regex;
    private HashType hashType;
    private boolean disabled;

    public PasswordFieldFactoryImpl(ItemStackService itemStackService, AttributeController attributeController) {
        super(itemStackService, attributeController);
    }

    // 在這裏設置默認數值
    @Override
    protected void defaultProperties() {
        this.showPasswordTxt = "&a顯示密碼";
        this.hidePasswordTxt = "&c隱藏密碼";
        this.inputMessage = "請在聊天欄輸入你的密碼。";
        this.invalidMessage = "無效的密碼格式。";
        this.maxWait = 200L;
        this.regex = Pattern.compile("\\d+"); // 僅限數字的密碼格式
        this.hashType = HashType.MD5;
        this.disabled = false;
    }

    @Override
    public Component build(ItemStackService.ItemFactory itemFactory) {
        // 暫時漏空
        return null;
    }

    @Override
    public PasswordFieldFactory bindInput(String field) {
        bind(AttributeController.FIELD_TAG, field);
        bind(AttributeController.VALUE_TAG, null); // 設置初始賦值為 null
        return this;
    }

    @Override
    public PasswordFieldFactory showPasswordTxt(String show) {
        this.showPasswordTxt = show;
        return this;
    }

    @Override
    public PasswordFieldFactory hidePasswordTxt(String hide) {
        this.hidePasswordTxt = hide;
        return this;
    }

    @Override
    public PasswordFieldFactory hashType(HashType type) {
        this.hashType = type;
        return this;
    }

    @Override
    public PasswordFieldFactory label(String label) {
        return editItemByFactory(f -> f.display(label)); // 調用 editItemByFactory 來更改物品的顯示
    }

    @Override
    public PasswordFieldFactory inputMessage(String input) {
        this.inputMessage = input;
        return this;
    }

    @Override
    public PasswordFieldFactory invalidMessage(String invalid) {
        this.invalidMessage = invalid;
        return this;
    }

    @Override
    public PasswordFieldFactory regex(String regex) {
        this.regex = Pattern.compile(regex);
        return this;
    }

    @Override
    public PasswordFieldFactory maxWait(long maxWait) {
        this.maxWait = maxWait;
        return this;
    }

    @Override
    public PasswordFieldFactory disabled() {
        this.disabled = true;
        return this;
    }

    // 定義 hash 方法
    static String hash(String plain, HashType type) {
        try {
            MessageDigest digest = MessageDigest.getInstance(hashConvert.get(type));
            byte[] hashed = digest.digest(plain.getBytes(StandardCharsets.UTF_8));
            return bytesToHex(hashed);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }

    // 將 bytes 轉變為 hex string 的方法
    private static final byte[] HEX_ARRAY = "0123456789ABCDEF".getBytes(StandardCharsets.US_ASCII);

    private static String bytesToHex(byte[] bytes) {
        byte[] hexChars = new byte[bytes.length * 2];
        for (int j = 0; j < bytes.length; j++) {
            int v = bytes[j] & 0xFF;
            hexChars[j * 2] = HEX_ARRAY[v >>> 4];
            hexChars[j * 2 + 1] = HEX_ARRAY[v & 0x0F];
        }
        return new String(hexChars, StandardCharsets.UTF_8);
    }
}
```

接著，創建一個 `PasswordField` 組件，但這次將不需要創建接口。而同樣，為了使創建組件更簡單，框架內置了抽象類別 `AbstractComponent<T>`。

密碼輸入組件除了需要監聽玩家輸入外，還需要可點擊以切換密碼顯示狀態。因此，我們需要實作 `Listenable<T>` 和 `Clickable`。

```java
public class PasswordField extends AbstractComponent implements Listenable<AsyncChatEvent>, Clickable {

    // 獲取剛才的屬性
    private final String showPasswordTxt;
    private final String hidePasswordTxt;
    private final String inputMessage;
    private final String invalidMessage;
    private final long maxWait;
    private final Pattern regex;
    private final PasswordFieldFactory.HashType hashType;
    private final boolean disabled;

    private String plainText; // 純文字密碼
    private boolean showTxt = false; // 切換顯示

    public PasswordField(
            AttributeController attributeController,
            ItemStackService.ItemFactory itemFactory,
            String showPasswordTxt,
            String hidePasswordTxt,
            String inputMessage,
            String invalidMessage,
            long maxWait,
            Pattern regex,
            PasswordFieldFactory.HashType hashType,
            boolean disabled
    ) {
        super(attributeController, itemFactory);
        this.showPasswordTxt = showPasswordTxt;
        this.hidePasswordTxt = hidePasswordTxt;
        this.inputMessage = inputMessage;
        this.invalidMessage = invalidMessage;
        this.maxWait = maxWait;
        this.regex = regex;
        this.hashType = hashType;
        this.disabled = disabled;

        this.plainText = attributeController.getAttribute(getItem(), AttributeController.VALUE_TAG);
        this.updateItem(); // 更新物品顯示
    }

    @Override
    public void onClick(InventoryClickEvent event) {
        if (event.getClick() != ClickType.MIDDLE) return;
        this.showTxt = !showTxt;
        this.updateItem();  // 更新物品顯示
    }

    @Override
    public boolean isDisabled() {
        return disabled;
    }

    @Override
    public void onListen(Player player) {
        player.sendMessage(inputMessage);
    }

    @Override
    public long getMaxWaitingTime() {
        return maxWait;
    }

    @Override
    public void callBack(AsyncChatEvent event) {
        String message = ((TextComponent) event.message()).content();
        if (!regex.matcher(message).find()) {
            event.getPlayer().sendMessage(invalidMessage);
            return;
        }
        this.plainText = message;
        final String value = PasswordFieldFactoryImpl.hash(this.plainText, hashType);
        attributeController.setAttribute(getItem(), AttributeController.VALUE_TAG, value); // 設置密碼 hash 值
        updateItem();  // 更新物品顯示
    }

    @Override
    public Class<AsyncChatEvent> getEventClass() {
        return AsyncChatEvent.class;
    }

    @Override
    public boolean shouldActivate(InventoryClickEvent e) {
        return e.getClick() != ClickType.MIDDLE; // 非中鍵點擊才會觸發監聽輸入
    }


    private void updateItem() {
        // 如果 plainText 為 null, 則顯示 NONE
        // 如果 showTxt 為 true, 則顯示純文字密碼
        // 如果 showTxt 為 false, 則顯示 ****
        itemFactory.lore(List.of(
                "-> " + (plainText == null ? "NONE" : showTxt ? plainText : "*".repeat(plainText.length())),
                "&b中鍵以 "+ (showTxt ? hidePasswordTxt : showPasswordTxt) // 提示字眼
        ));
        this.updateInventory(); // 必須調用 updateInventory 以重新渲染組件顯示
    }
}
```

{% hint style="danger" %}
假若你不使用框架內置的抽象類別來創建組件/組件工廠，除了需要自行實作預設的東西之外，你還需要創建一個與框架內置抽象類別相同參數的構造器以成功初始化你的組件/組件工廠。
{% endhint %}

然後，回到組件工廠實作創建方法。

```java
    @Override
    public Component build(ItemStackService.ItemFactory itemFactory) {
        return new PasswordField(
                attributeController,
                itemFactory,
                showPasswordTxt,
                hidePasswordTxt,
                inputMessage,
                invalidMessage,
                maxWait,
                regex,
                hashType,
                disabled
        );
    }
```

最後，獲取 `MVCInstallation` 並註冊你的組件。

```java
@ELDPlugin(
        registry = TesterRegistry.class,
        lifeCycle = TesterLifeCycle.class
)
public class ELDTester extends ELDBukkitPlugin {


    @Override
    protected void bindServices(ServiceCollection serviceCollection) {
        MVCInstallation mvc = serviceCollection.getInstallation(MVCInstallation.class);
        // 註冊組件
        mvc.addComponentFactory(PasswordFieldFactory.class, PasswordFieldFactoryImpl.class);
    }

    @Override
    protected void manageProvider(ManagerProvider provider) {

    }
}
```

然後，就可以開始使用:

```java
@UseTemplate(
        template = "main",
        groupResource = GUITemplate.class
)
public class MainView implements View<String> { // 此界面裝載 String 作為數據

    @Override
    public void renderView(String s, UIContext context) {
        PasswordFieldFactory password = context.factory(PasswordFieldFactory.class); // 獲取工廠接口
        context.pattern('A') // 指定 Pattern A
                .components( // 放入組件
                        password
                                .icon(Material.PAPER)
                                .label("&e輸入密碼")
                                .bindInput("password")
                                .hashType(PasswordFieldFactory.HashType.SHA_256)
                                .create()
                );
    }
}
```

{% hint style="info" %}
由於大部分屬性都已經賦予了默認數值，因此使用者可以只調用他們需要的建造方法。
{% endhint %}

### 使用演示

{% code title="GUI/main.yaml" %}
```yaml
name: "Main View"
rows: 1
pattern:
  - ZZZZAZZZB
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
  B:
    material: DIAMOND_BLOCK
    name: "&a遞交"
```
{% endcode %}

{% code title="MainController.java 中的自定義方法" %}
```java
    @ClickMapping(pattern = 'B', view = MainView.class)
    public void onSubmit(Player player, @MapAttribute('A') Map<String, Object> map) {
        var passwordHashed = (String) map.get("password"); // 獲取組件屬性數值
        player.sendMessage(passwordHashed == null ? "null" : passwordHashed);
    }
```
{% endcode %}

![](../.gitbook/assets/9adabf3b0fbb113aceaf012aee1f6590.gif)
