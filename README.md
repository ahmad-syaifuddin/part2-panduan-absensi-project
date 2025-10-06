# 📋 Part 2 - Panduan Absensi Karyawan

> **Fokus Bagian Ini:** Membangun logik dan view blade untuk menyimpan data absensi karyawan

---

## 🎯 Gambaran Umum

Sekarang kita akan fokus membangun fitur utama untuk karyawan: **melakukan absensi**. Kita akan mulai dengan merombak halaman Dashboard agar menjadi halaman utama bagi karyawan untuk melakukan absen masuk dan pulang.

---

# 📊 Tahap 7: Membangun Dashboard Absensi Karyawan

Dashboard akan kita buat **dinamis**. Tampilannya akan berbeda tergantung role pengguna:
- **Admin** → melihat dashboard ringkasan umum
- **Karyawan** → melihat panel absensi

Sebelum memulai kita edit dulu isi dari model ``app/Models/Attendance.php`` copy semua kode dibawah ini lalu paste kan:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Attendance extends Model
{
    use HasFactory;
    
    protected $fillable = [
        'employee_id',
        'date',
        'time_in',
        'time_out',
        'status',
        'notes',
    ];

    public function employee()
    {
        return $this->belongsTo(Employee::class);
    }
}
```

dan sebelum lanjut ada yg perlu kita revisi yaitu kita harus mengubah tipe kolom status dari ENUM menjadi VARCHAR. Ini memberikan fleksibilitas untuk menyimpan teks apa pun, termasuk status gabungan.

Langkah-langkahnya persis seperti solusi yang saya berikan sebelumnya. Mari kita lakukan lagi dengan benar.

### 1. Buat File Migration
Jika Anda belum membuatnya, jalankan perintah ini di terminal:
```Bash
php artisan make:migration change_status_to_varchar_in_attendances_table --table=attendances
```
### 2. Isi File Migration
Buka file migration yang baru dibuat di database/migrations/ dan pastikan isinya seperti ini:

```PHP
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('attendances', function (Blueprint $table) {
            // Mengubah kolom ENUM menjadi VARCHAR dengan panjang 50
            $table->string('status', 50)->change();
        });
    }

    public function down(): void
    {
        Schema::table('attendances', function (Blueprint $table) {
            // Jika ingin bisa rollback, definisikan kembali ENUM di sini
            // Namun, untuk kasus ini, kita bisa kosongkan atau kembalikan ke VARCHAR lama
            $table->string('status', 20)->change(); // Sesuaikan jika perlu
        });
    }
};
```
### Poin Kunci:
$table->string('status', 50): Ini akan mengubah kolom status menjadi tipe VARCHAR dengan panjang 50 karakter.
->change(): Perintah untuk memodifikasi kolom yang sudah ada.

### 3. Jalankan Migration
Terakhir, jalankan perintah ini untuk menerapkan perubahan ke database Anda:

```Bash
php artisan migrate
```
## Kesimpulan
Jadi, kesimpulannya adalah:

Hindari ENUM untuk kasus ini karena Anda butuh fleksibilitas untuk menggabungkan status.

Gunakan VARCHAR sebagai gantinya.

Setelah Anda menjalankan migrasi di atas, kolom status Anda akan siap menerima teks yang lebih panjang dan bervariasi. Coba lagi fitur "Pulang Cepat", dan error tersebut dijamin sudah hilang.

---

## 🔧 Langkah 15: Membuat Controller untuk Dashboard

Daripada menggunakan fungsi sederhana di file rute, kita akan membuat controller khusus untuk mengatur logika dashboard.

### Buat DashboardController:

```bash
php artisan make:controller DashboardController
```

### Isi Logika di DashboardController:

Buka `app/Http/Controllers/DashboardController.php` dan isi dengan kode berikut. Logika ini akan memeriksa role pengguna dan menampilkan view yang sesuai.

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class DashboardController extends Controller
{
    public function index()
    {
        // Ambil pengguna yang sedang login
        $user = Auth::user();

        // Cek role pengguna
        if ($user->role === 'admin') {
            // Jika admin, tampilkan dashboard admin
            return view('dashboard-admin'); 
        } elseif ($user->role === 'karyawan') {
            // Jika karyawan, tampilkan dashboard karyawan
            // Nanti kita akan tambahkan logika untuk mengambil data absensi di sini
            return view('dashboard-karyawan');
        }

        // Fallback jika role tidak terdefinisi (seharusnya tidak terjadi)
        return redirect('/');
    }
}
```

---

## 🛣️ Langkah 16: Menyesuaikan Rute Dashboard

Sekarang, kita ubah rute `/dashboard` di `routes/web.php` agar menggunakan `DashboardController` yang baru kita buat.

### Langkah-langkahnya:

1. Buka `routes/web.php`
2. Cari dan ubah rute `/dashboard`

**Ubah dari ini:**

```php
// use App\Http\Controllers\DashboardController; // <-- Jangan lupa tambahkan ini di atas

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');
```

**Menjadi seperti ini:**

```php
use App\Http\Controllers\DashboardController; // <-- Tambahkan ini di atas

Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware(['auth', 'verified'])->name('dashboard');
```

---

## 📄 Langkah 17: Membuat View untuk Masing-Masing Dashboard

Kita perlu membuat dua file view baru yang dipanggil oleh `DashboardController`.

### 1️⃣ Buat View Admin (`dashboard-admin.blade.php`)

