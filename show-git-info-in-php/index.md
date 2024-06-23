---
title: Show Git Info in PHP
slug: show-git-info-in-php
description: How to show the current Git branch and commit hash in a PHP application
excerpt: When working on a project, it can be useful to show the current Git branch and commit hash in the application. This can be helpful for debugging, tracking down issues, or just keeping track of what version of the code is running.
date: 2024-06-23
tags: [php, git]
sources: []
---

# Show Git Info in PHP

When working on a project, it can be useful to show the current Git branch and commit hash in the application. This can be helpful for debugging, tracking down issues, or just keeping track of what version of the code is running.

```php file="app/Version.php"
<?php
namespace App;

use Carbon\Carbon;

class Version
{
    public readonly ?string $commit;

    public readonly ?string $commit_short;

    public readonly ?string $tag;

    public readonly ?Carbon $date;

    protected static array $instances = [];
    
    public function __construct(?string $commit = null, ?string $tag = null, ?Carbon $date = null)
    {
        $this->commit = $commit ?? static::getCurrentGitCommitHash();
        $this->commit_short = substr($this->commit, 0, 7);
        $this->tag = $this->tag();
        $this->date = $this->date();
    }

    public static function make(?string $commit = null, ?string $tag = null, ?Carbon $date = null): self
    {
        if(! isset(static::$instances[$commit])) {
            static::$instances[$commit] = new static($commit);
        }

        return static::$instances[$commit];
    }

    public function toArray(): array
    {
        return [
            'commit' => $this->commit,
            'commit_short' => $this->commit_short,
            'tag' => $this->tag,
            'label' => $this->label(),
            'date' => $this->date,
        ];
    }

    protected function hasTag(): bool
    {
        return $this->tag !== null;
    }

    protected function label(): string
    {
        if(! $this->hasTag()) {
            return $this->commit_short;
        }

        return str_starts_with($this->tag, 'v')
            ? "{$this->tag} ({$this->commit_short})"
            : "v{$this->tag} ({$this->commit_short})";
    }

    protected static function getCurrentGitCommitHash(): ?string
    {
        $path = base_path('.git/');
    
        if (! file_exists($path)) {
            return null;
        }
    
        $head = trim(substr(file_get_contents($path . 'HEAD'), 4));
    
        $hash = trim(file_get_contents(sprintf($path . $head)));
    
        return $hash;
    }

    protected function tag(): ?string
    {
        $tag = exec('git tag --contains ' . $this->commit);

        return $tag === '' ? null : $tag;
    }

    public function date(): ?Carbon
    {
        $date = exec('git show -s --format=%ci ' . $this->commit);
        
        return $date === '' ? null : Carbon::parse($date);
    }
}
```

```php file="config/version.php"
$version = \App\Version::make();

return [
    ...$version->toArray(),
    'changelog' => "https://github.com/<project>/<name>/releases/tag/{$version->tag}",
];

/**
 * Example output:
 * 
 * "commit" => "21c08ad0accb6607f8f347be972640c5a6c99aed"
 * "commit_short" => "21c08ad"
 * "tag" => "v0.1.0-beta"
 * "label" => "v0.1.0-beta (21c08ad)"
 * "date" => <Carbon> "2024-06-23 14:00:00" 
 * "changelog" => "https://github.com/project/name/releases/tag/v0.1.0-beta"
 */
```

```php file="resources/views/layout/footer.php"
<footer class="py-16 text-center text-sm text-black dark:text-white/70">
    @env(['staging', 'production'])
        <a href="{{ config('version.changelog') }}">{{ config('version.label') }}</a>
    @else
        Laravel {{ Illuminate\Foundation\Application::VERSION }} (PHP v{{ PHP_VERSION }})
    @endenv
</footer>
```