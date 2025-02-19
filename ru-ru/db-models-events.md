---
layout: default
title: 'Model Events'
keywords: 'models, events, events manager'
---

# Model Events
- - -
![](/assets/images/document-status-stable-success.svg) ![](/assets/images/version-{{ pageVersion }}.svg)

## Введение
Models allow you to implement events that will be thrown while performing an insert/update/delete which can be used to define business rules. The following are the events supported by [Phalcon\Mvc\Model][mvc-model-query] and their order of execution:

| Операция      | Название                   | Stop? | Пояснение                                                                                             |
| ------------- | -------------------------- |:-----:| ----------------------------------------------------------------------------------------------------- |
| Insert        | `afterCreate`              |  Нет  | Runs after creating a record                                                                          |
| Delete        | `afterDelete`              |  Нет  | Runs after deleting records                                                                           |
| Fetch         | `afterFetch`               |  Нет  | Runs after fetching records                                                                           |
| Insert/Update | `afterSave`                |  Нет  | Runs after saving a record                                                                            |
| Update        | `afterUpdate`              |  Нет  | Runs after updating a record                                                                          |
| Insert/Update | `afterValidation`          |  Да   | Is executed after the fields are validated for not `null`/empty strings or foreign keys               |
| Insert        | `afterValidationOnCreate`  |  Да   | Is executed after the fields are validated for not `null`/empty strings or foreign keys on an insert  |
| Update        | `afterValidationOnUpdate`  |  Да   | Is executed after the fields are validated for not `null`/empty strings or foreign keys on an update  |
| Insert        | `beforeCreate`             |  Да   | Runs before creating a record                                                                         |
| Delete        | `beforeDelete`             |  Да   | Runs before deleting records                                                                          |
| Insert/Update | `beforeSave`               |  Да   | Runs before saving a record                                                                           |
| Update        | `beforeUpdate`             |  Да   | Runs before updating a record                                                                         |
| Insert/Update | `beforeValidation`         |  Да   | Is executed before the fields are validated for not `null`/empty strings or foreign keys              |
| Insert        | `beforeValidationOnCreate` |  Да   | Is executed before the fields are validated for not `null`/empty strings or foreign keys on an insert |
| Update        | `beforeValidationOnUpdate` |  Да   | Is executed before the fields are validated for not `null`/empty strings or foreign keys on an update |
| Delete        | `notDeleted`               |  Нет  | Runs when records are not deleted (fail)                                                              |
| Save          | `notSaved`                 |  Нет  | Runs when records are not saved (fail)                                                                |
| Insert/Update | `onValidationFails`        |  Да   | Is executed after an integrity validator fails                                                        |
| Insert/Update | `prepareSave`              |  Нет  | Is executed before saving and allows data manipulation                                                |
| Insert/Update | `validation`               |  Да   | Is executed before the fields are validated for not nulls/empty strings or foreign keys on an update  |

### События
Models act as listeners to the events manager. Therefore, we only need to implement the events above in the models directly as public methods:

```php
<?php

namespace MyApp\Models;

use Phalcon\Mvc\Model;

/**
 * Class Invoices
 *
 * @property string $inv_created_at
 * @property int    $inv_cst_id
 * @property int    $inv_id
 * @property string $inv_number
 * @property string $inv_title
 * @property float  $inv_total
 */
class Invoices extends Model
{
    /**
     * @var int
     */
    public $inv_cst_id;

    /**
     * @var string
     */
    public $inv_created_at;

    /**
     * @var int
     */
    public $inv_id;

    /**
     * @var string
     */
    public $inv_number;

    /**
     * @var string
     */
    public $inv_title;

    /**
     * @var float
     */
    public $inv_total;

    public function beforeValidationOnCreate()
    {
        if ($this->inv_total < 1) {
            $this->inv_total = 0;
        }
    }
}

```

Events can be used to assign values before performing an operation, for example:

```php
<?php

namespace MyApp\Models;

use Phalcon\Mvc\Model;
use function str_pad;

/**
 * Class Invoices
 *
 * @property string $inv_created_at
 * @property int    $inv_cst_id
 * @property int    $inv_id
 * @property string $inv_number
 * @property string $inv_title
 * @property float  $inv_total
 */
class Invoices extends Model
{
    /**
     * @var int
     */
    public $inv_cst_id;

    /**
     * @var string
     */
    public $inv_created_at;

    /**
     * @var int
     */
    public $inv_id;

    /**
     * @var string
     */
    public $inv_number;

    /**
     * @var string
     */
    public $inv_title;

    /**
     * @var float
     */
    public $inv_total;

    public function beforeCreate()
    {
        $date     = date('YmdHis');
        $customer = substr(
            str_pad(
                $this->inv_cst_id, 6, '0', STR_PAD_LEFT
            ),
            -6
        );

        $this->inv_number = 'INV-' . $customer . '-' . $date;
    }
}
```

### Custom Events Manager
Additionally, this component is integrated with [Phalcon\Events\Manager][events-manager], this means we can create listeners that run when an event is triggered.

```php
<?php

namespace MyApp\Models;

use Phalcon\Mvc\Model;
use Phalcon\Events\Manager;

/**
 * Class Invoices
 *
 * @property string $inv_created_at
 * @property int    $inv_cst_id
 * @property int    $inv_id
 * @property string $inv_number
 * @property string $inv_title
 * @property float  $inv_total
 */
class Invoices extends Model
{
    /**
     * @var int
     */
    public $inv_cst_id;

    /**
     * @var string
     */
    public $inv_created_at;

    /**
     * @var int
     */
    public $inv_id;

    /**
     * @var string
     */
    public $inv_number;

    /**
     * @var string
     */
    public $inv_title;

    /**
     * @var float
     */
    public $inv_total;

    public function initialize()
    {
        $eventsManager = new Manager();

        $eventsManager->attach(
            'model:beforeSave',
            function (Event $event, $invoice) {
                if ($invoice->inv_total < 1) {
                    return false;
                }

                return true;
            }
        );

        $this->setEventsManager($eventsManager);
    }
}
```

