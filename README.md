## Релиз v2 — Заказы и товарные позиции

Реализовано оформление заказов и привязка к ним списка товаров.

---

### Заказы (Orders)

- Создание заказа (`total_price` изначально равен `0`).
- Получение заказа по `ID`.
- Получение всех заказов пользователя по `email`.
- Валидация входных данных и возврат ошибок `400`, `404`, `500`.

**Storage:**
```go
CreateOrder(order models.Order) (int, error)
GetOrderByID(id int) (models.Order, error)
GetOrdersByUserEmail(email string) ([]models.Order, error)
```

**HTTP endpoints:**
- `POST /orders` — создание заказа
- `GET /orders/{id}` — получение по ID
- `GET /users/orders?email=...` — заказы пользователя

---

### Позиции заказа (Order Items)

- Добавление товара в заказ и получение всех позиций заказа.
- Проверка `product_id` и `quantity`.
- Извлечение `order_id` из path-параметров URL.
- Автоматический пересчет итоговой стоимости через `UpdateOrderTotalPrice(orderID int) error`.

**Storage:**
```go
AddOrderItem(orderItem models.OrderItem) error
GetOrderItemsByOrderID(orderID int) ([]models.OrderItem, error)
UpdateOrderTotalPrice(orderID int) error
```

**HTTP endpoints:**
- `POST /orders/{id}/items` — добавить позицию
- `GET /orders/{id}/items` — список позиций

**Пример запроса (`POST /orders/1/items`):**
```json
{
  "ProductID": 1,
  "Quantity": 2
}
```

---

### Расчет стоимости

Итоговая сумма вычисляется в БД через единый SQL-запрос (`JOIN`, `SUM`, `COALESCE`):

$$\text{product.price} \times \text{order\_items.quantity}$$

Если позиций в заказе нет, сумма равна 0.

---

### Ключевые темы в v2

- Связи «один-ко-многим», внешние ключи (`FOREIGN KEY`) и SQL `JOIN`.
- Чтение query- и path-параметров в HTTP.
- Разграничение ошибок `400`, `404`, `500`.
- Юнит-тестирование хэндлеров с изолированными моками.

---

### Команды

```bash
# Тесты и качество кода
go test ./...
golangci-lint run ./...
go fmt ./...

# Запуск
go mod tidy
docker compose up --build
```