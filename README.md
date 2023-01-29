<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

<p align="center">
<a href="https://github.com/laravel/framework/actions"><img src="https://github.com/laravel/framework/workflows/tests/badge.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

## Generalize API Response Laravel

### 1. Create Traits
create traits with the name ```ApiResponse.php``` in the ```App/Traits``` folder
```php
trait ApiResponse{

    protected function successResponse($data, $message = null, $code = 200)
    {
        return response()->json([
            'status'=> 'Success',
            'message' => $message,
            'data' => $data
        ], $code);
    }

    protected function errorResponse($message = null, $code)
    {
        return response()->json([
            'status'=>'Error',
            'message' => $message,
            'data' => null
        ], $code);
    }

}

```

### 2. Call Traits in Controller Parent
here I call the ApiResponse trait on the parent controller ```App/Http/Controllers/Controller.php```
```php
class Controller extends BaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests, ApiResponse;
}
```

### 3. Using in Controller
```php
return $this->successResponse($user);
```
```php
return $this->successResponse($user,'User Created', 201);
```
```php
return $this->errorResponse($validator->messages(), 422);
```

### 4. Result
>SUCCESS
```json
{
    "status": "Success",
    "message": null,
    "data": [
        {
            "id": 1,
            "name": "Item 1",
            "email": "email@gmail.com",
            "email_verified_at": null,
            "created_at": "2023-01-29T12:22:08.000000Z",
            "updated_at": "2023-01-29T12:22:08.000000Z"
        }
    ]
}
```
>ERROR
```json
{
    "status": "Error",
    "message": {
        "email": [
            "The email has already been taken."
        ]
    },
    "data": null
}
```

## Handling Error API Response
### 1. Edit Exception
Open ```App/Exceptions/Handler.php``` 

add this code
```php
use App\Traits\ApiResponse;
```
then add and you can edit what error you want to handle
```php
    public function render($request, \Throwable $exception)
    {
        $response = $this->handleException($request, $exception);
        return $response;
    }

    public function handleException($request, Throwable $exception)
    {

        if ($exception instanceof MethodNotAllowedHttpException) {
            return $this->errorResponse('Method Not Allowed', 405);
        }

        if ($exception instanceof NotFoundHttpException) {
            return $this->errorResponse('The specified URL cannot be found', 404);
        }

        if ($exception instanceof HttpException) {
            return $this->errorResponse($exception->getMessage(), $exception->getStatusCode());
        }

        if (config('app.debug')) {
            return parent::render($request, $exception);
        }

        return $this->errorResponse('Unexpected Exception. Try later', 500);

    }
```
### 2. Result
```json
{
    "status": "Error",
    "message": "Method Not Allowed",
    "data": null
}
```
## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
