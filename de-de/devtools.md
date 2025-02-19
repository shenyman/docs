---
layout: default
title: 'Devtools'
keywords: 'devtools, developer tools, models, controllers'
---

# Phalcon Devtools
- - -
![](/assets/images/document-status-stable-success.svg) ![](/assets/images/version-{{ pageVersion }}.svg)

## Overview
These tools help you to generate skeleton code, maintain your database structure and helps to speedup development. Core components of your application can be generated with a simple command, allowing you to easily develop applications using Phalcon.

You can use the Phalcon Devtools either from the command line (terminal) or the web interface.

## Installation

Phalcon Devtools can be installed using [composer](composer). Make sure you have installed first.

Install Phalcon Devtools globally
```bash
composer global require phalcon/devtools
```
Or only inside your project
```bash
composer require phalcon/devtools
```

Test your installation by typing: `phalcon`
```bash
$ phalcon

Phalcon DevTools (5.0.0)

Available commands:
  info             (alias of: i)
  commands         (alias of: list, enumerate)
  controller       (alias of: create-controller)
  module           (alias of: create-module)
  model            (alias of: create-model)
  all-models       (alias of: create-all-models)
  project          (alias of: create-project)
  scaffold         (alias of: create-scaffold)
  migration        (alias of: create-migration)
  webtools         (alias of: create-webtools)
  serve            (alias of: server)
  console          (alias of: shell, psysh)
```

The devtools are also available as phar download on our GitHub [repository](github_devtools).

## Usage
### Available Commands
You can get a list of available commands in Phalcon tools by typing: `phalcon commands`

```bash
$ phalcon commands

Phalcon DevTools (5.0.0)

Available commands:
  info             (alias of: i)
  commands         (alias of: list, enumerate)
  controller       (alias of: create-controller)
  module           (alias of: create-module)
  model            (alias of: create-model)
  all-models       (alias of: create-all-models)
  project          (alias of: create-project)
  scaffold         (alias of: create-scaffold)
  migration        (alias of: create-migration)
  webtools         (alias of: create-webtools)
  serve            (alias of: server)
  console          (alias of: shell, psysh)
```

### Generating a Project Skeleton
You can use Phalcon tools to generate pre-defined project skeletons for your applications with Phalcon framework. By default, the project skeleton generator will use mod_rewrite for Apache. Type the following command on your web server document root:

```bash
$ phalcon create-project store
```

The above recommended project structure was generated:

![](/assets/images/content/v4/devtools-store-dirstructure.png)

You could add the parameter `--help` to get help on the usage of a certain script:

```bash
$ phalcon project --help

Phalcon DevTools (5.0.0)

Help:
  Creates a project

Usage:
  project [name] [type] [directory] [enable-webtools]

Arguments:
  help  Shows this help text

Example
  phalcon project store simple

Options:
 --name=s               Name of the new project
 --enable-webtools      Determines if webtools should be enabled [optional]
 --directory=s          Base path on which project will be created [optional]
 --type=s               Type of the application to be generated (cli, micro, simple, modules)
 --template-path=s      Specify a template path [optional]
 --template-engine=s    Define the template engine, default phtml (phtml, volt) [optional]
 --use-config-ini       Use a ini file as configuration file [optional]
 --trace                Shows the trace of the framework in case of exception [optional]
 --help                 Shows this help [optional]
```

Accessing the project from the web server will show you:

![](/assets/images/content/v4/devtools-store-localhost.png)

### Generating Controllers
The command `create-controller` generates controller skeleton structures. It's important to invoke this command inside a directory that already has a Phalcon project.

```bash
$ phalcon create-controller --name test
```

The following code is generated by the script:

```php
<?php
declare(strict_types=1);

class TestController extends \Phalcon\Mvc\Controller
{
    public function indexAction()
    {

    }
}

```

### Preparing Database Settings
When a project is generated using developer tools. A configuration file can be found in `app/config/config.php`. To generate models or scaffold, you will need to change the settings used to connect to your database.

Change the database section in your config.php file:

```php
<?php

/*
 * Modified: prepend directory path of current file, because of this file own different ENV under between Apache and command line.
 * NOTE: please remove this comment.
 */
defined('BASE_PATH') || define('BASE_PATH', getenv('BASE_PATH') ?: realpath(dirname(__FILE__) . '/../..'));
defined('APP_PATH') || define('APP_PATH', BASE_PATH . '/app');

return new \Phalcon\Config([
    'database' => [
        'adapter'     => 'Mysql',
        'host'        => 'localhost',
        'username'    => 'root',
        'password'    => '',
        'dbname'      => 'test',
        'charset'     => 'utf8',
    ],
    'application' => [
        'appDir'         => APP_PATH . '/',
        'controllersDir' => APP_PATH . '/controllers/',
        'modelsDir'      => APP_PATH . '/models/',
        'migrationsDir'  => APP_PATH . '/migrations/',
        'viewsDir'       => APP_PATH . '/views/',
        'pluginsDir'     => APP_PATH . '/plugins/',
        'libraryDir'     => APP_PATH . '/library/',
        'cacheDir'       => BASE_PATH . '/cache/',
        'baseUri'        => '/',
    ]
]);
```