- Ubah nama file lama `resources/views/dashboard.blade.php` menjadi `dashboard-admin.blade.php`
- Kita bisa biarkan isinya seperti bawaan Breeze untuk saat ini
- Nantinya, halaman ini bisa diisi dengan ringkasan data absensi, jumlah karyawan aktif, dll.

### 2️⃣ Buat View Karyawan (`dashboard-karyawan.blade.php`)

- Buat file baru di `resources/views/` bernama `dashboard-karyawan.blade.php`
- Isi dengan kode berikut. Ini adalah tampilan inti untuk absensi:

```html
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Absensi Harian') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">

                    {{-- Menampilkan pesan sukses atau error --}}
                    @if (session('success'))
                    <div class="mb-4 bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded relative"
                        role="alert">
                        <strong class="font-bold">Sukses!</strong>
                        <span class="block sm:inline">{{ session('success') }}</span>
                    </div>
                    @endif
                    @if (session('error'))
                    <div class="mb-4 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative"
                        role="alert">
                        <strong class="font-bold">Gagal!</strong>
                        <span class="block sm:inline">{{ session('error') }}</span>
                    </div>
                    @endif

                    <div class="text-center">
                        <h3 class="text-lg font-medium text-gray-900">
                            {{ \Carbon\Carbon::now('Asia/Makassar')->isoFormat('dddd, D MMMM Y') }}
                        </h3>
                        <div id="jam-digital" class="text-5xl font-bold text-gray-800 my-4">00:00:00</div>

                        <div class="mt-6">
                            {{-- LOGIKA BARU DIMULAI DI SINI --}}
                            @php
                            $currentTime = \Carbon\Carbon::now('Asia/Makassar');
                            $endOfDay = $currentTime->copy()->setTime(14, 0, 0);
                            $isPastOfficeHours = $currentTime->gt($endOfDay);
                            @endphp

                            @if (isset($todaysAttendance))
                            {{-- JIKA SUDAH ADA DATA ABSENSI HARI INI --}}
                            @if (in_array($todaysAttendance->status, ['Izin', 'Sakit']))
                            <div class="bg-yellow-100 text-yellow-800 font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                Anda hari ini tercatat: <strong>{{ $todaysAttendance->status }}</strong>.
                            </div>
                            @elseif ($todaysAttendance->status === 'Alpa')
                            {{-- Jika statusnya Alpa --}}
                            <div class="bg-red-100 text-red-800 font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                Anda hari ini tercatat: <strong>Alpa</strong>.
                            </div>
                            @elseif ($todaysAttendance->time_out === null)
                            <form method="POST" action="{{ route('attendance.clockout') }}" class="inline-block">
                                @csrf
                                <button type="submit"
                                    class="bg-red-500 hover:bg-red-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                    <i class="fas fa-sign-out-alt mr-2"></i> Absen Pulang
                                </button>
                            </form>
                            @else
                            <div class="bg-blue-100 text-blue-800 font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                Absensi hari ini telah selesai.
                            </div>
                            @endif
                            @else
                            {{-- JIKA BELUM ADA DATA ABSENSI HARI INI --}}
                            @if ($isPastOfficeHours)
                            {{-- Jika sudah lewat jam 14:00 --}}
                            <div class="bg-red-100 text-red-800 font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                Waktu untuk melakukan absensi hari ini telah berakhir.
                            </div>
                            @else
                            {{-- Jika masih dalam jam kerja --}}
                            <div class="space-x-4">
                                <form method="POST" action="{{ route('attendance.clockin') }}" class="inline-block">
                                    @csrf
                                    <button type="submit"
                                        class="bg-green-500 hover:bg-green-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                        <i class="fas fa-fingerprint mr-2"></i> Absen Masuk
                                    </button>
                                </form>
                                <button onclick="openPermissionModal('Izin')"
                                    class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                    <i class="fas fa-file-alt mr-2"></i> Ajukan Izin
                                </button>
                                <button onclick="openPermissionModal('Sakit')"
                                    class="bg-orange-500 hover:bg-orange-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                    <i class="fas fa-notes-medical mr-2"></i> Ajukan Sakit
                                </button>
                            </div>
                            @endif
                            @endif
                        </div>
                    </div>

                    {{-- Ringkasan Absensi --}}
                    <div class="mt-10 border-t pt-6">
                        <h4 class="text-md font-semibold mb-4">Ringkasan Hari Ini:</h4>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <div class="bg-blue-100 p-4 rounded-lg">
                                <p class="text-sm text-blue-700">Jam Masuk</p>
                                <p class="text-lg font-bold text-blue-900">{{ $todaysAttendance?->time_in ?? '--:--' }}
                                </p>
                            </div>
                            <div class="bg-purple-100 p-4 rounded-lg">
                                <p class="text-sm text-purple-700">Jam Pulang</p>
                                <p class="text-lg font-bold text-purple-900">{{ $todaysAttendance?->time_out ?? '--:--'
                                    }}</p>
                            </div>
                            <div class="bg-yellow-100 p-4 rounded-lg">
                                <p class="text-sm text-yellow-700">Status</p>
                                <p class="text-lg font-bold text-yellow-900">{{ $todaysAttendance?->status ?? 'Belum
                                    Absen' }}</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    {{-- MODAL UNTUK IZIN/SAKIT --}}
    <div id="permissionModal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden">
        <div class="relative top-20 mx-auto p-5 border w-96 shadow-lg rounded-md bg-white">
            <div class="mt-3 text-center">
                <h3 class="text-lg leading-6 font-medium text-gray-900" id="modalTitle">Form Pengajuan</h3>
                <div class="mt-2 px-7 py-3">
                    <form id="permissionForm" method="POST" action="{{ route('attendance.store_permission') }}">
                        @csrf
                        <input type="hidden" name="status" id="statusInput">
                        <textarea name="notes"
                            class="w-full px-3 py-2 text-gray-700 border rounded-lg focus:outline-none" rows="4"
                            placeholder="Tuliskan keterangan Anda di sini..." required></textarea>
                        <div class="items-center px-4 py-3">
                            <button type="submit"
                                class="px-4 py-2 bg-green-500 text-white text-base font-medium rounded-md w-full shadow-sm hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-300">
                                Kirim Pengajuan
                            </button>
                        </div>
                    </form>
                </div>
                <div class="items-center px-4 py-1">
                    <button onclick="closePermissionModal()"
                        class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-md w-full shadow-sm hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-300">
                        Batal
                    </button>
                </div>
            </div>
        </div>
    </div>

    @push('scripts')
    <script>
        function updateClock() {
            const now = new Date(new Date().toLocaleString('en-US', { timeZone: 'Asia/Makassar' }));
            const hours = String(now.getHours()).padStart(2, '0');
            const minutes = String(now.getMinutes()).padStart(2, '0');
            const seconds = String(now.getSeconds()).padStart(2, '0');
            document.getElementById('jam-digital').textContent = `${hours}:${minutes}:${seconds}`;
        }
        setInterval(updateClock, 1000);
        updateClock();

        // --- JavaScript untuk Modal ---
        const modal = document.getElementById('permissionModal');
        const modalTitle = document.getElementById('modalTitle');
        const statusInput = document.getElementById('statusInput');

        function openPermissionModal(status) {
            modal.classList.remove('hidden');
            modalTitle.textContent = `Form Pengajuan ${status}`;
            statusInput.value = status;
        }

        function closePermissionModal() {
            modal.classList.add('hidden');
        }
    </script>
    @endpush
</x-app-layout>
```

