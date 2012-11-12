PHP Option Type
===============
This adds an Option type for PHP.

The Option type is intended for cases where you sometimes might return a value
(typically an object), and sometimes you might return no value (typically null)
depending on arguments, or other runtime factors.

Often times, you forget to handle the case where no value is returned. Not intentionally
of course, but maybe you did not account for all possible states of the sytem; or maybe you
indeed covered all cases, then time goes on, code is refactored, some of these your checks 
might become invalid, or incomplete. Suddenly, without noticing, the no value case is not
handled anymore. As a result, you might sometimes get fatal PHP errors telling you that 
you called a method on a non-object; users might see blank pages, or worse.

On one hand, the Option type forces a developer to consciously think about both cases
(returning a value, or returning no value). That in itself will already make your code more
robust. On the other hand, the Option type also allows the API developer to provide
more concise API methods, and empowers the API user in how he consumes these methods.

Installation
============
Installation is super-easy via composer

```
composer require phpoption/phpoption
```

or add it to your composer.json file.


Usage
=====

Using the Option Type in your API
---------------------------------
```php
class MyRepository
{
    public function findSomeEntity($criteria)
    {
        if (null !== $entity = $this->em->find(...)) {
            return new \PhpOption\Some($entity);
        }

        // We use a singleton, for the None case.
        return \PhpOption\None::create();
    }
}
```

If you are consuming an existing library, you can also use a shorter version
which treats ``null`` as ``None``, and everything else as ``Some`` case:

```php
class MyRepository
{
    public function findSomeEntity($criteria)
    {
        return \PhpOption\Option::notNull($this->em->find(...));
    }
}
```

Case 1: You always Require an Entity in Calling Code
----------------------------------------------------
```php
$entity = $repo->findSomeEntity(...)->get(); // returns entity, or throws exception
```

Case 2: Fallback to Default Value If Not Available
--------------------------------------------------
```php
$entity = $repo->findSomeEntity(...)->getOrElse(new Entity());

// Or, if you want to lazily create the entity.
$entity = $repo->findSomeEntity(...)->getOrCall(function() {
    return new Entity();
});
```

Lazy-Evaluating Options
-----------------------

For example, if you like to try to find an entity, and only if an entity is not
found, let the repository create that entity, you can use code such as

```php
return $this->findSomeEntity()->orElse(LazyOption::create(array($this, 'createEntity')));
```

The lazy option will only be evaluated if the option that is returned by ``findSomeEntity`` was empty.


More Examples
=============

No More Boiler Plate Code
-------------------------
```php
// Before
if (null === $entity = $this->findSomeEntity()) {
    throw new NotFoundException();
}
echo $entity->name;

// After
echo $this->findSomeEntity()->get()->name;
```

No More Control Flow Exceptions
-------------------------------
```php
// Before
try {
    $entity = $this->findSomeEntity();
} catch (NotFoundException $ex) {
    $entity = new Entity();
}

// After
$entity = $this->findSomeEntity()->getOrElse(new Entity());
```

More Concise Null Handling
--------------------------
```php
// Before
$entity = $this->findSomeEntity();
if (null === $entity) {
    return new Entity();
}

return $entity;

// After
return $this->findSomeEntity()->getOrElse(new Entity());
```

Performance Considerations
==========================
Of course, performance is important. Attached is a performance benchmark which
you can run on a machine of your choosing.

The overhead incurred by the Option type comes down to the time that it takes to
create one object, our wrapper. Also, we need to perform one additional method call
to retrieve the value from the wrapper.

* Overhead: Creation of 1 Object, and 1 Method Call
* Average Overhead per Invocation (some case/value returned): 0.000000761s (that is 761 nano seconds)
* Average Overhead per Invocation (none case/null returned): 0.000000368s (that is 368 nano seconds)

The benchmark was run under Ubuntu precise with PHP 5.4.6. As you can see the
overhead is suprisingly low, almost negligble. 

So in conclusion, unless you plan to call a method thousands of times during a
request, there is no reason to stick to the ``object|null`` return value; better give
your code some options!