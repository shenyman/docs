---
layout: default
upgrade: '#logger'
title: 'Журналирование'
keywords: 'logger, adapters, noop, stream, syslog'
---

# Журналирование
- - -
![](/assets/images/document-status-stable-success.svg) ![](/assets/images/version-{{ pageVersion }}.svg)

## Введение
[Phalcon\Logger\Logger][logger-logger] is a component providing logging services for applications. It offers logging to different back-ends using different adapters. It also offers transaction logging, configuration options and different logging formats. You can use the [Phalcon\Logger\Logger][logger-logger] for any logging need your application has, from debugging processes to tracing application flow.

The [Phalcon\Logger\Logger][logger-logger] implements methods that are inline with [PSR-3][psr-3], but does not implement the particular interface. A package that implements [PSR-3][psr-3] is available, that uses [Phalcon\Logger\Logger][logger-logger]. The package is located [here][proxy-psr3]. To use it, you will need to have Phalcon installed and then using composer you can install the proxy package.

```sh
composer require phalcon/proxy-psr3
```

Using the proxy classes allows you to follow [PSR-3][psr-3] and use it with any other package that needs that interface.

The [Phalcon\Logger\Logger][logger-logger] implements only the logging functionality and accepts one or more adapters that would be responsible for doing the work of logging. This implementation separates the responsibilities of the component and offers an easy way to attach more than one adapter to the logging component so that logging to multiple adapters can be achieved.

## Adapters
This component makes use of adapters to store the logged messages. The use of adapters allows for a common logging interface which provides the ability to easily switch back-ends, or use multiple adapters if necessary. The adapters supported are:

| Адаптер                                                   | Описание                                     |
| --------------------------------------------------------- | -------------------------------------------- |
| [Phalcon\Logger\Adapter\Noop][logger-adapter-noop]     | Black hole adapter (used for testing mostly) |
| [Phalcon\Logger\Adapter\Stream][logger-adapter-stream] | Logs messages on a file stream               |
| [Phalcon\Logger\Adapter\Syslog][logger-adapter-syslog] | Logs messages to the Syslog                  |

### Stream
This adapter is used when we want to log messages to a particular file stream. This adapter combines the v3 `Stream` and `File` ones. Usually this is the most used one: logging to a file in the file system.

### Syslog
This adapter sends messages to the system log. The syslog behavior may vary from one operating system to another.

### Noop
This is a black hole adapter. It sends messages to *infinity and beyond*! This adapter is used mostly for testing or if you want to joke with a colleague.

## Фабрика
You can use the [Phalcon\Logger\LoggerFactory][logger-loggerfactory] component to create a logger. For the [Phalcon\Logger\LoggerFactory][logger-loggerfactory] to work, it needs to be instantiated with a [Phalcon\Logger\AdapterFactory][logger-adapterfactory]:

```php
<?php

use Phalcon\Logger\LoggerFactory;
use Phalcon\Logger\AdapterFactory;

$adapterFactory = new AdapterFactory();
$loggerFactory  = new LoggerFactory($adapterFactory);
```

### `load()`
[Phalcon\Logger\LoggerFactory][logger-loggerfactory] offers the `load` method, that constructs a logger based on supplied configuration. The configuration can be an array or a [Phalcon\Config](config) object.

> **NOTE**: Use Case: Create a Logger with two Stream adapters. One adapter will be called `main` for logging all messages, while the second one will be called `admin`, logging only messages generated in the admin area of our application 
> 
> {: .alert .alert-info}

```php
<?php

use Phalcon\Logger\AdapterFactory;
use Phalcon\Logger\LoggerFactory;
use Phalcon\Storage\SerializerFactory;

$config = [
    "name"     => "prod-logger",
    "adapters" => [
        "main"  => [
            "adapter" => "stream",
            "name"    => "/storage/logs/main.log",
            "options" => []
        ],
        "admin" => [
            "adapter" => "stream",
            "name"    => "/storage/logs/admin.log",
            "options" => []
        ],
    ],
];

$serializerFactory = new SerializerFactory();
$adapterFactory    = new AdapterFactory();
$loggerFactory     = new LoggerFactory($adapterFactory);

$logger = $loggerFactory->load($config);
```

### `newInstance()`
The [Phalcon\Logger\LoggerFactory][logger-loggerfactory] also offers the `newInstance()` method, that constructs a logger based on the supplied name and array of relevant adapters. Using the use case above:

```php
<?php

use Phalcon\Logger\Logger\Logger\Adapter\Stream;
use Phalcon\Logger\AdapterFactory;
use Phalcon\Logger\LoggerFactory;
use Phalcon\Storage\SerializerFactory;

$adapters = [
    "main"  => new Stream("/storage/logs/main.log"),
    "admin" => new Stream("/storage/logs/admin.log"),
];

$serializerFactory = new SerializerFactory();
$adapterFactory    = new AdapterFactory($serializerFactory);
$loggerFactory     = new LoggerFactory($adapterFactory);

$logger = $loggerFactory->newInstance('prod-logger', $adapters);
```

## Creating a Logger
Creating a logger is a multistep process. First you create the logger object, and then you attach an adapter to it. After that you can start logging messages according to the needs of your application.

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter = new Stream('/storage/logs/main.log');
$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong');
```

The above example creates a [Stream][logger-adapter-stream] adapter. We then create a logger object and attach this adapter to it. Each adapter attached to a logger needs to have a unique name, for the logger to be able to know where to log the messages. When calling the `error()` method on the logger object, the message is going to be stored in the `/storage/logs/main.log`.

Since the logger component implements PSR-3, the following methods are available:

```php
<?php

use Phalcon\Logger\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter = new Stream('/storage/logs/main.log');
$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->alert("This is an alert message");
$logger->critical("This is a critical message");
$logger->debug("This is a debug message");
$logger->error("This is an error message");
$logger->emergency("This is an emergency message");
$logger->info("This is an info message");
$logger->log(Logger::CRITICAL, "This is a log message");
$logger->notice("This is a notice message");
$logger->warning("This is a warning message");

```
The log generated is as follows:

```bash
[Tue, 25 Dec 18 12:13:14 -0400][ALERT] This is an alert message
[Tue, 25 Dec 18 12:13:14 -0400][CRITICAL] This is a critical message
[Tue, 25 Dec 18 12:13:14 -0400][DEBUG] This is a debug message
[Tue, 25 Dec 18 12:13:14 -0400][ERROR] This is an error message
[Tue, 25 Dec 18 12:13:14 -0400][EMERGENCY] This is an emergency message
[Tue, 25 Dec 18 12:13:14 -0400][INFO] This is an info message
[Tue, 25 Dec 18 12:13:14 -0400][CRITICAL] This is a log message
[Tue, 25 Dec 18 12:13:14 -0400][NOTICE] This is a notice message
[Tue, 25 Dec 18 12:13:14 -0400][WARNING] This is warning message
```

## Multiple Adapters
[Phalcon\Logger\Logger][logger-logger] can send messages to multiple adapters with a just single call:

```php
<?php

use Phalcon\Logger\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter1 = new Stream('/logs/first-log.log');
$adapter2 = new Stream('/remote/second-log.log');
$adapter3 = new Stream('/manager/third-log.log');

$logger = new Logger(
    'messages',
    [
        'local'   => $adapter1,
        'remote'  => $adapter2,
        'manager' => $adapter3,
    ]
);

$logger->error('Something went wrong');
```

The messages are sent to the handlers in the order they were registered using the [first in first out][fifo] principle.

### Excluding Adapters
[Phalcon\Logger\Logger][logger-logger] also offers the ability to exclude logging to one or more adapters when logging a message. This is particularly useful when in need to log a `manager` related message in the `manager` log but not in the `local` log without having to instantiate a new logger.

With the following setup:

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter1 = new Stream('/logs/first-log.log');
$adapter2 = new Stream('/remote/second-log.log');
$adapter3 = new Stream('/manager/third-log.log');

$logger = new Logger(
    'messages',
    [
        'local'   => $adapter1,
        'remote'  => $adapter2,
        'manager' => $adapter3,
    ]
);
```

we have the following:

```php
<?php

$logger->error('Something went wrong');
```
Log to all adapters

```php
<?php

$logger
    ->excludeAdapters(['local'])
    ->info('This does not go to the "local" logger');
```
Log only to remote and manager

> **NOTE** Internally, the component loops through the registered adapters and calls the relevant logging method to achieve logging to multiple adapters. If one of them fails, the loop will break and the remaining adapters (from the loop) will not log the message. In future versions of Phalcon we will be introducing asynchronous logging to alleviate this problem. 
> 
> {: .alert .alert-warning }