### 3️⃣ Tambahkan `@stack('scripts')` ke Layout Utama

Agar Javascript untuk jam digital bisa berjalan, kita perlu menambahkan "slot" untuk script di layout utama.

- Buka `resources/views/layouts/app.blade.php`
- Tambahkan `@stack('scripts')` tepat sebelum tag penutup `</body>`

```html
        </main>
    </div>

    @stack('scripts')
</body>
</html>
```

---

## ✅ Uji Coba

1. **Login sebagai admin** (`admin@gmail.com`)
   - Anda seharusnya melihat halaman dashboard default Breeze yang lama (sekarang dari `dashboard-admin.blade.php`)
   - Navigasi untuk admin akan muncul

2. **Logout, lalu login sebagai karyawan** (`karyawan@gmail.com`)
   - Anda akan disambut oleh halaman dashboard absensi yang baru
   - Lengkap dengan jam digital yang berjalan
   - Navigasi untuk karyawan akan muncul

---

---

# ⚙️ Tahap 8: Membuat Logika Absensi (Controller & Rute)

> Sekarang saatnya menghidupkan tombol-tombol absensi itu dengan membuat "mesin" di belakangnya. Ini melibatkan pembuatan Controller baru untuk menangani semua logika absensi dan Rute sebagai alamatnya.

---

## 🎮 Langkah 18: Membuat AttendanceController

Controller ini akan menjadi **pusat komando** untuk semua aksi terkait absensi, seperti absen masuk dan absen pulang.

Jalankan perintah ini di terminal:

```bash
php artisan make:controller AttendanceController
```

---

## 🛣️ Langkah 19: Membuat Rute untuk Aksi Absensi

Kita perlu dua alamat baru:
- Satu untuk mengirim data **"Absen Masuk"**
- Satu lagi untuk **"Absen Pulang"**

### Langkah-langkahnya:

1. Buka file `routes/web.php`
2. Tambahkan rute berikut di dalam grup `middleware('auth')`
3. Anda bisa meletakkannya bersama rute profil

```php
// routes/web.php

use App\Http\Controllers\AttendanceController; // <-- Jangan lupa tambahkan ini

// ... Rute lainnya ...

Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');

    // RUTE UNTUK ABSENSI
    Route::post('/attendance/clock-in', [AttendanceController::class, 'clockIn'])->name('attendance.clockin');
    Route::post('/attendance/clock-out', [AttendanceController::class, 'clockOut'])->name('attendance.clockout');

    // RUTE BARU UNTUK IZIN/SAKIT
    Route::post('/attendance/permission', [AttendanceController::class, 'storePermission'])->name('attendance.store_permission');
});
// ... Rute admin ...
```

---

## 🔵 Langkah 20: Implementasi Logika Absen Masuk (clockIn)

Sekarang kita isi `AttendanceController` dengan logika untuk `clockIn`.

Buka `app/Http/Controllers/AttendanceController.php` dan isi dengan kode berikut:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Models\Attendance; // <-- Import model Attendance
use Carbon\Carbon;          // <-- Import Carbon untuk manajemen waktu

