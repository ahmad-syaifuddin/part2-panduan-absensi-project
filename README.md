# 📋 Part 2 - Panduan Absensi Karyawan

> **🎯 Fokus Bagian Ini:** Membangun logik dan view blade untuk menyimpan data absensi karyawan

---

## 🌟 Gambaran Umum

Nah, sekarang kita mau fokus bikin fitur utama buat karyawan nih: **sistem absensi!** Kita bakal mulai dengan ngerombak halaman Dashboard biar jadi tempat karyawan buat absen masuk dan pulang. Seru kan? Let's go!

---

# 📊 Tahap 7: Membangun Dashboard Absensi Karyawan

Dashboard yang kita bikin bakal **dinamis** banget guys! Tampilannya bakal beda-beda tergantung siapa yang login:
- **Admin** → bakal liat dashboard ringkasan umum
- **Karyawan** → bakal liat panel absensi

---

## 🔧 Persiapan Awal

### Langkah 1: Edit Model Attendance

Sebelum mulai, kita perlu edit dulu isi dari model `app/Models/Attendance.php`. Copy semua kode di bawah ini terus paste:

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

---

### Langkah 2: Ubah Tipe Kolom Status

Nah, ada yang perlu kita revisi nih. Kita harus ngubah tipe kolom `status` dari ENUM jadi VARCHAR. Kenapa? Biar lebih fleksibel dan bisa nyimpen teks apa aja, termasuk status gabungan kayak "Terlambat, Pulang Cepat". 

#### 2.1 Buat File Migration

Jalanin perintah ini di terminal:

```bash
php artisan make:migration change_status_to_varchar_in_attendances_table --table=attendances
```

#### 2.2 Isi File Migration

Buka file migration yang baru dibuat di `database/migrations/` dan pastikan isinya kayak gini:

```php
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
            // Kalau mau rollback, bisa kembaliin ke VARCHAR lama
            $table->string('status', 20)->change();
        });
    }
};
```

**💡 Poin Kunci:**
- `$table->string('status', 50)`: Ini bakal ngubah kolom status jadi VARCHAR dengan panjang 50 karakter
- `->change()`: Perintah buat modifikasi kolom yang udah ada

#### 2.3 Jalankan Migration

Terakhir, jalanin perintah ini buat apply perubahan ke database:

```bash
php artisan migrate
```

**✅ Kesimpulan:**
- Hindari ENUM buat kasus kayak gini karena kita butuh fleksibilitas
- Gunakan VARCHAR sebagai gantinya
- Setelah jalanin migrasi di atas, kolom status siap nerima teks yang lebih panjang dan bervariasi

---

## 🎮 Langkah 15: Membuat Dashboard Controller

Daripada pake fungsi sederhana di file rute, mendingan kita bikin controller khusus buat ngatur logika dashboard. Lebih rapi dan profesional! 

### 15.1 Buat DashboardController

```bash
php artisan make:controller DashboardController
```

### 15.2 Isi Logika di DashboardController

