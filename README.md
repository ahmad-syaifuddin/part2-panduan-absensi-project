# Part 2 panduan absensi

# Langkah ini melibatkan pembuatan Model dan Migrasi untuk menyimpan data absensi karyawan dan daftar hari libur.

## Sekarang kita akan fokus membangun fitur utama untuk karyawan: melakukan absensi. Kita akan mulai dengan merombak halaman Dashboard agar menjadi halaman utama bagi karyawan untuk melakukan absen masuk dan pulang.

# Tahap 7: Membangun Dashboard Absensi Karyawan
Dashboard akan kita buat dinamis. Tampilannya akan berbeda tergantung role pengguna. Admin akan melihat dashboard ringkasan umum, sedangkan karyawan akan melihat panel absensi.

## Langkah 15: Membuat Controller untuk Dashboard
Daripada menggunakan fungsi sederhana di file rute, kita akan membuat controller khusus untuk mengatur logika dashboard.

Buat DashboardController:

```Bash
php artisan make:controller DashboardController
```
Isi Logika di DashboardController:
Buka app/Http/Controllers/DashboardController.php dan isi dengan kode berikut. Logika ini akan memeriksa role pengguna dan menampilkan view yang sesuai.

```PHP
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

## Langkah 16: Menyesuaikan Rute Dashboard
Sekarang, kita ubah rute /dashboard di routes/web.php agar menggunakan DashboardController yang baru kita buat.

Buka routes/web.php.

Cari dan ubah rute /dashboard.

Ubah baris ini:
```PHP
// use App\Http\Controllers\DashboardController; // <-- Jangan lupa tambahkan ini di atas

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');
```
Menjadi seperti ini:

```PHP
use App\Http\Controllers\DashboardController; // <-- Tambahkan ini di atas

Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware(['auth', 'verified'])->name('dashboard');
```    
## Langkah 17: Membuat View untuk Masing-Masing Dashboard
Kita perlu membuat dua file view baru yang dipanggil oleh DashboardController.

Buat View Admin (dashboard-admin.blade.php):

Ubah nama file lama resources/views/dashboard.blade.php menjadi dashboard-admin.blade.php.

Kita bisa biarkan isinya seperti bawaan Breeze untuk saat ini. Nantinya, halaman ini bisa diisi dengan ringkasan data absensi, jumlah karyawan aktif, dll.

Buat View Karyawan (dashboard-karyawan.blade.php):

Buat file baru di resources/views/ bernama dashboard-karyawan.blade.php.

Isi dengan kode berikut. Ini adalah tampilan inti untuk absensi.

```HTML
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

                    {{-- Akan kita tambahkan nanti setelah membuat logika absen --}}

                    <div class="text-center">
                        <h3 class="text-lg font-medium text-gray-900">
                            {{ \Carbon\Carbon::now()->isoFormat('dddd, D MMMM Y') }}
                        </h3>

                        <div id="jam-digital" class="text-5xl font-bold text-gray-800 my-4">
                            00:00:00
                        </div>

                        <div class="mt-6 space-x-4">
                            {{-- Logika tombol akan ditambahkan pada langkah selanjutnya --}}

                            <form method="POST" action="#" class="inline-block">
                                @csrf
                                <button type="submit" class="bg-green-500 hover:bg-green-700 text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg transition duration-300 ease-in-out transform hover:scale-105">
                                    <i class="fas fa-fingerprint mr-2"></i> Absen Masuk
                                </button>
                            </form>

                            <button disabled class="bg-gray-400 cursor-not-allowed text-white font-bold py-4 px-8 rounded-lg text-xl shadow-lg">
                                <i class="fas fa-sign-out-alt mr-2"></i> Absen Pulang
                            </button>
                        </div>

                    </div>

                    <div class="mt-10 border-t pt-6">
                        <h4 class="text-md font-semibold mb-4">Ringkasan Hari Ini:</h4>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                            <div class="bg-blue-100 p-4 rounded-lg">
                                <p class="text-sm text-blue-700">Jam Masuk</p>
                                <p class="text-lg font-bold text-blue-900">--:--</p>
                            </div>
                            <div class="bg-purple-100 p-4 rounded-lg">
                                <p class="text-sm text-purple-700">Jam Pulang</p>
                                <p class="text-lg font-bold text-purple-900">--:--</p>
                            </div>
                            <div class="bg-yellow-100 p-4 rounded-lg">
                                <p class="text-sm text-yellow-700">Status</p>
                                <p class="text-lg font-bold text-yellow-900">Belum Absen</p>
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
            const now = new Date();
            const hours = String(now.getHours()).padStart(2, '0');
            const minutes = String(now.getMinutes()).padStart(2, '0');
            const seconds = String(now.getSeconds()).padStart(2, '0');
            document.getElementById('jam-digital').textContent = `${hours}:${minutes}:${seconds}`;
        }
        setInterval(updateClock, 1000);
        updateClock(); // initial call
    </script>
    @endpush