class AttendanceController extends Controller
{
    /**
     * Store a newly created resource in storage for clock in.
     */
    public function clockIn(Request $request)
    {
        $user = Auth::user();
        // Pastikan user memiliki relasi employee
        if (!$user->employee) {
            return redirect()->route('dashboard')->with('error', 'Data karyawan tidak ditemukan.');
        }

        $timezone = 'Asia/Makassar'; // WITA
        $now = Carbon::now($timezone);
        $today = $now->format('Y-m-d');
        $time = $now->format('H:i:s');

        // 1. Cek apakah sudah absen masuk hari ini
        $existingAttendance = Attendance::where('employee_id', $user->employee->id)
                                        ->where('date', $today)
                                        ->first();

        if ($existingAttendance) {
            return redirect()->route('dashboard')->with('error', 'Anda sudah melakukan absen masuk hari ini.');
        }

        // 2. Cek apakah hari ini adalah hari kerja (Senin-Jumat)
        if ($now->isWeekend()) {
            return redirect()->route('dashboard')->with('error', 'Absensi hanya bisa dilakukan pada hari kerja.');
        }

        // 3. Tentukan status (Terlambat atau Hadir)
        $lateTime = Carbon::createFromTimeString('08:01:00', $timezone);
        $status = 'Hadir';
        if ($now->gt($lateTime)) {
            $status = 'Terlambat';
        }

        // 4. Simpan data absensi
        Attendance::create([
            'employee_id' => $user->employee->id,
            'date' => $today,
            'time_in' => $time,
            'status' => $status,
        ]);

        return redirect()->route('dashboard')->with('success', 'Berhasil melakukan absen masuk.');
    }
}
```

---

## 🔴 Langkah 21: Implementasi Logika Absen Pulang (clockOut)
Revisi:
Kita perlu menambahkan logika untuk memeriksa apakah karyawan absen sebelum jam pulang resmi (14:00 WITA). Untuk melakukan ini, kita harus menambahkan kolom baru di attendances table untuk status pulang, atau memodifikasi status yang ada. Namun, untuk simplisitas, kita bisa memodifikasi status yang sudah ada saat clockOut.

Saran Implementasi:

Tambahkan definisi jam pulang resmi.

Saat clockOut, bandingkan waktu saat ini dengan jam pulang resmi.

Update kolom status jika karyawan pulang cepat.

Tambahkan method `clockOut` di dalam `AttendanceController.php`.

```php
// app/Http/Controllers/AttendanceController.php

// ... (method clockIn) ...

/**
 * Update the specified resource in storage for clock out.
 */
public function clockOut(Request $request)
{
    $user = Auth::user();
    if (!$user->employee) {
        return redirect()->route('dashboard')->with('error', 'Data karyawan tidak ditemukan.');
    }

    $timezone = 'Asia/Makassar'; // WITA
    $now = Carbon::now($timezone);
    $today = $now->format('Y-m-d');
    $time = $now->format('H:i:s');

    // 1. Cari data absensi masuk hari ini
    $attendance = Attendance::where('employee_id', $user->employee->id)
                              ->where('date', '>=', $today)
                              ->whereNull('time_out') // Pastikan belum absen pulang
                              ->first();


    // 2. Jika tidak ditemukan data absen masuk
    if (!$attendance) {
        return redirect()->route('dashboard')->with('error', 'Anda belum melakukan absen masuk hari ini.');
    }

    // 3. Jika sudah ada data absen pulang (sebagai double check)
    if ($attendance->time_out) {
        return redirect()->route('dashboard')->with('error', 'Anda sudah melakukan absen pulang hari ini.');
    }

    // --- REVISI DIMULAI DI SINI ---
    // 4. Tentukan status Pulang Cepat
    $officialTimeOut = Carbon::createFromTimeString('14:00:00', $timezone);

    // Ambil status jam masuk sebelumnya (Hadir atau Terlambat)
    $statusMasuk = $attendance->status;

    // Default status pulang adalah status saat masuk
    $newStatus = $statusMasuk;

    if ($now->lt($officialTimeOut)) {
        // Gabungkan status masuk dengan status pulang cepat
        // Contoh: "Hadir, Pulang Cepat" atau "Terlambat, Pulang Cepat"
        $newStatus .= ', Pulang Cepat';
    }
    // --- REVISI SELESAI ---


    // 5. Update data absen pulang dengan status baru
    $attendance->update([
        'time_out' => $time,
        'status' => $newStatus, // Gunakan status yang sudah diperbarui
    ]);

    return redirect()->route('dashboard')->with('success', 'Berhasil melakukan absen pulang. Selamat beristirahat!');
}
```

---

## 🔗 Langkah 22: Menghubungkan Data dan Logika ke Dashboard

Terakhir, kita sempurnakan `DashboardController` dan view `dashboard-karyawan` agar bisa menampilkan data absensi hari ini dan tombol yang dinamis.

### 1️⃣ Update DashboardController

Buka `app/Http/Controllers/DashboardController.php` dan modifikasi method `index` untuk mengambil data absensi.

```php
// app/Http/Controllers/DashboardController.php
use App\Models\Attendance; // <-- Tambahkan ini
use Carbon\Carbon;          // <-- Tambahkan ini

// ...

