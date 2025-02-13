# Проектирование продажи ОСАГО

В соответствии с требованиями бизнеса и принятыми решениями, можно разработать следующую схему реализации `osago-aggregator` и интеграцию между сервисами.

1. `osago-aggregator` - требует отдельного хранилища данных, так как в случае возникновения каких либо проблем с внешними API, можно пробовать отправлять повторно запросы, а также хранить историю заявок
2. API для `core-app`:
   - `POST /create-osago-request` - отправка заявки на ОСАГО в страховые компании
   - `GET /get-osago-proposals/{requestId}` - получение предложений по заявке с идентификатором `requestId`.
3. средство интеграции между `core-app` и `osago-aggregator` - будем использовать асинхронные неблокирующие запросы через `Kafka`, так как при пике нагрузки, сможем по чуть чуть обрабатывать запросы, в фоновом режиме
4. паттерны отказоустойчивости `osago-aggregator`:
   - `Rate Limiting` - будем контролировать обьем запросов к внешним сервисам в соответствии с договоренностями
   - `Circuit Breaker` - дополнительная защита от сбоев в API страховых компаний. При превышении порога ошибок, запросы будут перенаправлены на запасной вариант или обработчик ошибок
   - `Retry` - в случае получения ошибки при отправке заявки или получении предложений, `osago-aggregator` будет повторять запросы с задержкой, до достижения максимума попыток
   - `Timeout` - установка максимального времени ожидания ответа от страховых компаний (60 секунд) для предотвращения зависания приложения.
