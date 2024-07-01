
# Laravel Middleware for Ensuring Correct Data Types in API Responses

This repository contains a Laravel middleware that ensures the data types in the API response match the expected model data types. Specifically, it converts JSON strings representing arrays back into PHP arrays.

# Developed By Eng. Mohamed Hekal 

 ##Noouh For Integrated Solutions
   
## Installation

1. Clone the repository or download the code.

2. Navigate to the project directory:

   ```bash
   cd your-project-directory
   ```

3. Install the dependencies:

   ```bash
   composer install
   ```

## Middleware Setup

1. **Create Middleware:**

   Generate a new middleware using the Artisan command:

   ```bash
   php artisan make:middleware EnsureCorrectDataType
   ```

2. **Define the Middleware Logic:**

   In the generated middleware file `EnsureCorrectDataType.php`, implement the logic to check and fix the data types of the response data:

   ```php
   namespace App\Http\Middleware;

   use Closure;
   use Illuminate\Http\JsonResponse;

   class EnsureCorrectDataType
   {
       /**
        * Handle an incoming request.
        *
        * @param  \Illuminate\Http\Request  $request
        * @param  \Closure  $next
        * @return mixed
        */
       public function handle($request, Closure $next)
       {
           $response = $next($request);

           // Check if the response is a JSON response
           if ($response instanceof JsonResponse) {
               $data = $response->getData();

               // Fix data types in the response data
               $data = $this->fixDataTypes($data);

               // Set the corrected data back to the response
               $response->setData($data);
           }

           return $response;
       }

       /**
        * Fix data types in the given data.
        *
        * @param  mixed  $data
        * @return mixed
        */
       protected function fixDataTypes($data)
       {
           if (is_array($data)) {
               foreach ($data as $key => $value) {
                   if (is_string($value) && $this->isJsonArray($value)) {
                       $data[$key] = json_decode($value, true);
                   } elseif (is_array($value) || is_object($value)) {
                       $data[$key] = $this->fixDataTypes($value);
                   }
               }
           } elseif (is_object($data)) {
               foreach ($data as $key => $value) {
                   if (is_string($value) && $this->isJsonArray($value)) {
                       $data->{$key} = json_decode($value, true);
                   } elseif (is_array($value) || is_object($value)) {
                       $data->{$key} = $this->fixDataTypes($value);
                   }
               }
           }

           return $data;
       }

       /**
        * Check if a string is a JSON array.
        *
        * @param  string  $string
        * @return bool
        */
       protected function isJsonArray($string)
       {
           json_decode($string);
           return (json_last_error() == JSON_ERROR_NONE && is_array(json_decode($string, true)));
       }
   }
   ```

3. **Register the Middleware:**

   Register the middleware in `app/Http/Kernel.php`. Add it to the `$routeMiddleware` array:

   ```php
   protected $routeMiddleware = [
       // Other middleware
       'ensure.correct.data.type' => \App\Http\Middleware\EnsureCorrectDataType::class,
   ];
   ```

4. **Apply Middleware to Routes:**

   Apply the middleware to your routes or controllers. For example, in your `routes/api.php`:

   ```php
   Route::group(['middleware' => ['ensure.correct.data.type', ' any other middlewares'], 'namespace' => 'Api\V1'], function () {
       // Define your routes here
       Route::get('/example', [ExampleController::class, 'exampleMethod']);
       // Add other routes here
   });
   ```

## Usage

With this setup, any array data returned as a JSON string in the API response will be properly converted back to a PHP array. Adjust the `fixDataTypes` method as needed to handle other specific data type corrections based on your requirements.

## Contributing

Feel free to submit pull requests or open issues to improve this middleware.

## License

This project is open-source and available under the [MIT License](LICENSE).
