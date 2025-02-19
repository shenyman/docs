---
layout: default
upgrade: '#flash'
title: 'Flash Messages'
keywords: 'flash, flash messages, flash direct, flash session, templates'
---
# Flash Messages
- - -
![](/assets/images/document-status-stable-success.svg) ![](/assets/images/version-{{ pageVersion }}.svg)

## Overview
Flash messages are used to notify the user about the state of actions he/she made or simply show information to the users. These kinds of messages can be generated using this component.

## Adapters
This component uses adapters that dictate how messages are displayed or sent to the view. There are two adapters available, but you can easily create your own adapter using the [Phalcon\Flash\FlashInterface][flash-flashinterface] interface.

| Adapter                                | Description                                                                                  |
|----------------------------------------|----------------------------------------------------------------------------------------------|
| [Phalcon\Flash\Direct][flash-direct]   | Directly outputs the messages passed to the flasher                                          |
| [Phalcon\Flash\Session][flash-session] | Temporarily stores the messages in session, then messages can be printed in the next request |

### Direct
[Phalcon\Flash\Direct][flash-direct] can be used to directly output messages set in the component. This is useful in instances where you need to show data back to the user in the current request that does not employ any redirects.

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

$flash->error('Something went wrong');
```

### Session
[Phalcon\Flash\Session][flash-session] can be used to output messages set in the component. The component transparently stores the messages in the session to be used after a redirect. 

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Session as FlashSession;
use Phalcon\Session\Adapter\Stream;
use Phalcon\Session\Manager;

$session = new Manager();
$files = new Stream(
    [
        'savePath' => '/tmp',
    ]
);
$session->setHandler($files);

$escaper = new Escaper();
$flash   = new FlashSession($escaper, $session);

$flash->error('Something went wrong');
```

and in your view

```php
<?php echo $flash->output(); ?>
```

or when using Volt

```twig
{% raw %}{{ flash.output() }}{% endraw %}
```

Imagine a login form that you need to validate the username and password and inform the user if their credentials are correct. The [Phalcon\Flash\Session][flash-session] can be used to perform this task as follows:  
- User enters credentials and clicks `Login`
- The application posts the data to the `loginAction` of our controller
- The application checks the data and the combination is not correct
- The application stores the `Incorrect Credentials` message in the flash messenger
- The application redirects the user back to the login page (`/login`)
- The Flash messenger still holds the message `Incorrect Credentials` and will display it on the screen.

The example below displays this behavior in the controller. If an error occurs, whether this is an actual application error or a result of incorrect credentials, the code sets the messages using `$this->flash->error()`.

```php
<?php

namespace MyApp;

use Phalcon\Flash\Session;
use Phalcon\Http\Request;
use Phalcon\Http\Response;
use Phalcon\Mvc\Controller;
use Phalcon\Mvc\View;
use Vokuro\Auth\Auth; 
use Vokuro\Models\Users;
use Vokuro\Models\ResetPasswords;

/**
 * Controller used handle non-authenticated session actions like 
 * login/logout, user signup, and forgotten passwords
 *
 * @property Auth     $auth
 * @property Request  $request
 * @property Response $response
 * @property Session  $flashSession
 * @property View     $view
 */
class SessionController extends Controller
{
    /**
     * Starts a session in the admin backend
     */
    public function loginAction()
    {
        $form = new LoginForm();

        try {
            if (true !== $this->request->isPost()) {
                // ....
            } else {
                $postData = $this->request->getPost();
                if (true !== $form->isValid($postData)) {
                    // Flash
                    foreach ($form->getMessages() as $message) {
                        $this->flashSession->error($message);
                    }
                } else {
                    $email    = $this->request->getPost('email');
                    $password = $this->request->getPost('password');
                    $remember = $this->request->getPost('remember');

                    $this->auth->check(
                        [
                            'email'    => $email,
                            'password' => $password,
                            'remember' => $remember,
                        ]
                    );
                    
                    return $this->response->redirect('users');
                }
            }
        } catch (AuthException $e) {
            // Flash
            $this->flashSession->error($e->getMessage());
        }

        $this->view->form = $form;
    }
}
```

and in your view

```php
<?php echo $flashSession->output(); ?>
```

or when using Volt

```twig
{% raw %}{{ flashSession.output() }}{% endraw %}
```

> **NOTE**: In the above example, the `flashSession` service has been already registered in the DI container. For more information about this please check the relevant section below.
{: .alert .alert-info }