public function index()
{
    $user = Auth::user();

    if ($user->role === 'admin') {
        return view('dashboard-admin');
    } elseif ($user->role === 'karyawan' && $user->employee) {
        // Ambil data absensi HARI INI untuk karyawan yang login
        $today = Carbon::now('Asia/Makassar')->format('Y-m-d');
        $todaysAttendance = Attendance::where('employee_id', $user->employee->id)
                                      ->where('date', $today)
                                      ->first();

        // Kirim data absensi ke view
        return view('dashboard-karyawan', compact('todaysAttendance'));
    }

    // Jika karyawan tapi data employee belum ada, atau role lain.
    return view('dashboard-karyawan');
}
```

### 2️⃣ Update View dashboard-karyawan.blade.php

Ganti seluruh isi `resources/views/dashboard-karyawan.blade.php` dengan kode yang sudah dinamis berikut:

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Absensi Harian') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">

                    @if (session('success'))
                        <div class="mb-4 bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded relative" role="alert">
                            <strong class="font-bold">Sukses!</strong>
                            <span class="block sm:inline">{{ session('success') }}</span>
                        </div>
                    @endif
                    @if (session('error'))
                        <div class="mb-4 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative" role="alert">
                            <strong class="font-bold">Gagal!</strong>
                            <span class="block sm:inline">{{ session('error') }}</span>
                        </div>
                    @endif

                    <div class="text-center">
                        <h3 class="text-lg font-medium text-gray-900">
                            {{ \Carbon\Carbon::now('Asia/Makassar')->isoFormat('dddd, D MMMM Y') }}
                        </h3>
                        <div id="jam-digital" class="text-5xl font-bold text-gray-800 my-4">00:00:00</div>

                        <div class="mt-6 space-x-4">
                            @if (isset($todaysAttendance))
                                @if ($todaysAttendance->time_out === null)
                                    <form method="POST" action="{{ route('attendance.clockout') }}" class="inline-block">
                                        @csrf
                                        <button type="submit" class="bg-red-500 hover:bg-red-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg transition duration-300 ease-in-out transform hover:scale-105">
                                            <i class="fas fa-sign-out-alt mr-2"></i> Absen Pulang
                                        </button>
                                    </form>
                                @else
                                    <div class="bg-blue-100 text-blue-800 font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                        Absensi hari ini telah selesai.
                                    </div>
                                @endif
                            @else
                                <form method="POST" action="{{ route('attendance.clockin') }}" class="inline-block">
                                    @csrf
                                    <button type="submit" class="bg-green-500 hover:bg-green-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg transition duration-300 ease-in-out transform hover:scale-105">
                                        <i class="fas fa-fingerprint mr-2"></i> Absen Masuk
                                    </button>
                                </form>
                            @endif
                        </div>
                    </div>

                    <div class="mt-10 border-t pt-6">
                        <h4 class="text-md font-semibold mb-4">Ringkasan Hari Ini:</h4>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <div class="bg-blue-100 p-4 rounded-lg">
                                <p class="text-sm text-blue-700">Jam Masuk</p>
                                <p class="text-lg font-bold text-blue-900">{{ $todaysAttendance?->time_in ?? '--:--' }}</p>
                            </div>
                            <div class="bg-purple-100 p-4 rounded-lg">
                                <p class="text-sm text-purple-700">Jam Pulang</p>
                                <p class="text-lg font-bold text-purple-900">{{ $todaysAttendance?->time_out ?? '--:--' }}</p>
                            </div>
                            <div class="bg-yellow-100 p-4 rounded-lg">
                                <p class="text-sm text-yellow-700">Status</p>
                                <p class="text-lg font-bold text-yellow-900">{{ $todaysAttendance?->status ?? 'Belum Absen' }}</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    @push('scripts')
    <script>
        function updateClock() {
            // WITA is UTC+8
            const now = new Date(new Date().toLocaleString('en-US', { timeZone: 'Asia/Makassar' }));
            const hours = String(now.getHours()).padStart(2, '0');
            const minutes = String(now.getMinutes()).padStart(2, '0');
            const seconds = String(now.getSeconds()).padStart(2, '0');
            document.getElementById('jam-digital').textContent = `${hours}:${minutes}:${seconds}`;
        }
        setInterval(updateClock, 1000);
        updateClock();
    </script>
    @endpush
</x-app-layout>
```

buat rute baru di // routes/web.php :

```php
// ... rute lainnya

Route::middleware('auth')->group(function () {
    // ... (rute profile & absensi sebelumnya) ...

    // RUTE UNTUK ABSENSI
    Route::get('/attendance', [AttendanceController::class, 'index'])->name('attendance.index');
    Route::post('/attendance/clock-in', [AttendanceController::class, 'clockIn'])->name('attendance.clockin');
    Route::post('/attendance/clock-out', [AttendanceController::class, 'clockOut'])->name('attendance.clockout');
    
    // RUTE BARU UNTUK IZIN/SAKIT
    Route::post('/attendance/permission', [AttendanceController::class, 'storePermission'])->name('attendance.store_permission');
});
```
tambahkan method controller nya di app/Http/Controllers/AttendanceController.php dan tambah kan method baru bernama storePermission:
```php
// app/Http/Controllers/AttendanceController.php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Models\Attendance;
use Carbon\Carbon;

class AttendanceController extends Controller
{
    // ... (method index, clockIn, clockOut) ...


    /**
     * Store a newly created resource in storage for permission (Izin/Sakit).
     */
    public function storePermission(Request $request)
    {
        // 1. Validasi input
        $request->validate([
            'status' => 'required|in:Izin,Sakit',
            'notes'  => 'required|string|max:255',
        ]);

        $user = Auth::user();
        if (!$user->employee) {
            return redirect()->route('dashboard')->with('error', 'Data karyawan tidak ditemukan.');
        }

        $timezone = 'Asia/Makassar';
        $today = Carbon::now($timezone)->format('Y-m-d');

        // 2. Cek apakah sudah ada absensi hari ini (masuk/pulang/izin/sakit)
        $existingAttendance = Attendance::where('employee_id', $user->employee->id)
                                        ->where('date', $today)
                                        ->first();

        if ($existingAttendance) {
            return redirect()->route('dashboard')->with('error', 'Anda sudah memiliki catatan absensi untuk hari ini.');
        }

        // 3. Simpan data izin/sakit
        Attendance::create([
            'employee_id' => $user->employee->id,
            'date' => $today,
            'status' => $request->status,
            'notes' => $request->notes,
            'time_in' => null, // Tidak ada jam masuk
            'time_out' => null, // Tidak ada jam pulang
        ]);

        return redirect()->route('dashboard')->with('success', 'Pengajuan ' . $request->status . ' berhasil disimpan.');
    }
}
```

