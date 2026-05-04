## Выбранный сценарий

**Обработка результата пожертвования после оплаты через внешний платёжный шлюз.**

Суть сценария такая. Пользователь на сайте делает пожертвование и его переводят в платёжный сервис. Система сама не хранит данные карты, поэтому вся оплата проходит через внешний шлюз. В этот момент нельзя сразу понять, прошёл платёж или нет. Итог приходит позже от платёжного сервиса и уже тогда система обновляет статус пожертвования.

Это асинхронное взаимодействие, потому что платёж обрабатывает внешняя система и мы не контролируем, когда она вернёт результат. Пользователь может закрыть страницу или потерять соединение, а платёж может подтвердиться с задержкой или даже прийти повторно. Поэтому нельзя полагаться на один запрос от клиента.

В итоге процесс делится на две части. Сначала пользователь создаёт пожертвование и получает ответ, что платёж начат. Потом платёжный сервис отправляет результат, и система отдельно обрабатывает это событие и меняет статус на успешный или неуспешный.

**Выбор технологии**

Для этого сценария я выбрала RabbitMQ. Это подходит лучше всего, потому что у нас не поток огромного количества событий, а конкретная бизнес-задача: получить результат оплаты и передать его дальше в систему. RabbitMQ удобен для таких очередей, где важно не потерять сообщение и при необходимости обработать его повторно.

С точки зрения платформы у нас обычное веб-приложение с внешним платёжным шлюзом. Пользователь делает пожертвование через сайт, а потом платёжный сервис отдельно присылает результат. Для такой архитектуры не нужен WebSocket, потому что здесь нет постоянного обмена с фронтом в реальном времени. Kafka тоже будет избыточной, потому что она больше подходит для очень больших потоков событий и более сложной инфраструктуры. gRPC со стримами здесь тоже не нужен, так как для MVP это слишком сложное решение. RabbitMQ в этом случае проще, понятнее и логичнее.

С точки зрения стека такой вариант тоже выглядит нормально. У нас веб-система с REST API и интеграцией со сторонним сервисом оплаты. RabbitMQ можно поставить между сервисом, который принимает callback от платёжного шлюза, и сервисом пожертвований, который меняет статус платежа. Это позволит не держать всю логику в одном синхронном запросе и не завязывать систему на мгновенный ответ от внешнего провайдера.

С точки зрения характера взаимодействия RabbitMQ подходит потому, что результат оплаты может прийти позже, с задержкой или даже повторно. Значит нам нужен не мгновенный ответ, а надёжная доставка события и возможность обработать его отдельно. 

По формату контракта здесь логично использовать AsyncAPI, потому что обмен можно описать в JSON.

```Plain Text
asyncapi: 3.0.0
info:
  title: Donation Payment Result API
  version: 1.0.0
  description: Асинхронная обработка результата пожертвования после оплаты через внешний платёжный шлюз

defaultContentType: application/json

servers:
  rabbitmq:
    host: rabbitmq:5672
    protocol: amqp
    description: RabbitMQ broker

channels:
  donationPaymentResult:
    address: donation.payment.result
    messages:
      paymentResult:
        $ref: '#/components/messages/PaymentResultMessage'

operations:
  sendPaymentResult:
    action: send
    channel:
      $ref: '#/channels/donationPaymentResult'
    messages:
      - $ref: '#/channels/donationPaymentResult/messages/paymentResult'

  receivePaymentResult:
    action: receive
    channel:
      $ref: '#/channels/donationPaymentResult'
    messages:
      - $ref: '#/channels/donationPaymentResult/messages/paymentResult'

components:
  messages:
    PaymentResultMessage:
      name: PaymentResultMessage
      title: Payment result
      summary: Результат оплаты пожертвования
      payload:
        $ref: '#/components/schemas/PaymentResult'

  schemas:
    PaymentResult:
      type: object
      additionalProperties: false
      required:
        - eventId
        - donationId
        - paymentId
        - status
        - amount
        - currency
      properties:
        eventId:
          type: string
          description: Уникальный идентификатор события
          example: EVT-001

        donationId:
          type: string
          description: Идентификатор пожертвования в системе
          example: DON-001

        paymentId:
          type: string
          description: Идентификатор платежа во внешнем платёжном сервисе
          example: PAY-123456

        status:
          type: string
          description: Статус оплаты
          enum:
            - succeeded
            - failed

        amount:
          type: number
          format: double
          description: Сумма пожертвования
          example: 1000.0

        currency:
          type: string
          description: Валюта платежа
          example: RUB

        failureReason:
          type: string
          nullable: true
          description: Причина ошибки, если платёж неуспешен
          example: insufficient_funds
```