## Styling
The component (irrespective of adapter) offers automatic styling of messages on screen. This means that messages will be wrapped in `<div>` tags. There is also a mapping of message type to CSS class that you can take advantage of based on the stylesheet you use in your application. By default, the component uses the following mapping:

| Type      | Name of CSS class |
|-----------|-------------------|
| `error`   | `errorMessage`    |
| `notice`  | `noticeMessage`   |
| `success` | `successMessage`  |
| `warning` | `warningMessage`  |

By using the default classes, you can style `errorMessage` accordingly in the stylesheet of your application to make it appear the way you want it to. It is common for error messages to have a red background for instance so that they stand out. 

An error message:

```php
$flash->error('Error message');
```

will produce:

```html
<div class="errorMessage">Error message</div>
```

If you do not wish to use the default classes, you can use the `setCssClasses()` method to replace the mapping of type of message to class name.

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

var_dump(
    $flash->getCssClasses()
);

// [
//     "error"   => "errorMessage",
//     "notice"  => "noticeMessage",
//     "success" => "successMessage",
//     "warning" => "warningMessage",
// ];


$cssClasses = [
    'error'   => 'alert alert-danger',
    'success' => 'alert alert-success',
    'notice'  => 'alert alert-info',
    'warning' => 'alert alert-warning',
];

$flash->setCssClasses($cssClasses);
```

and then calling 

```php
$flash->error('Error message');
```

will produce:

```html
<div class="alert alert-danger">Error message</div>
```

> **NOTE**: The `setCssClasses()` returns back the object, so you can use in a more fluent interface by chaining calls.
{: .alert .alert-info }

The component also allows you to specify a different template, so that you can control the HTML produced by the component. The `setCustomTemplate()` and `getCustomTemplate()` expose this functionality. The template needs to have two placeholders:

| Placeholder  | Description                          |
|:------------:|--------------------------------------|
| `%cssClass%` | where the CSS class will be injected |
| `%message%`  | where the message will be injected   |

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

$template = '<span class="%cssClass%">%message%</span>';

$flash->setCustomTemplate($template);
```

and then calling 

```php
$flash->error('Error message');
```

will produce:

```html
<span class="myErrorClass">Error message</span>
```

> **NOTE**: The `setCustomTemplate()` returns back the object, so you can use in a more fluent interface by chaining calls.
{: .alert .alert-info }

You can also set the icon class for each CSS class by using `setCssIconClasses()`. This is particularly useful when working with CSS libraries such as [Bootstrap][bootstrap].

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

$iconClasses = [
    'error'   => 'alert alert-error',
    'success' => 'alert alert-success',
    'notice'  => 'alert alert-notice',
    'warning' => 'alert alert-warning',
];

$flash->setCssIconClasses($iconClasses);
```

and then calling

```php
$flash->error('Error message');
```

will produce:

```html
<div class="errorMessage"><i class="alert alert-error"></i>Error message
```

> **NOTE**: The `setCssIconClasses()` returns back the object, so you can use in a more fluent interface by chaining calls.
{: .alert .alert-info }

An example of how the `setCssClasses`, `setCssIconClasses` and `setCustomTemplate` can be used to output flash messages that can be _closed_ is below:

```php
<?php

use Phalcon\Di\FactoryDefault;
use Phalcon\Flash\Session;

$container = new FactoryDefault();

$container->set(
    'flashSession',
    function () {
        $flash = new Session();

        $flash->setCssClasses(
            [
                'error'   => 'error_message callout alert radius flashSession',
                'success' => 'success_message callout success radius flashSession',
                'warning' => 'warning_message callout warning radius flashSession',
                'notice'  => 'notice_message callout secondary radius flashSession'
            ]
        );

        $flash->setCssIconClasses(
            [
                'error'   => 'fi-alert',
                'success' => 'fi-check',
                'notice'  => 'fi-star',
                'warning' => 'fi-flag',
            ]
        );

        $template = '<div class="%cssClass%">
    <i class="%cssIconClass%"></i> %message%
    <button class="close-button" aria-label="Close" type="button" data-close>
        <span aria-hidden="true">&times;</span>
    </button>
</div>';
        $flash->setCustomTemplate($template);

        $flash->setAutoescape(false);

        return $flash;
    }
);
```
If you then call:

```php
$this->flashSession->error('An error has occurred. Please contact support.')
```
will produce the following HTML snippet in your view (when calling `$flashSession->output()`:
```html
<div class="error_message callout alert radius flashSession">
    <i class="fi-alert"></i> An error has occurred. Please contact support.
    <button class="close-button" aria-label="Close" type="button" data-close>
        <span aria-hidden="true">&times;</span>
    </button>