## Константы
The class offers a number of constants that can be used to distinguish between log levels. These constants can also be used as the first parameter in the `log()` method.

| Константа   | Value |
| ----------- |:-----:|
| `EMERGENCY` |   0   |
| `CRITICAL`  |   1   |
| `ALERT`     |   2   |
| `ERROR`     |   3   |
| `WARNING`   |   4   |
| `NOTICE`    |   5   |
| `INFO`      |   6   |
| `DEBUG`     |   7   |
| `CUSTOM`    |   8   |


## Log Levels
[Phalcon\Logger\Logger][logger-logger] allows you to set the minimum log level for the logger(s) to log. If you set this integer value, any level higher than the one set will not be logged. Check the values of the constants in the previous section for the order that the levels are being set.

In the following example, we set the log level to `ALERT`. We will only see `EMERGENCY`, `CRITICAL` **and** `ALERT` messages.

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter = new Stream('/storage/logs/main.log');
$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->setLogLevel(Logger::ALERT);

$logger->alert("This is an alert message");
$logger->critical("This is a critical message");
$logger->debug("This is a debug message");
$logger->error("This is an error message");
$logger->emergency("This is an emergency message");
$logger->info("This is an info message");
$logger->log(Logger::CRITICAL, "This is a log message");
$logger->notice("This is a notice message");
$logger->warning("This is a warning message");

```
The log generated is as follows:

```bash
[Tue, 25 Dec 18 12:13:14 -0400][ALERT] This is an alert message
[Tue, 25 Dec 18 12:13:14 -0400][CRITICAL] This is a critical message
[Tue, 25 Dec 18 12:13:14 -0400][EMERGENCY] This is an emergency message
[Tue, 25 Dec 18 12:13:14 -0400][CRITICAL] This is a log message
```

The above can be used in situations where you want to log messages above a certain severity based on conditions in your application such as development mode vs. production.

> **NOTE**: The log level set is included in the logging. Anything **below** that level (i.e. higher number) will not be logged 
> 
> {: .alert .alert-info }

> **NOTE**: It is **never** a good idea to suppress logging levels in your application, since even warning errors do require CPU cycles to be processed and neglecting these errors could potentially lead to unintended circumstances 
> 
> {: .alert .alert-danger }

## Транзакции
[Phalcon\Logger\Logger][logger-logger] also offers the ability to queue the messages in your logger, and then _commit_ them all together in the log file. This is similar to a database transaction with `begin` and `commit`. Each adapter exposes the following methods:

| Название                | Описание                                       |
| ----------------------- | ---------------------------------------------- |
| `begin(): void`         | begins the logging transaction                 |
| `inTransaction(): bool` | if you are in a transaction or not             |
| `commit(): void`        | writes all the queued messages in the log file |

Since the functionality is available at the adapter level, you can program your logger to use transactions on a per-adapter basis.

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter1 = new Stream('/logs/first-log.log');
$adapter2 = new Stream('/remote/second-log.log');
$adapter3 = new Stream('/manager/third-log.log');

$logger = new Logger(
    'messages',
    [
        'local'   => $adapter1,
        'remote'  => $adapter2,
        'manager' => $adapter3,
    ]
);

$logger->getAdapter('manager')->begin();

$logger->error('Something happened');

$logger->getAdapter('manager')->commit();
```
In the example above, we register three adapters. We set the `manager` logger to be in transaction mode. As soon as we call:

```php
$logger->error('Something happened');
```
the message will be logged in both `local` and `remote` adapters. It will be queued for the `manager` adapter and not logged until we call the `commit` method in the `manager` adapter.

> **NOTE**: If you set one or more adapters to be in transaction mode (i.e. call `begin`) and forget to call `commit`, The adapter will call `commit` for you right before it is destroyed. 
> 
> {: .alert .alert-info }
## Message Formatting
This component makes use of `formatters` to format messages before sending them to the backend. The formatters available are:

| Адаптер                                                   | Описание                                     |
| --------------------------------------------------------- | -------------------------------------------- |
| [Phalcon\Logger\Formatter\Line][logger-formatter-line] | Formats the message on a single line of text |
| [Phalcon\Logger\Formatter\Json][logger-formatter-json] | Formats the message in a JSON string         |

