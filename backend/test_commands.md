# Руководство по тестированию FastAPI Shop API в Postman

## Содержание
1. [Настройка окружения](#настройка-окружения)
2. [Эндпоинты Categories](#categories)
3. [Эндпоинты Products](#products)
4. [Эндпоинты Cart](#cart)
5. [Типичные ошибки и коды ответов](#коды-ответов)
6. [Примеры тестовых сценариев](#тестовые-сценарии)

---

## Настройка окружения

### 1. Создание Environment в Postman

1. Нажмите **Environments** → **Add**
2. Назовите окружение: `FastAPI Shop - Local`
3. Добавьте переменную:

| Variable     | Initial Value           | Current Value           |
|-------------|------------------------|------------------------|
| `base_url`  | `http://localhost:8000` | `http://localhost:8000` |

4. Нажмите **Save**, затем выберите это окружение в правом верхнем углу.

### 2. Запуск сервера

```bash
cd backend
python run.py
```

Сервер будет доступен по адресу `http://localhost:8000`.
Swagger-документация: `http://localhost:8000/api/docs`

---

## Health Check

### GET / — Корневой эндпоинт

| Параметр  | Значение                          |
|-----------|----------------------------------|
| **URL**   | `{{base_url}}/`                  |
| **Метод** | `GET`                            |

**Ожидаемый ответ (200 OK):**
```json
{
    "message": "Welcome to fastapi shopw API",
    "docs": "api/docs"
}
```

---

### GET /health — Проверка состояния

| Параметр  | Значение                          |
|-----------|----------------------------------|
| **URL**   | `{{base_url}}/health`            |
| **Метод** | `GET`                            |

**Ожидаемый ответ (200 OK):**
```json
{
    "status": "healthy"
}
```

---

## Categories

### GET /api/categories — Получить все категории

| Параметр  | Значение                              |
|-----------|--------------------------------------|
| **URL**   | `{{base_url}}/api/categories`        |
| **Метод** | `GET`                                |
| **Headers** | не требуются                       |

**Ожидаемый ответ (200 OK):**
```json
[
    {
        "name": "Electronics",
        "slug": "electronics",
        "id": 1
    },
    {
        "name": "Clothing",
        "slug": "clothing",
        "id": 2
    },
    {
        "name": "Books",
        "slug": "books",
        "id": 3
    },
    {
        "name": "Home & Garden",
        "slug": "home-garden",
        "id": 4
    }
]
```

---

### GET /api/categories/{id} — Получить категорию по ID

| Параметр  | Значение                                    |
|-----------|---------------------------------------------|
| **URL**   | `{{base_url}}/api/categories/1`            |
| **Метод** | `GET`                                       |

**Ожидаемый ответ (200 OK):**
```json
{
    "name": "Electronics",
    "slug": "electronics",
    "id": 1
}
```

**Тест — несуществующая категория:**

URL: `{{base_url}}/api/categories/999`

**Ожидаемый ответ (404 Not Found):**
```json
{
    "detail": "Category with id 999 not found"
}
```

---

## Products

### GET /api/products — Получить все товары

| Параметр  | Значение                           |
|-----------|-----------------------------------|
| **URL**   | `{{base_url}}/api/products`       |
| **Метод** | `GET`                             |

**Ожидаемый ответ (200 OK):**
```json
{
    "products": [
        {
            "name": "Wireless Headphones",
            "description": "High-quality wireless headphones...",
            "price": 299.99,
            "category_id": 1,
            "image_url": "https://images.unsplash.com/...",
            "id": 1,
            "created_at": "2025-01-01T00:00:00",
            "category": {
                "name": "Electronics",
                "slug": "electronics",
                "id": 1
            }
        }
    ],
    "total": 13
}
```

---

### GET /api/products/{id} — Получить товар по ID

| Параметр  | Значение                              |
|-----------|--------------------------------------|
| **URL**   | `{{base_url}}/api/products/1`        |
| **Метод** | `GET`                                |

**Ожидаемый ответ (200 OK):**
```json
{
    "name": "Wireless Headphones",
    "description": "High-quality wireless headphones with noise cancellation...",
    "price": 299.99,
    "category_id": 1,
    "image_url": "https://images.unsplash.com/photo-1551028719-00167b16eac5?w=400",
    "id": 1,
    "created_at": "2025-01-01T00:00:00",
    "category": {
        "name": "Electronics",
        "slug": "electronics",
        "id": 1
    }
}
```

**Тест — несуществующий товар:**

URL: `{{base_url}}/api/products/999`

**Ожидаемый ответ (404 Not Found):**
```json
{
    "detail": "Product with id 999 not found"
}
```

---

### GET /api/products/category/{category_id} — Товары по категории

> ⚠️ **Важно:** Из-за порядка роутов в FastAPI эндпоинт `/category/{id}` может конфликтовать с `/{product_id}`. Если получаете неожиданный ответ — проверьте порядок объявления роутов в `products.py`.

| Параметр  | Значение                                          |
|-----------|--------------------------------------------------|
| **URL**   | `{{base_url}}/api/products/category/1`           |
| **Метод** | `GET`                                            |

**Ожидаемый ответ (200 OK):**
```json
{
    "products": [
        {
            "name": "Wireless Headphones",
            "price": 299.99,
            "category_id": 1,
            ...
        }
    ],
    "total": 5
}
```

**Тест — несуществующая категория:**

URL: `{{base_url}}/api/products/category/999`

**Ожидаемый ответ (404 Not Found):**
```json
{
    "detail": "Category with id 999 not found"
}
```

---

## Cart

> **Принцип работы корзины:** сервер не хранит состояние корзины. Клиент передаёт текущее содержимое (`cart`) при каждом запросе, получает обновлённое состояние и сохраняет его на своей стороне.

### POST /api/cart/add — Добавить товар в корзину

| Параметр      | Значение                          |
|---------------|----------------------------------|
| **URL**       | `{{base_url}}/api/cart/add`      |
| **Метод**     | `POST`                           |
| **Headers**   | `Content-Type: application/json` |

**Body (raw JSON) — добавить первый товар:**
```json
{
    "product_id": 1,
    "quantity": 2,
    "cart": {}
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "cart": {
        "1": 2
    }
}
```

**Body — добавить ещё один товар в уже непустую корзину:**
```json
{
    "product_id": 3,
    "quantity": 1,
    "cart": {"1": 2}
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "cart": {
        "1": 2,
        "3": 1
    }
}
```

**Body — добавить товар, которого нет в базе:**
```json
{
    "product_id": 999,
    "quantity": 1,
    "cart": {}
}
```

**Ожидаемый ответ (404 Not Found):**
```json
{
    "detail": "Product with id 999 not found"
}
```

---

### POST /api/cart — Получить детали корзины

| Параметр      | Значение                          |
|---------------|----------------------------------|
| **URL**       | `{{base_url}}/api/cart`          |
| **Метод**     | `POST`                           |
| **Headers**   | `Content-Type: application/json` |

**Body (raw JSON):**
```json
{
    "1": 2,
    "3": 1
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "items": [
        {
            "product_id": 1,
            "name": "Wireless Headphones",
            "price": 299.99,
            "quantity": 2,
            "subtotal": 599.98,
            "image_url": "https://images.unsplash.com/..."
        },
        {
            "product_id": 3,
            "name": "Laptop Stand",
            "price": 49.99,
            "quantity": 1,
            "subtotal": 49.99,
            "image_url": "https://images.unsplash.com/..."
        }
    ],
    "total": 650,
    "items_count": 3
}
```

**Body — пустая корзина:**
```json
{}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "items": [],
    "total": 0.0,
    "items_count": 0
}
```

---

### PUT /api/cart/update — Обновить количество товара

| Параметр      | Значение                              |
|---------------|--------------------------------------|
| **URL**       | `{{base_url}}/api/cart/update`       |
| **Метод**     | `PUT`                                |
| **Headers**   | `Content-Type: application/json`     |

**Body (raw JSON):**
```json
{
    "product_id": 1,
    "quantity": 5,
    "cart": {"1": 2, "3": 1}
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "cart": {
        "1": 5,
        "3": 1
    }
}
```

**Тест — товар не в корзине:**
```json
{
    "product_id": 99,
    "quantity": 1,
    "cart": {"1": 2}
}
```

**Ожидаемый ответ (404 Not Found):**
```json
{
    "detail": "Product with id 99 not found in cart"
}
```

---

### DELETE /api/cart/remove/{product_id} — Удалить товар из корзины

| Параметр      | Значение                                     |
|---------------|---------------------------------------------|
| **URL**       | `{{base_url}}/api/cart/remove/1`            |
| **Метод**     | `DELETE`                                    |
| **Headers**   | `Content-Type: application/json`            |

**Body (raw JSON):**
```json
{
    "cart": {"1": 2, "3": 1}
}
```

**Ожидаемый ответ (200 OK):**
```json
{
    "cart": {
        "3": 1
    }
}
```

**Тест — удалить товар, которого нет в корзине:**

URL: `{{base_url}}/api/cart/remove/99`
```json
{
    "cart": {"1": 2}
}
```

**Ожидаемый ответ (404 Not Found):**
```json
{
    "detail": "Product with id 99 not found in cart"
}
```

---

## Коды ответов

| Код  | Описание                                         |
|------|--------------------------------------------------|
| `200` | Успешный запрос                                 |
| `404` | Ресурс не найден (товар, категория, итем корзины) |
| `422` | Ошибка валидации — неверный формат данных       |
| `500` | Внутренняя ошибка сервера                       |

---

## Тестовые сценарии

### Сценарий 1: Полный цикл покупки

Выполните запросы в следующем порядке:

1. `GET /api/categories` — получить список категорий
2. `GET /api/products/category/1` — просмотреть товары категории "Electronics"
3. `GET /api/products/1` — открыть карточку товара
4. `POST /api/cart/add` — добавить товар `id=1`, количество `2`, `cart: {}`
5. `POST /api/cart/add` — добавить товар `id=2`, количество `1`, `cart: {"1": 2}`
6. `POST /api/cart` — получить детали корзины с body `{"1": 2, "2": 1}`
7. `PUT /api/cart/update` — изменить количество товара `id=1` на `3`
8. `DELETE /api/cart/remove/2` — удалить товар `id=2`
9. `POST /api/cart` — финальная проверка корзины

---

### Сценарий 2: Проверка граничных случаев

| Шаг | Запрос                                    | Ожидаемый результат |
|-----|------------------------------------------|---------------------|
| 1   | `GET /api/products/0`                   | 404                 |
| 2   | `GET /api/products/category/0`          | 404                 |
| 3   | `POST /api/cart/add` с `quantity: 0`    | 422 (validation)    |
| 4   | `POST /api/cart/add` с `quantity: -1`   | 422 (validation)    |
| 5   | `POST /api/cart/add` с несуществующим product_id | 404          |
| 6   | `PUT /api/cart/update` с product_id не в корзине | 404          |

---

### Сценарий 3: Добавление одного товара несколько раз

Проверяет, что количество суммируется:

1. `POST /api/cart/add`: `product_id=1, quantity=2, cart={}`  
   → Результат: `{"1": 2}`

2. `POST /api/cart/add`: `product_id=1, quantity=3, cart={"1": 2}`  
   → Результат: `{"1": 5}` (количество суммируется)

---

## Советы по работе с Postman

- Сохраняйте актуальное состояние корзины в **Postman Variable** (`cart_state`) и подставляйте его через `{{cart_state}}` в Body запросов.
- Используйте вкладку **Tests** для автоматической проверки кодов ответа:
  ```javascript
  pm.test("Status 200", () => pm.response.to.have.status(200));
  pm.test("Has products", () => {
      const json = pm.response.json();
      pm.expect(json.products).to.be.an('array');
  });
  ```
- Для Cart-запросов удобно использовать **Pre-request Script**, чтобы автоматически подставлять сохранённую корзину из переменных окружения.