Buka `app/Http/Controllers/DashboardController.php` dan isi dengan kode berikut:

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
            return view('dashboard-karyawan');
        }

        // Fallback jika role tidak terdefinisi
        return redirect('/');
    }
}
```

---

## 🛣️ Langkah 16: Menyesuaikan Rute Dashboard

Sekarang, kita ubah rute `/dashboard` di `routes/web.php` biar pake `DashboardController` yang baru kita bikin.

**📝 Ubah dari ini:**

```php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');
```

**🎯 Menjadi seperti ini:**

```php
use App\Http\Controllers\DashboardController; // Jangan lupa tambahkan ini di atas

Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware(['auth', 'verified'])->name('dashboard');
```

---

## 📄 Langkah 17: Membuat View untuk Masing-Masing Dashboard

Kita perlu bikin dua file view baru yang bakal dipanggil sama `DashboardController`.

### 17.1 Buat View Admin

- Ubah nama file lama `resources/views/dashboard.blade.php` jadi `dashboard-admin.blade.php`
- Kita biarkan dulu isinya kayak bawaan Breeze
- Nanti halaman ini bisa diisi dengan ringkasan data absensi, jumlah karyawan aktif, dll.

### 17.2 Buat View Karyawan

Buat file baru di `resources/views/` bernama `dashboard-karyawan.blade.php` dengan isi berikut:

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
                            <div class="bg-red-100 text-red-800 font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                Waktu untuk melakukan absensi hari ini telah berakhir.
                            </div>
                            @else
                            <div class="space-x-4">
                                <button onclick="openPermissionModal('Izin')"
                                    class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                    <i class="fas fa-file-alt mr-2"></i> Ajukan Izin
                                </button>
                                <form method="POST" action="{{ route('attendance.clockin') }}" class="inline-block">
                                    @csrf
                                    <button type="submit"
                                        class="bg-green-500 hover:bg-green-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                        <i class="fas fa-fingerprint mr-2"></i> Absen Masuk
                                    </button>
                                </form>
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

        // JavaScript untuk Modal
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

---

### 17.3 Tambahkan Script Stack ke Layout

Biar Javascript buat jam digital bisa jalan, kita perlu nambahin "slot" buat script di layout utama.

Buka `resources/views/layouts/app.blade.php` dan tambahin `@stack('scripts')` tepat sebelum tag penutup `</body>`:

```html
        </main>
    </div>

    @stack('scripts')
</body>
</html>
```

---

## ✅ Testing Langkah 17

Saatnya test hasil kerja kita! 

1. **Login sebagai admin** (`admin@gmail.com`)
   - Lo harusnya liat halaman dashboard default Breeze yang lama
   - Navigasi untuk admin bakal muncul

2. **Logout, terus login sebagai karyawan** (`karyawan@gmail.com`)
   - Lo bakal disambut sama halaman dashboard absensi yang baru
   - Lengkap dengan jam digital yang berjalan
   - Navigasi untuk karyawan bakal muncul

---

# ⚙️ Tahap 8: Membuat Logika Absensi (Controller & Rute)

> Nah sekarang saatnya ngehidupin tombol-tombol absensi itu dengan bikin "mesin" di belakangnya. Ini melibatkan pembuatan Controller baru buat nanganin semua logika absensi dan Rute sebagai alamatnya.

---

## 🎮 Langkah 18: Membuat AttendanceController

Controller ini bakal jadi **pusat komando** buat semua aksi terkait absensi, kayak absen masuk dan absen pulang.

Jalanin perintah ini di terminal:

```bash
php artisan make:controller AttendanceController
```

Easy peasy!

---

## 🛣️ Langkah 19: Membuat Rute untuk Aksi Absensi

Kita butuh beberapa alamat baru nih:
- Satu buat ngirim data **"Absen Masuk"**
- Satu lagi buat **"Absen Pulang"**
- Satu lagi buat **"Izin/Sakit"**

### Setup Rute

Buka file `routes/web.php` dan tambahin rute berikut di dalam grup `middleware('auth')`:

```php
// routes/web.php

use App\Http\Controllers\AttendanceController; // Jangan lupa tambahkan ini

