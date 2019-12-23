# Cookbook - Scaffolding

## Installation
To install extension:

```bash
$ composer require spiral/scaffolder
```

Make sure to add `Spiral\Scaffolder\Bootloader\ScaffolderBootloader` to your App class:

```php
use Spiral\Scaffolder\Bootloader\ScaffolderBootloader;

class App extends Kernel
{
    // ...

    protected const APP = [
        // ...
        ScaffolderBootloader::class
    ];
}
```

> Attention, the extension will invoke `TokenizerConfig`, make sure it is add after the end of bootload chain.

## Configuration
You can customize scaffolder component by replacing declaration generators and their options using `scaffolder` configuration file.
Default configuration is located in the `ScaffolderBootloader`

## Available Commands
Command            | Description
---                | ---
create:bootloader  | Create bootloader declaration
create:command     | Create command declaration
create:config      | Create config declaration
create:controller  | Create controller declaration
create:filter      | Create HTTP Request Filter declaration
create:middleware  | Create middleware declaration
create:migration   | Create migration declaration
create:repository  | Create Entity Repository declaration
create:entity      | Create Entity declaration

### Create bootloader
```bash
$ php app.php create:bootloader <name>
```

`<Name>Bootloader` class will be created.

#### Example
```bash
$ php app.php create:bootloader my
```

Output is:
```php
use Spiral\Boot\Bootloader\Bootloader;

class MyBootloader extends Bootloader
{
    protected const BINDINGS = [];

    protected const SINGLETONS = [];

    protected const DEPENDENCIES = [];

    public function boot(): void
    {
    }
}
```

### Create command
```bash
$ php app.php create:command <name> [<alias>]
```

`<Name>Command` class will be created. Command executable name will be set to `name` or `alias` if alias is set.

#### Example without `alias` option
```bash
$ php app.php create:bootloader my
```

Output is:
```php
use Spiral\Console\Command;

class MyCommand extends Command
{
    protected const NAME = 'my';

    protected const DESCRIPTION = '';

    protected const ARGUMENTS = [];

    protected const OPTIONS = [];

    /**
     * Perform command
     */
    protected function perform(): void
    {
    }
}
```

#### Example with `alias` option
```bash
$ php app.php create:bootloader my alias
```

Output is:
```php
use Spiral\Console\Command;

class MyCommand extends Command
{
    protected const NAME = 'alias';

    protected const DESCRIPTION = '';

    protected const ARGUMENTS = [];

    protected const OPTIONS = [];

    /**
     * Perform command
     */
    protected function perform(): void
    {
    }
}
```

### Create config
```bash
$ php app.php create:config <name>
```

`<Name>Config` class will be created. Also, `<app directory>/config/<name>.php` file will be created if doesn't exists.
Available options:
* `reverse (r)` - Using this flag, scaffolder will look for `<app directory>/config/<name>.php` file and create a rich `<Name>Config` class based on the given config file.
Class will include default values and getters, in some cases it will also include by-key-getters for array values. Details below:

If an array-value consists of more than 1 sub-values with the same types for keys and sub-values,
scaffolder will try to create a by-key-getter method.
If a generated key is conflicting with an existing method, by-key-getter will be omitted.
#### Example with empty config file
```bash
$ php app.php create:config my
```

Output config file:
```php
return [];
```

Output config class:
```php
use Spiral\Core\InjectableConfig;

class MyConfig extends InjectableConfig
{
    public const CONFIG = 'my';

    /**
     * @internal For internal usage. Will be hydrated in the constructor.
     */
    protected $config = [];
}
```

#### Example with reversing
```php
//...existing "my.php" config file:
return [
    //will create "getParam()" by-key-getter (successfully singularized name)
    'params'      => [
        'one' => 'param',
        'two' => 'another param',
    ],
    //will create "getParameterBy()" by-key-getter (unsuccessfully singularized name)
    'parameter'   => [
        'one' => 'parameter',
        'two' => 'another parameter',
    ],
    //will create "getValueBy()" by-key-getter (because "getValue()" conflicts with the next "value" field)
    'values'      => [
        1 => 'value',
        2 => 'another value',
    ],
    'value'       => 'third value',
    //won't create by-key-getter due to only 1 sub-value
    'few'        => [
        'one' => 'value',
    ],
    //won't create by-key-getter due to mixed values
    'mixedValues' => [
        'one' => 'value',
        'two' => 2,
    ],
    //won't create by-key-getter due to mixed keys
    'mixedKeys'   => [
        'one' => 'value',
        2     => 'another value',
    ],
    //won't create by-key-getter to name conflicts
    //(because "getConflict()" and "getConflictBy" conflicts with the next "conflict" and "conflictBy" field)
    'conflicts'   => [
        'one' => 'conflict',
        'two' => 'another conflict',
    ],
    'conflict'    => 'third conflic',
    'conflictBy'  => 'fourth conflic',
];
```