</x-app-layout>
```
Tambahkan @stack('scripts') ke Layout Utama:
Agar Javascript untuk jam digital bisa berjalan, kita perlu menambahkan "slot" untuk script di layout utama. Buka resources/views/layouts/app.blade.php dan tambahkan @stack('scripts') tepat sebelum tag penutup </body>.

```HTML
</main>
    </div>

    @stack('scripts') </body>
</html>
```
Uji Coba
Login sebagai admin (admin@gmail.com). Anda seharusnya melihat halaman dashboard default Breeze yang lama (sekarang dari dashboard-admin.blade.php). Navigasi untuk admin akan muncul.

Logout, lalu login sebagai karyawan (karyawan@gmail.com). Anda akan disambut oleh halaman dashboard absensi yang baru, lengkap dengan jam digital yang berjalan. Navigasi untuk karyawan akan muncul.

---

# Sekarang saatnya menghidupkan tombol-tombol absensi itu dengan membuat "mesin" di belakangnya.
###Ini melibatkan pembuatan Controller baru untuk menangani semua logika absensi dan Rute sebagai alamatnya.

## Tahap 8: Membuat Logika Absensi (Controller & Rute)
Langkah 18: Membuat AttendanceController
Controller ini akan menjadi pusat komando untuk semua aksi terkait absensi, seperti absen masuk dan absen pulang.

Jalankan perintah ini di terminal:

```Bash
php artisan make:controller AttendanceController
```
### Langkah 19: Membuat Rute untuk Aksi Absensi
Kita perlu dua alamat baru: satu untuk mengirim data "Absen Masuk" dan satu lagi untuk "Absen Pulang".

Buka file routes/web.php.

Tambahkan rute berikut di dalam grup middleware('auth'). Anda bisa meletakkannya bersama rute profil.

```PHP
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
});
// ... Rute admin ...
```
### Langkah 20: Implementasi Logika Absen Masuk (clockIn)
Sekarang kita isi AttendanceController dengan logika untuk clockIn.

Buka app/Http/Controllers/AttendanceController.php dan isi dengan kode berikut:

```PHP
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
        $lateTime = Carbon::createFromTimeString('08:00:00', $timezone);
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
### Langkah 21: Implementasi Logika Absen Pulang (clockOut)
Tambahkan method clockOut di dalam AttendanceController.php.

```PHP
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
                              ->where('date', $today)
                              ->first();

    // 2. Jika tidak ditemukan data absen masuk
    if (!$attendance) {
        return redirect()->route('dashboard')->with('error', 'Anda belum melakukan absen masuk hari ini.');
    }

    // 3. Jika sudah ada data absen pulang
    if ($attendance->time_out) {
        return redirect()->route('dashboard')->with('error', 'Anda sudah melakukan absen pulang hari ini.');
    }

    // 4. Update data absen pulang
    $attendance->update([
        'time_out' => $time,
    ]);

    return redirect()->route('dashboard')->with('success', 'Berhasil melakukan absen pulang. Selamat beristirahat!');
}
```
### Langkah 22: Menghubungkan Data dan Logika ke Dashboard
Terakhir, kita sempurnakan DashboardController dan view dashboard-karyawan agar bisa menampilkan data absensi hari ini dan tombol yang dinamis.

Update DashboardController:
Buka app/Http/Controllers/DashboardController.php dan modifikasi method index untuk mengambil data absensi.