In the example given above, the Events Manager only acts as a bridge between an object and a listener (the anonymous function). Events will be fired to the listener when `Invoices` are saved:

```php
<?php

use MyApp\Models\Invoices;

$invoice = new Invoices();
$invoice->inv_cst_id = 10;
$invoice->inv_title = 'Invoice for ACME Inc.';

$invoice->save();
```

If we want all objects created in our application use the same EventsManager, then we need to assign it to the Models Manager when setting it in the DI container:

```php
<?php

use MyApp\Models\Invoices;
use Phalcon\Di\FactoryDefault;
use Phalcon\Events\Event;
use Phalcon\Events\Manager;
use Phalcon\Mvc\Model\Manager as ModelsManager;

$container = new FactoryDefault();
$container->setShared(
    'modelsManager',
    function () {
        $eventsManager = new Manager();

        $eventsManager->attach(
            'model:beforeSave',
            function (Event $event, $model) {
                if (get_class($model) === Invoices::class) {
                    if ($model->inv_total < 1) {
                        return false;
                    }
                }

                return true;
            }
        );

        $modelsManager = new ModelsManager();
        $modelsManager->setEventsManager($eventsManager);

        return $modelsManager;
    }
);
```

If a listener returns false that will stop the operation that is executing currently.

## Logging SQL Statements
When using high-level abstraction components such as [Phalcon\Mvc\Model][mvc-model] to access a database, it is difficult to understand which statements are finally sent to the database system. [Phalcon\Mvc\Model][mvc-model] is supported internally by [Phalcon\Db][db]. [Phalcon\Logger](logger) interacts with [Phalcon\Db][db], providing logging capabilities on the database abstraction layer, thus allowing us to log SQL statements as they happen.

```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql;
use Phalcon\Di\FactoryDefault;
use Phalcon\Events\Manager;
use Phalcon\Logger;
use Phalcon\Logger\Adapter\Stream;

$container = new FactoryDefault();
$container->set(
    'db',
    function () {
        $eventsManager = new Manager();
        $adapter = new Stream('/storage/logs/db.log');
        $logger  = new Logger(
            'messages',
            [
                'main' => $adapter,
            ]
        );

        $eventsManager->attach(
            'db:beforeQuery',
            function ($event, $connection) use ($logger) {
                $logger->info(
                    $connection->getSQLStatement()
                );
            }
        );

        $connection = new Mysql\(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'phalcon',
            ]
        );

        $connection->setEventsManager($eventsManager);

        return $connection;
    }
);
```

As models access the default database connection, all SQL statements that are sent to the database system will be logged in the file:

```php
<?php

use MyApp\Models\Invoices;

$invoice = new Invoices();
$invoice->inv_cst_id = 10;
$invoice->inv_title  = 'Invoice for ACME Inc.';
$invoice->inv_total  = 10000;

if ($invoice->save() === false) {
    echo 'Cannot save robot';
}
```

As above, the file */storage/logs/db.log* will contain something like this:


> `[Mon, 30 Apr 12 13:47:18 -0500][DEBUG][Resource Id #77] INSERT INTO co_invoices` `(inv_cst_id, inv_title, inv_total) VALUES (10, 'Invoice for ACME Inc.', 10000)`

## Profiling SQL Statements
Using the [Phalcon\Db][db], the underlying component of [Phalcon\Mvc\Model][mvc-model], it is possible to profile the SQL statements generated by the ORM in order to analyze the performance of database operations. Analyzing the logs will help in identifying bottlenecks in your SQL code:

```php
<?php

use Phalcon\Db\Profiler;
use Phalcon\Di\FactoryDefault;
use Phalcon\Events\Manager;
use Phalcon\Db\Adapter\Pdo;

$container = new FactoryDefault();
$container->set(
    'profiler',
    function () {
        return new Profiler();
    },
    true
);

$container->set(
    'db',
    function () use ($container) {
        $manager  = new Manager();
        $profiler = $container->getProfiler();

        $manager->attach(
            'db',
            function ($event, $connection) use ($profiler) {
                if ($event->getType() === 'beforeQuery') {
                    $profiler->startProfile(
                        $connection->getSQLStatement()
                    );
                }

                if ($event->getType() === 'afterQuery') {
                    $profiler->stopProfile();
                }
            }
        );

        $connection = new Mysql(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'phalcon',
            ]
        );

        $connection->setEventsManager($manager);

        return $connection;
    }
);
```

Profiling some queries:

```php
<?php

use MyApp\Models\Invoices;

Invoices::find();
Invoices::find(
    [
        'order' => 'inv_cst_id, inv_title',
    ]
);
Invoices::find(
    [
        'limit' => 30,
    ]
);

$profiles = $container->get('profiler')->getProfiles();

foreach ($profiles as $profile) {
    echo 'SQL: ', 
        $profile->getSQLStatement(), 
        PHP_EOL,
        'Start: ',
        $profile->getInitialTime(),
        PHP_EOL,
        'Final: ',
        $profile->getFinalTime(),
        PHP_EOL,
        'Elapsed: ',
        $profile->getTotalElapsedSeconds(),
        PHP_EOL
    );
}
```

Each generated profile contains the duration in milliseconds that each instruction takes to complete as well as the generated SQL statement.

[db]: api/phalcon_db
[events-manager]: api/phalcon_events#events-manager
[mvc-model]: api/phalcon_mvc#mvc-model
[mvc-model-query]: api/phalcon_mvc#mvc-model-query