Output is:
```php
use Spiral\Core\InjectableConfig;

class MyConfig extends InjectableConfig
{
    public const CONFIG = 'my';

    /**
     * @internal For internal usage. Will be hydrated in the constructor.
     */
    protected $config = [
        'params'      => [],
        'parameter'   => [],
        'values'      => [],
        'value'       => '',
        'few'         => [],
        'mixedValues' => [],
        'mixedKeys'   => [],
        'conflicts'   => [],
        'conflict'    => '',
        'conflictBy'  => ''
    ];

    /**
     * @return array|string[]
     */
    public function getParams(): array
    {
        return $this->config['params'];
    }

    /**
     * @return array|string[]
     */
    public function getParameter(): array
    {
        return $this->config['parameter'];
    }

    /**
     * @return array|string[]
     */
    public function getValues(): array
    {
        return $this->config['values'];
    }

    /**
     * @return string
     */
    public function getValue(): string
    {
        return $this->config['value'];
    }

    /**
     * @return array|string[]
     */
    public function getFew(): array
    {
        return $this->config['few'];
    }

    /**
     * @return array
     */
    public function getMixedValues(): array
    {
        return $this->config['mixedValues'];
    }

    /**
     * @return array|string[]
     */
    public function getMixedKeys(): array
    {
        return $this->config['mixedKeys'];
    }

    /**
     * @return array|string[]
     */
    public function getConflicts(): array
    {
        return $this->config['conflicts'];
    }

    /**
     * @return string
     */
    public function getConflict(): string
    {
        return $this->config['conflict'];
    }

    /**
     * @return string
     */
    public function getConflictBy(): string
    {
        return $this->config['conflictBy'];
    }

    /**
     * @param string param
     * @return string
     */
    public function getParam(string $param): string
    {
        return $this->config['params'][$param];
    }

    /**
     * @param string parameter
     * @return string
     */
    public function getParameterBy(string $parameter): string
    {
        return $this->config['parameter'][$parameter];
    }

    /**
     * @param int value
     * @return string
     */
    public function getValueBy(int $value): string
    {
        return $this->config['values'][$value];
    }
}
```

### Create controller
```bash
$ php app.php create:controller <name>
```
`<Name>Controller` class will be created.
You can optionally specify controller actions using `action (a)` option (multiple values allowed).

#### Example with empty actions list
```bash
$ php app.php create:controller my
```

Output is:
```php
class MyController
{
}
```

#### Example with actions list
```bash
$ php app.php create:controller my -a index -a create -a update -a delete
```

Output is:
```php
class MyController
{
    public function index()
    {
    }

    public function create()
    {
    }

    public function update()
    {
    }

    public function delete()
    {
    }
}
```

### Create HTTP request filter
```
$ php app.php create:filter <name>
```
`<Name>Filter` class will be created.
Using option `entity (e)` you can pass an `EntityClass` and the filter command will fetch the all the given
class properties into the filter and try to define each property's type based on its type declaration (if php74),
default value or a PhpDoc. Otherwise you can optionally specify filter schema using `field (f)` option (multiple values allowed).<br/>
Full field format is `name:type(source:origin)`. `type`, `origin` and `source:origin` are optional and can be omitted, defaults are:
* type=string
* source=data
* origin=\<name\>
> See more about filters in [filters](https://github.com/spiral/filters) package

### Create middleware
```
$ php app.php create:middleware <name>
```
`<Name>Middlweare` class will be created.

### Create migration
```
$ php app.php create:migration <name>
```
`<Name>Migration` class will be created.
You can optionally specify table name using `table (t)` option and columns using `column (col)` option (multiple values allowed).
Column format is `name:type`.
> See more about migrations in [migrations](https://github.com/spiral/migrations) package

### Create repository
```
$ php app.php create:repository <name>
```
`<Name>Repository` class will be created.
 
### Create entity
```
$ php app.php create:repository <name> [<format>]
```
`<Name>Entity` class will be created.
`format` is responsible for the declaration format, currently only [annotations](https://github.com/cycle/annotated) is supported. 
Available options:
* `role (r)` - Entity role, defaults to lowercase class name without a namespace
* `mapper (m)` - Mapper class name, defaults to Cycle\ORM\Mapper\Mapper
* `table (t)` - Entity source table, defaults to plural form of entity role
* `accessibility (a)` - accessibility accessor (public, protected, private), defaults to public
* `inflection (i)` - Optional column name inflection, allowed values: tableize (or t), camelize (or c). See [Doctrine inflector](https://github.com/doctrine/inflector)
* `field (f)` - Add field in a format "name:type" (multiple values allowed)
* `repository (e)` - Repository class to represent read operations for an entity, defaults to `Cycle\ORM\Select\Repository`
* `database (d)` - Database name, defaults to null (default database)
