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

    /**
     * Relasi ke model Attendance.
     */
    public function attendances(): HasMany
    {
        return $this->hasMany(Attendance::class);
    }
```
## 6.4. Jalankan Migrasi
Terakhir, jalankan migrasi untuk membuat kedua tabel baru di database Anda.

```Bash
php artisan migrate
```
