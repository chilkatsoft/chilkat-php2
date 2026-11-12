# chilkat/chilkat2

[Chilkat2 PHP](https://www.chilkatsoft.com/chilkat2-php.asp) as a Composer package: a command that installs the
`chilkat2` PHP extension for your PHP, and the stub that tells your IDE and static analyzers about its classes.

```
composer require chilkat/chilkat2
vendor/bin/chilkat2-install
```

`chilkat2-install` asks the PHP it runs under which version, architecture and thread-safety it is, downloads the
matching build of the extension (`chilkat2.so` / `php_chilkat2.dll`) from chilkatdownload.com, verifies its SHA-256,
copies it into PHP's `extension_dir`, enables it in the right place (`phpenmod` on Debian/Ubuntu, a `chilkat2.ini` in
the conf.d directory on Alpine, RHEL/Fedora, Homebrew and the official Docker images, or an `extension=` line in
`php.ini` — created next to `php.exe` on Windows when there is none), and verifies that a fresh PHP loads it.
Supported: **PHP 8.1 – 8.5** on Linux (x86_64, x86, arm64, armv7l; arm from PHP 8.2), Alpine Linux (x86_64, arm64),
macOS (Apple silicon, Intel) and Windows (x64, x86; TS and NTS).

The package version is the Chilkat version (`11.7.0` installs Chilkat 11.7.0; a fourth number, e.g. `11.7.0.1`, is a
fix to the package itself and still installs Chilkat 11.7.0), so the stub and the extension always match.
Updating is `composer update chilkat/chilkat2` followed by `vendor/bin/chilkat2-install` again. Let `composer require`
write its usual caret constraint (`"chilkat/chilkat2": "^11.7"`) rather than pinning an exact version, so those
updates come through.

## Usage

The classes come from the extension, so there is nothing to `require` for them (your project's
`vendor/autoload.php` is unaffected):

```php
<?php
declare(strict_types=1);

use Chilkat2\GlobalSettings;
use Chilkat2\Http;

(new GlobalSettings())->UnlockBundle('Anything for 30-day trial');   // your unlock code after purchase

try {
    $http = new Http(['ConnectTimeout' => 10]);
    $http->onPercentDone(function (int $pct): bool { echo "$pct%\n"; return false; });
    $html = $http->QuickGetStr('https://www.chilkatsoft.com/');
    echo strlen($html), " bytes\n";
} catch (\Chilkat2\Exception $e) {
    echo $e->getMessage(), "\n";        // the object's LastErrorText
}
```

A failed call throws `Chilkat2\Exception`; create an object with `['RaiseError' => false]` to get `false` / `null`
instead. Lookups that find nothing (`$json->StringOf('missing')`, `$xml->FindChild('a')`) return `null` without
throwing. Download page and getting started: https://www.chilkatsoft.com/chilkat2-php.asp, reference documentation:
https://www.chilkatsoft.com/refdoc/.

Chilkat2 PHP is a separate extension from the classic Chilkat PHP extension (`chilkat/chilkat`: `extension=chilkat`,
`CkHttp`, ...). The two can be installed side by side; `chilkat2-install` never touches the classic one.

## IDE and static analysis

`stubs/chilkat2.stub.php` declares every class, method and property of the extension with types and PHPDoc. Never
include it at run time — the extension already defines the classes.

* **PhpStorm** and **VS Code with Intelephense** index `vendor/` by default, so completion and hover documentation
  work as soon as the package is installed.
* **PHPStan** (`phpstan.neon`):
  ```neon
  parameters:
      scanFiles:
          - vendor/chilkat/chilkat2/stubs/chilkat2.stub.php
  ```
* **Psalm** (`psalm.xml`):
  ```xml
  <stubs>
      <file name="vendor/chilkat/chilkat2/stubs/chilkat2.stub.php"/>
  </stubs>
  ```

## chilkat2-install

```
vendor/bin/chilkat2-install [--dry-run] [--no-ini] [--dir=DIR] [--version=X.Y.Z] [--uninstall]
```

* It installs for **the PHP that runs it**. To target a specific PHP (the one PHP-FPM uses, or one of several
  versions) run that binary explicitly: `/usr/bin/php8.4 vendor/bin/chilkat2-install`.
* `--dry-run` prints what would be done — which build, where it goes, which ini file — and changes nothing.
* Installing into a system `extension_dir` usually needs root: the script tells you when, and prints the exact
  `sudo` command to run (Windows: an Administrator prompt). `--dir=DIR` installs somewhere writable instead; the
  ini line then uses the absolute path.
* `--no-ini` installs the file and prints the `extension=` line for you to add yourself.
* `--uninstall` removes the extension file and the ini entry it wrote.
* If PHP runs under a web server or PHP-FPM, restart it after installing so it picks up the extension.

## Docker and CI

In a Dockerfile based on the official `php` images (Alpine or Debian; cli, fpm, apache) the build runs as root and
the extension is enabled through `$PHP_INI_DIR/conf.d` for every SAPI, so:

```Dockerfile
FROM php:8.4-cli-alpine
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
WORKDIR /app
COPY . .
RUN composer install --no-dev --no-interaction && vendor/bin/chilkat2-install
```

On Alpine the installer also adds the `libstdc++` package, which the minimal `php:*-alpine` images do not ship and
`chilkat2.so` needs. In a multi-stage build, run the installer in the final stage. The `php:*-zts` images get the
ZTS build automatically.

GitHub Actions (PHP from `shivammathur/setup-php` keeps a root-owned `extension_dir`, hence `sudo`):

```yaml
- uses: shivammathur/setup-php@v2
  with:
    php-version: '8.4'
- run: composer install --no-interaction
- run: sudo php vendor/bin/chilkat2-install
```

## Why `ext-chilkat2` is not a requirement

Composer refuses to install a package whose `require` names an extension the running PHP does not have — and before
`chilkat2-install` has run, it does not have it. So the package only *suggests* `ext-chilkat2`. Once it is installed,
a project can add `"ext-chilkat2": "*"` to its own `require` so that Composer checks every environment for it, as is
usual for native extensions (`ext-gd`, `ext-intl`, ...).

## License

Chilkat is commercial software: the extension works in trial mode for 30 days, after which a
[license](https://www.chilkatsoft.com/purchase2.asp) is required. See LICENSE.

## Support

https://www.chilkatforum.com/ · support@chilkatsoft.com