### Line Formatter
Formats the messages using a one-line string. The default logging format is:

```bash
[%date%][%type%] %message%
```

#### Message Format
If the default format of the message does not fit the needs of your application you can change it using the `setFormat()` method. The log format variables allowed are:

| Variable    | Описание                                 |
| ----------- | ---------------------------------------- |
| `%message%` | The message itself expected to be logged |
| `%date%`    | Date the message was added               |
| `%type%`    | Uppercase string with message type       |

The following example demonstrates how to change the message format:

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;
use Phalcon\Logger\Formatter\Line;

$formatter = new Line('[%type%] - [%date%] - %message%');
$adapter   = new Stream('/storage/logs/main.log');

$adapter->setFormatter($formatter);

$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong');
```

which produces:

```bash
[ALERT] - [Tue, 25 Dec 18 12:13:14 -0400] - Something went wrong
```

If you do not want to use the constructor to change the message, you can always use the `setFormat()` on the formatter:

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;
use Phalcon\Logger\Formatter\Line;

$formatter = new Line();
$formatter->setFormat('[%type%] - [%date%] - %message%');

$adapter = new Stream('/storage/logs/main.log');

$adapter->setFormatter($formatter);

$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong');
```

#### Date Format
The default date format is:

```bash
"D, d M y H:i:s O"
```

If the default format of the message does not fit the needs of your application you can change it using the `setDateFormat()` method. The method accepts a string with characters that correspond to date formats. For all available formats, please consult [this page][date-formats].

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;
use Phalcon\Logger\Formatter\Line;

$formatter = new Line();
$formatter->setDateFormat('Ymd-His');

$adapter = new Stream('/storage/logs/main.log');

$adapter->setFormatter($formatter);

$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong'); 
```
which produces:

```bash
[ERROR] - [20181225-121314] - Something went wrong
```

### JSON Formatter
Formats the messages returning a JSON string:

```json
{
    "type"      : "Type of the message",
    "message"   : "The message",
    "timestamp" : "The date as defined in the date format"
}
```

> The `format()` method encodes JSON with the following options by default (79): - `JSON_HEX_TAG` - `JSON_HEX_APOS` - `JSON_HEX_AMP` - `JSON_HEX_QUOT` - `JSON_UNESCAPED_SLASHES` - `JSON_THROW_ON_ERROR` 
> 
> {: .alert .alert-info }

#### Date Format
The default date format is:

```bash
"D, d M y H:i:s O"
```

If the default format of the message does not fit the needs of your application you can change it using the `setDateFormat()` method. The method accepts a string with characters that correspond to date formats. For all available formats, please consult [this page][date-formats].

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;
use Phalcon\Logger\Formatter\Line;

$formatter = new Line();
$formatter->setDateFormat('Ymd-His');

$adapter = new Stream('/storage/logs/main.log');
$adapter->setFormatter($formatter);

$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong');
```

which produces:

```json
{
    "type"      : "error",
    "message"   : "Something went wrong",
    "timestamp" : "20181225-121314"
}
```

### Custom Formatter
The [Phalcon\Logger\Formatter\FormatterInterface][logger-formatter-formatterinterface] interface must be implemented in order to create your own formatter or extend the existing ones. Additionally, you can reuse the [Phalcon\Logger\Formatter\AbstractFormatter][logger-formatter-abstractformatter] abstract class.

## Interpolation
The logger also supports interpolation. There are times that you might need to inject additional text in your logging messages; text that is dynamically created by your application. This can be easily achieved by sending an array as the second parameter of the logging method (i.e. `error`, `info`, `alert` etc.). The array holds keys and values, where the key is the placeholder in the message and the value is what will be injected in the message.

The following example demonstrates interpolation by injecting in the message the parameter "framework" and "secs".

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter = new Stream('/storage/logs/main.log');
$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$message = '{framework} executed the "Hello World" test in {secs} second(s)';
$context = [
    'framework' => 'Phalcon',
    'secs'      => 1,
];