// ... Rute lainnya ...

Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');

    // RUTE UNTUK ABSENSI
    Route::get('/attendance', [AttendanceController::class, 'index'])->name('attendance.index');
    Route::post('/attendance/clock-in', [AttendanceController::class, 'clockIn'])->name('attendance.clockin');
    Route::post('/attendance/clock-out', [AttendanceController::class, 'clockOut'])->name('attendance.clockout');
    
    // RUTE BARU UNTUK IZIN/SAKIT
    Route::post('/attendance/permission', [AttendanceController::class, 'storePermission'])->name('attendance.store_permission');
});
```

---

## 🟢 Langkah 20: Implementasi Logika Absen Masuk (clockIn)

Sekarang kita isi `AttendanceController` dengan logika buat `clockIn`. Ini yang bakal ngecek apakah lo terlambat atau nggak!

Buka `app/Http/Controllers/AttendanceController.php` dan isi dengan kode berikut:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Models\Attendance;
use Carbon\Carbon;

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

**💡 Penjelasan Logika:**
- Ngecek apakah user punya data employee
- Ngecek apakah udah absen hari ini (biar ga double absen)
- Ngecek apakah hari kerja (Senin-Jumat)
- Ngecek jam masuk (kalau lewat 08:01 = Terlambat)
- Simpan data absensi ke database

---

## 🔴 Langkah 21: Implementasi Logika Absen Pulang (clockOut)

Nah ini yang paling seru! Kita bakal bikin logika buat ngecek apakah karyawan pulang cepat atau nggak. Sistem bakal otomatis nge-tag status "Pulang Cepat" kalau pulang sebelum jam 14:00.

Tambahin method `clockOut` di dalam `AttendanceController.php`:

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

    // 5. Update data absen pulang dengan status baru
    $attendance->update([
        'time_out' => $time,
        'status' => $newStatus, // Gunakan status yang sudah diperbarui
    ]);

    return redirect()->route('dashboard')->with('success', 'Berhasil melakukan absen pulang. Selamat beristirahat!');
}
```

**💡 Penjelasan Logika:**
- Ngecek apakah udah absen masuk dulu
- Ngecek apakah udah absen pulang (biar ga double)
- Bandingkan jam pulang dengan jam resmi (14:00)
- Kalau pulang sebelum jam 14:00, tambahin status "Pulang Cepat"
- Status bakal jadi kayak "Hadir, Pulang Cepat" atau "Terlambat, Pulang Cepat"

---

## 🟡 Langkah 22: Implementasi Logika Izin/Sakit (storePermission)

Ini buat karyawan yang mau ajuin izin atau sakit. Mereka bisa langsung submit lewat dashboard tanpa perlu dateng ke kantor.

Tambahin method `storePermission` di dalam `AttendanceController.php`:

```php
// app/Http/Controllers/AttendanceController.php

// ... (method clockIn dan clockOut) ...

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
```

**💡 Penjelasan Logika:**
- Validasi input (harus Izin atau Sakit, notes wajib diisi)
- Ngecek apakah udah ada catatan absensi hari ini
- Simpan data izin/sakit tanpa jam masuk dan pulang
- Notes disimpen buat keterangan

---

## 🔗 Langkah 23: Menghubungkan Data dan Logika ke Dashboard

Nah, terakhir kita sempurnain `DashboardController` dan view `dashboard-karyawan` biar bisa nampilin data absensi hari ini dan tombol yang dinamis.

### 23.1 Update DashboardController

Buka `app/Http/Controllers/DashboardController.php` dan modifikasi method `index`:

```php
// app/Http/Controllers/DashboardController.php
use App\Models\Attendance; // Tambahkan ini
use Carbon\Carbon;          // Tambahkan ini

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

    // Jika karyawan tapi data employee belum ada, atau role lain
    return view('dashboard-karyawan');
}
```

**Penjelasan:**
- Ambil data absensi hari ini berdasarkan employee_id
- Kirim data ke view pake `compact('todaysAttendance')`
- View bakal otomatis bisa akses variabel `$todaysAttendance`

---

## Testing Tahap 8

**Ini momen pembuktian!** Waktunya test semua fitur yang udah kita bikin!

### Test Scenario 1: Absen Masuk Normal
1. Login sebagai **karyawan** (`karyawan@gmail.com`)
2. Buka halaman **Dashboard**
3. Klik tombol **"Absen Masuk"**
4. Cek apakah:
   - Halaman refresh dan muncul pesan sukses
   - Ringkasan terisi (Jam Masuk, Status)
   - Tombol berubah jadi **"Absen Pulang"**

