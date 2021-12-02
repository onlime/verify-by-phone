# Verify your users by call or SMS

It's a common practice: a user signs up, you send an SMS to their phone with a code, they enter that code in your application
and they're off to the races.

This package makes it simple to add this capability to your Laravel application.

## Installation

You can install the package via composer:

```bash
composer require worksome/verify-by-phone
```

You can publish the config file by running:

```bash
php artisan vendor:publish --tag="verify-by-phone-config"
```

## Configuration

This package is built to support multiple verification services. The primary service is [Twilio](https://www.twilio.com/).
You may configure the service in the config file at `config/verify-by-phone.php` under `driver`, or by using the dedicated `.env` variable: `VERIFY_BY_PHONE_DRIVER`.

### `twilio`

To use our Twilio integration, you'll need to provide an `account_sid`, `auth_token` and `verify_sid`. All of these
can be set in the `config/verify-by-phone.php` file under `services.twilio`.

## Usage

To use this package, you'll want to inject the `\Worksome\VerifyByPhone\Contracts\PhoneVerificationService`
contract into your application. Let's imagine that you want to send the verification code in a controller method:

```php
public function sendVerificationCode(Request $request, PhoneVerificationService $verificationService)
{
    // Send a verification code to the given number
    $verificationService->send(new PhoneNumber($request->input('phone')));
    
    return redirect(route('home'))->with('message', 'Verification code sent!');
}
```

It's as simple as that! Note that we are using `\Propaganistas\LaravelPhone\PhoneNumber` to safely parse
numbers in different formats.

Now, when a user receives their verification code, you'll want to check that it is valid. Use the `verify` method for this:

```php
public function verifyCode(Request $request, PhoneVerificationService $verificationService)
{
    // Send a verification code to the given number
    $valid = $verificationService->verify(
        new PhoneNumber($request->input('phone')), 
        $request->input('code')
    );
    
    if ($valid) {
        // Mark your user as valid
    }
}
```

The first parameter is the phone number (again using `\Propaganistas\LaravelPhone\PhoneNumber`), and the second is the 
verification code provided by the user.

## Testing

When writing tests, you likely do not want to make real requests to services such as Twilio. To support testing, we provide a 
`FakeVerificationService` that can be used to mock the verification service. To use it, you should set an `env` variable
in your `phpunit.xml` with the following value:

```xml
<env name='VERIFY_BY_PHONE_DRIVER' value='null'/>
```

Alternatively, you may manually swap out the integration in your test via using the `swap` method:

```php
it('tests something to do with SMS verification', function () {
    $this->swap(PhoneVerificationService::class, new FakeVerificationService());
    
    // The rest of your test
});
```

You may execute this project's tests by running:

```bash
composer test
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](.github/CONTRIBUTING.md) for details.

## Security Vulnerabilities

Please review [our security policy](../../security/policy) on how to report security vulnerabilities.

## Credits

- [Luke Downing](https://github.com/lukeraymonddowning)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
