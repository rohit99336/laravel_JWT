### I have implemented JWT Auth system in Laravel in this project, whose complete procedure is given here, by following this you can easily implement JWT in Laravel easily

<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="200"></a>
</p>

## Learning Laravel

Laravel has the most extensive and thorough [documentation](https://laravel.com/docs) and video tutorial library of all modern web application frameworks, making it a breeze to get started with the framework.<br><br><br>

<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://jwt.io/img/logo.svg" width="70"></a>
</p>

## JWT token

JSON Web Token Authentication for Laravel & Lumen.follow this [documentation](https://jwt-auth.readthedocs.io/en/develop/auth-guard/#payload) for more information regarding to JWT.

## Installtion Process

I am follow this [documentation](https://hackthestuff.com/article/laravel-8-jwt-authentication-tutorial) for install jwt token.<br>

JWT is a encoded string which contains three parts saperated with . sign. The three parts are Header, Payload and Verify Signature. The format of the JWT is like s1ksDk8sd2.sdpcSd79a1.sda81eq

In this article, we will go through how to create authentication API using Jason Web Token. We will use tymondesigns/jwt-auth library to create authentication. We will go through step by step from the fresh application.

## The installation process below steps

1. Create fresh Laravel application
2. Install and configure JWT library
3. Configuration of database in .env file
4. Update User model
5. Configure default authentication guard
6. Add Authentication routes
7. Create JWTController controller class
8. Test application in Postman

### Step 1: Create fresh Laravel application

In the tutorial, the first step is to create new Laravel application. We will use following Composer command to create latest version of Laravel application. You can install Composer by following this article. Open the Terminal and run the following command.

    composer create-project laravel/laravel projectName --prefer-dist

<center>or</center>

    laravel new project

if you getting version problem and istall diffrent version of laravel use bellow command.

    composer create-project laravel/laravel project name version

for example -

    composer create-project laravel/laravel jwt 8.x    

After the application is created, change Terminal working directory to project.

    cd jwt

### Step 2: Install and configure JWT library<br>

In the second step, install JWT library using below Composer command.

    composer require tymon/jwt-auth

Now register the library service provider to config/app.php file. Open file and add the following lines into providers array.

    'providers' => [
    ....
    Tymon\JWTAuth\Providers\LaravelServiceProvider::class,
],

Publish JWT config file using vendor:command command into terminal.

    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"

This will copy configuration file from vendor to config/jwt.php.

We also need to generate token secret. Run the below artisan command.

    php artisan jwt:secret
This will create JWT token secret to .env file.

    JWT_SECRET=RGffekr......hGKasf5u

### Step 3: Configuration of database in .env file <br>

Now we will need to configure database connection. Open .env file from the root directory and change below database credentials with your MySQL.

    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=jwt
    DB_USERNAME=root
    DB_PASSWORD=secret

We will use default users table to authenticate API. Laravel already have users table. You can customize users table field at database/migrations directory. Lastly migrate users table into database using following command.

    php artisan migrate

### Step 4: Update User model

Now we need to modify User model. Open App/Models/User.php file and implement Tymon\JWTAuth\Contracts\JWTSubject interface. We also need to add two model methods getJWTIdentifier() and getJWTCustomClaims().

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Tymon\JWTAuth\Contracts\JWTSubject;

    class User extends Authenticatable implements JWTSubject
    {
        use HasFactory, Notifiable;

        /**
        * Get the identifier that will be stored in the subject claim of the JWT.
        *
        * @return mixed
        */
        public function getJWTIdentifier()
        {
            return $this->getKey();
        }

        /**
        * Return a key value array, containing any custom claims to be added to the JWT.
        *
        * @return array
        */
        public function getJWTCustomClaims()
        {
            return [];
        }
    }

### Step 5: Configure default authentication guard

The default authentication guard is web. We need to change it to api. Open config/auth.php file and change default guard to api.

    <?php

    return [

        'defaults' => [
            'guard' => 'api',
            'passwords' => 'users',
        ],

        'guards' => [
            'web' => [
                'driver' => 'session',
                'provider' => 'users',
            ],

            'api' => [
                'driver' => 'jwt',
                'provider' => 'users',
            ],
        ],
    ];

### Step 6: Add Authentication routes<br>

In this step, we need to register authentication routes into routes/api.php file. Open the file and add below routes into it.

    <?php

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Route;
    use App\Http\Controllers\JWTController;

    Route::group(['middleware' => 'api'], function($router) {
        Route::post('/register', [JWTController::class, 'register']);
        Route::post('/login', [JWTController::class, 'login']);
        Route::post('/logout', [JWTController::class, 'logout']);
        Route::post('/refresh', [JWTController::class, 'refresh']);
        Route::post('/profile', [JWTController::class, 'profile']);
    });

### Step 7: Create JWTController controller class <br>

We have defined routes for authentication so far. We need to create controller class to build application logic. The below Artisan command will generate controller class at App/Http/Controllers directory.

    php artisan make:controller JWTController

In the controller class, add the methods as per routes.

    <?php

    namespace App\Http\Controllers;

    use Auth;
    use Validator;
    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class JWTController extends Controller
    {
        /**
        * Create a new AuthController instance.
        *
        * @return void
        */
        public function __construct()
        {
            $this->middleware('auth:api', ['except' => ['login', 'register']]);
        }

        /**
        * Register user.
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function register(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'name' => 'required|string|min:2|max:100',
                'email' => 'required|string|email|max:100|unique:users',
                'password' => 'required|string|confirmed|min:6',
            ]);

            if($validator->fails()) {
                return response()->json($validator->errors(), 400);
            }

            $user = User::create([
                    'name' => $request->name,
                    'email' => $request->email,
                    'password' => Hash::make($request->password)
                ]);

            return response()->json([
                'message' => 'User successfully registered',
                'user' => $user
            ], 201);
        }

        /**
        * login user
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function login(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'email' => 'required|email',
                'password' => 'required|string|min:6',
            ]);

            if ($validator->fails()) {
                return response()->json($validator->errors(), 422);
            }

            if (!$token = auth()->attempt($validator->validated())) {
                return response()->json(['error' => 'Unauthorized'], 401);
            }

            return $this->respondWithToken($token);
        }

        /**
        * Logout user
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function logout()
        {
            auth()->logout();

            return response()->json(['message' => 'User successfully logged out.']);
        }

        /**
        * Refresh token.
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function refresh()
        {
            return $this->respondWithToken(auth()->refresh());
        }

        /**
        * Get user profile.
        *
        * @return \Illuminate\Http\JsonResponse
        */
        public function profile()
        {
            return response()->json(auth()->user());
        }

        /**
        * Get the token array structure.
        *
        * @param  string $token
        *
        * @return \Illuminate\Http\JsonResponse
        */
        protected function respondWithToken($token)
        {
            return response()->json([
                'access_token' => $token,
                'token_type' => 'bearer',
                'expires_in' => auth()->factory()->getTTL() * 60
            ]);
        }
    }

We have created methods for authenticating APIs for Login, Register, Profile, Token Refresh and Logout routes.

### Step 8: Test application in Postman <br>

We have completed the application coding. Start the Laravel server using below Artisan command.

    php artisan serve

For testing APIs, we will use Postman application.  Postman is an API platform for building and using APIs. We will test all API. Lets start from register API.

Register API
All API routes are prefixed with api namespace. In the postman use <http://localhost:8000/api/register> API endpoint. Pass name, email, password and password_confirmation parameters into request. You will get message and user details into response.

Login API
Use <http://localhost:8000/api/login> API endpoint with email password parameter with request. If the email and password matches with registered user, you will receive token json object into response.