### Test Scenario 2: Absen Masuk Terlambat
1. Ubah waktu sistem jadi lewat jam 08:01
2. Login sebagai karyawan
3. Klik **"Absen Masuk"**
4. Cek apakah status jadi **"Terlambat"**

### Test Scenario 3: Absen Pulang Normal
1. Setelah absen masuk, tunggu beberapa saat
2. Klik tombol **"Absen Pulang"**
3. Cek apakah:
   - Pesan sukses muncul
   - Jam Pulang terisi
   - Status tetep sesuai jam masuk (Hadir/Terlambat)

### Test Scenario 4: Pulang Cepat
1. Absen masuk dulu
2. Langsung klik **"Absen Pulang"** sebelum jam 14:00
3. Cek apakah status berubah jadi: 
   - "Hadir, Pulang Cepat" atau
   - "Terlambat, Pulang Cepat"

### Test Scenario 5: Ajukan Izin/Sakit
1. Login di hari baru (belum absen)
2. Klik tombol **"Ajukan Izin"** atau **"Ajukan Sakit"**
3. Isi form catatan
4. Klik **"Kirim Pengajuan"**
5. Cek apakah:
   - Data tersimpan dengan status Izin/Sakit
   - Tidak ada jam masuk dan pulang
   - Catatan tersimpan

### Test Scenario 6: Validasi Error
1. Coba absen masuk 2x di hari yang sama → harus error
2. Coba absen pulang tanpa absen masuk → harus error
3. Coba ajuin izin setelah udah absen → harus error
4. Coba absen di hari libur (Sabtu/Minggu) → harus error

---

# Tahap 9: Membuat Halaman Riwayat Absensi

> Setelah karyawan bisa melakukan absensi, tentu mereka perlu liat rekap atau histori dari absensi yang udah mereka lakuin. Sekarang kita bakal bikin halaman **"Riwayat Absensi"**.

Halaman ini bakal berisi tabel yang nampilin **semua catatan absensi** milik karyawan yang lagi login, diurutkan dari yang paling baru.

---

## Langkah 24: Logika Menampilkan Riwayat di Controller

Kita bakal nambahin method baru bernama `index` ke dalam `AttendanceController` buat ngambil data riwayat absensi.

### Setup Method Index

Buka `app/Http/Controllers/AttendanceController.php` dan tambahin method `index`:

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

    // ... (method clockIn, clockOut, storePermission) ...
}
```

**Penjelasan:**
- Ngecek apakah user adalah karyawan
- Ambil semua data absensi milik karyawan tersebut
- Urutkan dari yang terbaru (`latest()`)
- Paginasi 15 data per halaman
- Kirim ke view `attendance.index`

---

## Langkah 25: Membuat View Riwayat Absensi

Saatnya bikin tampilan buat halaman riwayat yang keren dan user-friendly!

### Setup View

1. Buat folder baru bernama `attendance` di dalam `resources/views`
2. Di dalam folder `attendance` tersebut, buat file baru bernama `index.blade.php`
3. Isi file `index.blade.php` dengan kode berikut:

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

**Fitur di View Ini:**
- Tabel yang responsive dan rapi
- Badge berwarna buat setiap status (Hadir = hijau, Terlambat = kuning, dll)
- Status gabungan ditampilin terpisah (contoh: badge "Hadir" + badge "Pulang Cepat")
- Pagination otomatis buat data yang banyak
- Empty state kalau belum ada data

---

## Langkah 26: Mengaktifkan Link di Navigasi

Langkah terakhir adalah **"menyalakan"** link "Riwayat Absensi" di menu navigasi!

### Update Navigation

Buka file `resources/views/layouts/navigation.blade.php`:

#### Untuk Desktop Navigation

**Cari dan ubah dari ini:**

```html
<x-nav-link :href="'#'">
    {{ __('Riwayat Absensi') }}
