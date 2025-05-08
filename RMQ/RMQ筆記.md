# RabbitMQ 基本筆記

## 📌 RabbitMQ 是什麼？
RabbitMQ 是一個基於 AMQP（Advanced Message Queuing Protocol）的開源訊息代理（Message Broker），用來實現系統之間的解耦與非同步通訊。

---

## 🧱 基本元件 (Concepts)

### 1. Producer
- 發送訊息的應用程式（訊息的來源）。

### 2. Consumer
- 接收並處理訊息的應用程式。

### 3. Queue
- 訊息的儲存區。Producer 發送的訊息會被送到某個 queue，等待 consumer 消費。

### 4. Exchange
- 接收 producer 的訊息並根據 routing rule 決定要送到哪個 queue。
- 有幾種常見類型：

| Type       | 說明 |
|------------|------|
| direct     | 精準匹配 routing key。 |
| topic      | 模糊匹配 routing key（支援 * 和 #）。 |
| fanout     | 廣播，忽略 routing key，訊息會發送給所有綁定的 queue。 |
| headers    | 根據 headers 而非 routing key 進行路由（較少用）。 |

### 5. Binding
- exchange 和 queue 之間的綁定關係，可以指定 routing key。

### 6. Routing Key
- Producer 發送訊息時指定的 key，用來讓 exchange 做路由判斷。

---

## 🔄 訊息流程（Message Flow）

-Producer --> Exchange --> Queue --> Consumer
1. Producer 發送訊息到 Exchange。
2. Exchange 根據類型與 routing key，將訊息分發到對應的 Queue。
3. Consumer 從 Queue 中讀取並處理訊息。

## 🔁 消費模式（Consumer Mode）

- **自動確認（auto-ack）**
  - Consumer 收到訊息後即視為已處理，若中途錯誤會導致訊息遺失。
- **手動確認（manual-ack）**
  - Consumer 處理完成後明確發出 ack，若處理失敗未 ack，訊息可重送。