### Generating Models
There are several ways to create models. You can create all models from the default database connection or some selectively. Models can have public attributes for the field representations or setters/getters can be used.

```bash
Options:
 --name=s             Table name
 --schema=s           Name of the schema [optional]
 --config=s           Configuration file [optional]
 --namespace=s        Model's namespace [optional]
 --get-set            Attributes will be protected and have setters/getters [optional]
 --extends=s          Model extends the class name supplied [optional]
 --excludefields=l    Excludes fields defined in a comma separated list [optional]
 --doc                Helps to improve code completion on IDEs [optional]
 --directory=s        Base path on which project is located [optional]
 --output=s           Folder where models are located [optional]
 --force              Rewrite the model [optional]
 --camelize           Properties is in camelCase [optional]
 --trace              Shows the trace of the framework in case of exception [optional]
 --mapcolumn          Get some code for map columns [optional]
 --abstract           Abstract Model [optional]
 --annotate           Annotate Attributes [optional]
 --help               Shows this help [optional]
```

The simplest way to generate a model for a table called `customers` is:

```bash
$ phalcon model customers
```

If your database looks like this:

```sql
create table customers
(
    `cst_id`          int(10) auto_increment primary key,
    `cst_status_flag` tinyint(1)   null,
    `cst_name_last`   varchar(100) null,
    `cst_name_first`  varchar(50)  null
);

create index customers_cst_status_flag_index
    on `customers` (`cst_status_flag`);

create index customers_cst_name_last_index
    on `customers` (`cst_name_last`);

create index customers_cst_name_first_index
    on `customers` (`cst_name_first`);
```

It will result in

```php
<?php

/**
 * This file is part of the Phalcon Framework.
 *
 * (c) Phalcon Team <team@phalcon.io>
 *
 * For the full copyright and license information, please view the LICENSE.txt
 * file that was distributed with this source code.
 */

declare(strict_types=1);

namespace Phalcon\Tests\Models;

use Phalcon\Mvc\Model;
use Phalcon\Filter\Validation;
use Phalcon\Filter\Validation\Validator\PresenceOf as EmailValidator;

/**
 * @property int    $cst_id
 * @property int    $cst_status_flag
 * @property string $cst_name_last
 * @property string $cst_name_first
 * @property array  $cst_data;
 */
class Customers extends Model
{
    /**
     * @var int 
     */
    public $cst_id;

    /**
     * @var int 
     */
    public $cst_status_flag;

    /**
     * @var string 
     */
    public $cst_name_last;

    /**
     * @var string 
     */
    public $cst_name_first;

    public function initialize()
    {
        $this->setSource('customers');
    }

    /**
     * @return bool
     */
    public function validation()
    {
        $validator = new Validation();

        $validator->add(
            'cst_name_last',
            new PresenceOf(
                [
                    'model'   => $this,
                    'message' => 'Please enter a valid last name',
                ]
            )
        );

        return $this->validate($validator);
    }
}
```

Options for generating different type of model blueprints can be found using

```bash
phalcon model --help
```

### Scaffold a CRUD
Scaffolding is a quick way to generate some major pieces of an application. If you want to create the models, views, and controllers for a new resource in a single operation, scaffolding is the tool for the job.

Once the code is generated, it will have to be customized to meet your needs. Many developers avoid scaffolding entirely, opting to write all or most of their source code from scratch. The generated code can serve as a guide to better understand of how the framework works or develop prototypes. The code below shows a scaffold based on the table `customers`:

```bash
$ phalcon scaffold --table-name customers
```

The scaffold generator will build several files in your application, along with some folders. Here's a quick overview of what will be generated:

| File                                      | Purpose                      |
| ----------------------------------------- | ---------------------------- |
| `app/controllers/CustomersController.php` | The Customers controller     |
| `app/models/Customers.php`                | The Customers model          |
| `app/views/layout/customers.phtml`        | Controller layout for Users  |
| `app/views/products/search.phtml`         | View for the action `search` |
| `app/views/products/new.phtml`            | View for the action `new`    |
| `app/views/products/edit.phtml`           | View for the action `edit`   |

When browsing the recently generated controller, you will see a search form and a link to create a new Product:

![](/assets/images/content/devtools-usage-03.png)

The `create page` allows you to create products applying validations on the Products model. Phalcon will automatically validate not null fields producing warnings if any of them is required.

![](/assets/images/content/devtools-usage-04.png)

After performing a search, a pager component is available to show paged results. Use the "Edit" or "Delete" links in front of each result to perform such actions.

![](/assets/images/content/devtools-usage-05.png)

### Web Interface to Tools
Also, if you prefer, it's possible to use Phalcon Developer Tools from a web interface. Check out the following screencast to figure out how it works:

<div align="center">
<iframe src="https://player.vimeo.com/video/42367665" width="500" height="266" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>
</div>

### Integrating Tools with PhpStorm IDE
The screencast below shows how to integrate developer tools with the [PhpStorm IDE][phpstorm]. The configuration steps could be easily adapted to other IDEs for PHP.

<div align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/UbUx_6Cs6r4" frameborder="0" allowfullscreen></iframe>
</div>
[phpstorm]: https://www.jetbrains.com/phpstorm/
