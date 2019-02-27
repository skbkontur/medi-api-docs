```yaml
swagger: "2.0"
info:
  description: "Medi Api v2.0"
  version: "2.0"
  title: "Medi"
host: "medi"
basePath: "/api/V2"
tags:
- name: "Parties"
  description: "Организации"
- name: "Events"
  description: "События в ящике"
- name: "Messages"
  description: "Сообщения"
schemes:
- "https"
paths:
  /parties:
    get:
      tags: 
      - "Parties"
      summary: "Получение списка организаций, доступных пользователю"
      produces:
      - "application/json"
      responses:
        200:
          description: "Успешное выполнение"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/PartyInfo"
        401:
          description: "Не авторизован"
        403:
          description: "Нет доступа ни к одной организации"
  /parties/{medId}:
    get:
      tags:
      - "Parties"
      summary: "Информация об организации"
      produces:
      - "application/json"
      parameters:
      - name: "medId"
        in: "path"
        description: "MediId организации"
        required: true
        type: "string"
      responses:
        200:
          description: "Успешное выполнение"
          schema:
            $ref: "#/definitions/PartyInfo"
        401:
          description: "Не авторизован"
        403:
          description: "Нет доступа к указанной организации, либо нет доступа ни к одной организации"
        404:
          description: "Организация с указанным MediId не существует"
  /events:
    get:
      tags:
      - "Events"
      summary: "Получение cписка событий в ящике организации"
      produces:
      - "application/json"
      parameters:
      - name: "mediId"
        in: "query"
        description: "MediId"
        required: true
        type: "string"
      - name: "lastEventPointer"
        in: "query"
        description: "Идентификатор последнего полученного события"
        type: "string"
      - name: "count"
        in: "query"
        default: 100
        description: "Количество запрашиваемых событий в диапазоне от 1 до 1000. Сервер вернет count событий или меньше. Если не передано, то по умолчанию равно 100."
        type: integer
      responses:
        200:
          description: "Успешное выполнение"
          schema:
            $ref: "#/definitions/BoxEventsBatch"
        401:
          description: "Не авторизован"
        403:
          description: "Нет доступа к организации"
        400:
          description: "Count лежит вне доступного диапазона либо lastEventPointer невалиден"
  /messages/{messageId}:
    get:
      tags: 
      - "Messages"
      summary: "Получение сообщения по идентификатору"
      produces:
      - "application/json"
      parameters:
      - name: "messageId"
        in: "path"
        description: "Идентификатор сообщения"
        required: true
        type: "string"
      responses:
        200:
          description: "Успешное выполнение"
          schema:
            $ref: "#/definitions/Message"
        400:
          description: "messageId имеет неверный формат"
        401:
          description: "Не авторизован"
        403:
          description: "Нет доступа ни к одной организации"
        404:
          description: "Сообщение не найдено"
  /messages:
    post:
      tags:
      - "Messages"
      summary: "Отправка сообщения"
      produces:
      - "application/json"
      parameters:
      - in: body
        name: message
        description: Сообщение для отправки.
        schema:
          $ref: "#/definitions/MessageForSendData"
      responses:
        200:
          description: "Успешное выполнение"
          schema:
            $ref: "#/definitions/BasicMessageMeta"
        401:
          description: "Не авторизован"
        403:
          description: "Нет доступа ни к одной организации"
definitions:
  PartyInfo:
    type: "object"
    properties:
      mediId:
        type: "string"
      name: 
        type: "string"
      inn:
        type: "string"
      kpp: 
        type: "string"
      partyTypeCode:
        type: "string"
        enum:
        - "clinic"
        - "insurer"
        - "employer"
  BoxEventsBatch:
    type: "object"
    properties:
      events:
        type: "array"
        items:
          $ref: "#/definitions/Event"
      count: 
        type: integer
        description: "Количество событий в наборе"
      lastEventPointer:
        type: string
        description: "Идентификатор последнего события в наборе: при следующем вызове метода 'GET events' необходимо передать именно его."
  Event:
    type: "object"
    properties:
      mediId:
        type: string
      eventId:
        type: string
      eventPointer:
        type: string
      eventDateTime:
        type: string
        format: date-time
      eventType:
        type: string
        enum:
        - "NewOutboxMessage"
        - "NewInboxMessage"
        - "MessageDelivered"
        - "MessageUndelivered"
        - "ProcessingTimesReport"
      eventContent:
        type: object
  BasicMessageMeta:
    type: object
    properties:
      mediId: 
        type: string
      messageId:
        type: string
      documentCirculationId:
        type: string
  FullMessageMeta:
    type: object
    properties:
      mediId: 
        type: string
      messageId:
        type: string
      documentCirculationId:
        type: string
      routingInfo:
        $ref: "#/definitions/RoutingInfo"
      documentDetails:
        $ref: "#/definitions/DocumentDetails"
  RoutingInfo:
    type: object
    properties:
      senderMediId:
        type: string
      recipientMediId:
        type: string
  DocumentDetails:
    type: object
    properties:
      documentType:
        $ref: "#/definitions/DocumentType"
      documentNumber:
        type: string
      documentDate:
        type: string
        format: date-time
  DocumentType:
    type: string
    enum:
    - "Unknown"
    - "Any"
    - "Ihcebi"
    - "Ihcebr"
    - "Ihclme"
    - "Relist"
  Message:
    type: object
    properties:
      meta:
        $ref: "#/definitions/FullMessageMeta"
      data:
        type: object
        properties:
          content:
            type: string
            format: byte
  MessageForSendData:
    type: object
    properties:
      mediId:
        type: string
        description: "mediId отправителя"
      message:
        type: string
        format: byte
  NewOutboxMessageEventContent:
    type: object
    properties:
      basicMessageMeta:
        $ref: "#/definitions/BasicMessageMeta"
  MessageDeliveredEventContent:
    type: object
    properties:
      fullMessageMeta:
        $ref: "#/definitions/FullMessageMeta"
  MessageUndeliveredEventContent:
    type: object
    properties:
      basicMessageMeta:
        $ref: "#/definitions/BasicMessageMeta"
      reasons:
        type: array
        items:
          type: string
  NewInboxMessageEventContent:
    type: object
    properties:
      fullMessageMeta:
        $ref: "#/definitions/FullMessageMeta"
  ProcessingTimesEventContent:
    type: object
    properties:
      documentCirculationStartTimestamp:
        type: string
        format: date-time
      documentCirculationCompletionTimestamp:
        type: string
        format: date-time
      processingTimes:
        $ref: "#/definitions/ProcessingTimes"
  ProcessingTimes:
    type: object
    properties:
      totalWorkTime:
        type: string
        format: date-time
      deliveryTime:
        type: string
        format: date-time
      deliveryNotificationWaitTime:
        type: string
        format: date-time
  EventType:
        type: string
        enum:
        - "NewOutboxMessage"
        - "NewInboxMessage"
        - "MessageDelivered"
        - "MessageUndelivered"
        - "ProcessingTimesReport"
    
  ErrorResponse:
    type: object
    properties:
      error:
        $ref: "#/definitions/ApiError"
      innerErrors:
        type: array
        items:
          $ref: "#/definitions/ApiError"
    
  ApiError:
    type: object
    properties:
      errorCode:
        $ref: "#/definitions/ApiErrorCode"
      message:
        type: string
    
  ApiErrorCode:
    type: string
    enum: [HasNoAccessToParty,
        PartyDoesNotExist,
        ClientHasNoAccessibleParties,

        MediIdDoesNotExist,
        HasNoAccessToMediId,
        MessageNotFound,

        ValidationErrorCode,
        BadEventsCount,
        InvalidLastEventPointer]
```
