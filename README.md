[![Latest Stable Version](https://poser.pugx.org/sanmai/later/v/stable)](https://packagist.org/packages/sanmai/later)
[![Build Status](https://travis-ci.com/sanmai/later.svg?branch=master)](https://travis-ci.com/sanmai/later)
![CI](https://github.com/sanmai/later/workflows/CI/badge.svg)
[![Coverage Status](https://coveralls.io/repos/github/sanmai/later/badge.svg?branch=master)](https://coveralls.io/github/sanmai/later?branch=master)
[![Mutation testing badge](https://badge.stryker-mutator.io/github.com/sanmai/later/master)](https://infection.github.io/)
[![Type Coverage](https://shepherd.dev/github/sanmai/later/coverage.svg)](https://shepherd.dev/github/sanmai/later)

This rigorously tested fully-typed library just works. It neither defines nor throws any exceptions.

# Install

    composer require sanmai/later

The latest version requires PHP 7.1 or greater.

# Use 

To use this pattern you need a generator function, yielding a single item of type you want to produce lazily. Pass it to `later()`, a static wrapper returning a `Deferred` object:

For example:

```php
use function Later\later;

$deferred = later(function () {
    $deepThought = new DeepThought();
    $deepThought->solveTheQuestion();

    yield $deepThought;
});
```

And then call `get()` when needed, as many times as needed:

```php
$deferred->get()->getAnswer(); // 42
$deferred->get()->getAnswer(); // same 42
```

Using a generator instead of a traditional callback comes with a major benefits: any generator comes with a guarantee from the language that it will be used exactly once. You can be sure that it won't be called twice.

But that's not all: read on.

## No Callbacks Required

Making a closure generator on the spot isn't always convenient. And not to say these closures are much different from all-too-familiar callbacks. Not at all different from the looks of them.

The power of this pattern is in its ability to make use of any function, previously returning a single value, without any need for any additional callbacks or closures.

Consider this diff:

```diff
 private function makeFooBar()
 {
    //...

-    return $foo;
+    yield $foo;
 }
```

After adding `Deferred` to the mix:

```diff
 use function Later\lazy;

 public function __construct()
 {
-    $this->fooBar = $this->makeFooBar();
+    $this->lazyFooBar = lazy($this->makeFooBar());
 }

 public function usesFooBar()
 {
     if ($fooBarReallyRequired) {
-        $this->fooBar->getResult();
+        $this->lazyFooBar->get()->getResult();
     }
 }
```

We can see, this simple, single-line, change in the original method enabled us to free our program from creating things it may not need, postponing this process until the last moment, while also avoiding any use of callbacks.

## Discussion

The library is completely typed. [PHPStan](https://github.com/phpstan/phpstan), [Psalm](https://github.com/vimeo/psalm), and [Phan](https://github.com/phan/phan) are all routinely supported.

To exploit this capability it is recommended to declare a variable holding this object as `\Later\Interfaces\Deferred<Type>`.

In this example it will be `Deferred<DeepThought>`:

```php
use Later\Interfaces\Deferred;

final class HyperIntelligentMice
{
    /** @var Deferred<DeepThought> */
    private $supercomputer;

    public function __construct(DeepThought $deepThought)
    {
        $this->supercomputer = later(static function () use ($deepThought): iterable {
            $deepThought->solveTheQuestion();

            yield $deepThought;
        });
    }

    public function getAnswer(): int
    {
        return $this->supercomputer->getSupercomputer()->getAnswer();
    }
}
```

Following this approach, a static analyzer will be able to understand what is called, and what is returned.

# Writing Tests

The underlying `Deferred` object is fairly lax about input types. It will be happy to accept any `iterable`, not just generators.

This makes it super easy to use in mocks:

```php
$this->lazyDependency = lazy([$myObject]);
```

And that's it. No need to go through loops assembling closures and whatnot.

If nothing else, one can make a common mock for it:

```php
$this->createMock(\Later\Interfaces\Deferred::class);
```