Kita akan mengimplementasikan fungsionalitas ini dengan membuat sebuah tombol khusus di dashboard admin. Saat tombol ini diklik, sistem akan menjalankan skrip untuk menandai semua karyawan yang belum absen hari itu sebagai "Alpa".

Mari kita mulai.

## Langkah 1: Buat Rute Khusus Admin
Pertama, kita definisikan "alamat" atau rute yang akan dituju saat tombol di-klik. Rute ini harus aman dan hanya bisa diakses oleh admin.

Buka file routes/web.php dan tambahkan rute berikut. Tempatkan di dalam grup rute admin Anda jika ada, atau buat yang baru.

```PHP
// routes/web.php
use App\Http\Controllers\AttendanceController;

// ... rute lainnya

// Pastikan rute ini dilindungi oleh middleware untuk admin
Route::middleware(['auth'])->group(function () {
    // ... rute-rute yang sudah ada ...
    
    // RUTE BARU UNTUK ADMIN: Menandai karyawan yang alpa
    // Kita tambahkan middleware 'role:admin' jika Anda sudah membuatnya
    // Jika belum, logika di controller akan memvalidasi role
    Route::post('/admin/attendance/mark-alpa', [AttendanceController::class, 'markAlpa'])
            ->name('admin.attendance.mark_alpa');
});
```
Catatan: Idealnya, rute ini dilindungi oleh middleware role admin. Jika Anda belum membuatnya, logika di dalam controller yang akan kita buat selanjutnya akan tetap memeriksa role pengguna.

## Langkah 2: Tambahkan Method di Controller
Sekarang kita buat logikanya. Kita akan menambahkan method markAlpa ke dalam AttendanceController. Logikanya hampir sama dengan command yang kita bahas sebelumnya.

Buka file app/Http/Controllers/AttendanceController.php dan tambahkan method baru di bawah ini:

```PHP
// app/Http/Controllers/AttendanceController.php

use App\Models\Employee; // Pastikan ini sudah ada di atas

class AttendanceController extends Controller
{
    // ... (method index, clockIn, clockOut, storePermission) ...


    /**
     * Mark absent employees as 'Alpa' for today.
     * This action is triggered manually by an admin.
     */
    public function markAlpa(Request $request)
    {
        // 1. Pastikan hanya admin yang bisa mengakses
        if (Auth::user()->role !== 'admin') {
            return redirect()->route('dashboard')->with('error', 'Anda tidak memiliki hak akses.');
        }

        $timezone = 'Asia/Makassar';
        $today = Carbon::now($timezone)->format('Y-m-d');
        $now = Carbon::now($timezone);

        // 2. Jangan jalankan jika hari libur (Sabtu/Minggu)
        if ($now->isWeekend()) {
            return redirect()->route('dashboard')->with('info', 'Tidak ada tindakan yang diambil pada hari libur.');
        }

        // 3. Dapatkan semua ID karyawan yang statusnya 'aktif'
        $activeEmployeeIds = Employee::where('status', 'aktif')->pluck('id');

        // 4. Dapatkan ID karyawan yang sudah punya catatan absensi hari ini
        $attendedEmployeeIds = Attendance::where('date', $today)->pluck('employee_id');

        // 5. Cari ID karyawan yang aktif tapi belum absen (dianggap Alpa)
        $absentEmployeeIds = $activeEmployeeIds->diff($attendedEmployeeIds);

        if ($absentEmployeeIds->isEmpty()) {
            return redirect()->route('dashboard')->with('info', 'Semua karyawan aktif sudah memiliki catatan absensi hari ini.');
        }

        // 6. Buat catatan 'Alpa' untuk setiap karyawan yang absen
        foreach ($absentEmployeeIds as $employeeId) {
            Attendance::create([
                'employee_id' => $employeeId,
                'date' => $today,
                'status' => 'Alpa',
                'notes' => 'Tidak melakukan absensi masuk hingga akhir hari kerja.',
            ]);
        }

        return redirect()->route('dashboard')->with('success', $absentEmployeeIds->count() . ' karyawan telah ditandai Alpa.');
    }
}
```

