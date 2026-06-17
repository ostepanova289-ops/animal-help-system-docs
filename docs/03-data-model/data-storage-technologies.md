# Определение сущностей и технологий хранения данных

## 1. Основные сущности системы

### User — пользователь системы

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| user_id | BIGINT | PK | Нет | Уникальный идентификатор пользователя |
| last_name | VARCHAR(100) | — | Нет | Фамилия пользователя |
| first_name | VARCHAR(100) | — | Нет | Имя пользователя |
| middle_name | VARCHAR(100) | — | Да | Отчество пользователя, если указано |
| email | VARCHAR(255) | UNIQUE | Нет | Email |
| phone | VARCHAR(20) | UNIQUE | Да | Телефон пользователя |
| password_hash | VARCHAR(255) | — | Нет | Хэш пароля |
| status | ENUM(active, blocked, pending_verification) | — | Нет | Статус аккаунта пользователя |
| city | VARCHAR(100) | — | Да | Город пользователя |
| avatar_url | VARCHAR(500) | — | Да | Ссылка на аватар |
| profile_description | TEXT | — | Да | Описание пользователя, например опыт помощи животным |
| consent_pd | BOOLEAN | — | Нет | Согласие на обработку персональных данных |
| created_at | TIMESTAMP | — | Нет | Дата создания аккаунта |
| last_login_at | TIMESTAMP | — | Да | Дата последнего входа |

### Role — роль пользователя

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| role_id | SMALLINT | PK | Нет | Уникальный идентификатор роли |
| code | VARCHAR(50) | UNIQUE | Нет | Код роли |
| name | VARCHAR(100) | — | Нет | Название роли |

Таблица Role является справочником ролей пользователей. В ней хранятся значения admin, curator, volunteer, helper, adopter, shelter. Связь пользователя с ролями реализуется через таблицу UserRole, так как один пользователь может иметь несколько ролей.

### UserRole — связь пользователя и роли

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| user_id | BIGINT | PK, FK → User.user_id | Нет | Пользователь |
| role_id | SMALLINT | PK, FK → Role.role_id | Нет | Роль пользователя |

### Shelter — приют

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| shelter_id | BIGINT | PK | Нет | Уникальный идентификатор приюта |
| name | VARCHAR(255) | — | Нет | Название приюта |
| city | VARCHAR(100) | — | Нет | Город |
| address | VARCHAR(500) | — | Да | Адрес приюта |
| contact_phone | VARCHAR(20) | — | Да | Контактный телефон |
| description | TEXT | — | Да | Описание приюта |
| status | ENUM(active, blocked, archived) | — | Нет | Статус |
| created_at | TIMESTAMP | — | Нет | Дата добавления приюта |

### Animal — животное

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| animal_id | BIGINT | PK | Нет | Уникальный идентификатор животного |
| shelter_id | BIGINT | FK → Shelter.shelter_id | Да | Приют, к которому относится животное. Nullable, если животное у частного куратора |
| curator_user_id | BIGINT | FK → User.user_id | Да | Пользователь, ответственный за животное |
| name | VARCHAR(100) | — | Нет | Кличка животного |
| species | ENUM(dog, cat, other) | — | Нет | Вид животного |
| breed | VARCHAR(100) | — | Да | Порода, если известна |
| sex | ENUM(male, female, unknown) | — | Нет | Пол животного |
| age | SMALLINT | — | Да | Возраст животного |
| description | TEXT | — | Да | Описание животного |
| health_status | TEXT | — | Да | Информация о здоровье |
| sterilized_flag | BOOLEAN | — | Нет | Признак стерилизации |
| vaccinated_flag | BOOLEAN | — | Нет | Признак вакцинации |
| current_status | ENUM(treatment, looking_for_home, adopted, archived) | — | Нет | Текущий статус животного |
| adoption_ready_flag | BOOLEAN | — | Нет | Готово ли животное к пристройству |
| created_at | TIMESTAMP | — | Нет | Дата добавления |
| updated_at | TIMESTAMP | — | Нет | Дата |

Для животного должно быть заполнено хотя бы одно из полей: shelter_id или curator_user_id.

