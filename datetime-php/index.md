---
title: DateTime Formats
slug: datetime-formats
description: This article is everything you need to know about date/time in Laravel.
excerpt: We will cover important things like immutability, formatting, exending Carbon with macros as well as helpful tips like a format cheat sheet, youtube duration formats and constants.
date: 2021-08-16
tags: [PHP, datetime, macros, mixins, constants, immutability, carbon, rfc, formatting, TTL, duration]
sources:
    - https://twitter.com/AlexVanderbist/status/1276444417709297664
---

# DateTime Formats

This article is everything you need to know about date/time in Laravel.

We will cover important things like immutability, formatting, exending Carbon with macros as well as helpful tips like a format cheat sheet, youtube duration formats and constants.

- [Cheat Sheet](#cheat-sheet)
  - [Formats](#formats)
  - [Examples](#examples)
  - [Carbon constants](#carbon-constants)
- [Custom date formats](#custom-date-formats)
  - [Extending Carbon](#extending-carbon)
  - [Custom constants](#custom-constants)
  - [Blade stringables](#blade-stringables)
  - [Immutable dates by default](#immutable-dates-by-default)
  - [Creating Dates](#creating-dates)
- [Duration](#duration)
  - [Simple Duration string](#simple-duration-string)
  - [Youtube Duration](#youtube-duration)
    - [Simple](#simple)
    - [Advanced](#advanced)
    - [CarbonInterval Macro](#carboninterval-macro)
    - [CarbonInterval Mixin](#carboninterval-mixin)
    - [CarbonInterval Mixin (Trait)](#carboninterval-mixin-trait)
  - [Create Time From Format](#create-time-from-format)
- [Avoiding Magic Time](#avoiding-magic-time)
  - [Fluent Interval](#fluent-interval)
  - [Setting Time](#setting-time)
  - [Convert DateTime to Carbon](#convert-datetime-to-carbon)
- [Periods](#periods)

## Cheat Sheet

### Formats
| Day Format  | Value/Example |
| ------------- | ------------- |
| d  | 01 through 31  |
| D  | Mon through Sun  |
| j  | 1 through 31  |
| l (lowercase 'L')  | Sunday throguh Saturday  |
| N  | 1 (Monday) to 7 (Sunday) |
| S  | st, nd, rd, th  |
| w  | 0 (sunday) through 6 (Saturday)  |
| z  | 0 through 365  |

| Week Format  | Value/Example |
| ------------- | ------------- |
| W  | 42 (42nd week in year)  |

| Month Format  | Value/Example |
| ------------- | ------------- |
| F  | January through December  |
| m  | 01 through 12  |
| M  | Jan through Dec  |
| n  | 1 through 12  |
| t  | 28 through 31  |

| Year Format  | Value/Example |
| ------------- | ------------- |
| L  | 1 (leap year), 0 otherwise  |
| o  | same as Y, except that it use week number to decide which year it falls onto |
| Y  | 1991, 2012, 2014, ...  |
| y  | 91, 12, 14, ...  |

| Time Format  | Value/Example |
| ------------- | ------------- |
| a  | am or pm  |
| A  | AM or PM |
| B  | 000 through 999  |
| g  | 1 through 12  |
| G  | 0 through 23  |
| h  | 01 through 12  |
| H  | 01 through 23  |
| i  | 00 through 59  |
| u  | 123456 (microseconds)  |

| Timezone Format  | Value/Example | Description
| ------------- | ------------- | --- |
| e  | UTC, GMT, Atlantic/Azores  | 	Timezone identifier
| T  | EST, BST, ...  | Timezone abbreviation (if known) or the GMT offset.
| I (capital i)  | 1 (daylight), 0 otherwise | Is date in daylight saving time
| O  | +0200  | Difference to Greenwich time (GMT) without colon between hours and minutes
| P  | +02:00  | The same as O with colon
| Z  | -43200 through 50400 (timezone offset)  | Timezone offset in seconds. West UTC = negative, East UTC = positive

| Date/Time Format  | Value/Example | Description
| ------------- | ------------- | --- |
| c  | 2004-02-12T15:19:21+00:00  | ISO 8601 date
| r  | Thu, 21 Dec 2000 16:01:07 +0200 |RFC 2822 formatted date
| U  | 1617265800 | Seconds since the Unix Epoch (January 1 1970 00:00:00 GMT)

### Examples
Here are some common formats using `2021-04-01 09:30:00`

| Format  | Code | Example
| --- | --- | ---|
| RFC2822 | DateTime::RFC2822 | Thu, 01 Apr 2021 09:30:00 +0000
| ISO | c | 2021-04-01T09:30:00+00:00
| ISO Date | Y-m-d | 2021-04-01
| ISO Week | Y-\WW | 2021-W33
| ISO Weekday | Y-\WW-N | 2021-W33-2
| Ordinal (Year-Day) | Y-z | 2021-90
| RFC7231 | Carbon::RFC7231_FORMAT | Thu, 01 Apr 2021 09:30:00 GMT
| RFC7231 | D, d M Y H:i:s \G\M\T | Thu, 01 Apr 2021 09:30:00 GMT
| UK (short)  | d/m/y | 01/04/21
| UK (medium)   | j M Y | 1 Apr 2021
| UK (long)   | d F Y | 01 April 2021
| Letter   | F d, Y | April 01, 2021
| 12 Hour | g:i a | 9:30 am
| 24 Hour | H:i | 09:30
| Military | Hi | 0930
| Words   | l, jS F Y \a\\t g:i A (T) | Thursday, 1st April 2021 at 9:30 AM (UTC)

### Carbon constants
Carbon comes with a number of [constants](#carbon-constants) you can to avoid 'magic' dates/times.

```php

// days
Carbon::SUNDAY = 0;
Carbon::MONDAY = 1
Carbon::SATURDAY = 6;

// months
Carbon::JANUARY = 1;
Carbon::DECEMBER = 12;

// x in y
Carbon::YEARS_PER_MILLENNIUM = 1000;
Carbon::YEARS_PER_CENTURY = 100;
Carbon::YEARS_PER_DECADE = 10;
Carbon::MONTHS_PER_YEAR = 12;
Carbon::MONTHS_PER_QUARTER = 3;
Carbon::QUARTERS_PER_YEAR = 4;
Carbon::WEEKS_PER_YEAR = 52;
Carbon::WEEKS_PER_MONTH = 4;
Carbon::DAYS_PER_YEAR = 365;
Carbon::DAYS_PER_WEEK = 7;
Carbon::HOURS_PER_DAY = 24;
Carbon::MINUTES_PER_HOUR = 60;
Carbon::SECONDS_PER_MINUTE = 60;
Carbon::MILLISECONDS_PER_SECOND = 1000;
Carbon::MICROSECONDS_PER_MILLISECOND = 1000;
Carbon::MICROSECONDS_PER_SECOND = 1000000;

$minutesInAWeek = Carbon::DAYS_PER_WEEK * Carbon::HOURS_PER_DAY * Carbon::MINUTES_PER_HOUR;
```

## Custom date formats

Carbon already comes with a number of different conversion methods in `Carbon\Traits\Converter.php`. Here's a few:

- `toDateString`
- `toCookieString`
- `toRfc822String`
- `toRfc7231String`
- `toArray`
- `toIsoString`

### Extending Carbon

Using custom format constants might be useful if you use a number of different formats throughout your application, for example a finance system generating customer invoices, sending reports, working with accounting periods etc.

But if you usually use the same format throughout your application you might want to keep it at your fingertips.

We could use the Carbon's converter 'method' approach for our own common [date formats listed in the examples section](#examples).

You can extend `Illuminate\Support\Carbon` in a service provider with macros.

```php
// AppServiceProvider

use Illuminate\Support\Carbon;

Carbon::macro('ukShortFormat', function(){
    return $this->format('d/m/y');
});

Carbon::macro('toDefaultFormat', function(){
    return $this->ukShortFormat();
});
```

### Custom constants

If you have a number of different formats or you just want to clean up your service provider's macro calls you could set the string formats as enum or class constants.

Here we are using Enums (PHP 8.1+)
```php
enum DateTimeFormats: string
{
    case ISO_DATE = 'Y-m-d';
    case UK_SHORT = 'd/m/y';
    case UK_LONG = 'd F Y';
}

# usage
now()->format(DateTimeFormat::UK_SHORT->value);
```

but you could always use class constants

```php
class DateTimeFormat
{
    public const ISO_DATE = 'Y-m-d';
    public const UK_SHORT = 'd/m/y';
    public const UK_LONG = 'd F Y';
}

# usage
now()->format(DateTimeFormat::UK_SHORT);
```

or in a macro:

```php
Carbon::macro('ukShortFormat', function(){
    return $this->format(DateTimeFormat::UK_SHORT);
});
```

### Blade stringables

You could set the default blade carbon output format by changing the stringable implementation in a service provider.

```php
use Illuminate\Support\Carbon;

Blade::stringable(fn (Carbon $carbon) =>
    $carbon->format(DateTimeFormat::UK_SHORT)
);
```

### Immutable dates by default

Immutable dates prevent modifications to an instance following the original instance.

Making dates immutable is sort of like copying a version the previous instance before making any changes.

We can force Laravel to use immutable dates by default by calling the `Date` facade in the boot method of a service provider.

```php
use Illuminate\Support\Facades\Date;

public function boot(){
    Date::use(CarbonImmutable::class);

    // then add your custom carbon formatters
    Date::macro('ukShortFormat', function(){
        return $this->format('d/m/y');
    });

    Date::macro('toDefaultFormat', function(){
        return $this->ukShortFormat();
    });
}
```

Now any eloquent model datetime attributes will be immutable by default 🎉

Just remember to use Date instead of Carbon when instantiating dates manually!

This means you no longer have to remember to use the `CarbonImmutable` class, or cast your models:
```php
protected $casts = [
    // ...
    'created_at' => 'immutable_datetime' // no longer required
    // ...
];
```

### Creating Dates

```php
// last Friday of current month
now()->lastOfMonth(Carbon\Carbon::FRIDAY)

// first Friday of next month
now()->addMonth()->firstOfMonth(Carbon\Carbon::FRIDAY)
```

## Duration
### Simple Duration string
```php
gmdate('H:i:s', $seconds = 817); // 00:13:37
// Don't go over 24 hours (86,400 seconds)

gmdate("j\d H:i:s", $seconds = 86410); // 2d 00:00:10

gmdate("j:H:i:s", $seconds = 86410); // 2:00:00:10
```

### Youtube Duration

#### Simple

```php
// Will not work over 24 hours

function simpleTimeFormat($seconds)
{
    return $time->totalHours >= 1
  	    ? gmdate('g:i:s', $time->totalSeconds)
        : gmdate('i:s', $time->totalSeconds);
}
```

#### Advanced
This will allow you to pass `$seconds` as an integer or a `CarbonInterval` object.
```php
function youtubeFormat($seconds)
{
    $time = $seconds instanceof Carbon\CarbonInterval
        ? $seconds
        : Carbon\CarbonInterval::seconds($seconds);

    return sprintf("%02d:%02d:%02d",
                $time->totalHours,
                $time->totalMinutes % 60,
                $time->totalSeconds % 60
            );
}

youtubeFormat(CarbonInterval::seconds(817)) // "13:27"
youtubeFormat(8170) // "02:16:10"
```

#### CarbonInterval Macro

```php
// in service provider
Carbon\CarbonInterval::macro('youtubeFormat', function ()
{
      return sprintf("%02d:%02d:%02d",
               self::this()->totalHours,
               self::this()->totalMinutes % 60,
               self::this()->totalSeconds % 60
            );
});

CarbonInterval::seconds(86471)->youtubeFormat(); // "24:01:11"
```

#### CarbonInterval Mixin
```php
// in support folder
class YoutubeFormatCarbonMixin
{
    public function youtubeFormat()
    {
        return function () {
            // self::this()->...
            // return YoutubeFormat function code here
        };
    }
}

// in service provider
Carbon\CarbonInterval::mixin(new YoutubeFormatCarbonMixin());

// in your application
return Carbon\CarbonInterval::seconds($seconds)
    ->youtubeFormat();
```

#### CarbonInterval Mixin (Trait)

```php
trait YoutubeFormatCarbonMixin
{
    public function youtubeFormat()
    {
        // self::this()->...
        // return YoutubeFormat function code here
    }
}

// in service provider
Carbon::mixin(YoutubeFormatCarbonMixin::class);
```

### Create Time From Format

```php
CarbonInterval::createFromFormat('H:i:s', '10:20:00');
// 10 hours 20 minutes
```

## Magic Time

### Fluent Interval
```php
Carbon\CarbonInterval::day()->totalSeconds
// 86400

Carbon\CarbonInterval::week()->totalDays
// 7

Carbon\CarbonInterval::days(2)->hours(1)->totalHours;
// 25 (two days + 1 hour)

 Carbon\CarbonInterval::seconds(150)->totalMinutes;
 // 2.5

 (int) round(Carbon\CarbonInterval::seconds(150)->totalMinutes);
 // 3

 Carbon\CarbonInterval::seconds(817)->cascade()->forHumans(['short' => true]);
// "13m 37s"

Carbon\CarbonInterval::days(1)
  	->hours(3)
  	->minutes(45)
  	->seconds(3)
    ->youtubeFormat();
// "27:45:03"
// 27 hours, 45 minutes and 3 seconds
```

### Setting Time

```php
now()->setSeconds(0);

now()->setSeconds(0)->setMinutes(0);

now()->startOfHour();

now()->endOfHour();

now()->startOfMonth();

now()->endOfMonth()->next(Carbon\Carbon::WEDNESDAY);
```

### Convert DateTime to Carbon

```php
$dt = new \DateTime('first day of January 2008');
$carbon = Carbon::instance($dt);
```

## Periods

Generate an array of dates for a the next 30 days:
```php
CarbonPeriod::create(
    Carbon::now(),
    Carbon::now()->addDays(30)
);
```