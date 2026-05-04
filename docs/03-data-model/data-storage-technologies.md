Основные сущности системы

|**Сущность**|**Ключевые атрибуты**|**Акторы**|**Характер данных**|
|-|-|-|-|
|**Пользователь (User)**|user_id<br>role (приют / куратор / волонтёр / помощник / потенциальный владелец / админ)<br>full_name<br>email<br>phone<br>password_hash<br>status (active / blocked / pending_verification)<br>created_at  <br>avatar_url<br>last_login_at<br>consent_pd <br>city<br>profile_description<br>organization_id (nullable)|Все роли, кроме приютов|Транзакционные|
|Приют ( Shelter)|organization_id<br>name<br>city<br>address<br>contact_phone<br>description<br>created_at<br>status|Приют, Администратор|Транзакционные|
|**Животное (Animal)**|animal_id<br>organization_id (nullable, если животное у частного куратора)<br>user_id<br>name<br>species (собака / кошка / др.)<br>breed<br>sex<br>age<br>description<br>health_status<br>sterilized_flag<br>vaccinated_flag<br>current_status (на лечении / ищет дом / пристроен / архив)<br>adoption_ready_flag<br>created_at<br>updated_at<br>description|Приют, Куратор, Администратор|Транзакционные + Аналитические|
|**Медиафайл животного (**AnimalМediaFile)|media_id<br>animal_id<br>uploaded_by<br>file_url<br>file_type<br>mime_type<br>size_bytes<br>sort_order<br>created_at|Приют, куратор, админ||
|**Заявка на помощь (HelpRequest)**|help_request_id<br>animal_id<br>created_by<br>request_type (лечение / корм / перевозка / передержка / др.)<br>title<br>description<br>priority<br>required_by_date<br>location<br>status (новая / в работе / закрыта / отменена / просрочена)<br>created_at<br>closed_at|Приют, Куратор, Волонтер, помощник, Администратор|Транзакционные + аналитические|
|**Отклик на заявку (HelpResponse)**|response_id<br>help_request_id<br>volunteer_id<br>response_comment<br>response_status (откликнулся / назначен / отказался / выполнил)<br>assigned_at<br>completed_at<br>completion_comment<br>proof_media_url — при необходимости|Волонтёр, Куратор, Приют, Админ|Транзакционные + аналитические|
|**Заявка на пристройство (AdoptionRequest)**|adoption_application_id<br>animal_id<br>applicant_user_id<br>message<br>contact_phone<br>housing_type<br>has_other_animals_flag<br>application_status (новая / на рассмотрении / одобрена / отклонена / завершена)<br>created_at<br>reviewed_at<br>review_comment|Потенциальный владелец, Куратор, Приют, Админ|Транзакционные + аналитические|
|**Пожертвование (Donation)**|donation_id<br>donor_user_id (nullable, если гостевой сценарий допустим)<br>animal_id (nullable)<br>help_request_id (nullable)<br>amount<br>currency<br>payment_provider<br>provider_payment_id<br>payment_status (pending / succeeded / failed / refunded)<br>created_at<br>paid_at<br>receipt_url|Помощник, Бухгалтер, Администратор|Транзакционные + аналитические|
|**История изменений (ChangeLog)**|audit_event_id<br>entity_type (animal / help_request / adoption_application / donation / user)<br>entity_id<br>action_type (create / update / status_change / assign / close / moderate)<br>changed_by_user_id<br>old_value_json<br>new_value_json<br>created_at|Система (автоматически), Админ|Аналитические|

 

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
