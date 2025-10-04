# Part2 panduan absensi

## Langkah ini melibatkan pembuatan Model dan Migrasi untuk menyimpan data absensi karyawan dan daftar hari libur.

### 🛠️ Langkah 6: Fondasi Database Absensi
Kita akan membuat dua entitas baru: Attendance (Absensi) dan Holiday (Hari Libur).

## 6.1. Model dan Migrasi Attendance (Absensi)
Model ini akan menyimpan setiap catatan check-in dan check-out harian dari setiap pengguna.

Perintah Artisan:
Kita buat Model dan Migrasi secara bersamaan.

```Bash
php artisan make:model Attendance -m
```
Kode Migrasi:
Buka file migrasi create_attendances_table (di database/migrations/) dan isi dengan skema berikut. Kita akan menggunakan Foreign Key untuk menghubungkan setiap absensi ke user_id.

```PHP
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('attendances', function (Blueprint $table) {
            $table->id();
            // Foreign Key ke tabel users
            $table->foreignId('user_id')->constrained()->onDelete('cascade');

            $table->date('date');
            $table->time('time_in')->nullable();
            $table->time('time_out')->nullable();

            // Status Absensi: Hadir, Terlambat, Izin, Sakit, Cuti, Alpa
            $table->enum('status', ['Hadir', 'Terlambat', 'Izin', 'Sakit', 'Cuti', 'Alpa'])->default('Alpa');

            $table->text('notes')->nullable();

            // Kolom tambahan untuk melacak lokasi (jika diperlukan di masa depan)
            $table->string('location_in')->nullable();
            $table->string('location_out')->nullable();

            $table->timestamps();

            // Memastikan satu user hanya bisa memiliki satu record absensi per hari
            $table->unique(['user_id', 'date']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('attendances');
    }
};
```

Kode Model:
Buka app/Models/Attendance.php dan definisikan properti fillable serta relasi ke Model User.

```PHP
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Attendance extends Model
{
    use HasFactory;

    protected $fillable = [
        'user_id',
        'date',
        'time_in',
        'time_out',
        'status',
        'notes',
        'location_in',
        'location_out',
    ];

    protected $casts = [
        'date' => 'date',
        'time_in' => 'datetime',
        'time_out' => 'datetime',
    ];

    /**
     * Relasi ke model User.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

## 6.2. Model dan Migrasi Holiday (Hari Libur)
Model ini akan digunakan untuk mencatat hari libur perusahaan atau nasional, yang nantinya akan memengaruhi perhitungan absensi.

Perintah Artisan:

```Bash
php artisan make:model Holiday -m
```
Kode Migrasi:
Buka file migrasi create_holidays_table dan isi dengan skema berikut:

```PHP
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('holidays', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->date('date');
            $table->text('description')->nullable();
            $table->timestamps();

            $table->unique('date'); // Hari libur hanya bisa satu kali pada tanggal tertentu
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('holidays');
    }
};
```
Kode Model:
Buka app/Models/Holiday.php dan definisikan properti fillable.

```PHP
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Holiday extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'date',
        'description',
    ];

    protected $casts = [
        'date' => 'date',
    ];
}
```

## 6.3. Relasi Balik ke Model User
Jangan lupa untuk menambahkan relasi ke Model Attendance di dalam Model User (app/Models/User.php).

Tambahkan metode ini ke dalam App\Models\User.php:

```PHP
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'role',
        'phone',
        'gender',
    ];


    /**
     * Relasi ke model Attendance.
     */
    public function attendances(): HasMany
    {
        return $this->hasMany(Attendance::class);
    }

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array<int, string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
    ];
}
```
## 6.4. Jalankan Migrasi
Terakhir, jalankan migrasi untuk membuat kedua tabel baru di database Anda.

```Bash
php artisan migrate
```


# Langkah Selanjutnya: Fungsionalitas Absensi Karyawan
Dengan fondasi database yang sudah siap, kita akan fokus pada sisi pengguna (karyawan) terlebih dahulu.

Langkah berikutnya: 
1. Membuat Controller dan Rute untuk Check-in dan Check-out sederhana. Ini akan mencakup:

2. Membuat EmployeeController

3. Menambahkan Rute untuk checkIn dan checkOut.

4. Membuat View Dashboard Karyawan yang menampilkan tombol absensi.


## 💻 Langkah 7: Fungsionalitas Absensi Karyawan
Langkah ini akan melibatkan pembuatan Controller baru untuk karyawan dan penambahan logika sederhana untuk absensi.

### 7.1. Membuat Controller Karyawan
Kita akan menggunakan Controller yang sederhana bernama EmployeeController untuk menangani aksi absensi yang spesifik.

Perintah Artisan:

```Bash
php artisan make:controller EmployeeController
```
Kode Controller:
Buka app/Http/Controllers/EmployeeController.php dan tambahkan logika untuk menampilkan dashboard karyawan, checkIn, dan checkOut. Kita akan menerapkan logika pengecekan status absensi harian di sini.

```PHP

