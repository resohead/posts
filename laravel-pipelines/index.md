---
title: Laravel Pipelines
slug: laravel-pipelines
description: Use pipeline classes run steps in sequence with an object oriented approach.
date: 2022-08-15
tags: [laravel, pipelines, middleware]
sources: []
---

# Laravel Pipelines
Laravel pipelines allow you to replace procedural code with an object orientated approach. Simply pass a something (e.g. Model) through a series of steps (pipes) which performs an action (handle) at each stage and finally returns a result.

You could use it anywhere you need a run a sequence of steps. The Laravel framework uses this for handling request middleware but could be used to cleanse forum posts, set up an environment/onboarding new users, query filters and even sub-pipelines.

It also allows you to unit test the function.

## Method Signature

```php
app(Pipeline::class)
    ->send($something)
    ->through([
        \App\Pipes\FirstPipe::class,
        \App\Pipes\SecondPipe::class,
        \App\Pipes\ThirdPipe::class,
    ])
    ->then($callback);
```

| parameter | description |
| --- | --- |
| $something | passable object, string, query builder, model etc.
| $pipes | array of classes with 'handle' method
| $callback | the function to run once all pipes have been 'handled'

## Pipe Classes

You need a new class for each pipeline task with a single `handle` method and `input` and `closure` arguments.

```php
class FirstPipe {
    public function handle($something, Closure $next)
    {
        // $something = ...

        return $next($something);
    }
}
```

### Optional Interface
You could create an interface to enforce the `handle` method via a contract.

```php
interface Pipe
{
    public function handle($something, Closure $next);
}
```

Using the original pipe class from the previous example:
```php
class FirstPipe implements Pipe {
    public function handle($something, Closure $next)
    {
        // $something = ...

        return $next($something);
    }
}
```

### Pipe Parameters

You can pass parameters to the pipe class constructor by appending a comma separated string to to the end of the pipe with a semi-colon prefix.

In the example below, 'sean' and 'foo' will be passed to `$name` parameter on the `ParameterPipeline::class`.

```php
app(Pipeline::class)
    ->send($something)
    ->through([
        BasicPipeClass::class,
        ParameterPipeClass::class.':sean, foo',
    ])
```

```php
class ParameterPipeClass {

    public function handle($something, Closure $next, $name, $foo)
    {
        // $name = 'sean'
        // $foo = 'foo'

        return $next($something);
    }
}
```

## Pipeline Classes

Instead of building the pipes on demand, you can create a dedicated pipeline class to DRY up your code.

All you need is to add some Pipes to the $pipes property array.

```php
use Illuminate\Pipeline\Pipeline;

class DoSomethingPipeline extends Pipeline
{
    protected $pipes = [
        BasicPipeClass::class,
        ParameterPipeClass::class.':sean, foo',
    ];
}
```

### Dynamic Pipes

You can use the `pipes` method to dynamically set pipes:

```php
protected function pipes(): array
{
    // dynamically create pipes here...

    $pipes = [
        BasicPipeClass::class,
        ParameterPipeClass::class.':sean, foo',
    ];

    return $pipes;
}
```

or combine Pipeline and Collections to dynamically decide which pipes should be executed:

```php
$html = app(Pipeline::class)
            ->send($model)
            ->through(collect([
                FirstPipe::class => true,
                SecondPipe::class => $this->shouldRunFirstPipe(),
                ThirdPipe::class => false,
            ])->filter()->keys()->all())
            ->thenReturn();
```

Alternatively, the pipe class itself could determine whether or not it needs to run.

### Trigger via static method

To keep things 'Laravel' you could even add custom methods to use the pipeline without resolving from the container each time and calling the `then` or `thenReturn` methods.

```php
use Illuminate\Pipeline\Pipeline;

class DoSomethingPipeline extends Pipeline
{
    protected $pipes = [
        BasicPipeClass::class,
        ParameterPipeClass::class.':sean, foo',
    ];

    public static function sendAndReturn($something)
    {
        return (new static)
            ->send($something)
            ->thenReturn();
    }
}
```

and used:

```php
$sometingElse = DoSomethingPipeline::sendAndReturn($something);
```

## Error handling with transactions
You could wrap your pipeline in a try catch and add transactions to your method. This will exit the pipeline and avoid database problems with partial completion.

> :tip:
> Remember to throw errors from your pipes!

```php
    try {
        DB::beginTransaction();
        return app(Pipeline::class)
            ->send($user)
            ->through($pipes)
            ->then(function ($traveler) {
                DB::commit();
                // return to dashboard
            });
    } catch (Exception $e) {
        DB::rollback();
        // return a message to the user
    }
```

## Testing

```php
class SuffixBar implements Pipe
{
    public function handle($input, Closure $next)
    {
        $input = $input . 'bar';
        return $next($input);
    }
}

/**
 * @pest
 */
it('appends bar', function () {
    app(SuffixBar::class)->handle('foo', function ($result) {
        $this->assertEquals('foobar', $result);
   });
});
```

## Sources
- https://jeffochoa.me/understanding-laravel-pipelines
- https://github.com/archtechx/jobpipeline
- https://github.com/laravel/framework/pull/30346