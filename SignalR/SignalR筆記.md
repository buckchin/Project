# 📡 SignalR 學習筆記

## 🧩 什麼是 SignalR？

SignalR 是 .NET 提供的一套**即時通訊框架**，可以讓 **Server 主動推送訊息給 Client**，不需要 Client 持續發出輪詢請求。  
這有效解決了傳統輪詢方式造成的：
- 延遲高
- 頻寬浪費
- 效能差

📌 常見應用場景：
- 即時聊天室
- 即時通知（如紅點提醒）
- 多人線上協作編輯（如 Google Docs）
- 股票/即時數據看板

---

## 🚀 SignalR 支援的傳輸協定（Transport Protocols）

SignalR 根據客戶端與伺服器的能力，自動選擇以下傳輸方式：

1. **WebSocket**
   - ✅ 最優先、效能最好
   - ✅ 支援雙向通訊
   - ❗️要求 Server 和 Client 都支援 WebSocket（需 HTTP/1.1+，無代理阻擋）
   
2. **Server-Sent Events (SSE)**
   - ➡️ 僅支援 **Server → Client** 單向推播
   - ✅ 相容 Chrome / Firefox
   - ❌ 不支援 IE / Edge

3. **Long Polling**
   - 📦 最基本、相容性最好
   - 📉 效能最差：Client 請求後需等待 Server 回應，再重新連線

---

## 🧱 核心架構：Hub vs PersistentConnection

| API | 特性 | 適合用途 |
|-----|------|----------|
| `Hub` | 高階 API，可像本地方法一樣呼叫 Client/Server 方法，隱藏底層細節 | 一般應用場景（聊天室、推播等） |
| `PersistentConnection` | 低階 API，需自行解析訊息格式與方法 | 特殊協定場景，如遊戲、IoT 等 |

---

## 🔐 認證與授權（Authentication & Authorization）

### ✅ 認證方式（Authentication）
- 使用 Cookie 或 JWT（JSON Web Token）
- 連線時會自動攜帶認證資訊
- 需在 Startup 設定中啟用認證中介軟體

### ✅ 授權方式（Authorization）
- 透過 `Context.User` 判斷角色、Claims
- 可在 `OnConnectedAsync` 中加入邏輯拒絕未授權用戶

### ✍️ 範例：
```csharp
public class ChatHub : Hub
{
    public override async Task OnConnectedAsync()
    {
        if (!Context.User.Identity.IsAuthenticated)
        {
            Context.Abort(); // 拒絕連線
        }

        await base.OnConnectedAsync();
    }

    public async Task SendMessage(string message)
    {
        var user = Context.User.Identity.Name;
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```
Startup
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR();
    services.AddAuthentication(options => {
        // 設定認證方式，如 JWT 或 Cookie
    });
}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseAuthentication();
    app.UseRouting();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chatHub");
    });
}
```
```csharp
public class ChatHub : Hub
{
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

```javascript
<script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/6.0.0/signalr.min.js"></script>
<script>
    const connection = new signalR.HubConnectionBuilder()
        .withUrl("/chatHub")
        .build();

    connection.on("ReceiveMessage", (user, message) => {
        console.log(`${user}: ${message}`);
    });

    connection.start().then(() => {
        connection.invoke("SendMessage", "Alice", "Hello from browser!");
    });
</script>
```