$logger->info($message, $context);
```

## Item
The formatter classes above accept a [Phalcon\Logger\Item][logger-item] object. The object contains all the necessary data required for the logging process. It is used as transport of data from the logger to the formatter.

> **NOTE**: In v5 the object now accepts a `\DateTimeImmutable` object as the `$dateTime` parameter 
> 
> {: .alert .alert-warning }

## Exceptions
Any exceptions thrown in the Logger component will be of type [Phalcon\Logger\Exception][logger-exception]. You can use this exception to selectively catch exceptions thrown only from this component.

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;
use Phalcon\Logger\Exception;

try {
    $adapter = new Stream('/storage/logs/main.log');
    $logger  = new Logger(
        'messages',
        [
            'main' => $adapter,
        ]
    );

    // Log to all adapters
    $logger->error('Something went wrong');
} catch (Exception $ex) {
    echo $ex->getMessage();
}
```

## Examples

### Stream
Logging to a file:

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter = new Stream('/storage/logs/main.log');
$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

// Log to all adapters
$logger->error('Something went wrong');
```

The stream logger writes messages to a valid registered stream in PHP. A list of streams is available [here][stream-wrappers]. Logging to a stream

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$adapter = new Stream('php://stderr');
$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong');
```

### Syslog

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Syslog;

// Setting identity/mode/facility
$adapter = new Syslog(
    'ident-name',
    [
        'option'   => LOG_NDELAY,
        'facility' => LOG_MAIL,
    ]
);

$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong');
```

### Noop

```php
<?php

use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Noop;

$adapter = new Noop('nothing');
$logger  = new Logger(
    'messages',
    [
        'main' => $adapter,
    ]
);

$logger->error('Something went wrong');
```

### Custom Adapters
The [Phalcon\Logger\AdapterInterface][logger-adapter-adapterinterface] interface must be implemented in order to create your own logger adapters or extend the existing ones. You can also take advantage of the functionality in [Phalcon\Logger\Adapter\AbstractAdapter][logger-adapter-abstractadapter] abstract class.

### Abstract Classes
There are three abstract classes that offer useful functionality when creating custom objects:
- [Phalcon\Logger\AbstractLogger][logger-abstractlogger]
- [Phalcon\Logger\Adapter\AbstractAdapter][logger-adapter-abstractadapter]
- [Phalcon\Logger\Formatter\AbstractFormatter][logger-formatter-abstractformatter].

## Dependency Injection
You can register as many loggers as you want in the \[Phalcon\Di\FactoryDefault\]\[factorydefault\] container. An example of the registration of the service as well as accessing it is below:

```php
<?php

use Phalcon\Di;
use Phalcon\Logger\Logger;
use Phalcon\Logger\Adapter\Stream;

$container = new Di();

$container->set(
    'logger',
    function () {
        $adapter = new Stream('/storage/logs/main.log');
        $logger  = new Logger(
            'messages',
            [
                'main' => $adapter,
            ]
        );

        return $logger;
    }
);

// accessing it later:
$logger = $container->getShared('logger');

```

[date-formats]: https://www.php.net/manual/en/function.date.php
[fifo]: https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics)
[logger-logger]: api/phalcon_logger#logger-logger
[logger-abstractlogger]: api/phalcon_logger#logger-abstractlogger
[logger-adapter-noop]: api/phalcon_logger#logger-adapter-noop
[logger-adapter-stream]: api/phalcon_logger#logger-adapter-stream
[logger-adapter-stream]: api/phalcon_logger#logger-adapter-stream
[logger-adapter-syslog]: api/phalcon_logger#logger-adapter-syslog
[logger-loggerfactory]: api/phalcon_logger#logger-loggerfactory
[logger-adapterfactory]: api/phalcon_logger#logger-adapterfactory
[logger-formatter-line]: api/phalcon_logger#logger-formatter-line
[logger-formatter-json]: api/phalcon_logger#logger-formatter-json
[logger-formatter-formatterinterface]: api/phalcon_logger#logger-formatter-formatterinterface
[logger-adapter-adapterinterface]: api/phalcon_logger#logger-adapter-adapterinterface
[logger-formatter-abstractformatter]: api/phalcon_logger#logger-formatter-abstractformatter
[logger-adapter-abstractadapter]: api/phalcon_logger#logger-adapter-abstractadapter
[logger-exception]: api/phalcon_logger#logger-exception
[logger-item]: api/phalcon_logger#logger-item
[proxy-psr3]: https://github.com/phalcon/proxy-psr3
[psr-3]: https://www.php-fig.org/psr/psr-3/
[stream-wrappers]: https://php.net/manual/en/wrappers.php
