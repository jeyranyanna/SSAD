# Лабораторная работа №4
**Тема:** Проектирование REST API

**Цель работы:** Получить опыт проектирования программного интерфейса.

## Выбранный сервис
**Подсистема работы с моделями** используется для создания и редактирования моделей, а также осуществления их трансформаций вида "Модель-Текст".

## Документация по API
### Проектные решения 
#### 1. RESTful-архитектура
- **Принятое решение**: Использование REST-подхода с ресурсно-ориентированными URL.
- **Детали**:
  - URL вида: `/api/{ресурс}/{id}`
  - Пример:  
    ```
    GET /api/v1/models/model_123
    ```
#### 2. Структура идентификаторов ресурсов
- **Принятое решение**: Идентификаторы моделей генерируются в формате `model_{число}`.
- **Детали**:
  - При создании модели ей автоматически присваивается ID вида `model_1`, `model_2` и так далее.
  - Это гарантирует уникальность идентификаторов внутри сервиса.
  - Пример:  
    ```json
    {
      "id": "model_3"
    }
    ```
    
#### 3. Версионирование API
- **Принятое решение**: Версия API указывается в URL.
- **Детали**:
  - Все эндпоинты начинаются с `/api/v1/`
  - Пример:  
    ```
    POST /api/v1/models
    ```

#### 4. Использование HTTP-методов
| Метод  | Назначение                             | Пример использования              |
|--------|---------------------------------------|-----------------------------------|
| GET    | Получение данных                      | `GET /api/v1/models/model_123`    |
| POST   | Создание сущностей                    | `POST /api/v1/models`          |
| PUT    | Полное обновление сущности            | `PUT /api/v1/models/model_123`          |
| DELETE | Удаление сущности                     | `DELETE /api/v1/models/model_123`       |

#### 5. Стандартизация HTTP-статус-кодов
- **Принятое решение**: Четкое соответствие HTTP-статус-кодов результатам операций в соответствии с REST-практиками.
  
| Код  | Назначение                               | 
|--------|---------------------------------------|
| 200    | Успешное выполнение GET, PUT, DELETE или валидации     | 
| 201    | Успешное создание ресурса                     | 
| 400    | Некорректный запрос (ошибки валидации, неверный формат данных)         | 
| 401    | Отсутствие или невалидность аутентификационных данных                     | 
| 404    | Ресурс не найден                  | 
| 500    | Внутренняя ошибка сервера (универсальный код для необработанных исключений)       | 

#### 6. Формат данных
- **Принятое решение**: JSON для всех запросов и ответов.
- **Пример запроса**:
  ```json
  {
    "name": "Sales Dashboard",
    "type": "graph",
    "data_source": "database",
    "filters": [],
    "elements": {}
  }
  ```

#### 7. Обработка ошибок 
- **Принятое решение**: Стандартизированные HTTP-коды + тело с описанием.
- **Пример ошибки**:
  ```json
  {
    "error": "Model not found"
  }
  ```

#### 8. Аутентификация
- **Принятое решение**: JWT-токен в заголовке ``Authorization``.
- **Пример запроса с токеном**:
  ```http
  GET /api/v1/models/123 HTTP/1.1
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  ```




### Эндпоинты API 
#### 0. Получение токена
**Метод: POST /api/v1/token**

##### Параметры запроса: 
- **Тело (application/json):**
  ```json
  {
    "username": "name",
    "password": "pass"
  }
  ```

#### Пример успешного ответа (200 OK):
```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTc0MTA3NTg2NiwianRpIjoiNDYyNzJiMjktYzAzNS00YzhmLWFiYWYtZjllMTg2MzQ3MTM1IiwidHlwZSI6ImFjY2VzcyIsInN1YiI6ImFkbWluIiwibmJmIjoxNzQxMDc1ODY2LCJjc3JmIjoiMTdjMjhkMDktNTNjZC00MGJiLWFjNmYtNGMxNjdmZjIwNWVhIiwiZXhwIjoxNzQxMDc2NzY2fQ.QBB8fhQi1Cj0FeERspQxyRu_S8sepmiBbA3x98onU1o"
}
```

#### Пример ошибки (401 Invalid credentials):
```json
{
  "error": "InvalidCredentials"
}
```

#### 1. Создание новой модели диаграммы 
**Метод: POST /api/v1/models**

Модуль: Модуль создания визуальных моделей 