</x-nav-link>
```

**Menjadi seperti ini:**

```html
<x-nav-link :href="route('attendance.index')" :active="request()->routeIs('attendance.*')">
    {{ __('Riwayat Absensi') }}
</x-nav-link>
```

#### Untuk Mobile/Responsive Navigation

**Cari dan ubah dari ini:**

```html
<x-responsive-nav-link :href="'#'">
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

## Testing Tahap 9

**Waktunya Testing Riwayat!** 

1. Login sebagai **karyawan**
2. Klik menu **"Riwayat Absensi"** di navigasi
3. Cek apakah:
   - Data absensi tampil lengkap
   - Status badge punya warna yang sesuai
   - Pagination berfungsi (kalau data lebih dari 15)
   - Format tanggal dan waktu tampil dengan benar
   - Kalau ada status "Pulang Cepat", muncul badge terpisah

---

# Tahap 10: Fitur Admin - Tandai Karyawan Alpa

> Nah ini fitur penting buat admin! Admin bisa otomatis tandain karyawan yang ga absen sama sekali di hari itu sebagai "Alpa". Praktis banget kan?

Kita bakal bikin tombol khusus di dashboard admin yang cuma muncul setelah jam kerja berakhir (14:00 WITA).

---

## Langkah 27: Buat Method markAlpa di Controller

Kita bakal nambahin method baru di `AttendanceController` buat ngecek dan nandain karyawan yang alpa.

### Setup Method

Buka `app/Http/Controllers/AttendanceController.php` dan tambahin method baru:

```php
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

**Penjelasan Logika:**
- Hanya admin yang bisa akses method ini
- Ga jalan kalau hari libur (weekend)
- Ambil semua karyawan aktif
- Bandingkan dengan yang udah absen hari ini
- Yang belum absen = otomatis jadi Alpa
- Bikin catatan absensi dengan status Alpa

---

## Langkah 28: Tambahkan Rute untuk Admin

Sekarang kita bikin rute khusus buat admin buat trigger method `markAlpa`.

Buka `routes/web.php` dan tambahin rute ini:

```php
// routes/web.php

// ... rute lainnya