<?php

namespace App\Http\Controllers;

use App\Models\Attendance;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class EmployeeController extends Controller
{
    /**
     * Menampilkan dashboard karyawan dan status absensi hari ini.
     */
    public function dashboard()
    {
        $user = Auth::user();
        $today = Carbon::today()->toDateString();

        // Cek status absensi hari ini
        $attendanceToday = Attendance::where('user_id', $user->id)
                                    ->whereDate('date', $today)
                                    ->first();

        return view('employee.dashboard', compact('user', 'attendanceToday'));
    }

    /**
     * Proses Check-in Karyawan.
     */
    public function checkIn()
    {
        $user = Auth::user();
        $today = Carbon::today()->toDateString();
        $currentTime = Carbon::now();

        // 1. Cek duplikasi
        if (Attendance::where('user_id', $user->id)->whereDate('date', $today)->exists()) {
            return redirect()->back()->with('error', 'Anda sudah melakukan Check-in hari ini.');
        }

        // 2. Logika Sederhana Status (Asumsi jam masuk standar 08:00)
        $standardCheckInTime = Carbon::parse($today . ' 08:00:00');

        $status = $currentTime->greaterThan($standardCheckInTime) ? 'Terlambat' : 'Hadir';
        $notes = $status === 'Terlambat' ? 'Check-in terlambat dari jam 08:00.' : null;

        // 3. Simpan Absensi
        Attendance::create([
            'user_id' => $user->id,
            'date' => $today,
            'time_in' => $currentTime,
            'status' => $status,
            'notes' => $notes,
        ]);

        return redirect()->back()->with('success', 'Check-in berhasil pada pukul ' . $currentTime->format('H:i:s'));
    }

    /**
     * Proses Check-out Karyawan.
     */
    public function checkOut()
    {
        $user = Auth::user();
        $today = Carbon::today()->toDateString();
        $currentTime = Carbon::now();

        // 1. Cek apakah sudah Check-in
        $attendance = Attendance::where('user_id', $user->id)->whereDate('date', $today)->first();

        if (!$attendance) {
            return redirect()->back()->with('error', 'Anda harus Check-in terlebih dahulu!');
        }

        // 2. Cek duplikasi Check-out
        if ($attendance->time_out) {
            return redirect()->back()->with('error', 'Anda sudah melakukan Check-out hari ini.');
        }

        // 3. Update Waktu Pulang
        $attendance->time_out = $currentTime;
        $attendance->save();

        return redirect()->back()->with('success', 'Check-out berhasil pada pukul ' . $currentTime->format('H:i:s'));
    }
}
```
## 7.2. Rute Absensi Karyawan
Kita akan merevisi rute dashboard bawaan Breeze dan menambahkan rute untuk checkIn dan checkOut.

Buka file routes/web.php dan ubah rute dashboard seperti berikut:

```PHP
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Rute Karyawan (Pengganti Dashboard default)
|--------------------------------------------------------------------------
| Menggunakan EmployeeController@dashboard jika role='karyawan' atau AdminController@dashboard jika role='admin'
*/

