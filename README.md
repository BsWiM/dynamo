[![Build Status](https://travis-ci.org/tebru/dynamo.svg?branch=v0.1.0)](https://travis-ci.org/tebru/dynamo)
[![Coverage Status](https://coveralls.io/repos/tebru/dynamo/badge.svg?branch=master&service=github)](https://coveralls.io/github/tebru/dynamo?branch=master)

# Dynamo

This library allows you to take an interface annotated with Doctrine annotations and generate a class.  It
handles all of the parsing, and provides events to hook into in order to create the method body based
on the annotations.

## Installation

``` bash
composer require tebru/dynamo
```
    
## Usage

Create a new generator object using the builder

```php
$generator = \Tebru\Dynamo\Generator::builder()
    ->namespacePrefix('My\Custom\Library')
    ->setCacheDir('path/to/cache/vendor-name')
    ->build();
```
        
There are many different options to use with the builder, however, for most all cases, the defaults outside
of the namespace prefix and cache dir will be fine.

The namespace prefix is required in order to get around class name conflicts.  The generator uses the full
interface name plus the prefix as the generated class name.  The prefix defaults to `Dynamo`.

The cache directory defaults to `/system/dir/dynamo`

After you have a generator, you can pass your interface name into it and it will create a file in your
cache directory

```php
$generator->createAndWrite(MyInterface::class);
```

## Events

It's essential to subscribe to at least the `MethodEvent` as it is what allows you to add a method
body to the method.  The `MethodModel` and `AnnotationCollection` are available on the event.

The two other events are `StartEvent` and `EndEvent`, both of which provide access to the `ClassModel`.

```php
$eventDispatcher = new \Symfony\Component\EventDispatcher\EventDispatcher();
$eventDispatcher->addListener(new MethodListener());
    
$generator = \Tebru\Dynamo\Generator::builder()
    ->namespacePrefix('My\Custom\Library')
    ->setCacheDir('path/to/cache/vendor-name')
    ->setEventDispatcher($eventDispatcher)
    ->build();
```

## Sample listener

Here is a skeleton of a method listener

```php
<?php
    
namespace Foo;

use Tebru\Dynamo\Event\MethodEvent;

class MethodListener
{
    public function __invoke(MethodEvent $event)
    {
        $methodModel = $event->getMethodModel();
        $annotationCollection = $event->getAnnotationCollection();
        
        $body = [];
        if ($annotation->collection->exists(MyAnnotation::class)) {
            $body[] = '$var = "annotation exists";';
        } else {
            $body[] = '$var = "annotation not exists";';
        }
        
        $methodModel->setBody(implode($body));
    }
}
```
