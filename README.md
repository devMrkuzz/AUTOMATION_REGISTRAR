# FULLSTACK DOCUMENTATION

<!--Installing Laravel Backend-->

# Create new Laravel Project

1. composer crete-project laravel/laravel laravel-api
2. cd laravel-api

# Install Laravel Sanctum for API authentication

1. composer require laravel/sanctum
2. php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
3. php artisan migrate
4. php artisan key:generate
5. php artisan serve

<!--Installing Laravel Backend-->

# Create a new Next.js project (in a different folder)

1. npx create-next-app@latest nextjs-app --typescript --tailwind --eslint
2. cd nextjs-app

# Install axios for API calls

1. npm install axios
2. npm run dev

# Configure Laravel Sanctum for Next.js

Edit `laravel-api/.env`:
SANCTUM_STATEFUL_DOMAINS=localhost:3000,localhost:3000
SESSION_DOMAIN=localhost
CORS_ALLOWED_ORIGINS=http://localhost:3000

Edit `laravel-api/config/sanctum.php`
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost,localhost:3000')),

# Create Laravel Authentication Endpoints

1. php artisan make:controller AuthController
2. Edit `laravel-api/app/Http/Controllers/AuthController.php`:
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
// Register
public function register(Request $request)
{
$validated = $request->validate([
'name' => 'required|string|max:255',
'email' => 'required|email|unique:users',
'password' => 'required|min:8|confirmed',
]);

        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        return response()->json([
            'message' => 'User registered successfully',
            'user' => $user,
        ], 201);
    }

    // Login
    public function login(Request $request)
    {
        $validated = $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        $user = User::where('email', $validated['email'])->first();

        if (!$user || !Hash::check($validated['password'], $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'message' => 'Login successful',
            'token' => $token,
            'user' => $user,
        ]);
    }

    // Logout
    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message' => 'Logged out successfully']);
    }

    // Get current user
    public function user(Request $request)
    {
        return response()->json($request->user());
    }

}