```PHP
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
Update View dashboard-karyawan.blade.php:
Ganti seluruh isi resources/views/dashboard-karyawan.blade.php dengan kode yang sudah dinamis berikut:

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
### Uji Coba
Ini adalah momen pembuktian!

Pastikan Anda login sebagai karyawan.

Buka halaman Dashboard. Anda akan melihat tombol "Absen Masuk".

Klik tombol "Absen Masuk". Halaman akan me-refresh, pesan sukses akan muncul, ringkasan di bawah akan terisi, dan tombol akan berubah menjadi "Absen Pulang".

Tunggu beberapa saat, lalu klik "Absen Pulang". Halaman akan me-refresh lagi, pesan sukses muncul, ringkasan jam pulang terisi, dan tombol akan hilang digantikan pesan bahwa absensi telah selesai.

Coba refresh halaman, statusnya akan tetap sama.

---

# Setelah karyawan bisa melakukan absensi, tentu mereka perlu melihat rekap atau histori dari absensi yang telah mereka lakukan. Sekarang kita akan membuat halaman "Riwayat Absensi".

## Tahap 9: Membuat Halaman Riwayat Absensi
Halaman ini akan berisi tabel yang menampilkan semua catatan absensi milik karyawan yang sedang login, diurutkan dari yang paling baru.

### Langkah 23: Logika Menampilkan Riwayat di Controller
Kita akan menambahkan method baru bernama index ke dalam AttendanceController untuk mengambil data riwayat absensi.

Buka app/Http/Controllers/AttendanceController.php.

Tambahkan method index berikut ini:

```PHP
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
### Langkah 24: Membuat Rute untuk Halaman Riwayat
Sekarang, kita buat "alamat" atau rute untuk mengakses method index yang baru saja kita buat.

Buka file routes/web.php.

Tambahkan rute GET berikut di dalam grup middleware('auth').

```PHP
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
### Langkah 25: Membuat View Riwayat Absensi
Saatnya membuat tampilan untuk halaman riwayat.

Buat folder baru bernama attendance di dalam resources/views.

Di dalam folder attendance tersebut, buat file baru bernama index.blade.php.

Isi file index.blade.php tersebut dengan kode berikut:

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
                                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">No</th>
                                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Tanggal</th>
                                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Jam Masuk</th>
                                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Jam Pulang</th>
                                    <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                                </tr>
                            </thead>
                            <tbody class="bg-white divide-y divide-gray-200">
                                @forelse ($attendances as $attendance)
                                    <tr>
                                        <td class="px-6 py-4 whitespace-nowrap">{{ $loop->iteration + $attendances->firstItem() - 1 }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">
                                            {{ \Carbon\Carbon::parse($attendance->date)->isoFormat('dddd, D MMMM Y') }}
                                        </td>
                                        <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->time_in }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->time_out ?? 'Belum Absen' }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">
                                            @if ($attendance->status == 'Hadir')
                                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800">
                                                    Hadir
                                                </span>
                                            @elseif ($attendance->status == 'Terlambat')
                                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-yellow-100 text-yellow-800">
                                                    Terlambat
                                                </span>
                                            @elseif ($attendance->status == 'Izin')
                                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-blue-100 text-blue-800">
                                                    Izin
                                                </span>
                                            @else
                                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-red-100 text-red-800">
                                                    Alpa
                                                </span>
                                            @endif
                                        </td>
                                    </tr>
                                @empty
                                    <tr>
                                        <td colspan="5" class="px-6 py-4 whitespace-nowrap text-center text-sm text-gray-500">
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
### Langkah 26: Mengaktifkan Link di Navigasi
Langkah terakhir adalah "menyalakan" link "Riwayat Absensi" di menu navigasi yang sebelumnya sudah kita siapkan.

Buka file resources/views/layouts/navigation.blade.php.

Cari bagian menu untuk karyawan dan hapus komentar serta ganti href-nya.

Ubah ini (di bagian desktop):

```HTML
<x-nav-link :href="'#'" {{-- :href="route('attendance.index')" :active="request()->routeIs('attendance.*')" --}}>
    {{ __('Riwayat Absensi') }}
</x-nav-link>
```
Menjadi:

```HTML
<x-nav-link :href="route('attendance.index')" :active="request()->routeIs('attendance.*')">
    {{ __('Riwayat Absensi') }}
</x-nav-link>
```
Dan ubah juga ini (di bagian mobile/responsif):

```HTML
<x-responsive-nav-link :href="'#'" {{-- :href="route('attendance.index')" :active="request()->routeIs('attendance.*')" --}}>
    {{ __('Riwayat Absensi') }}
</x-responsive-nav-link>
```
Menjadi:

```HTML
<x-responsive-nav-link :href="route('attendance.index')" :active="request()->routeIs('attendance.*')">
    {{ __('Riwayat Absensi') }}
</x-responsive-nav-link>
```
## Uji Coba
Login sebagai karyawan.

Di menu navigasi atas, link "Riwayat Absensi" sekarang sudah bisa diklik.

Klik link tersebut.

Anda akan diarahkan ke halaman /attendance yang menampilkan tabel berisi catatan absensi yang sudah Anda buat sebelumnya.
---