### AnimalMediaFile — медиафайл животного

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| media_id | BIGINT | PK | Нет | Уникальный идентификатор медиафайла |
| animal_id | BIGINT | FK → Animal.animal_id | Нет | Животное, к которому относится файл |
| uploaded_by_user_id | BIGINT | FK → User.user_id | Нет | Пользователь, загрузивший файл |
| file_url | VARCHAR(500) | — | Нет | Ссылка на файл |
| file_type | ENUM(photo, video) | — | Нет | Тип файла |
| mime_type | VARCHAR(100) | — | Нет | Технический формат файла |
| size_bytes | BIGINT | — | Нет | Размер файла в байтах |
| sort_order | SMALLINT | — | Да | Порядок отображения файлов в карточке животного |
| created_at | TIMESTAMP | — | Нет | Дата загрузки файла |

### HelpRequest — заявка на помощь

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| help_request_id | BIGINT | PK | Нет | Уникальный идентификатор заявки |
| animal_id | BIGINT | FK → Animal.animal_id | Нет | Животное, для которого создана заявка |
| created_by_user_id | BIGINT | FK → User.user_id | Нет | Пользователь, создавший заявку |
| request_type | ENUM(treatment, food, transport, temporary_home, other) | — | Нет | Тип требуемой помощи |
| title | VARCHAR(255) | — | Нет | Краткое название заявки |
| description | TEXT | — | Нет | Подробное описание ситуации |
| priority | ENUM(low, medium, high, urgent) | — | Нет | Приоритет заявки |
| required_by_date | DATE | — | Да | Дата, до которой желательно оказать помощь |
| location | VARCHAR(255) | — | Да | Место оказания помощи |
| status | ENUM(new, in_progress, closed, canceled, overdue) | — | Нет | Статус заявки |
| created_at | TIMESTAMP | — | Нет | Дата создания заявки |
| closed_at | TIMESTAMP | — | Да | Дата закрытия заявки |

### HelpResponse — отклик на заявку

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| response_id | BIGINT | PK | Нет | Уникальный идентификатор отклика |
| help_request_id | BIGINT | FK → HelpRequest.help_request_id | Нет | Заявка, на которую оставлен отклик |
| volunteer_user_id | BIGINT | FK → User.user_id | Нет | Волонтер, который откликнулся на заявку |
| response_comment | TEXT | — | Да | Комментарий волонтера |
| response_status | ENUM(responded, assigned, declined, completed) | — | Нет | Статус отклика |
| assigned_at | TIMESTAMP | — | Да | Дата назначения волонтера исполнителем |
| completed_at | TIMESTAMP | — | Да | Дата выполнения помощи |
| completion_comment | TEXT | — | Да | Комментарий по итогам выполнения |
| proof_media_url | VARCHAR(500) | — | Да | Ссылка на подтверждающий файл, если нужен |

### AdoptionRequest — заявка на пристройство

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| adoption_request_id | BIGINT | PK | Нет | Уникальный идентификатор заявки на пристройство |
| animal_id | BIGINT | FK → Animal.animal_id | Нет | Животное, на которое подана заявка |
| applicant_user_id | BIGINT | FK → User.user_id | Нет | Пользователь, который хочет забрать животное |
| message | TEXT | — | Да | Сообщение заявителя |
| contact_phone | VARCHAR(20) | — | Да | Контактный телефон для связи |
| housing_type | ENUM(apartment, house, other) | — | Да | Тип жилья заявителя |
| has_other_animals_flag | BOOLEAN | — | Нет | Есть ли у заявителя другие животные |
| status | ENUM(new, reviewing, approved, rejected, completed) | — | Нет | Статус заявки |
| created_at | TIMESTAMP | — | Нет | Дата создания заявки |
| reviewed_at | TIMESTAMP | — | Да | Дата рассмотрения заявки |
| review_comment | TEXT | — | Да | Комментарий по итогам рассмотрения |

### Donation — пожертвование

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| donation_id | BIGINT | PK | Нет | Уникальный идентификатор пожертвования |
| donor_user_id | BIGINT | FK → User.user_id | Да | Пользователь, совершивший пожертвование. Nullable, если разрешены гостевые пожертвования |
| animal_id | BIGINT | FK → Animal.animal_id | Да | Животное, на которое направлено пожертвование |
| help_request_id | BIGINT | FK → HelpRequest.help_request_id | Да | Заявка на помощь, к которой относится пожертвование |
| amount | DECIMAL(10,2) | — | Нет | Сумма пожертвования |
| currency | ENUM(RUB) | — | Нет | Валюта пожертвования |
| payment_provider | ENUM(yookassa, cloudpayments, manual) | — | Нет | Платежный провайдер или ручное внесение |
| provider_payment_id | VARCHAR(255) | UNIQUE | Да | Идентификатор платежа у провайдера |
| payment_status | ENUM(pending, succeeded, failed, canceled) | — | Нет | Статус платежа |
| created_at | TIMESTAMP | — | Нет | Дата создания платежа |
| paid_at | TIMESTAMP | — | Да | Дата успешной оплаты |
| receipt_url | VARCHAR(500) | — | Да | Ссылка на чек или квитанцию |

