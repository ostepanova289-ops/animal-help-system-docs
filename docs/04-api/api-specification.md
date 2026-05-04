```Plain Text
openapi: 3.0.3

info:
  title: Animal Help UI API
  version: 1.0.0
  description: API для веб-интерфейса системы координации помощи бездомным животным.

servers:
  - url: https://animal-help.com

paths:
  /api/animals:
    get:
      summary: Получить список карточек животных
      operationId: getAnimals
      parameters:
        - name: search
          in: query
          required: false
          description: Поиск по кличке
          schema:
            type: string
        - name: species
          in: query
          required: false
          description: Фильтр по виду животного
          schema:
            $ref: '#/components/schemas/AnimalSpecies'
        - name: city
          in: query
          required: false
          description: Фильтр по городу
          schema:
            type: string
      responses:
        '200':
          description: Список карточек животных
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AnimalListResponse'

    post:
      summary: Создать карточку животного
      operationId: createAnimal
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateAnimalRequest'
      responses:
        '201':
          description: Карточка животного создана
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AnimalResponse'

  /api/animals/{animalId}:
    get:
      summary: Получить карточку животного
      operationId: getAnimalById
      parameters:
        - $ref: '#/components/parameters/AnimalId'
      responses:
        '200':
          description: Карточка животного
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AnimalResponse'
        '404':
          description: Карточка не найдена

    patch:
      summary: Изменить карточку животного
      operationId: updateAnimal
      parameters:
        - $ref: '#/components/parameters/AnimalId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateAnimalRequest'
      responses:
        '200':
          description: Карточка животного обновлена
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AnimalResponse'
        '404':
          description: Карточка не найдена

    delete:
      summary: Удалить карточку животного
      operationId: deleteAnimal
      parameters:
        - $ref: '#/components/parameters/AnimalId'
      responses:
        '204':
          description: Карточка удалена
        '404':
          description: Карточка не найдена

  /api/animals/{animalId}/status:
    patch:
      summary: Изменить статус животного
      operationId: updateAnimalStatus
      parameters:
        - $ref: '#/components/parameters/AnimalId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateAnimalStatusRequest'
      responses:
        '200':
          description: Статус животного обновлён
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AnimalResponse'
        '404':
          description: Карточка не найдена

  /api/animals/{animalId}/photos:
    post:
      summary: Загрузить фотографию животного
      operationId: uploadAnimalPhoto
      parameters:
        - $ref: '#/components/parameters/AnimalId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateAnimalPhotoRequest'
      responses:
        '201':
          description: Фотография загружена
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AnimalPhotoResponse'
        '404':
          description: Карточка не найдена

  /api/animals/{animalId}/photos/{photoId}:
    delete:
      summary: Удалить фотографию животного
      operationId: deleteAnimalPhoto
      parameters:
        - $ref: '#/components/parameters/AnimalId'
        - $ref: '#/components/parameters/PhotoId'
      responses:
        '204':
          description: Фотография удалена
        '404':
          description: Карточка или фотография не найдены

  /api/requests:
    get:
      summary: Получить список заявок на помощь
      operationId: getRequests
      parameters:
        - name: animal
          in: query
          required: false
          description: Поиск по животному
          schema:
            type: string
        - name: helpType
          in: query
          required: false
          description: Фильтр по типу помощи
          schema:
            $ref: '#/components/schemas/HelpType'
        - name: status
          in: query
          required: false
          description: Фильтр по статусу заявки
          schema:
            $ref: '#/components/schemas/RequestStatus'
        - name: city
          in: query
          required: false
          description: Фильтр по городу
          schema:
            type: string
      responses:
        '200':
          description: Список заявок на помощь
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RequestListResponse'

    post:
      summary: Создать заявку на помощь
      operationId: createRequest
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateRequestRequest'
      responses:
        '201':
          description: Заявка создана
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RequestResponse'

  /api/requests/{requestId}:
    get:
      summary: Получить карточку заявки
      operationId: getRequestById
      parameters:
        - $ref: '#/components/parameters/RequestId'
      responses:
        '200':
          description: Карточка заявки
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RequestResponse'
        '404':
          description: Заявка не найдена

    patch:
      summary: Изменить заявку
      operationId: updateRequest
      parameters:
        - $ref: '#/components/parameters/RequestId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateRequestRequest'
      responses:
        '200':
          description: Заявка обновлена
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RequestResponse'
        '404':
          description: Заявка не найдена

    delete:
      summary: Удалить заявку
      operationId: deleteRequest
      parameters:
        - $ref: '#/components/parameters/RequestId'
      responses:
        '204':
          description: Заявка удалена
        '404':
          description: Заявка не найдена

  /api/requests/{requestId}/status:
    patch:
      summary: Изменить статус заявки
      operationId: updateRequestStatus
      parameters:
        - $ref: '#/components/parameters/RequestId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateRequestStatusRequest'
      responses:
        '200':
          description: Статус заявки обновлён
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RequestResponse'
        '404':
          description: Заявка не найдена

components:
  parameters:
    AnimalId:
      name: animalId
      in: path
      required: true
      schema:
        type: string
        example: ANIMAL-001

    RequestId:
      name: requestId
      in: path
      required: true
      schema:
        type: string
        example: REQUEST-001

    PhotoId:
      name: photoId
      in: path
      required: true
      schema:
        type: string
        example: PHOTO-001

  schemas:
    AnimalSpecies:
      type: string
      enum:
        - cat
        - dog
        - other

    AnimalSex:
      type: string
      enum:
        - male
        - female

    AnimalStatus:
      type: string
      enum:
        - treatment
        - looking_for_home
        - adopted

    HelpType:
      type: string
      enum:
        - treatment
        - food
        - transport
        - foster_care
        - financial_help
        - other

    RequestUrgency:
      type: string
      enum:
        - urgent
        - normal

    RequestStatus:
      type: string
      enum:
        - open
        - in_progress
        - closed

    AnimalPhoto:
      type: object
      required:
        - id
        - url
      properties:
        id:
          type: string
          example: PHOTO-001
        url:
          type: string
          format: uri
          example: https://example.com/photos/photo-001.jpg

    HealthInfo:
      type: object
      required:
        - vaccinated
        - sterilized
      properties:
        vaccinated:
          type: boolean
          example: true
        sterilized:
          type: boolean
          example: false
        diseases:
          type: string
          nullable: true
          example: Травма лапы

    Animal:
      type: object
      required:
        - id
        - name
        - species
        - sex
        - age
        - status
        - city
        - healthInfo
        - photos
      properties:
        id:
          type: string
          example: ANIMAL-001
        name:
          type: string
          nullable: true
          example: Барсик
        withoutName:
          type: boolean
          example: false
        species:
          $ref: '#/components/schemas/AnimalSpecies'
        sex:
          $ref: '#/components/schemas/AnimalSex'
        age:
          type: string
          example: 2 года
        status:
          $ref: '#/components/schemas/AnimalStatus'
        description:
          type: string
          example: Дружелюбный кот
        city:
          type: string
          example: Москва
        healthInfo:
          $ref: '#/components/schemas/HealthInfo'
        photos:
          type: array
          items:
            $ref: '#/components/schemas/AnimalPhoto'

    CreateAnimalRequest:
      type: object
      required:
        - species
        - sex
        - age
        - status
        - city
      properties:
        name:
          type: string
          nullable: true
          example: Барсик
        withoutName:
          type: boolean
          example: false
        species:
          $ref: '#/components/schemas/AnimalSpecies'
        sex:
          $ref: '#/components/schemas/AnimalSex'
        age:
          type: string
          example: 2 года
        status:
          $ref: '#/components/schemas/AnimalStatus'
        description:
          type: string
          example: Дружелюбный кот
        city:
          type: string
          example: Москва
        healthInfo:
          $ref: '#/components/schemas/HealthInfo'

    UpdateAnimalRequest:
      type: object
      properties:
        name:
          type: string
          nullable: true
        withoutName:
          type: boolean
        species:
          $ref: '#/components/schemas/AnimalSpecies'
        sex:
          $ref: '#/components/schemas/AnimalSex'
        age:
          type: string
        description:
          type: string
        city:
          type: string
        healthInfo:
          $ref: '#/components/schemas/HealthInfo'

    UpdateAnimalStatusRequest:
      type: object
      required:
        - status
      properties:
        status:
          $ref: '#/components/schemas/AnimalStatus'

    CreateAnimalPhotoRequest:
      type: object
      required:
        - url
      properties:
        url:
          type: string
          format: uri
          example: https://example.com/uploads/photo.jpg

    AnimalPhotoResponse:
      type: object
      required:
        - data
      properties:
        data:
          $ref: '#/components/schemas/AnimalPhoto'

    AnimalResponse:
      type: object
      required:
        - data
      properties:
        data:
          $ref: '#/components/schemas/Animal'

    AnimalListResponse:
      type: object
      required:
        - items
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/Animal'

    Fundraising:
      type: object
      required:
        - needed
      properties:
        needed:
          type: boolean
          example: true
        purpose:
          type: string
          nullable: true
          example: Лечение
        targetAmount:
          type: number
          format: float
          nullable: true
          example: 10000
        description:
          type: string
          nullable: true
          example: Сбор на оплату лечения
        currentAmount:
          type: number
          format: float
          readOnly: true
          example: 0

    RequestAnimalRef:
      type: object
      required:
        - animalId
        - animalName
      properties:
        animalId:
          type: string
          example: ANIMAL-001
        animalName:
          type: string
          example: Барсик

    HelpRequest:
      type: object
      required:
        - id
        - animal
        - helpType
        - urgency
        - status
        - city
      properties:
        id:
          type: string
          example: REQUEST-001
        animal:
          $ref: '#/components/schemas/RequestAnimalRef'
        helpType:
          $ref: '#/components/schemas/HelpType'
        urgency:
          $ref: '#/components/schemas/RequestUrgency'
        status:
          $ref: '#/components/schemas/RequestStatus'
        city:
          type: string
          example: Москва
        description:
          type: string
          example: Нужна помощь на лечение
        fundraising:
          $ref: '#/components/schemas/Fundraising'

    CreateRequestRequest:
      type: object
      required:
        - animalId
        - helpType
        - urgency
        - city
      properties:
        animalId:
          type: string
          example: ANIMAL-001
        helpType:
          $ref: '#/components/schemas/HelpType'
        urgency:
          $ref: '#/components/schemas/RequestUrgency'
        city:
          type: string
          example: Москва
        description:
          type: string
          example: Нужна помощь на лечение
        fundraising:
          $ref: '#/components/schemas/Fundraising'

    UpdateRequestRequest:
      type: object
      properties:
        animalId:
          type: string
        helpType:
          $ref: '#/components/schemas/HelpType'
        urgency:
          $ref: '#/components/schemas/RequestUrgency'
        city:
          type: string
        description:
          type: string
        fundraising:
          $ref: '#/components/schemas/Fundraising'

    UpdateRequestStatusRequest:
      type: object
      required:
        - status
      properties:
        status:
          $ref: '#/components/schemas/RequestStatus'

    RequestResponse:
      type: object
      required:
        - data
      properties:
        data:
          $ref: '#/components/schemas/HelpRequest'

    RequestListResponse:
      type: object
      required:
        - items
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/HelpRequest'
```

