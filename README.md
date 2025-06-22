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
static void load_trap_state() {
    FILE *f = fopen(state_file, "r");
    if (f) {
        if (fscanf(f, "%d", &trap_triggered) != 1) {
            trap_triggered = 0;
        }
        fclose(f);
    } else {
        trap_triggered = 0;
    }
    fprintf(stderr, "TrollFS: Status jebakan awal (dimuat): %d\n", trap_triggered);
}

static void save_trap_state() {
    FILE *f = fopen(state_file, "w");
    if (f) {
        fprintf(f, "%d", trap_triggered);
        fclose(f);
    } else {
        fprintf(stderr, "TrollFS: PERINGATAN: Tidak bisa menyimpan status jebakan ke %s: %s\n", state_file, strerror(errno));
    }
    fprintf(stderr, "TrollFS: Status jebakan disimpan: %d\n", trap_triggered);
}

static uid_t get_uid_from_username(const char *username) {
    struct passwd *pw = getpwnam(username);
    if (pw) {
        return pw->pw_uid;
    }
    return (uid_t)-1;
}
```
Penjelasan Fungsi Pembantu:

load_trap_state(): Membaca status trap_triggered dari file .troll_state saat FUSE dimulai, memungkinkan jebakan untuk tetap aktif antar sesi. Jika file tidak ada atau gagal dibaca, status diatur ke 0.
save_trap_state(): Menulis status trap_triggered saat ini ke file .troll_state ketika FUSE dihentikan, memastikan persistensi.
get_uid_from_username(const char *username): Mencari UID dari nama pengguna yang diberikan, digunakan untuk mengidentifikasi pengguna 'daintontas'.

```
static int troll_getattr(const char *path, struct stat *stbuf) {
    int res;
    char fpath[PATH_MAX];

    snprintf(fpath, PATH_MAX, "%s%s", source_path, path);

    res = lstat(fpath, stbuf);
    if (res == -1) {
        return -errno;
    }

    uid_t current_uid = fuse_get_context()->uid;

    if (strcmp(path + 1, secret_file_name) == 0) {
        if (trap_triggered && current_uid == daintontas_uid) {
            stbuf->st_size = strlen(ascii_art_troll);
            fprintf(stderr, "TrollFS: getattr untuk %s (DainTontas, jebakan aktif): ukuran ASCII art %zu.\n", path, stbuf->st_size);
        } else if (current_uid == daintontas_uid) {
            stbuf->st_size = strlen("Very spicy internal developer information: leaked roadmap.docx");
            fprintf(stderr, "TrollFS: getattr untuk %s (DainTontas, jebakan tidak aktif): ukuran konten DainTontas %zu.\n", path, stbuf->st_size);
        } else {
            stbuf->st_size = strlen("DainTontas' personal secret!!.txt");
            fprintf(stderr, "TrollFS: getattr untuk %s (User lain): ukuran konten spesifik %zu.\n", path, stbuf->st_size);
        }
    }
    else if (strstr(path, ".txt") != NULL && trap_triggered) {
        stbuf->st_size = strlen(ascii_art_troll);
        fprintf(stderr, "TrollFS: getattr untuk %s (file .txt lain, jebakan aktif): ukuran ASCII art %zu.\n", path, stbuf->st_size);
    }
    else {
        fprintf(stderr, "TrollFS: getattr untuk %s (default): ukuran asli %ld.\n", path, stbuf->st_size);
    }

    return 0;
}
```
Penjelasan troll_getattr:
- Secara default, ia mendapatkan atribut file dari source_path.
- Jika file adalah very_spicy_info.txt, ukurannya disesuaikan:
- Jika trap_triggered aktif dan pengguna adalah daintontas, ukuran disetel ke panjang ASCII art.
- Jika pengguna adalah daintontas tetapi trap_triggered tidak aktif, ukuran disetel ke panjang pesan khusus daintontas.
- Jika pengguna lain, ukuran disetel ke panjang pesan umpan.
- Jika file adalah .txt lain dan trap_triggered aktif, ukurannya disetel ke panjang ASCII art.
- Ini memastikan cat atau perintah lain mengalokasikan buffer yang benar untuk konten yang dimodifikasi.

  ```
  static int troll_readdir(const char *path, void *buf, fuse_fill_dir_t filler,
                         off_t offset, struct fuse_file_info *fi) {
    DIR *dp;
    struct dirent *de;
    (void) offset;
    (void) fi;

    char fpath[PATH_MAX];
    snprintf(fpath, PATH_MAX, "%s%s", source_path, path);
    fprintf(stderr, "TrollFS: readdir dipanggil untuk path: %s (dipetakan ke %s)\n", path, fpath);

    dp = opendir(fpath);
    if (dp == NULL) {
        fprintf(stderr, "TrollFS: Error membuka direktori sumber %s: %s\n", fpath, strerror(errno));
        return -errno;
    }

    while ((de = readdir(dp)) != NULL) {
        fprintf(stderr, "TrollFS: Menemukan entri: %s (tipe: %d)\n", de->d_name, de->d_type);

        if (strcmp(de->d_name, ".troll_state") == 0) {
            fprintf(stderr, "TrollFS: Melewatkan .troll_state\n");
            continue;
        }

        struct stat st;
        memset(&st, 0, sizeof(st));
        st.st_ino = de->d_ino;
        st.st_mode = DTTOIF(de->d_type);

        if (de->d_type == DT_DIR) {
            st.st_mode |= 0755;
        } else {
            st.st_mode |= 0644;
        }

        if (strcmp(path, "/") == 0) {
            if (!(strcmp(de->d_name, secret_file_name) == 0 ||
                  strcmp(de->d_name, upload_file_name) == 0 ||
                  strcmp(de->d_name, ".") == 0 ||
                  strcmp(de->d_name, "..") == 0)) {
                fprintf(stderr, "TrollFS: Melewatkan entri yang disaring: %s\n", de->d_name);
                continue;
            }
        }

        fprintf(stderr, "TrollFS: Menambahkan entri ke FUSE: %s\n", de->d_name);
        if (filler(buf, de->d_name, &st, 0)) {
            fprintf(stderr, "TrollFS: Filler mengembalikan non-nol (buffer penuh?)\n");
            break;
        }
    }

    closedir(dp);
    fprintf(stderr, "TrollFS: readdir selesai untuk path: %s\n", path);
    return 0; }

Penjelasan troll_readdir:
- Membuka direktori asli di source_path.
- Menyembunyikan file .troll_state dari daftar.
- Jika direktori yang dibaca adalah root FUSE (/), hanya secret_file_name, upload_file_name, . dan .. yang akan ditampilkan kepada pengguna. File lain yang ada di direktori sumber akan disembunyikan.

## Output Code

User Dain tontas: Untuk User dain tontas ini merupakan target dari jebakan kita. jadi dia akan kita fokuskan untuk mencoba soal ini.