### ChangeLog — история изменений

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| audit_event_id | BIGINT | PK | Нет | Уникальный идентификатор события аудита |
| entity_type | ENUM(animal, help_request, adoption_request, donation, user) | — | Нет | Тип измененной сущности |
| entity_id | BIGINT | — | Нет | Идентификатор измененной записи |
| action_type | ENUM(create, update, status_change, assign, close, moderate) | — | Нет | Тип действия |
| changed_by_user_id | BIGINT | FK → User.user_id | Да | Пользователь, выполнивший изменение. Nullable, если действие выполнено системой автоматически |
| old_value_json | JSON | — | Да | Состояние данных до изменения |
| new_value_json | JSON | — | Да | Состояние данных после изменения |
| created_at | TIMESTAMP | — | Нет | Дата и время изменения |

### HelpRequestAnalytics — аналитика по заявкам на помощь

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| analytics_id | BIGINT | PK | Нет | Уникальный идентификатор аналитической записи |
| period_date | DATE | — | Нет | Дата или начало периода, за который рассчитана аналитика |
| request_type | ENUM(treatment, food, transport, temporary_home, other) | — | Да | Тип заявки, если аналитика считается в разрезе типа помощи |
| created_count | INTEGER | — | Нет | Количество созданных заявок за период |
| closed_count | INTEGER | — | Нет | Количество закрытых заявок за период |
| overdue_count | INTEGER | — | Нет | Количество просроченных заявок за период |
| avg_close_time_hours | DECIMAL(10,2) | — | Да | Среднее время закрытия заявки в часах |

### DonationAnalytics — аналитика по пожертвованиям

| Поле | Тип данных | Ключ | Nullable | Назначение |
| - | - | - | - | - |
| analytics_id | BIGINT | PK | Нет | Уникальный идентификатор аналитической записи |
| period_date | DATE | — | Нет | Дата или начало периода, за который рассчитана аналитика |
| total_amount | DECIMAL(12,2) | — | Нет | Общая сумма успешных пожертвований за период |
| donation_count | INTEGER | — | Нет | Количество успешных пожертвований за период |
| avg_donation_amount | DECIMAL(10,2) | — | Да | Средний размер пожертвования |
| unique_donors_count | INTEGER | — | Да | Количество уникальных доноров |

## Характер взаимодействия с данными

| Сущность | Характер данных |
| - | - |
| User | Транзакционные |
| Role | Транзакционные |
| UserRole | Транзакционные |
| Shelter | Транзакционные |
| Animal | Транзакционные + аналитические |
| AnimalMediaFile | Транзакционные |
| HelpRequest | Транзакционные + аналитические |
| HelpResponse | Транзакционные + аналитические |
| AdoptionRequest | Транзакционные + аналитические |
| Donation | Транзакционные + аналитические |
| ChangeLog | Аналитические |
| HelpRequestAnalytics | Аналитические |
| DonationAnalytics | Аналитические |

## Выбор технологии хранения

| Шаг | Пользователь | Животное | Заявка на помощь | Отклик на заявку | Заявка на пристройство |
| - | - | - | - | - | - |
| Сколько данных? | 1k–100k | 10k–100k | 50k–300k | 100k–500k | 10k–100k |
| Паттерн | OLTP, чтение по ID, фильтры по роли | OLTP, CRUD, фильтры по статусу | OLTP, CRUD, статусы, списки | OLTP, создание и смена статуса | OLTP, workflow |
| Консистентность | strong | strong | strong | strong | strong |
| Доступность | 99.9% | 99.9% | 99.9% | 99.9% | 99.9% |
| Мутабельность схемы | жесткая | жесткая | жесткая | жесткая | жесткая |
| транзакции | Да (регистрация, смена пароля) | Да (обновление статуса + история) | Да (смена статуса + отклик) | Да (атомарно с заявкой) | реляционная БД |
| Итоговое решение | реляционная БД | реляционная БД | реляционная БД | реляционная БД | реляционная БД |