### update view dashboard-admin.blade.php :
```blade
{{-- Tambahkan blok PHP untuk memeriksa waktu --}}
@php
$currentTime = \Carbon\Carbon::now('Asia/Makassar');
$endOfDay = $currentTime->copy()->setTime(14, 0, 0);
$isPastOfficeHours = $currentTime->gt($endOfDay);
@endphp
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Admin Dashboard') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    {{ __("You're logged in as Admin!") }}

                    {{-- Kotak Aksi Admin --}}
                    <div class="mt-8 border-t pt-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Aksi Cepat</h3>
                        {{-- Tampilkan tombol hanya jika sudah lewat jam 14:00 --}}
                        @if ($isPastOfficeHours)
                        <form action="{{ route('admin.attendance.mark_alpa') }}" method="POST"
                            onsubmit="return confirm('Apakah Anda yakin ingin menandai semua karyawan yang belum absen sebagai ALPA untuk hari ini? Tindakan ini tidak bisa dibatalkan.');">
                            @csrf
                            <button type="submit"
                                class="bg-red-600 hover:bg-red-800 text-white font-bold py-2 px-4 rounded-lg shadow-md transition duration-300">
                                <i class="fas fa-user-times mr-2"></i>
                                Tandai Karyawan yg Alpa Hari Ini
                            </button>
                        </form>
                        <p class="text-sm text-gray-500 mt-2">
                            * Klik tombol ini untuk secara otomatis mengisi status "Alpa" bagi karyawan yang tidak
                            melakukan absensi sama sekali hari ini.
                        </p>
                        @else
                        {{-- Tampilkan pesan jika belum waktunya --}}
                        <div class="bg-blue-100 border-l-4 border-blue-500 text-blue-700 p-4" role="alert">
                            <p class="font-bold">Informasi</p>
                            <p>Tombol untuk memproses absensi "Alpa" akan muncul di sini setelah jam kerja berakhir
                                (pukul 14:00 WITA).</p>
                        </div>
                        @endif
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

---

## ✅ Uji Coba

**Ini adalah momen pembuktian!** 🎉

1. Pastikan Anda login sebagai **karyawan**
2. Buka halaman **Dashboard**. Anda akan melihat tombol **"Absen Masuk"**
3. Klik tombol **"Absen Masuk"**:
   - Halaman akan me-refresh
   - Pesan sukses akan muncul
   - Ringkasan di bawah akan terisi
   - Tombol akan berubah menjadi **"Absen Pulang"**
4. Tunggu beberapa saat, lalu klik **"Absen Pulang"**:
   - Halaman akan me-refresh lagi
   - Pesan sukses muncul
   - Ringkasan jam pulang terisi
   - Tombol akan hilang digantikan pesan bahwa absensi telah selesai
5. Coba refresh halaman, statusnya akan tetap sama

---

---

# 📜 Tahap 9: Membuat Halaman Riwayat Absensi

> Setelah karyawan bisa melakukan absensi, tentu mereka perlu melihat rekap atau histori dari absensi yang telah mereka lakukan. Sekarang kita akan membuat halaman **"Riwayat Absensi"**.

Halaman ini akan berisi tabel yang menampilkan **semua catatan absensi** milik karyawan yang sedang login, diurutkan dari yang paling baru.

---

## 📊 Langkah 23: Logika Menampilkan Riwayat di Controller

Kita akan menambahkan method baru bernama `index` ke dalam `AttendanceController` untuk mengambil data riwayat absensi.

### Langkah-langkahnya:

1. Buka `app/Http/Controllers/AttendanceController.php`
2. Tambahkan method `index` berikut ini:

```php
// app/Http/Controllers/AttendanceController.php

// ... (use statements) ...

class AttendanceController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        $user = Auth::user();

        // Pastikan user adalah karyawan dan memiliki data employee
        if ($user->role !== 'karyawan' || !$user->employee) {
            // Redirect atau tampilkan error jika bukan karyawan
            return redirect()->route('dashboard')->with('error', 'Anda tidak memiliki akses ke halaman ini.');
        }

        // Ambil semua data absensi milik karyawan tersebut,
        // urutkan dari yang terbaru, dan paginasi (15 data per halaman)
        $attendances = Attendance::where('employee_id', $user->employee->id)
                                ->latest() // ini sama dengan orderBy('created_at', 'desc')
                                ->paginate(15);

        // Kirim data ke view
        return view('attendance.index', compact('attendances'));
    }

    // ... (method clockIn dan clockOut) ...
}
```

---

## 🛣️ Langkah 24: Membuat Rute untuk Halaman Riwayat

Sekarang, kita buat "alamat" atau rute untuk mengakses method `index` yang baru saja kita buat.

### Langkah-langkahnya:

1. Buka file `routes/web.php`
2. Tambahkan rute `GET` berikut di dalam grup `middleware('auth')`

```php
// routes/web.php

// ...

Route::middleware('auth')->group(function () {
    // ... (rute profile) ...

    // RUTE UNTUK ABSENSI
    Route::get('/attendance', [AttendanceController::class, 'index'])->name('attendance.index'); // <-- TAMBAHKAN INI
    Route::post('/attendance/clock-in', [AttendanceController::class, 'clockIn'])->name('attendance.clockin');
    Route::post('/attendance/clock-out', [AttendanceController::class, 'clockOut'])->name('attendance.clockout');
});

