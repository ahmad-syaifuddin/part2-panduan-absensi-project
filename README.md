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
