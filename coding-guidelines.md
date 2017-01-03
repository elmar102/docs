Coding Guidelines
=================

Do your best to follow these guidelines when writing code for Craft and Craft plugins.

- [Code Style](#code-style)
- [Best Practices](#best-practices)
- [Namespaces & Class Names](#namespaces--class-names)
- [Method Names](#method-names)
- [Docblocks](#docblocks)
  - [Interfaces vs. Implementation Classes](#interfaces-vs-implementation-classes)
- [Control Flow](#control-flow)
  - [Happy Paths](#happy-paths)
  - [`if`…`return`…`else`](#ifreturnelse)
  - [Service Action Methods](#service-action-methods)
- [Controllers](#controllers)
  - [Return Types](#return-types)
  - [JSON Actions](#json-actions)
- [Exceptions](#exceptions)
- [DB Queries](#db-queries)
  - [Conditions](#conditions)
- [Getters & Setters](#getters--setters)
  - [App Component Getters](#app-component-getters)

## Code Style

- Follow the [PSR-1](http://www.php-fig.org/psr/psr-1/) & [PSR-2](http://www.php-fig.org/psr/psr-2/) coding standards.
- Use the short array syntax (`['foo' => 'bar']`).
- Don’t fret too much over line lengths. Focus on readability.
- Chained method calls should each be placed on their own line, with the `->` operator at the beginning of each line.
- Strings that are concatenated across multiple lines should have the `.` operator at the ends of lines.
- Don’t put a space after type typecasts (`(int)$foo`).
- Put a blank line before `return` statements.

## Best Practices

- Declare method argument types whenever possible.

    ```php
    public function foo(Entry $entry, array $settings)
    ```

- Use strict comparison operators (`===` and `!==`) whenever possible.
- Use `$foo === null`/`$bar !== null` rather than `is_null($foo)`/`!is_null($bar)`.
- Use `(int)$foo`/`(float)$bar` rather than `intval($foo)`/`floatval($bar)`.
- Always pass `true`/`false` to the third argument of [`in_array()`](http://php.net/manual/en/function.in-array.php) to indicate whether the check should be type-script (and make it `true` whenever possible).
- Use `$obj->property !== null` rather than `isset($obj->property)` in conditions that check if an object property is set.
- Use `empty()`/`!empty()` in conditions that check if an array is/isn’t empty.
- Refer to class names using the [`::class` keyword](http://php.net/manual/en/language.oop5.basic.php#language.oop5.basic.class.class) (`Foo::class`) rather than as a string (`'some\nmspace\Foo'`) or Yii’s [`::className()`](http://www.yiiframework.com/doc-2.0/yii-base-object.html#className()-detail) static method (which is going to be removed in Yii 2.1).
- Initialize arrays explicitly (`$array = []`) rather than implicitly (e.g. `$array[] = 'foo'` where `$array` wasn’t defined yet).
- Use `self::_foo()` rather than `static::_foo()` when calling private static functions, since `static::` would break if the class is extended.
- Use `self::CONSTANT` rather than `static::CONSTANT` (unnecessary overhead).
- Only use the `parent::` keyword when calling a parent method with the exact same name as the current method. Otherwise use `$this->`.
- Always specify the visibility of class properties and methods (`public`, `protected`, or `private`).
- Private class property/method names should begin with an underscore (`private $_foo`).
- Don’t explicitly set class properties’ default values to `null` (e.g. `public $foo = null;`).
- Always use `require` or `include` when including a file that returns something, rather than `require_once` or `include_once`.
- Use `strpos($foo, $bar) === 0` rather than `strncmp($foo, $bar, $barLength) === 0` when checking if one string begins with another string, for short strings.
- Use `$str === ''` rather than `strlen($str) === 0` when checking if a string is empty.
- Avoid using `array_merge()` within loops when possible.
- Unset variables created by reference in foreach-loops after the loop is finished.

    ```php
    foreach ($array as &$value) {
        // ...
    }
    unset($value);
    ```

- Use `implode()` rather than `join()`.
- Use `in_array()` rather than `array_search(...) !== false` when the position of the needle isn’t needed.
- Don’t use a `switch` statement when a single `if` condition will suffice.
- Use single quotes (`'`) whenever double quotes (`"`) aren’t needed.
- Use shortcut operators (`+=`, `-=`, `*=`, `/=`, `%=`, `.=`, etc.) whenever possible.
- Use shortcut regex patterns (`\d`, `\D`, `\w`, `\W`, etc.) whenever possible.
- Use the `DIRECTORY_SEPARATOR` constant rather than `'/'` when defining file paths.

> {tip} The [Php Inspections (EA Extended)](https://plugins.jetbrains.com/idea/plugin/7622-php-inspections-ea-extended-) PhpStorm plugin can help you locate and fix these sorts of best practice issues.

## Namespaces & Class Names

- Follow the [PSR-4](http://www.php-fig.org/psr/psr-4/) specification, where a class’s file location can be inferred by its fully qualified name, given a known base namespace mapped to a base path.
- Namespaces should be all-lowercase.
- Class names should be `StudlyCase`.
- Only first party code should use the `craft\` and `pixelandtonic\` namespace roots. Third party plugins should use a namespace root that refers to the vendor name and plugin name (e.g. `acme\pluginhandle\`).

## Method Names

Getter methods (methods whose primary responsibility is to return something, rather than do something) that **don’t accept any arguments** should begin with `get` , and there should be a corresponding `@property` tag in the class’s docblock to document the corresponding magic getter property.

- `getAuthor()`
- `getIsSystemOn()`
- `getHasFreshContent()`

Getter methods that **accept one or more arguments** (regardless of whether they can be omitted) should only begin with `get` if it “sounds right”.

- `getError($attribute)`
- `hasErrors($attribute = null)`

Static methods should generally not start with `get`.

  - `className()`
  - `displayName()`

## Docblocks

- Methods that override subclass methods or implement an interface method, and don’t have anything relevant to add to the docblock, should only have `@inheritdoc` in the docblock.
- Use full sentences with proper capitalization, grammar, and punctuation in docblock descriptions.
- `@param` and `@return` tags should **not** have proper capitalization or punctuation.
- Use `boolean` and `integer` instead of `bool` and `int` in type declarations.
- Specify array members’ class names in array type declarations when it makes sense (`ElementInterface[]` rather than `array`).
- Chainable functions that return an instance of the current class should use `static` as the return type declaration.
- Functions that don’t ever return anything should have `@return void`.

### Interfaces vs. Implementation Classes

`@param` , `@return` , `@var` , `@method` and `@property` tags on public service methods should reference Interfaces (when applicable), not their implementation class:

```php
// Bad:
/**
 * @param Element $element
 * @param ElementInterface|Element $element
 */

// Better:
/**
 * @param ElementInterface $element
 */
```

Inline `@var` tags should reference implementation classes, not their interfaces:

```php
// Bad:
/** @var ElementInterface $element */
/** @var ElementInterface|Element $element */

// Better:
/** @var Element $element */
```

## Control Flow

### Happy Paths

Use [them](https://en.wikipedia.org/wiki/Happy_path). In general the execution of a method should only make it all the way to the end if everything went as expected.

```php
// Bad:
if ($condition) {
    // Do stuff

    return true;
}

return false;

// Better:
if (!$condition) {
    return false;
}

// Do stuff

return true;
```

### `if`…`return`…`else`

Don’t do this. There’s no point, and can be misleading at first glance.

```php
// Bad:
if ($condition) {
    return $foo;
} else {
    return $bar;
}

// Better:
if ($condition) {
    return $foo;
}

return $bar;
```

## Service Action Methods

Service methods that perform an action should generally follow the following control flow:

1. Perform **validation** on the model, if `$runValidation == true`
    - If validation fails, log the failure and return `false`
2. Fire a **beforeX event**
    - *(Note: We no longer care if *`*$event->isValid == false*`* )*
3. Start a **DB transaction** and begin a `try`…`catch` block
    - *(Note: Only necessary if multiple things are about to be changed or if it’s an interface-oriented operation)*
4. If it’s an interface-oriented operation so the model’s class isn’t known, call `beforeX()` on the model
    - If `beforeX()` returns `false`, then rollback the transaction and return `false`
5. **Perform the operation**
    - *(Note: there should be no additional validation at this point, since we’ve already dealt with that in step 1.)*
6. If it’s an interface-oriented operation, call `afterX()` on the model
7. If a DB transaction was created, commit it and close out the `try`…`catch` block
8. Fire an `afterX` event
9. Return `true`

#### Example 1: Class-oriented operation

This is an example of how to save a thing that needs to be of a certain class, where `beforeX()` and `afterX()` methods are not necessary on the model.

```php
public function saveGroup(FieldGroup $group, $runValidation = true)
{
    if ($runValidation && !$group->validate()) {
        Craft::info('Field group not saved due to validation error.', __METHOD__);

        return false;
    }

    $isNewGroup = !$group->id;

    // Fire a 'beforeSaveFieldGroup' event
    $this->trigger(self::EVENT_BEFORE_SAVE_FIELD_GROUP, new FieldGroupEvent([
        'group' => $group,
        'isNew' => $isNewGroup,
    ]));

    // ...
    // Save the field group here
    // ...

    // Fire an 'afterSaveFieldGroup' event
    $this->trigger(self::EVENT_AFTER_SAVE_FIELD_GROUP, new FieldGroupEvent([
        'group' => $group,
        'isNew' => $isNewGroup,
    ]));

    return true;
}
```

#### Example 2: Interface-oriented operation

This is an example of how to save a thing that needs to be of a certain interface, but the actual class is not known.

```php
public function saveField(FieldInterface $field, $runValidation = true)
{
    /** @var Field $field */

    if ($runValidation && !$field->validate()) {
        Craft::info('Field not saved due to validation error.', __METHOD__);

        return false;
    }

    $isNewField = !$field->id;

    // Fire a 'beforeSaveField' event
    $this->trigger(self::EVENT_BEFORE_SAVE_FIELD, new FieldEvent([
        'field' => $field,
        'isNew' => $isNewField,
    ]));

    $transaction = Craft::$app->getDb()->beginTransaction();

    try {
        if (!$field->beforeSave()) {
            $transaction->rollback();

            return false;
        }

        // ... Save the field here ...

        $field->afterSave();

        $transaction->commit();
    } catch (\Exception $e) {
        $transaction->rollBack();

        throw $e;
    }

    // Fire an 'afterSaveField' event
    $this->trigger(self::EVENT_AFTER_SAVE_FIELD, new FieldEvent([
        'field' => $field,
        'isNew' => $isNewField,
    ]));

    return true;
}
```

## Controllers

### Return Types

Controller actions that should complete the request must return either a string (HTML) or a Response object.

```php
// Bad:
$this->asJson($obj);
$this->renderTemplate($template, $variables);

// Better:
return $this->asJson($obj);
return $this->renderTemplate($template, $variables);
```

### JSON Actions

Controller actions that have the option of returning JSON should do so if the request explicitly accepts a JSON response; not if it’s an Ajax request.

```php
// Bad:
if (Craft::$app->getRequest()->getIsAjax()) {
    return $this->asJson([...]);
}

// Better:
if (Craft::$app->getRequest()->getAcceptsJson()) {
    return $this->asJson([...]);
}
```

Controller actions that *only* return JSON should require that the request accepts JSON.

```php
$this->requireAcceptsJson();
```

## Exceptions

- If an exception is likely to occur as a result of user error, use the `yii\base\UserException` class (or a subclass)
- Only translate exception messages with `Craft::t()` if it’s a `yii\base\UserException`.

## DB Queries

- Always wrap table names with `{{%` and `}}` (e.g. `{{%entries}}`) so it gets properly quoted and the table prefix gets inserted.
- Use the `['col1', 'col2']` syntax with `select()` and `groupBy()` instead of `'col1, col2'`,  even if only referencing a single column
- Use the `['{{%tablename}}']` syntax with `from()` instead of `'{{%tablename}}'`.
- Use the `['col1' => SORT_ASC, 'col2' => SORT_DESC]` syntax with `orderBy()` instead of `'col1, col2 desc'`.

### Conditions
- Always use Yii’s [declarative condition syntax](http://www.yiiframework.com/doc-2.0/yii-db-queryinterface.html#where()-detail) when possible, as it will automatically quote table/column names and values for you.
- For consistency, use:
  -  `['col' => $values]`  instead of `['in', 'col', $values]`
  - `['col' => $value]`  instead of `['=', 'col', $value]`
  - `['like', 'col', 'value']`  instead of `['like', 'col', '%value%', false]`
    *(unless the `%` is only needed on one side of `value`)*
- If searching for `NULL`, use the `['col' => null]` syntax.
- If searching for `NOT NULL`, use the `['not', ['col' => null]]` syntax.
- If you cannot use the declarative condition syntax (e.g. the condition is referencing another table/column name rather than a value, as is often the case with joins), make sure you’ve quoted all column names, and any values that you aren’t 100% confident are safe should be added as query params.

```php
// Bad:
$query->where('foo.thing is null');
$query->innerJoin('{{%bar}} bar', 'bar.fooId = foo.id');

// Better:
$query->where(['foo.thing' => null]);
$query->innerJoin('{{%bar}} bar', '[[bar.fooId]] = [[foo.id]]');
```

## Getters & Setters

Getter and setter methods should have a corresponding `@property` tag in the class’s docblock, so IDEs like PhpStorm can be aware of the magic properties.

```php
/**
 * @property User $author
 */
class Entry
{
    private $_author;

    /**
     * @return User
     */
    public function getAuthor()
    {
        return $this->_author;
    }
}
```

For a slight performance improvement and easier debugging, you should generally stick with calling the getter and setter methods directly rather than going through their magic properties.

```php
// Bad:
$oldAuthor = $entry->author;
$entry->author = $newAuthor;

// Better:
$oldAuthor = $entry->getAuthor();
$entry->setAuthor($newAuthor);
```

### App Component Getters

App components should have their own getter functions, which call the app component getter method [`get()`](http://www.yiiframework.com/doc-2.0/yii-di-servicelocator.html#get()-detail) directly:

```php
/**
 * @return Entries
 */
public function getEntries()
{
    return $this->get('entries');
}
```

And you should use those instead of their magic properties:

```php
// Bad:
Craft::$app->entries->saveEntry($entry);

// Better:
Craft::$app->getEntries()->saveEntry($entry);
```

If you will be referencing the same app component multiple times within the same method, save a local reference to it.

```php
// Bad:
Craft::$app->getEntries()->saveEntry($entry1);
Craft::$app->getEntries()->saveEntry($entry2);

// Better:
$entriesService = Craft::$app->getEntries();
$entriesService->saveEntry($entry1);
$entriesService->saveEntry($entry2);
```