# Документация API: АИС «Кафедральный кампус» (Учебный процесс и расписание)

В данном документе приведено описание и исходный код интеграционного контракта OpenAPI 3.0.3 для АИС «Кафедральный кампус» (Вариант 1). Спецификация спроектирована с учетом разделения ответственности (**Separation of Concerns**) между интерфейсом (**UI Layer**) и сетевым клиентом (**Service / API Layer**).

Контракт полностью провалидирован и успешно используется в рамках обеспечения характеристик **Удобства использования / UX (Usability)** и **Сопровождаемости (Maintainability)** системы по стандарту **ISO/IEC 25010**.

---

## 1. Спецификация OpenAPI (YAML)

Этот код можно скопировать и вставить в среду Swagger Editor для генерации интерактивной документации Swagger UI.

```yaml
openapi: 3.0.3
info:
  title: АИС Кафедральный кампус — API Учебного процесса
  version: 1.0.0
  description: >
    API шлюза интеграции для управления учебным процессом и расписанием кафедры. 
    Обеспечивает взаимодействие между UI Layer и Service/API Layer.
servers:
  - url: https://department-campus.university
    description: Тестовый сервер для интеграционного тестирования (Staging)

paths:
  /schedule:
    get:
      summary: Получение расписания учебной группы
      description: Запрос направляется через API Layer для чтения данных из БД кампуса и отрисовки в UI Layer.
      parameters:
        - name: groupCode
          in: query
          required: true
          description: Уникальный код академической группы
          schema:
            type: string
            example: "ИВТ-24-1"
      responses:
        '200':
          description: Успешное получение массива занятий для указанной группы
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Lesson'
        '400':
          description: Неверный формат кода группы
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '500':
          description: Внутренняя ошибка сервера (ошибка интеграции с базой данных)

  /schedule/lesson:
    post:
      summary: Добавление нового занятия в расписание кафедры
      description: Позволяет интерфейсу администратора (UI Layer) отправлять данные формы через Сетевой слой.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LessonCreateInput'
      responses:
        '201':
          description: Занятие успешно создано и добавлено в расписание
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  createdLessonId:
                    type: integer
                    example: 1024
                  message:
                    type: string
                    example: "Занятие успешно добавлено в расписание группы ИВТ-24-1"
        '400':
          description: Ошибка валидации входящих данных
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '409':
          description: Конфликт расписания (аудитория или преподаватель уже заняты)

components:
  schemas:
    Lesson:
      type: object
      required:
        - id
        - timeSlot
        - subject
        - room
        - lecturer
      properties:
        id:
          type: integer
          example: 1024
        timeSlot:
          type: string
          example: "09:00 - 10:30 (1 пара)"
        subject:
          type: string
          example: "Архитектура информационных систем"
        room:
          type: string
          example: "Ауд. 312"
        lecturer:
          type: string
          example: "Иванов И.И."

    LessonCreateInput:
      type: object
      required:
        - groupCode
        - timeSlot
        - subject
        - room
        - lecturer
      properties:
        groupCode:
          type: string
          example: "ИВТ-24-1"
        timeSlot:
          type: string
          example: "09:00 - 10:30 (1 пара)"
        subject:
          type: string
          example: "Архитектура информационных систем"
        room:
          type: string
          example: "Ауд. 312"
        lecturer:
          type: string
          example: "Иванов И.И."

    ErrorResponse:
      type: object
      properties:
        errorCode:
          type: string
          example: "BAD_REQUEST_PARAMS"
        errorMessage:
          type: string
          example: "Параметр groupCode не соответствует валидному шаблону кафедры."
```

---

## 2. Верификация контракта в Swagger UI

Для подтверждения успешной валидации и визуализации спроектированных методов ниже представлены скриншоты работы со спецификацией.

### 2.1 Валидация спецификации в Swagger Editor
Вставка скриншота общей рабочей области Swagger Editor. Слева находится код спецификации без ошибок синтаксиса, справа генерируется интерактивная веб-документация.


<img width="632" height="778" alt="image" src="https://github.com/user-attachments/assets/07f76831-1941-4e60-ae2a-2a8876d7992c" />

Рисунок 1 - Валидация контракта в интерактивном редакторе Swagger Editor.

### 2.2 Структура взаимосвязанных эндпоинтов
Вставка скриншота с развернутыми методами `GET /schedule` и `POST /schedule/lesson`, на котором видны примеры передаваемых параметров и JSON-структуры моделей (`Lesson`, `LessonCreateInput`).

<img width="876" height="775" alt="image" src="https://github.com/user-attachments/assets/93194aad-3dd6-4a0e-81ce-ef9009aa19e1" />

Рисунок 2 - Интерактивное описание эндпоинтов чтения и создания элементов расписания.