##### Параметры запроса: 
- **Тело (application/json):**
  ```json
  {
    "name": "Диаграмма продаж по регионам",
    "type": "bubble_chart",
    "data_source": {
      "type": "csv",
      "uri": "s3://data/sales-2023.csv"
    },
    "filters": [
      {
        "field": "region",
        "values": ["Север", "Юг"]
      }
    ],
    "elements": {
      "x_axis": {"field": "revenue", "title": "Выручка (млн руб.)"},
      "y_axis": {"field": "profit", "title": "Прибыль"},
      "bubbles": {
        "size_field": "volume",
        "color_field": "category"
      }
    }
  }
  ```

#### Пример успешного ответа (201 Created):
```json
{
  "id": "model_123",
}
```

#### Пример ошибки (400 Bad Request):
```json
{
  "error": "InvalidAxisConfig"
}
```

#### 2. Получение полной конфигурации ранее созданной модели 
**Метод: GET /api/v1/models/{model_id}**

Модуль: Редактор визуальных моделей 

##### Параметры запроса: 
- model_id (path, string) - идентификатор модели (например, bubble_123)
  
#### Пример успешного ответа (200 OK):
```json
{
  "id": "bubble_123",
  "name": "Диаграмма продаж",
  "type": "bubble_chart",
  "created_at": "2023-10-15T12:00:00Z",
  "updated_at": "2023-10-15T12:00:00Z",
  "elements": {
    "x_axis": {
      "field": "revenue",
      "title": "Выручка (млн руб.)",
      "scale": "linear"
    },
    "bubbles": {
      "size_field": "volume"
    }
  },
  "data_source": {
    "type": "xlsx"
  }
}
```

#### Пример ошибки (404 Not Found):
```json
{
  "error": "Model not found"
}
```

#### 3. Обновление параметров модели 
**Метод: PUT /api/v1/models/{model_id}**

Модуль: Редактор визуальных моделей 
##### Параметры запроса: 
- **Тело (application/json):**
```json
{
  "filters": [
    {
      "field": "region",
      "values": ["Север", "Юг", "Запад"]
    }
  ],
  "elements": {
    "x_axis": {"title": "Годовая выручка"}
  }
}
```
  
#### Пример успешного ответа (200 OK):
```json
{
  "id": "bubble_123",
  "name": "Диаграмма продаж",
  "type": "bubble_chart",
  "created_at": "2023-10-15T12:00:00Z",
  "updated_at": "2023-10-15T12:00:00Z",
  "filters": [
    {
      "field": "region",
      "values": ["Север", "Юг", "Запад"]
    }
  ],
  "elements": {
    "x_axis": {
      "field": "revenue",
      "title": "Годовая выручка",
      "scale": "linear"
    },
    "bubbles": {
      "size_field": "volume"
    }
  },
  "data_source": {
    "type": "xlsx"
  }
}
```

#### Пример ошибки (404 Not Found):
```json
{
  "error": "Model not found"
}
```

#### 4. Удаление модели и связанных данных
**Метод: DELETE /api/v1/models/{model_id}**

Модуль: Редактор визуальных моделей 
##### Параметры запроса: 
- model_id (path, string) - идентификатор модели (например, bubble_123)
  
#### Пример успешного ответа (200 OK):
```json
{
    "message": "Model deleted"
}
```

#### Пример ошибки (404 Not Found):
```json
{
  "error": "Model not found"
}
```

#### 5. Проверка корректности конифгурации модели
**Метод: POST /api/v1/validate**

Модуль: Модуль валидации моделей 
##### Параметры запроса: 
- **Тело (application/json):**
```json
{
  "model_id": "bubble_123"
}
```
  
#### Пример успешного ответа (200 OK) для валидной модели:
```json
{
  "status": "valid",
  "errors": []
}
```

#### Пример успешного ответа (200 OK) для невалидной модели:
```json
{
  "status": "invalid",
  "errors": ["Missing x_axis in content"]
}
```

#### Пример ошибки (404 Not Found)):
```json
{
  "error": "Model not found"
}
```

#### 6. Генерация кода визуализации
**Метод: POST /api/v1/transform**

Модуль: Модуль трансформации моделей 
##### Параметры запроса: 
- **Тело (application/json):**
```json
{
  "model_id": "bubble_123",
  "target": {
    "language": "python",
    "libraries": ["matplotlib"]
  }
}
```
  