Route::middleware(['auth', 'role:admin'])->group(function () {
    // ... rute yang sudah ada ...
    
    // RUTE KHUSUS ADMIN: Menandai karyawan yang alpa
    Route::post('/admin/attendance/mark-alpa', [AttendanceController::class, 'markAlpa'])
            ->name('admin.attendance.mark_alpa');
});
```

**Catatan:** Idealnya rute ini dilindungi middleware khusus admin. Tapi di sini kita udah ngecek role di dalam controller jadi tetep aman kok.

---

## Langkah 29: Update View Dashboard Admin

Terakhir, kita update dashboard admin biar ada tombol buat nandain karyawan yang alpa. Tombolnya cuma muncul setelah jam 14:00 ya biar ga asal klik.

Buka `resources/views/dashboard-admin.blade.php` dan ganti isinya dengan:

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

**Fitur di View Ini:**
- Ngecek waktu sekarang vs jam 14:00
- Kalau belum jam 14:00 = tampil info box biru
- Kalau udah lewat jam 14:00 = tampil tombol merah
- Ada konfirmasi popup sebelum eksekusi
- Ada penjelasan singkat buat admin

---

## Testing Tahap 10

**Waktunya Test Fitur Admin!**

### Test Scenario 1: Sebelum Jam 14:00
1. Login sebagai **admin** (`admin@gmail.com`)
2. Buka **Dashboard**
3. Cek apakah muncul info box biru yang bilang tombol belum bisa diklik

### Test Scenario 2: Setelah Jam 14:00
1. Ubah waktu sistem jadi lewat jam 14:00 (atau tunggu sampe jam segitu)
2. Login sebagai admin
3. Buka Dashboard
4. Cek apakah tombol merah **"Tandai Karyawan yg Alpa Hari Ini"** muncul

### Test Scenario 3: Eksekusi Tandai Alpa
1. Pastikan ada beberapa karyawan yang belum absen hari ini
2. Klik tombol **"Tandai Karyawan yg Alpa Hari Ini"**
3. Konfirmasi popup yang muncul
4. Cek apakah:
   - Muncul pesan sukses dengan jumlah karyawan yang ditandai
   - Data absensi dengan status "Alpa" tersimpan di database
   - Karyawan yang udah absen tidak terpengaruh

### Test Scenario 4: Klik 2x di Hari yang Sama
1. Klik tombol lagi setelah udah nandain alpa
2. Cek apakah muncul pesan: "Semua karyawan aktif sudah memiliki catatan absensi hari ini"

### Test Scenario 5: Di Hari Libur
1. Ubah tanggal sistem ke hari Sabtu/Minggu
2. Coba klik tombol
3. Cek apakah muncul pesan: "Tidak ada tindakan yang diambil pada hari libur"

---

# Selamat! Part 2 Selesai!

Mantap! Lo udah berhasil nyelesain **Part 2 - Panduan Absensi Karyawan** yang mencakup:

## Recap Apa Aja yang Udah Kita Bikin:

### Tahap 7: Dashboard Absensi Karyawan
- Dashboard dinamis berdasarkan role
- Jam digital real-time (WITA)
- Panel absensi yang interaktif
- Modal untuk ajukan izin/sakit

### Tahap 8: Logika Absensi
- Sistem clock in/out yang lengkap
- Deteksi otomatis status terlambat (lewat jam 08:01)
- Deteksi otomatis pulang cepat (sebelum jam 14:00)
- Validasi hari kerja (Senin-Jumat)
- Fitur ajukan izin dan sakit dengan catatan
- Status gabungan (contoh: "Terlambat, Pulang Cepat")

### Tahap 9: Riwayat Absensi
- Tabel riwayat yang responsive
- Badge berwarna untuk setiap status
- Pagination otomatis
- Filter berdasarkan karyawan yang login

### Tahap 10: Fitur Admin
- Tombol otomatis tandai karyawan alpa
- Proteksi waktu (cuma muncul setelah jam 14:00)
- Validasi hari libur
- Konfirmasi sebelum eksekusi

---

## Lanjut ke Part 3?

**Part 3** bakal ngebahas:
- Laporan Absensi Harian untuk Admin
- CRUD Management Hari Libur
- Dashboard Admin dengan statistik lengkap
- Export data ke Excel/PDF
- Dan masih banyak lagi!

Untuk melanjutkan, kunjungi panduan berikutnya:  
👉 [Part 3 - Panduan Absensi Project](https://github.com/ahmad-syaifuddin/part3-panduan-absensi-project.git)

---

## Tips Sebelum Lanjut ke Part 3:

1. **Test semua fitur** yang udah kita bikin
2. **Coba berbagai skenario** (happy path & error handling)
3. **Pastikan database** udah bersih dan terstruktur
4. **Backup project** lo sebelum lanjut ke part berikutnya
5. **Commit ke Git** biar aman kalau ada yang error

---

## Troubleshooting

Kalau lo nemuin masalah:

### Problem 1: Error "status column too short"
**Solusi:** Pastikan lo udah jalanin migration yang ngubah kolom status jadi VARCHAR(50)

### Problem 2: Jam digital ga jalan
**Solusi:** Cek apakah `@stack('scripts')` udah ada di `layouts/app.blade.php`

### Problem 3: Tombol admin ga muncul
**Solusi:** Cek timezone sistem dan pastikan role user adalah 'admin'

### Problem 4: Pagination ga jalan
**Solusi:** Pastikan lo pake `paginate()` bukan `get()` di controller

---

**Happy Coding!** Semoga lancar ya project absensinya!
