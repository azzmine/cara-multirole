###### ***Cara Download Laragon sampai install breeze***

========================================================

1. download laragon 6.0 dari website
2. download zip php 8.4 atau lebih
3. buat folder baru untuk php 8.4 di \\laragon\\bin\\php
4. extract zip php ke folder baru yg sudah di buat didalam \\laragon\\bin\\php
5. ke laragon, ganti versi php dgn versi 8.4 yang sudah di extract
6. masuk ke terminal dari laragon
7. pastikan nodejs sudah v18+
8. masukkan code "**composer**"
9. masukkan code "**composer create-project --prefer-dist laravel/laravel:^12.0 nama\_project**" untuk download Laravel dan membuat folder project baru. tunggu hingga process download selesai
10. jalan laravel 12 dengan code "**cd nama folder project baru**"
11. masukkan code "**composer require laravel/breeze --dev**" tunggu hingga selesai, bagian ini untuk membuat login, register
12. setelah itu "**php artisan breeze:install**", jika sudah pilih "**blade**", pilih tema, lalu "**Pest**", tunggu hingga selesai
13. lalu masukkan code "**npm install \&\& npm run dev**", tunggu hingga selesai
14. kemudian "**php artisan migrate**"
15. "**php artisan serve**"



###### ***Cara Membuat Database dan Multi Role***

===============================================

1. cek extention sqlite dgn code "**php -m | findstr /i sqlite**"
2. jika belum aktif buka **php.ini**, aktifkan **extension=sqlite3**, restart server PHP, jika sudah aktif bisa langsung membuat database
3. "**type nul > database\\database.sqlite"** untuk membuat database, bagian "**database.sqlite"** bisa di ganti nama sesuai kebutuhan
4. setelah selesai, set bagian **.env** ke SQLite, hapus komen pada bagian "**DB\_CONNECTION=sqlite**" tambahkan "**DB\_DATABASE=C:\\laragon\\www\\namaproject\\database\\database.sqlite**"
5. "**php artisan config:clear**" dan "**php artisan migrate**"
6. Buat Model dan Migration Role "**php artisan make:model Role -m**"
7. Lalu buka file migration, isi seperti ini:

   **public function up(): void
   {
   Schema::create('roles', function (Blueprint $table) {
   $table->id();
   $table->string('name'); // admin, guru, siswa
   $table->timestamps();
   });**

   **}**"

8. isi bagian model dengan
   "**public function users() {
   return $this->hasMany(User::class);
   }**"
9. Jalankan migrasi "**php artisan migrate**"
10. Buat seeder role "**php artisan make:seeder RoleSeeder**"
11. isi
    "use App\\Models\\Role;

    Role::insert([
    ['name' => 'admin'],
    ['name' => 'guru'],
    ['name' => 'siswa'],
    ]);"

12. Daftarkan di DatabaseSeeder.php "$this->call(RoleSeeder::class);"
13. Tambah Role ke User "php artisan make:migration add\_role\_id\_to\_users\_table --table=users"
14. isi migration

    "<?php

    use Illuminate\Database\Migrations\Migration;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    return new class extends Migration
    {
    public function up(): void
    {
    Schema::table('users', function (Blueprint $table) {
    $table->unsignedBigInteger('role_id')->nullable(); // kolom role\_id
    // $table->string('role')->default('siswa'); // opsional, sebenarnya bisa dihapus
    });
    }

    public function down(): void
            {
                Schema::table('users', function (Blueprint $table) {
                    $table->dropColumn('role_id');
                    // $table->dropColumn('role'); // jika pakai
                });
            }

    };"

15. Jalankan migrate: "php artisan migrate:fresh --seed"
16. Update model User:

    "public function role() {
    return $this->belongsTo(Role::class);
    }"