#### Пример успешного ответа (200 OK):
```json
{
  "code": [
    {
      "filename": "visualization.py",
      "content": "import pandas as pd\nimport matplotlib.pyplot as plt\n..."
    }
  ]
}
```

#### Пример ошибки (404 Not Found)):
```json
{
  "error": "Model not found"
}
```

#### 7. Экспорт документации по модели 
**Метод: GET /api/v1/artifacts/{model_id}**

Модуль: Модуль генерации текстовых артефактов 
##### Параметры запроса: 
- format (query, enum): markdown (default), html

#### Пример успешного ответа (200 OK):
```markdown
# Пузырьковая диаграмма продаж

**Type:** bubble_chart
```

#### Пример ошибки (400 Bad Request):
```json
{
  "error": "Unsupported format"
}
```

#### Пример ошибки (404 Not Found)):
```json
{
  "error": "Model not found"
}
```

## Реализация API
Пример упрощенного кода реализации API:

```python
from flask import Flask, jsonify, request, make_response
from flask_jwt_extended import JWTManager, jwt_required, create_access_token
from datetime import datetime
import json

app = Flask(__name__)
app.config["JWT_SECRET_KEY"] = "super-secret-key"
jwt = JWTManager(app)

# Хранение моделей в памяти
models = {}
current_id = 1

# Заглушка пользователя для аутентификации
USERS = {"admin": "admin"}

# Пример данных для трансформации
TEMPLATES = {
    "python": {
        "matplotlib": "import matplotlib.pyplot as plt\nplt.bar({x}, {y})\nplt.show()",
        "seaborn": "import seaborn as sns\nsns.barplot(x={x}, y={y})\nplt.show()"
    }
}

@app.route("/api/v1/token", methods=["POST"])
def login():
    username = request.json.get("username")
    password = request.json.get("password")
    
    if USERS.get(username) == password:
        access_token = create_access_token(identity=username)
        return jsonify(access_token=access_token)
    
    return jsonify({"error": "Invalid credentials"}), 401

@app.route("/api/v1/models", methods=["POST"])
@jwt_required()
def create_model():
    global current_id
    data = request.get_json()
    
    required_fields = ["name", "type", "data_source", "filters", "elements"]
    if not all(field in data for field in required_fields):
        return jsonify({"error": "Missing required fields"}), 400
    
    model_id = f"model_{current_id}"
    models[model_id] = {
        "id": model_id,
        **data,
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat()
    }
    
    current_id += 1
    return jsonify({"id": model_id}), 201

@app.route("/api/v1/models/<model_id>", methods=["GET"])
@jwt_required()
def get_model(model_id):
    model = models.get(model_id)
    if not model:
        return jsonify({"error": "Model not found"}), 404
    
    return jsonify(model)

@app.route("/api/v1/models/<model_id>", methods=["PUT"])
@jwt_required()
def update_model(model_id):
    model = models.get(model_id)
    if not model:
        return jsonify({"error": "Model not found"}), 404
    
    data = request.get_json()
    model.update({
        **data,
        "updated_at": datetime.now().isoformat()
    })
    
    return jsonify(model)

@app.route("/api/v1/models/<model_id>", methods=["DELETE"])
@jwt_required()
def delete_model(model_id):
    if model_id not in models:
        return jsonify({"error": "Model not found"}), 404
    
    del models[model_id]
    return jsonify({"message": "Model deleted"}), 200

@app.route("/api/v1/validate", methods=["POST"])
@jwt_required()
def validate_model():
    data = request.get_json()
    model_id = data.get("model_id")
    
    if model_id not in models:
        return jsonify({"error": "Model not found"}), 404
    
    # Пример валидации: проверка наличия осей
    model = models[model_id]
    errors = []
    if "x_axis" not in model["elements"]:
        errors.append("Missing x_axis in content")
    if "y_axis" not in model["elements"]:
        errors.append("Missing y_axis in content")
    return jsonify({
        "status": "valid" if not errors else "invalid",
        "errors": errors
    })

@app.route("/api/v1/transform", methods=["POST"])
@jwt_required()
def transform_model():
    data = request.get_json()
    model_id = data.get("model_id")
    target = data.get("target", {})
    
    if model_id not in models:
        return jsonify({"error": "Model not found"}), 404
    
    model = models[model_id]
    language = target.get("language", "python")
    library = target.get("libraries", ["matplotlib"])[0]
    
    # Пример генерации кода
    code = TEMPLATES[language][library].format(
        x=model["elements"].get("x_axis", "x"),
        y=model["elements"].get("y_axis", "y")
    )
    
    return jsonify({
        "code": [{
            "filename": f"visualization.{language}",
            "content": code
        }]
    })

@app.route("/api/v1/artifacts/<model_id>", methods=["GET"])
@jwt_required()
def get_artifacts(model_id):
    format = request.args.get("format", "markdown")
    
    if model_id not in models:
        return jsonify({"error": "Model not found"}), 404
    
    model = models[model_id]
    content = f"# {model['name']}\n\n**Type:** {model['type']}\n\n"
    
    # Генерация документации
    if format == "markdown":
        response = make_response(content)
        response.headers["Content-Type"] = "text/markdown"
    elif format == "html":
        html_content = f"<h1>{model['name']}</h1><p>Type: {model['type']}</p>"
        response = make_response(html_content)
        response.headers["Content-Type"] = "text/html"
    else:
        return jsonify({"error": "Unsupported format"}), 400
    
    return response

if __name__ == "__main__":
    app.run(debug=True)
```