// ...
```

---

## 📄 Langkah 25: Membuat View Riwayat Absensi

Saatnya membuat tampilan untuk halaman riwayat.

### Langkah-langkahnya:

1. Buat folder baru bernama `attendance` di dalam `resources/views`
2. Di dalam folder `attendance` tersebut, buat file baru bernama `index.blade.php`
3. Isi file `index.blade.php` tersebut dengan kode berikut:

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Riwayat Absensi Saya') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">

                    <div class="overflow-x-auto">
                        <table class="min-w-full divide-y divide-gray-200">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        No</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Tanggal</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Jam Masuk</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Jam Pulang</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Status</th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Catatan</th>
                                </tr>
                            </thead>
                            <tbody class="bg-white divide-y divide-gray-200">
                                @forelse ($attendances as $attendance)
                                <tr>
                                    <td class="px-6 py-4 whitespace-nowrap">{{ $loop->iteration + $attendances->firstItem() - 1 }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->date }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->time_in }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->time_out }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap">
                                        {{-- Cek status utama (Hadir, Terlambat, Izin, Alpa) --}}
                                        @if (str_contains($attendance->status, 'Hadir'))
                                        <span
                                            class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800">
                                            Hadir
                                        </span>
                                        @elseif (str_contains($attendance->status, 'Terlambat'))
                                        <span
                                            class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-yellow-100 text-yellow-800">
                                            Terlambat
                                        </span>
                                        @elseif (str_contains($attendance->status, 'Izin'))
                                        <span
                                            class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-blue-100 text-blue-800">
                                            Izin
                                        </span>
                                        @elseif (str_contains($attendance->status, 'Sakit'))
                                        <span
                                            class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-cyan-100 text-cyan-800">
                                            Sakit
                                        </span>
                                        @else
                                        <span
                                            class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-red-100 text-red-800">
                                            {{-- Menampilkan status 'Alpa' atau status lain yang tidak terdefinisi --}}
                                            {{ $attendance->status }}
                                        </span>
                                        @endif

                                        {{-- Cek status tambahan 'Pulang Cepat' secara terpisah --}}
                                        @if (str_contains($attendance->status, 'Pulang Cepat'))
                                        <span
                                            class="ml-2 px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-purple-100 text-purple-800">
                                            Pulang Cepat
                                        </span>
                                        @endif
                                    </td>
                                    <td>{{ $attendance->notes }}</td>
                                </tr>
                                @empty
                                <tr>
                                    <td colspan="6"
                                        class="px-6 py-4 whitespace-nowrap text-center text-sm text-gray-500">
                                        Data riwayat absensi tidak ditemukan.
                                    </td>
                                </tr>
                                @endforelse
                            </tbody>
                        </table>
                    </div>

                    <div class="mt-4">
                        {{ $attendances->links() }}
                    </div>

                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

---

## 🔗 Langkah 26: Mengaktifkan Link di Navigasi

Langkah terakhir adalah **"menyalakan"** link "Riwayat Absensi" di menu navigasi yang sebelumnya sudah kita siapkan.

### Langkah-langkahnya:

1. Buka file `resources/views/layouts/navigation.blade.php`
2. Cari bagian menu untuk karyawan dan hapus komentar serta ganti href-nya

### Untuk Desktop Navigation

**Ubah dari ini:**

```html
<x-nav-link :href="'#'" {{-- :href="route('attendance.index')" :active="request()->routeIs('attendance.*')" --}}>
    {{ __('Riwayat Absensi') }}
</x-nav-link>
```

**Menjadi seperti ini:**

```html
<x-nav-link :href="route('attendance.index')" :active="request()->routeIs('attendance.*')">
    {{ __('Riwayat Absensi') }}
</x-nav-link>
```

### Untuk Mobile/Responsive Navigation

**Ubah dari ini:**

```html
<x-responsive-nav-link :href="'#'" {{-- :href="route('attendance.index')" :active="request()->routeIs('attendance.*')" --}}>
    {{ __('Riwayat Absensi') }}
</x-responsive-nav-link>
```

**Menjadi seperti ini:**

```html
<x-responsive-nav-link :href="route('attendance.index')" :active="request()->routeIs('attendance.*')">
    {{ __('Riwayat Absensi') }}
</x-responsive-nav-link>
```

---

## ✅ Uji Coba

**Waktunya Testing!** 🧪

1. Login sebagai **karyawan**
2. Di menu navigasi atas, link **"Riwayat Absensi"** sekarang sudah bisa diklik
3. Klik link tersebut
4. Anda akan diarahkan ke halaman `/attendance` yang menampilkan tabel berisi catatan absensi yang sudah Anda buat sebelumnya
5. Cek apakah:
   - ✅ Data absensi tampil dengan benar
   - ✅ Status badge memiliki warna yang sesuai (hijau untuk Hadir, kuning untuk Terlambat)
   - ✅ Pagination berfungsi (jika data lebih dari 15)
   - ✅ Format tanggal dan waktu tampil dengan benar

---

## 🎉 Selamat! Part 2 Selesai!

Anda telah berhasil menyelesaikan **Part 2 - Panduan Absensi Karyawan** yang mencakup:

✅ **Tahap 7:** Dashboard Absensi Karyawan dengan jam digital  
✅ **Tahap 8:** Logika Absensi (Clock In & Clock Out)  
✅ **Tahap 9:** Halaman Riwayat Absensi dengan tabel dan pagination

---

## 🚀 Lanjut ke Part 3?

**Part 3** akan membahas:
- 📊 Laporan Absensi Harian untuk Admin
- 🏖️ CRUD (Management) Hari Libur
- 📈 Dashboard Admin dengan statistik lengkap

Untuk melanjutkan, kunjungi panduan berikutnya:  
👉 [Part 3 - Panduan Absensi Project](https://github.com/ahmad-syaifuddin/part3-panduan-absensi-project.git)

---

**💡 Tips:** Sebelum lanjut ke Part 3, pastikan semua fitur di Part 2 sudah berjalan dengan baik!
