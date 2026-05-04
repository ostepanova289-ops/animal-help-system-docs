Основные сущности системы

|**Сущность**|**Ключевые атрибуты**|**Акторы**|**Характер данных**|
|-|-|-|-|
|**Пользователь (User)**|user_id
role (приют / куратор / волонтёр / помощник / потенциальный владелец / админ)
full_name
email
phone
password_hash
status (active / blocked / pending_verification)
created_at  
avatar_url
last_login_at
consent_pd 
city
profile_description
organization_id (nullable)|Все роли, кроме приютов|Транзакционные|
|Приют ( Shelter)|organization_id
name
city
address
contact_phone
description
created_at
status|Приют, Администратор|Транзакционные|
|**Животное (Animal)**|animal_id
organization_id (nullable, если животное у частного куратора)
user_id
name
species (собака / кошка / др.)
breed
sex
age
description
health_status
sterilized_flag
vaccinated_flag
current_status (на лечении / ищет дом / пристроен / архив)
adoption_ready_flag
created_at
updated_at
description|Приют, Куратор, Администратор|Транзакционные + Аналитические|
|**Медиафайл животного (**AnimalМediaFile)|media_id
animal_id
uploaded_by
file_url
file_type
mime_type
size_bytes
sort_order
created_at|Приют, куратор, админ||
|**Заявка на помощь (HelpRequest)**|help_request_id
animal_id
created_by
request_type (лечение / корм / перевозка / передержка / др.)
title
description
priority
required_by_date
location
status (новая / в работе / закрыта / отменена / просрочена)
created_at
closed_at|Приют, Куратор, Волонтер, помощник, Администратор|Транзакционные + аналитические|
|**Отклик на заявку (HelpResponse)**|response_id
help_request_id
volunteer_id
response_comment
response_status (откликнулся / назначен / отказался / выполнил)
assigned_at
completed_at
completion_comment
proof_media_url — при необходимости|Волонтёр, Куратор, Приют, Админ|Транзакционные + аналитические|
|**Заявка на пристройство (AdoptionRequest)**|adoption_application_id
animal_id
applicant_user_id
message
contact_phone
housing_type
has_other_animals_flag
application_status (новая / на рассмотрении / одобрена / отклонена / завершена)
created_at
reviewed_at
review_comment|Потенциальный владелец, Куратор, Приют, Админ|Транзакционные + аналитические|
|**Пожертвование (Donation)**|donation_id
donor_user_id (nullable, если гостевой сценарий допустим)
animal_id (nullable)
help_request_id (nullable)
amount
currency
payment_provider
provider_payment_id
payment_status (pending / succeeded / failed / refunded)
created_at
paid_at
receipt_url|Помощник, Бухгалтер, Администратор|Транзакционные + аналитические|
|**История изменений (ChangeLog)**|audit_event_id
entity_type (animal / help_request / adoption_application / donation / user)
entity_id
action_type (create / update / status_change / assign / close / moderate)
changed_by_user_id
old_value_json
new_value_json
created_at|Система (автоматически), Админ|Аналитические|

 

## **Выбор технологии хранения**

|Шаг|Пользователь|Животное|Заявка на помощь|Отклик на заявку|Заявка на пристройство|Пожертвование|История изменений|Медиафайлы|
|-|-|-|-|-|-|-|-|-|
|**Сколько данных?**|1k–100k|10k–100k|50k–300k|100k–500k|10k–100k|10k–200k|500k–5M|100 GB – 10 TB|
|**Паттерн**|OLTP, чтение по ID, фильтры по роли|OLTP, CRUD, фильтры по статусу|OLTP, CRUD, статусы, списки|OLTP, создание и смена статуса|OLTP, workflow|OLTP, фиксация платежа|только записи - append, чтение по entity/date|write-once|
|**Консистентность**|strong|strong|strong|strong|strong|strong|eventual допустима|eventual допустима|
|**Доступность**|99.9%|99.9%|99.9%|99.9%|99.9%|99.9%|99.5%|99.9%|
|**Мутабельность схемы**|жесткая|жесткая|жесткая|жесткая|жесткая|жесткая|гибкая|гибкая|
|**транзакции**|Да (регистрация, смена пароля)|Да (обновление статуса + история)|Да (смена статуса + отклик)|Да (атомарно с заявкой)|да|Да(платёжные операции)|Нет|нет|
|**Итоговое решение**|реляционная БД|реляционная БД|реляционная БД|реляционная БД|реляционная БД|реляционная БД + платежный шлюз|реляционная БД на MVP, далее архив|object storage|