17. Buat Seeder User "**php artisan make:seeder UserSeeder"**
    "<?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\Hash;
    use App\\Models\User;
    use App\\Models\Role;

    class UserSeeder extends Seeder
    {
    public function run(): void
    {
    // Admin
    User::create(\[
    'name' => 'Admin',
    'email' => 'admincontoh@gmail.com',
    'password' => Hash::make('password'),
    'role_id' => Role::where('name', 'admin')->first()->id,
    ]);

       // Guru
                User::create([
                    'name' => 'Guru 1',
                    'email' => 'gurucontoh@gmail.com',
                    'password' => Hash::make('password'),
                    'role_id' => Role::where('name', 'guru')->first()->id,
                ]);

                // Siswa
                User::create([
                    'name' => 'Siswa 1',
                    'email' => 'siswacontoh@gmail.com',
                    'password' => Hash::make('password'),
                    'role_id' => Role::where('name', 'siswa')->first()->id,
                ]);
            }

    }"

18. daftarkan di database seeder

    "$this->call([
    RoleSeeder::class,
    UserSeeder::class,
    ]);"

19. Buat middleware checkrole "php artisan make:middleware CheckRole"
20. Isi middleware CheckRole.php

    "<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
use Illuminate\Support\Facades\Auth;


class CheckRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle($request, Closure $next, $role)
    {
        if (!Auth::check() || Auth::user()->role->name !== $role) {
            abort(403, 'Unauthorized');
        }

        return $next($request);
    }
}"

21. Daftarkan middleware di bootstrap/app.php:

    "->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
    'checkrole' => \App\Http\Middleware\CheckRole::class,
    ]);
    })"

    

1. buat controller untuk masing2 role "php artisan make:controller AdminController"
2. isi

   "public function index() {
   return view('admin.dashboard'); // atau sesuai folder
   }"

3. buat folder "resources/views/admin/dashboard.blade.php"
4. Routes web:

   "use App\Http\Controllers\AdminController;
   use App\Http\Controllers\GuruController;
   use App\Http\Controllers\SiswaController;
   use Illuminate\Support\Facades\Auth;

   

   Route::get('/admin/dashboard', [Admin\DashboardController::class, 'index'])->middleware(['auth', 'checkrole:admin'])->name('admin.dashboard');
   Route::get('/guru/dashboard', [Guru\DashboardController::class, 'index'])->middleware(['auth', 'checkrole:guru'])->name('guru.dashboard');
   Route::get('/siswa/dashboard', [Siswa\DashboardController::class, 'index'])->middleware(['auth', 'checkrole:siswa'])->name('siswa.dashboard');

   Route::get('/home', function () {
   $role = Auth::user()->role->name;
   return match($role) {
   'admin' => redirect()->route('admin.dashboard'),
   'guru' => redirect()->route('guru.dashboard'),
   default => redirect()->route('siswa.dashboard'),
   };
   })->middleware(['auth'])->name('dashboard');

5. php artisan serve

   

   

   

   

   

   ###### ***Bagian yg Sempat Bermasalah***

   ===================================

6. jika muncul error seperti "**could not find driver (Connection: sqlite, SQL: select \* from "sessions" where "id" = YOg1nMaK0DkRDtOMHxhVAwK9Cm2iJti5evamRKo3 limit 1)**", cek ke bagian laragon→php→php.ini→pastikan bagian (extension=pdo\_sqlite
7. extension=sqlite3) tidak di komentari, setelah itu restart laragon
8. jika ada muncul error seperti "**Database file at path \[/full/path/to/database.sqlite] does not exist. Ensure this is an absolute path to the database. (Connection: sqlite, SQL: select \* from "sessions" where "id" = E8EzNFNqU7jbWPEvvdZY0X5uVQSmgFyes73wM3Jl limit 1)**", coba code "type nul > database\\database.sqlite", pada bagian "database.sqlite" boleh di ganti nama lain, setelah itu, coba code " php artisan migrate", setelah itu coba "php artisan serve"
9. Laravel siap di gunakan

