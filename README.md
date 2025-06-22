# Drama Troll

## Penjelasan soal
### Latar Belakang Soal
Soal ini tampaknya adalah tugas atau tantangan di bidang keamanan siber, forensik digital, atau sistem operasi, di mana tujuannya adalah membuat sebuah "troll" atau jebakan pada sistem file Linux menggunakan FUSE (Filesystem in Userspace).
### Tujuan 
Membuat sebuah sistem file virtual yang memiliki perilaku tidak biasa dan menyembunyikan informasi tertentu, tetapi mengungkapkannya dalam kondisi yang spesifik, terutama saat pengguna "terjebak" atau melakukan tindakan tertentu.

### Komponen penting 
1. Direktori Asli (Source Path): /home/troll_source
 - Ini adalah direktori di sistem file nyata yang akan "di-overlay" atau "dilapisi" oleh sistem file FUSE. FUSE akan menggunakan file-file di sini sebagai dasar, tetapi akan memodifikasi perilakunya.

2. Mount Point FUSE: /mnt/troll
- Ini adalah tempat di mana sistem file FUSE akan terlihat oleh pengguna. Semua interaksi pengguna akan terjadi melalui direktori ini.

3. File Spesial 1: very_spicy_info.txt
- Ini adalah file kunci yang kontennya berubah tergantung pada siapa yang membacanya dan apakah "jebakan" telah terpicu.
4. File Spesial 2: upload.txt
- File ini berfungsi sebagai pemicu (trigger) untuk "jebakan".
- Perilaku: Ketika ada string yang mengandung kata "upload" (tidak peduli huruf besar/kecil) ditulis ke file ini, maka "jebakan" akan terpicu (trap_triggered menjadi 1).
5. Jebakan Ascii
- Ketika ada yang membuat file terpicu makan filenya akan mengeluarkan ASCII.

### Skenario soal 
daintontas yang ceroboh dan dua rekannya, sunnybolt dan ryeku, yang mencoba mencari informasi rahasia.
Awalnya, daintontas membaca very_spicy_info.txt dan melihat yang dia harapkan. sunnybolt dan ryeku mencoba membaca file yang sama dan melihat pesan umpan.
Suatu saat, daintontas atau rekannya secara tidak sengaja menemukan upload.txt dan mencoba menulis sesuatu ke sana, dan di dalamnya ada kata "upload". ketika daintontas membaca very_spicy_info.txt lagi, dia akan mendapatkan kejutan: ASCII Art troll! Dan bahkan jika sunnybolt atau ryeku membaca file .txt lain yang tadinya normal, mereka juga akan mendapatkan ASCII Art (kecuali very_spicy_info.txt yang tetap menjadi pesan umpan bagi mereka).

## Penjelasan Code

```
static const char *source_path = "/home/troll_source";
static const char *secret_file_name = "very_spicy_info.txt";
static const char *upload_file_name = "upload.txt";
static const char *state_file = "/home/troll_source/.troll_state";

static int trap_triggered = 0;

static uid_t daintontas_uid = (uid_t)-1;
static const char *ascii_art_troll =
"                                                               .-'''-.                                                                                         \n"
"                             .---..---.                       '   _    \\                                                                                       \n"
"               __.....__     |   ||   |                     /   /` '.   \\                   .--.                                               .--.   _..._    \n"
"     _.._  .-''         '.   |   ||   |           __| |__   `.   ` ..' /  | |  | |          .--.     .|                      /.''\\\\            .--..   .-.   . \n"
"   .' .._|/     .-''\"'-.  `. |   ||   |           |__   __|     '-...-'`   | |  | |          |  |   .' |_               __   | |  | |      __   | |  ||  '   '  | \n"
" __| |__ |                  ||   ||   |             | |                   | |  | |          |  | .'     |           .:--.'.  \\`-' /    .:--.'. |  ||  |   |  | \n"
"|__   __|\    .-------------'|   ||   |             | |                   | |  '-           |  |'--.  .-'          / |   \\ | /(\"'`    / |   \\ ||  ||  |   |  | \n"
"   | |    \\    '-.____...---.|   ||   |             | |                   | |               |  |   |  |            `\" __ | | \\ '---.  `\" __ | ||  ||  |   |  | \n"
"   | |     `.             .' |   ||   |             | |                   | |                      |  |             .'.''| |  /'\"\"'.\\  .'.''| ||__||  |   |  | \n"
"   | |       `''-...... -'   '---''---'             | |                   | |                      |  '.'          / /   | |_||     ||/ /   | |_   |  |   |  | \n"
"   | |                                              | |                   |_|                      |   /           \\ \\._,\\ '/\\'. __// \\ \\._,\\ '/   |  |   |  | \n"
"   |_|                                              |_|                                            `'-'             `--'  `\"  `'---'   `--'  `\"    '--'   '--' \n";
```
- static const char *source_path: Mendefinisikan direktori di sistem file host yang akan disimulasikan oleh FUSE.
- static const char *secret_file_name: Nama file yang akan memiliki perilaku khusus dan berbeda.
- static const char *upload_file_name: Nama file yang berfungsi sebagai pemicu untuk mengaktifkan jebakan.
- static const char *state_file: Nama file internal yang digunakan untuk menyimpan status jebakan secara persisten.
- static int trap_triggered = 0;: Variabel global yang menunjukkan apakah jebakan telah terpicu (1) atau belum (0).
- static uid_t daintontas_uid = (uid_t)-1;: Menyimpan User ID (UID) dari pengguna 'daintontas' untuk mengimplementasikan perilaku khusus berdasarkan pengguna.
- static const char *ascii_art_troll: Sebuah string yang berisi seni ASCII yang akan ditampilkan sebagai bagian dari jebakan.

  ```