## Тестирование API
1. Локально запущен Flask API:
   ![image](https://github.com/user-attachments/assets/2b2a3cca-35d9-4afc-87b5-630e70019c16)

2. API добавлен в Postman:
   - Создан environment `Local`
   - Создана коллекция `Model Subsystem`
3. Получен JWT-токен (все запросы, кроме `/api/v1/token`, требуют JWT-токен в заголовке `Authorization`). Токен добавлен в `variables` в среде `local`.
   ![image](https://github.com/user-attachments/assets/ebe754fe-e2a2-4b7d-b558-6a2009c3bba6)

4. Добавлены запросы для каждого эндпойнта:
   
   ![image](https://github.com/user-attachments/assets/b57d91e7-a768-4172-bf81-23f3bc9c2c75)


### Далее представлены результаты тестирования: 
#### 0. Получение токена
- **Тестируемое API**: /api/v1/auth/token
- **Метод**: POST
- **Строка запроса**:
  ```
  POST http://127.0.0.1:5000/api/v1/token
  ```
- **Тело запроса**:
  ```json
  {
    "username": "admin",
    "password": "admin"
  }
  ```
- **Заголовки и параметры**:
  - Authorization: нет
  - Content-type: application/json
- **Скрин заголовков и параметров**:
  ![Image](https://github.com/user-attachments/assets/76c99857-1c4b-4760-9254-9cb68ca0ea64)
- **Скрин полученного ответа**:
  - Ответ (Body):
    ![Image](https://github.com/user-attachments/assets/14db6a51-f39f-4fcf-bdce-584866a06c19)
  - Ответ (Headers):
    ![image](https://github.com/user-attachments/assets/1427197e-de76-40ee-b271-14c58f5ac798)
- **Код автотестов**:
  ```javascript
  pm.test("Токен успешно получен", function () {
    pm.response.to.have.status(200);
    var responseBody = pm.response.json();
    pm.expect(responseBody).to.have.property('access_token');
  });
  ```
- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/4f6796b9-bfca-4a66-8a0c-86542cc12355)

*Примечание: при дальнейшем тестировании запросов, при истечении срока дейтсвия токена, отображалась ошибка 401:*   
![image](https://github.com/user-attachments/assets/b7f9a510-3f31-47ad-99fa-7a1e1df6fc78)

#### 1. Создание модели
#### Тест №1 (успешный)
- **Тестируемое API**: /api/v1/models
- **Метод**: POST
- **Строка запроса**:
  ```
  POST http://127.0.0.1:5000/api/v1/models
  ```
- **Тело запроса**:
  ```json
  {
  "name": "Диаграмма продаж по регионам",
  "type": "bubble_chart",
  "data_source": {
    "type": "csv",
    "uri": "s3://data/sales-2023.csv"
  },
  "filters": [
    {
      "field": "region",
      "values": ["Север", "Юг"]
    }
  ],
  "elements": {
    "x_axis": {"field": "revenue", "title": "Выручка (млн руб.)"},
    "y_axis": {"field": "profit", "title": "Прибыль"},
    "bubbles": {
      "size_field": "volume",
      "color_field": "category"}}
  }
  ```
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
  ![image](https://github.com/user-attachments/assets/d71b39a6-074e-40de-af86-77ec7f485ff0)

- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/117ac6ff-3e3f-4c45-9b66-980b02352db7)

  - Ответ (Headers):
    
    ![image](https://github.com/user-attachments/assets/803dac20-d3ab-41f7-a32c-9ad2a417af01)

- **Код автотестов**:
  ```javascript
  // Успешный тест
  pm.test("Статус код 201 Created", function () {
      pm.response.to.have.status(201);
  });
  
  pm.test("Ответ содержит ID модели", function () {
      var jsonData = pm.response.json();
      pm.expect(jsonData).to.have.property("id");
  });
  ``` 
- **Скрин результатов тестирования**
  ![image](https://github.com/user-attachments/assets/02663e10-6adc-4a76-8b81-2fea43f30e9a)

#### Тест №2 (неуспешный)
- **Тестируемое API**: /api/v1/models
- **Метод**: POST
- **Строка запроса**:
  ```
  POST http://127.0.0.1:5000/api/v1/models
  ```
- **Тело запроса**:
  ```json
  {
  "name": "Диаграмма продаж по регионам",
  "type": "bubble_chart",
  "data_source": {
    "type": "csv",
    "uri": "s3://data/sales-2023.csv"}
  }
  ```
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
  
  ![image](https://github.com/user-attachments/assets/88ff7717-3095-4885-b877-80e98cb331de)

- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/51331276-91a7-4963-8927-f654df54649a)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/32a85cd1-5b9c-4f10-9488-73910c180162)


- **Код автотестов**:
  ```javascript
  pm.test("Статус код 400 Bad Request", function () {
      pm.response.to.have.status(400);
  });
  
  pm.test("Ответ содержит сообщение об ошибке", function () {
      var jsonData = pm.response.json();
      pm.expect(jsonData).to.have.property("error");
  });
  ``` 
- **Скрин результатов тестирования**

  ![image](https://github.com/user-attachments/assets/000a5dc7-90b1-47e3-b7c3-3feec897b9c4)



#### 2. Получение полной конфигурации модели
#### Тест №1 (Успешный)
- **Тестируемое API**: /api/v1/models/<model_id>
- **Метод**: GET
- **Строка запроса**:
  ```
  http://127.0.0.1:5000/api/v1/models/model_1
  ```
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
  ![image](https://github.com/user-attachments/assets/dc2269c5-9bb2-408c-a005-37df2212f673)

- **Скрин полученного ответа**:
  - Ответ (Body):
    
    ![image](https://github.com/user-attachments/assets/c0e6771f-65c4-4629-b86e-3a10d5063f1c)

  - Ответ (Headers):
    
    ![image](https://github.com/user-attachments/assets/45f9d1f6-ce2a-4ed9-a0d6-9197d19e860d)

- **Код автотестов**:
  ```javascript
  pm.test("Модель успешно получена", function () {
    pm.response.to.have.status(200);
    pm.response.to.have.jsonBody("id");
  });
  ``` 
- **Скрин результатов тестирования**
  ![image](https://github.com/user-attachments/assets/113c9ea9-e03f-4d0c-a083-a32bab94e208)


#### Тест №2 (Неуспешный)
- **Тестируемое API**: /api/v1/models/<model_id>
- **Метод**: GET
- **Строка запроса**:
  ```
  http://127.0.0.1:5000/api/v1/models/model_100
  ```
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
![image](https://github.com/user-attachments/assets/caaa2380-a546-4bff-8dab-89ae2d116c68)


- **Скрин полученного ответа**:
  - Ответ (Body):
    
  ![image](https://github.com/user-attachments/assets/6a313dcb-daa4-46b1-af2d-efc37f230028)

  - Ответ (Headers):
    
  ![image](https://github.com/user-attachments/assets/632eafa6-7ddf-41ed-9dec-90609899df12)

- **Код автотестов**:
  ```javascript
  pm.test("Модель успешно получена", function () {
    pm.response.to.have.status(200);
    pm.response.to.have.jsonBody("id");
  });
  ``` 
- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/533221e9-d326-427c-ad6c-2b77d25e6c06)


#### 3. Редактирование модели
#### Тест №1 (Успешный)
- **Тестируемое API**: /api/v1/models/<model_id>
- **Метод**: PUT
- **Строка запроса**:
  ```
  PUT http://127.0.0.1:5000/api/v1/models/model_1
  ```
- **Тело запроса**:
  ```json
  {
  "filters": [
    {
      "field": "region",
      "values": ["Север", "Юг", "Запад"]
    }
  ],
  "elements": {
    "x_axis": {"title": "Годовая выручка"}
  }
  }
  ```
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
![image](https://github.com/user-attachments/assets/589cac31-570b-4701-abda-19698ac3c803)


- **Скрин полученного ответа**:
  - Ответ (Body):
    
    ![image](https://github.com/user-attachments/assets/894d357c-5c76-4d18-af6f-e9e6788f67cc)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/81ce5663-808e-4a41-8425-36577be83395)


- **Код автотестов**:
  ```javascript
  pm.test("Статус код 200 OK", function () {
      pm.response.to.have.status(200);
  });
  
  pm.test("Ответ содержит обновленные данные", function () {
      var jsonData = pm.response.json();
      pm.expect(jsonData).to.have.property("id");
      pm.expect(jsonData).to.have.property("updated_at");
      pm.expect(jsonData.filters).to.be.an("array");
      pm.expect(jsonData.filters[0].values).to.include("Запад");
      pm.expect(jsonData.elements.x_axis.title).to.equal("Годовая выручка");
  });
  ``` 
- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/81fd6a30-9d85-4105-b79e-45d466e37862)



#### Тест №2 (Неуспешный)
Отличие от теста №1 в строке запроса 
  ```
  PUT http://127.0.0.1:5000/api/v1/models/model_100
  ```
- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/108cf1f4-956a-4317-a1b7-559d944c5217)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/be30f5f9-4e14-41d2-9057-502e5b66caef)

- **Код автотестов**:
  ```javascript
  pm.test("Статус код 404 Not Found", function () {
      pm.response.to.have.status(404);
  });
  ``` 
- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/6449c26d-bfce-4bde-9422-68c92429c90a)

#### 4. Удаление модели
#### Тест №1 (Успешный)
- **Тестируемое API**: /api/v1/models/<model_id>
- **Метод**: DELETE
- **Строка запроса**:
  ```
  DELETE http://127.0.0.1:5000/api/v1/models/model_3
  ```
  
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
  ![image](https://github.com/user-attachments/assets/277b4814-2c5a-4c82-b33b-71d89eb02b63)

- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/1498e52d-bbe3-49d9-bb20-38166cba19d3)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/86db948f-e199-47a7-8870-ca86710c8a17)

- **Код автотестов**:
  ```javascript
  pm.test("Модель успешно удалена", function () {
    pm.response.to.have.status(200);
    pm.response.to.have.jsonBody("message");
  });
  ```
  
- **Скрин результатов тестирования**
  ![image](https://github.com/user-attachments/assets/47ad04ca-d4c2-4b3c-a5bd-781e40ed36cc)

#### Тест №2 (Неуспешный)
Отличие от теста №1 в строке запроса 
  ```
  http://127.0.0.1:5000/api/v1/models/model_30
  ```
- **Скрин полученного ответа**:
  - Ответ (Body):
  
  ![image](https://github.com/user-attachments/assets/b27421f5-b97f-4860-ac5c-b064283f221e)

  - Ответ (Headers):
  
  ![image](https://github.com/user-attachments/assets/c1cc973b-a704-4225-a06b-91f6dd2300ac)


- **Код автотестов**:
  ```javascript
  pm.test("Модель успешно удалена", function () {
      pm.response.to.have.status(200);
      pm.response.to.have.jsonBody("message");
  });
  ``` 
- **Скрин результатов тестирования**
  ![image](https://github.com/user-attachments/assets/55302ca9-e0d2-4726-8318-54314ac985a7)


#### 5. Валидация модели
#### Тест №1 (Успешный)
- **Тестируемое API**: /api/v1/validate
- **Метод**: POST
- **Строка запроса**:
  ```
  POST http://127.0.0.1:5000/api/v1/validate
  ```
- **Тело запроса**:
  ```json
  {
    "model_id": "model_2"
  }
  ```
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
![image](https://github.com/user-attachments/assets/c1038df4-a56f-4ea8-b2e7-0c13b4f15abb)


- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/e81d5fa6-efad-43fa-8031-c7df60b1c668)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/d7ead2e4-57e0-4130-ad63-549edb4f92f5)



- **Код автотестов**:
  ```javascript
  pm.test("Модель валидна", function () {
      pm.response.to.have.status(200);
      pm.response.to.have.jsonBody("status", "valid");
  });
  ``` 
- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/abcda2ba-dbf4-4157-b638-e92e9f4dc983)



#### Тест №2 (Неуспешный)
Отличие от теста №1 в теле запроса 
 ```json
 {
    "model_id": "model_1"
 }
  ```
- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/87edeaf9-9957-482b-9db3-ed54aeaf3eb7)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/de37ddc5-530b-44e7-866b-630b5762cbfc)

- **Код автотестов**:
  ```javascript
  pm.test("Модель валидна", function () {
      pm.response.to.have.status(200);
      pm.response.to.have.jsonBody("status", "valid");
  });
  
  pm.test("Модель невалидна", function () {
      pm.response.to.have.status(200);
      pm.response.to.have.jsonBody("status", "invalid");
  });
  ``` 
- **Скрин результатов тестирования**
  ![image](https://github.com/user-attachments/assets/fc7c8e3b-c147-4e4f-b581-74c71b7d3620)
 
#### 6. Трансформация модели
- **Тестируемое API**: /api/v1/transform
- **Метод**: POST
- **Строка запроса**:
  ```
  POST http://127.0.0.1:5000/api/v1/transform
  ```
- **Тело запроса**:
  ```json
  {
    "model_id": "model_1",
    "target": {
        "language": "python",
        "libraries": ["seaborn"]
    }
  }
  ```
- **Заголовки и параметры**:
  - Authorization: Bearer <access_token>
  - Content-type: application/json

- **Скрин заголовков и параметров**:
![image](https://github.com/user-attachments/assets/9d54b6bc-09b1-45eb-bb79-f61956f8f369)

- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/b5c9b24c-ebad-4aac-a1ac-00274ca84abf)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/fb05a4b7-c3a8-470a-a1ad-40466458219c)

- **Код автотестов**:
  ```javascript
  pm.test("Код трансформации создан", function () {
      pm.response.to.have.status(200);
      pm.response.to.have.jsonBody("code");
  });
  ``` 
- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/d506f203-0c82-4b7c-9295-1f9d5b83b0b1)

#### 7. Генерация текстовых артефактов
#### Тест №1 (Успешный)
- **Тестируемое API**: /api/v1/artifacts/<model_id>
- **Метод**: GET
- **Строка запроса**:
  ```
  GET http://127.0.0.1:5000/api/v1/artifacts/model_1?format=markdown 
  ```
- **Заголовки и параметры**:
  - format: markdown
  - Authorization: Bearer <access_token>
  - Content-type: application/json
- **Скрин заголовков и параметров**:
![image](https://github.com/user-attachments/assets/7e2ebdbd-8589-4383-9314-3f34653a31f7)
![image](https://github.com/user-attachments/assets/a878f35d-5528-411e-bf4e-9030b1e720e7)
![image](https://github.com/user-attachments/assets/d0975d88-25a9-4f0d-9e9a-39dae832a521)


- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/2f3a3edb-83e2-45f0-8c88-0393919d2aa6)


  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/bde435d3-9975-4c6b-a7b8-782fb598f707)


- **Код автотестов**:
  ```javascript
  pm.test("Response status is 200", function () {
      pm.response.to.have.status(200);
  });
  
  pm.test("Content-Type is markdown", function () {
      pm.response.to.have.header("Content-Type", "text/markdown");
  });
  
  pm.test("Content-Type is html", function () {
      pm.response.to.have.header("Content-Type", "text/html");
  });
  ``` 
- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/e2ffeef7-731e-4a02-b942-fb1299b56c96)



#### Тест №2 (Неуспешный)
Отличие от теста №1 в параметре format 

![image](https://github.com/user-attachments/assets/d05a2bc2-590c-45e6-ab67-d5e7433d5f11)

- **Скрин полученного ответа**:
  - Ответ (Body):

    ![image](https://github.com/user-attachments/assets/ff7c4d34-c400-4f20-8166-2d65f08aabd1)

  - Ответ (Headers):

    ![image](https://github.com/user-attachments/assets/674896e0-d390-4fcf-9744-b7c5a2cfeff0)

- **Скрин результатов тестирования**
![image](https://github.com/user-attachments/assets/b170be3d-f7a9-440c-ae27-7fa4004d56d6)