</div>
```

## Messages
As mentioned above, the component has different types of messages. To add a message to the component you can call `message()` with the type as well as the message itself. The types of messages are:

- `error`
- `notice`
- `success`
- `warning`
 
```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

$flash->message('error', 'Error message');
```

While you can pass the type as the first parameter when calling `message()` you can also use the relevant helper methods that do that for you:

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

$flash->error('Error message');
$flash->notice('Notice message');
$flash->success('Success message');
$flash->warning('Warning message');
```

If your application requires it, you might want to clear the messages at some point when building the response. To do so you can use the `clear()` method. 

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

$flash->error('Error message');

$flash->clear();
```

> **NOTE**: `clear()` works only when the implicit flush is disabled (`setImplicitFlush(false)`)
{: .alert .alert-info }

## Implicit Flush
By default, implicit flushing is set to `true`. You can however turn it off by using `setImplicitFlush(false)`. The purpose of this method is to set whether the output must be implicitly flushed to the output or returned as string

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

echo $flash->getImplicitFlush(); // true

$flash->error('Error'); // No output

echo $flash
    ->setImplicitFlush(false) 
    ->error('Error Message') // 'Error Message'
;
```

> **NOTE**: The `setImplicitFlush()` returns back the object, so you can use in a more fluent interface by chaining calls.
{: .alert .alert-info }

> **NOTE**: When using the [Phalcon\Flash\Direct][flash-direct] component, to directly show results on the page you **must** set `setImplicitFlush()` to `false`.
{: .alert .alert-warning }

## Escaping
By default, the component will escape the contents of the message. There might be times however that you do not wish to escape the contents of your messages. You can use the `setAutoescape(false)`;

```php
<?php

use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$escaper = new Escaper();
$flash   = new Direct($escaper);

echo $flash->getAutoescape(); // true

$flash
    ->setAutoescape(false)
    ->error('<h1>Error</h1>')
;
```

will produce

```html
<div class="errorMessage">&lt;h1&gt;Error&lt;/h1&gt;</div>
```

> **NOTE**: The `setAutoescape()` returns back the object, so you can use in a more fluent interface by chaining calls.
{: .alert .alert-info }

## Dependency Injection
If you use the [Phalcon\Di\FactoryDefault][factorydefault] container, the [Phalcon\Flash\Direct][flash-direct] is already registered for you with the name `flash`. Additionally, the [Phalcon\Flash\Session][flash-session] is already registered for you with the name `flashSession`. 

An example of the registration of the service as well as accessing it is below:

**Direct**

```php
<?php

use Phalcon\Di;
use Phalcon\Escaper;
use Phalcon\Flash\Direct;

$container = new Di();
$escaper   = new Escaper();

$container->set(
    'flash',
    function () use ($escaper) {
        return new Direct($escaper);
    }
);
```

**Session**

```php
<?php

use Phalcon\Di;
use Phalcon\Escaper;
use Phalcon\Flash\Session as FlashSession;
use Phalcon\Session\Adapter\Stream;
use Phalcon\Session\Manager;

$container = new Di();
$escaper   = new Escaper();
$session   = new Manager();
$files     = new Stream(
    [
        'savePath' => '/tmp',
    ]
);
$session->setHandler($files);

$container->set(
    'flashSession',
    function () use ($escaper, $session) {
        return new FlashSession($escaper, $session);
    }
);
```

> **NOTE** You do not need to pass the escaper or the session in the constructor. If you use the Di container and those services are already register in it, they will be used internally. This is another way of instantiating the components.
{: .alert .alert-info }

You can now use the component in a controller (or a component that implements [Phalcon\Di\Injectable][di-injectable])

```php
<?php

namespace MyApp;

use Phalcon\Flash\Direct;
use Phalcon\Mvc\Controller;

/**
 * Invoices controller
 *
 * @property Direct $flash
 */
class InvoicesController extends Controller
{
    public function indexAction()
    {

    }

    public function saveAction()
    {
        $this->flash->success('The post was correctly saved!');
    }
}
```

[flash-abstractflash]: api/phalcon_flash#flash-abstractflash
[flash-direct]: api/phalcon_flash#flash-direct
[flash-exception]: api/phalcon_flash#flash-exception
[flash-flashinterface]: api/phalcon_flash#flash-flashinterface
[flash-session]: api/phalcon_flash#flash-session
[di-injectable]: api/phalcon_di#di-injectable
[factorydefault]: api/phalcon_di#di-factorydefault
