
# .NET 單元測試介紹

## 📘 單元測試（Unit Test）是什麼？

- **目標**：驗證單一邏輯單元（如一個 class 或 method）的行為。
- **特性**：不依賴資料庫、外部 API 或服務，測試執行快速，容易維護。
- **好處**：快速檢查程式邏輯是否正確，避免重構造成 bug。

---

## 🧪 範例說明：xUnit + NSubstitute

```csharp
[Fact(DisplayName = "follow時 user upsert")]
public void UpsertUser()
{
    var omniUser = new OmniUser
    {
        Id = 0,
        LineUserId = "testUserId",
        ProviderId = 1,
        ChannelId = 1,
    };

    _userDAO.Upsert(omniUser).Returns(9);

    _userModule.UpsertUser(omniUser);

    Assert.Equal(9, omniUser.Id);
    _userDAO.Received(1).Upsert(omniUser);
    _userDAO.Received(1).AddUserLog(omniUser);
    _userDAO.Received(1).UpsertInfo(omniUser);
}
```

### ✅ 驗證目標：
- 模擬 DAO 回傳 Id = 9
- 確保正確設定 `omniUser.Id`
- 檢查 DAO 方法都被正確呼叫

---

## 🧰 常見 .NET 單元測試工具

| 類別     | 工具 / 技術               | 功能說明 |
|----------|---------------------------|----------|
| 測試框架 | xUnit / NUnit / MSTest    | 撰寫與執行單元測試 |
| Mock 工具| NSubstitute / Moq / FakeItEasy | 模擬依賴，驗證呼叫 |
| 覆蓋率   | Coverlet / dotCover        | 測試覆蓋率報告產出 |
| 斷言語法 | FluentAssertions           | 提供更易讀的斷言語法 |
| 測試資料庫 | In-memory DB / SQLite    | 用於模擬 EF 的資料操作 |

---

## 📚 延伸資源

1. [xUnit 官方文件](https://xunit.net/)
2. [NSubstitute 官方](https://nsubstitute.github.io/)
3. [FluentAssertions](https://fluentassertions.com/)
4. [測試替身（Test Double）說明](https://martinfowler.com/bliki/TestDouble.html)
