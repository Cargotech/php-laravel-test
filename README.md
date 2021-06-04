**Разрешенные пакеты:**
- https://github.com/nWidart/laravel-modules
- https://github.com/spatie/data-transfer-object
- https://github.com/laravel/horizon
- https://github.com/nunomaduro/larastan

Для начала работы над тестовым заданием клонируйте репу и создайте приватный репозиторий. 

После завершения работы откройте доступ к репе https://github.com/smadrom

**Версия PHP: 8.0, Laravel 8, Postgres 13**

Общие требования для заданий:

1. По максимум используем возможности php 8.0 что-бы сократить кол-во кода.
2. По максимум используем возможности laravel ^8.4 что-бы сократить кол-во кода.
3. ./vendor/bin/phpstan analyse не должен ругаться.
4. В каждом нашем файле должен быть включён строгий режим **<?php declare(strict_types=1)** и использованы **typehint**;
- 4.1 И в каждом генерируемом нами файле через ``make:``;
- 4.1 Бесполезные куски doc блоков требуется удалить из кода и заменить их на типы. Для этого рекомендуется заранее настроить стабы;
5. Код по возможности требуется покрывать простыми тестами (необязательно);
6. Для настройки локального окружения желательно использовать docker и docker-compose (необязательно);

Вы можете не выполнять дополнительные пункты, но они крайне рекомендуются для общей оценки.

-------------------------

**Задание 1. Создать модуль Api.**

Модуль должен уметь работать с апи https://api.cargo.tech/*, например https://api.cargo.tech/v1/cargos (реализовать только GET).

**Требуемые функции:**
- Получить одну запись.
- Получить первые 5 страниц.
- Получить все страницы.

Например, если в апи 3000 записей, макс. лимит на страницу 100 записей, нам надо сделать 100 * 30 запросов к апи что-бы получить все данные

Данные надо отдавать ларавел коллекцией.

Для пагинации используются limit и offset, например ?limit=25,offset=25 - вторая страница.

```php
\Modules\Api\Services\ClientService()->get('/cargos');
```

-------------------------

**Задание 2. Создать модуль Cargo.**

С помощью модуля API раз в 5 минут мы должны обращаться к апи, забирать первые 2 страницы данных, добавлять их в базу через модель Cargo, с помощью очереди **(sync использовать только в отладочных целях)**.

**Этапы:**

1. Забираем первые 2 страницы данных.
2. Обрабатываем (как передавать данные между функциями — на ваш выбор).
3. Каждую запись по отдельности отправляем в очередь **отдельной задачей**.
4.  Задача либо добавляет груз, если его нет, либо обновляет, если он есть. В зависимости от типа действия, отправляется ивент **CargoCreatedEvent/CargoUpdatedEvent**.
- CargoCreatedEvent - отправляет email сообщение через **notifications**;
- CargoUpdatedEvent - через пять минут после события груз должен **мягко** удалиться из базы данных;

Дополнительно:

- Информацию о количестве грузов держать в отдельной таблице (например counters, table=cargos, count=10), и обновлять её каждый раз когда происходит какое-либо действие с данными через модель Cargo.

- Создать контроллер для Cargo модели, реализовать простой api crud (JsonResponse) для index//store/update/show (использовать FormRequest, апи всегда должно отвечать с помощью json, не зависимо от того какой заголовок передан). index должен иметь возможность пагинации через offset + limit.


-------------------------

Поля из апи которые надо использовать:

- id
- weight
- volume
- truck

Truck должен быть jsonb полем, с кастомным cast https://laravel.com/docs/8.x/eloquent-mutators#custom-casts

Т.е. при записи поля в базу оно должно быть строкой (json_encode), при чтении поле должно возвращать объект TruckCast или TruckDto (на ваш выбор).

Пример с кодом:

```php
Cargo::create(['truck' => ['tir' => true]]);// работает, на вход передаем массив, сеттер должен учесть это

$object = new TruckDto(['tir' => true]);
Cargo::create(['truck' => $object]); // работает, на вход передаем объект

class TruckDtoExample
{
	public bool $tir = false;
	public bool $t1 = false;
	public bool $cmr = false;
}
```

-------------------------

**Дополнительно.**

- Нам надо иметь возможность осуществлять операции >< с полем truck.belt_count.
- Нам надо иметь возможность искать какое-либо значение в поле ``truck`` (jsonb) с помощью @>.
- Запаковать приложение в контейнер докера (php-fpm), + сделать контейнер с nginx + конфиг для домена ``test.com``, и сделать к ним docker-compose.