Route::middleware('auth')->group(function () {
    // Rute Dashboard Utama (akan diarahkan berdasarkan Role)
    Route::get('/dashboard', [App\Http\Controllers\EmployeeController::class, 'dashboard'])->name('dashboard');

    // Rute Khusus Aksi Absensi (hanya untuk Karyawan)
    Route::post('/checkin', [App\Http\Controllers\EmployeeController::class, 'checkIn'])->name('employee.checkin');
    Route::post('/checkout', [App\Http\Controllers\EmployeeController::class, 'checkOut'])->name('employee.checkout');

    // Rute Profile (bawaan Breeze)
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});
```

// ... (Rute Admin di bawahnya tetap) ...
(Catatan: Meskipun Controller bernama EmployeeController, kita gunakan untuk dashboard utama agar karyawan langsung melihat tombol absensi. Nanti kita bisa refactor untuk admin menggunakan AdminController)

## 7.3. View Dashboard Karyawan
Kita perlu membuat tampilan dashboard yang akan menampilkan status absensi hari ini dan tombol untuk Check-in atau Check-out.

Buat folder baru employee di dalam resources/views.

Buat file baru di resources/views/employee/dashboard.blade.php.

📄 File: resources/views/employee/dashboard.blade.php

```HTML
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Dashboard Absensi Karyawan') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg p-6">
                <h3 class="text-2xl font-bold mb-4 text-gray-800">Selamat Datang, {{ $user->name }}!</h3>
                <p class="text-gray-600 mb-6">Tanggal Hari Ini: **{{ \Carbon\Carbon::now()->translatedFormat('l, d F Y') }}**</p>

                {{-- Status Pesan (Success/Error) --}}
                @if (session('success'))
                    <div class="bg-green-100 border-l-4 border-green-500 text-green-700 p-4 mb-4" role="alert">
                        <p class="font-bold">Sukses!</p>
                        <p>{{ session('success') }}</p>
                    </div>
                @endif
                @if (session('error'))
                    <div class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-4" role="alert">
                        <p class="font-bold">Error!</p>
                        <p>{{ session('error') }}</p>
                    </div>
                @endif
                
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    {{-- Status Card --}}
                    <div class="md:col-span-2 bg-gray-50 border-2 rounded-lg p-6 shadow-md">
                        <h4 class="text-xl font-semibold mb-3">Status Absensi Hari Ini:</h4>
                        
                        @if ($attendanceToday)
                            <p class="text-3xl font-bold mb-4 
                                @if ($attendanceToday->status === 'Terlambat') text-red-600 
                                @elseif ($attendanceToday->status === 'Hadir') text-green-600 
                                @else text-blue-600 @endif">
                                {{ $attendanceToday->status }}
                            </p>
                            <p class="text-lg">Jam Masuk: **{{ $attendanceToday->time_in ? \Carbon\Carbon::parse($attendanceToday->time_in)->format('H:i:s') : 'Belum Check-in' }}**</p>
                            <p class="text-lg">Jam Pulang: **{{ $attendanceToday->time_out ? \Carbon\Carbon::parse($attendanceToday->time_out)->format('H:i:s') : 'Belum Check-out' }}**</p>
                            
                            {{-- Tombol Check-out (hanya tampil jika sudah check-in dan belum check-out) --}}
                            @if ($attendanceToday->time_in && !$attendanceToday->time_out)
                                <form action="{{ route('employee.checkout') }}" method="POST" class="mt-6">
                                    @csrf
                                    <button type="submit" class="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-4 rounded-lg transition duration-200">
                                        <i class="fas fa-sign-out-alt mr-2"></i>
                                        CHECK OUT SEKARANG
                                    </button>
                                </form>
                            @endif

                            @if ($attendanceToday->time_out)
                                <p class="mt-4 text-green-700 font-semibold">Absensi hari ini Selesai!</p>
                            @endif

                        @else
                            <p class="text-3xl font-bold text-yellow-600 mb-4">Belum Absen</p>
                            
                            {{-- Tombol Check-in (hanya tampil jika belum ada record hari ini) --}}
                            <form action="{{ route('employee.checkin') }}" method="POST" class="mt-6">
                                @csrf
                                <button type="submit" class="w-full bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-4 rounded-lg transition duration-200">
                                    <i class="fas fa-sign-in-alt mr-2"></i>
                                    CHECK IN SEKARANG
                                </button>
                            </form>
                        @endif
                    </div>
                    
                    {{-- Informasi User --}}
                    <div class="bg-blue-50 border border-blue-200 rounded-lg p-6 shadow-md">
                        <h4 class="text-xl font-semibold mb-3 text-blue-800">Informasi Saya</h4>
                        <div class="space-y-2 text-gray-700">
                            <p><strong>Peran:</strong> {{ ucfirst($user->role) }}</p>
                            <p><strong>Email:</strong> {{ $user->email }}</p>
                            <p><strong>No. HP:</strong> {{ $user->phone ?? '-' }}</p>
                            <p><strong>Jenis Kelamin:</strong> {{ $user->gender ?? '-' }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```
