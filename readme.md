# Modular Extensions (HMVC) for CodeIgniter 3

> This is the fork of the awesome https://bitbucket.org/wiredesignz/codeigniter-modular-extensions-hmvc project.

Modular Extensions makes the CodeIgniter PHP framework modular. Modules are groups of independent components, 
typically model, controller and view, arranged in an application modules sub-directory that can be dropped into other CodeIgniter applications.

HMVC stands for Hierarchical Model View Controller.

Module Controllers can be used as normal Controllers or HMVC Controllers and they can be used as widgets to help you build view partials.

## Installation

**Step 1.** Install package with composer:

```bash
composer require taufikmaulanaa/ci-hmvc
```

**Step 2.** Create two files in your `application/core` directory:

*MY_Loader.php*
```php
<?php

class MY_Loader extends MX_Loader {}
```

*MY_Router.php*
```php
<?php

class MY_Router extends MX_Router {}
```

### Alternative installation

In case if you're not using composer, just download the repository and copy its `core` and `third_party` directories to your `application` directory.

## Configuration

You may want to configure where your modules should be loaded from.

Just set the `$config['modules_locations']` in your `application/config/config.php`:
```php
$config['modules_locations'] = array(
    APPPATH.'modules/' => '../modules/',
);
```

## Features

All controllers can contain an `$autoload` property, which holds an array of items to load prior to running the constructor.
This can be used together with `module/config/autoload.php`, however using the `$autoload` variable only works for that specific controller.

```php
class Xyz extends MX_Controller
{
    public $autoload = array(
        'helper'    => array('url', 'form'),
        'libraries' => array('email'),
    );
}
```

`Modules::run()` output is buffered, so any data returned or output directly from the controller is caught and
returned to the caller. In particular, `$this->load->view()` can be used as you would in a normal controller, without the need for return.

Controllers can be loaded as class variables of other controllers using `$this->load->module('module/controller')`;
or simply $this->load->module('module'); if the controller name matches the module name.

Any loaded module controller can then be used like a library, ie: `$this->controller->method()`, but it has access to its own models and libraries independently from the caller.

All module controllers are accessible from the URL via module/controller/method or simply module/method if the module and controller names match.
If you add the `_remap()` method to your controllers you can prevent unwanted access to them from the URL and redirect or flag an error as you like.

### Notes:

To use HMVC functionality, such as `Modules::run()`, controllers must extend the `MX_Controller` class.

To use Modular Separation only, without HMVC, controllers will extend the CodeIgniter Controller class.

You must use PHP5 style constructors in your controllers. ie:

```php
class Xyz extends MX_Controller
{
    public function __construct()
    {
        parent::__construct();
    }
}
```

Constructors are not required unless you need to load or process something when the controller is first created.

All `MY_` extension libraries should include (require) their equivalent MX library file and extend their equivalent `MX_` class.

Each module may contain a `config/routes.php` file where routing and a default controller can be defined for that module using:
	
```php
$route['module_name'] = 'controller_name';
```

Controllers may be loaded from `application/controllers` sub-directories.

Controllers may also be loaded from `module/controllers` sub-directories.
 
Resources may be cross loaded between modules. ie: `$this->load->model('module/model');`

`Modules::run()` is designed for returning view partials, and it will return buffered output (a view) from a controller. The syntax for using modules::run is a URI style segmented string and unlimited variables.
	
```php
/** module and controller names are different, you must include the method name also, including 'index' **/
Modules::run('module/controller/method', $params, ...$more);

/** module and controller names are the same but the method is not 'index' **/
Modules::run('module/method', $params, ...$more);

/** module and controller names are the same and the method is 'index' **/
Modules::run('module', $params, ...$more);

/** Parameters are optional, You may pass any number of parameters. **/
```

To call a module controller from within a controller you can use `$this->load->module()` or `Modules::load()` and PHP5 method chaining is available for any object loaded by MX, ie: `$this->load->library(‘validation’)->run()`.

To load languages for modules it is recommended to use the Loader method which will pass the active module name to the Lang instance; ie: `$this->load->language('language_file')`;

The PHP5 spl_autoload feature allows you to freely extend your controllers, models and libraries from `application/core` or `application/libraries` base classes without the need to specifically include or require them.

The library loader has also been updated to accommodate some CI 1.7 features: ie Library aliases are accepted in the same fashion as model aliases, and loading config files from the module config directory as library parameters (re: form_validation.php) have beed added.

`$config = $this->load->config(‘config_file’)` returns the loaded config array to your variable.

Models and libraries can also be loaded from sub-directories in their respective application directories.

When using form validation with MX you will need to extend the `CI_Form_validation` class as shown below:

```php
<?php

/** application/libraries/MY_Form_validation **/
class MY_Form_validation extends CI_Form_validation
{
    public $CI;
}
```

before assigning the current controller as the $CI variable to the form_validation library.

This will allow your callback methods to function properly. (This has been discussed on the CI forums also).
	
```php
<?php

class Xyz extends MX_Controller
{
    function __construct()
    {
        parent::__construct();

        $this->load->library('form_validation');
        $this->form_validation->CI =& $this;
    }
}
```

### View Partials

Using a Module as a view partial from within a view is as easy as writing:

```php
<?=Modules::run('module/controller/method', $param, ...$more);?>
```

Parameters are optional, You may pass any number of parameters.

### FAQ

Q. What are modules, why should I use them?

A. http://en.wikipedia.org/wiki/Module

   http://en.wikipedia.org/wiki/Modular_programming

   http://blog.fedecarg.com/2008/06/28/a-modular-approach-to-web-development


Q. What is Modular HMVC, why should I use it?

A. Modular HMVC = Hierarchy of multiple MVC triads

This is most useful when you need to load a view and its data within a view. Think about adding a shopping cart to a page. The shopping cart needs its own controller which may call a model to get cart data.
Then the controller needs to load the data into a view. So instead of the main controller handling the page and the shopping cart, the shopping cart MVC can be loaded directly in the page.
The main controller doesn’t need to know about it, and is totally isolated from it.

In CI we can’t call more than 1 controller per request. Therefore, to achieve HMVC, we have to simulate 
controllers. It can be done with libraries, or with this “Modular Extensions HMVC” contribution.

The differences between using a library and a “Modular HMVC” class is:
1. No need to get and use the CI instance within an HMVC class.
2. HMVC classes are stored in a modules directory as opposed to the libraries directory.

Q. Is Modular Extensions HMVC the same as Modular Separation?

A. Yes and No. Like Modular Separation, Modular Extensions makes modules “portable” to other installations. For example, if you make a nice self-contained model-controller-view set of files you can bring that MVC into another project by copying just one folder - everything is in one place instead of spread around model, view and controller folders.

Modular HMVC means modular MVC triads. Modular Separation and Modular Extensions allows related controllers, models, libraries, views, etc. to be grouped together in module directories and used like a mini application. But, Modular Extensions goes one step further and allows those modules to “talk” to each other. You can get controller output without having to go out through the http interface again.