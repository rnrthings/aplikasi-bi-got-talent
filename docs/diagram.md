# Diagram Sistem BiGotTalent

Dokumen ini berisi diagram UML dan ERD untuk aplikasi **BiGotTalent** (sistem pendaftaran lomba siswa), disusun berdasarkan kode di `routes/web.php`, `app/Http/Controllers`, `app/Models`, dan `database/migrations`.

Aktor sistem:

| Aktor | Keterangan |
|---|---|
| **Pengunjung** | Belum login. Dapat melihat halaman depan, registrasi, login, reset password. |
| **Siswa** | User dengan `role = 'siswa'` (default saat registrasi). |
| **Admin** | User dengan `role = 'admin'`. |

Status pendaftaran: `Dokumen dalam Tinjauan` (default) → `Jadwal Briefing` → `Tahap Pelatihan` → `Final`.

---

## 1. Use Case Diagram

```mermaid
flowchart LR
    Pengunjung(["👤 Pengunjung"])
    Siswa(["👤 Siswa"])
    Admin(["👤 Admin"])

    subgraph Sistem["Sistem BiGotTalent"]
        UC1(("Melihat Halaman Depan"))
        UC2(("Registrasi Akun"))
        UC3(("Login"))
        UC4(("Reset Password"))
        UC5(("Logout"))
        UC6(("Kelola Profil"))

        UC7(("Melihat Daftar Lomba"))
        UC8(("Mendaftar Lomba"))
        UC9(("Melihat Riwayat & Status Pendaftaran"))

        UC10(("Kelola Mata Lomba<br/>(Tambah/Edit/Hapus)"))
        UC11(("Kelola Akun Siswa<br/>(Tambah/Edit/Hapus)"))
        UC12(("Melihat Data Pendaftaran"))
        UC13(("Ubah Status Pendaftaran"))
    end

    Pengunjung --- UC1
    Pengunjung --- UC2
    Pengunjung --- UC3
    Pengunjung --- UC4

    Siswa --- UC5
    Siswa --- UC6
    Siswa --- UC7
    Siswa --- UC8
    Siswa --- UC9

    Admin --- UC5
    Admin --- UC6
    Admin --- UC10
    Admin --- UC11
    Admin --- UC12
    Admin --- UC13

    UC8 -. "«include»" .-> UC7
    UC13 -. "«include»" .-> UC12
    UC8 -. "«include»" .-> UC3
    UC10 -. "«include»" .-> UC3
```

> Mermaid tidak punya notasi use case resmi. Untuk laporan, diagram ini bisa digambar ulang di draw.io / StarUML dengan elemen yang sama.

---

## 2. Activity Diagram

### 2.1 Login & Pengarahan Berdasarkan Role

```mermaid
flowchart TD
    A([Mulai]) --> B[Pengguna membuka halaman Login]
    B --> C[Isi email & password]
    C --> D{Kredensial valid?}
    D -- Tidak --> E[Tampilkan pesan error] --> C
    D -- Ya --> F[Buat session login]
    F --> G[Redirect ke /dashboard]
    G --> H{role user?}
    H -- admin --> I[Tampilkan Dashboard Admin]
    H -- siswa --> J[Tampilkan Dashboard Siswa]
    I --> K([Selesai])
    J --> K
```

### 2.2 Siswa Mendaftar Lomba

```mermaid
flowchart TD
    subgraph Siswa
        A([Mulai]) --> B[Buka Dashboard Siswa]
        B --> D[Pilih lomba, isi kelas & no. WhatsApp]
        D --> E[Klik Daftar]
        L[Lihat pesan & riwayat pendaftaran] --> M([Selesai])
    end
    subgraph Sistem
        B --> C[Ambil daftar lomba & riwayat pendaftaran siswa]
        C --> D
        E --> F{Input valid?<br/>lomba_id, kelas ≤50, no_wa ≤20}
        F -- Tidak --> G[Tampilkan error validasi] --> D
        F -- Ya --> H{Sudah terdaftar<br/>di lomba ini?}
        H -- Ya --> I[Tampilkan pesan 'Anda sudah terdaftar'] --> L
        H -- Tidak --> J[Simpan pendaftaran<br/>status = 'Dokumen dalam Tinjauan']
        J --> K[Tampilkan pesan sukses] --> L
    end
```

### 2.3 Admin Mengubah Status Pendaftaran

```mermaid
flowchart TD
    A([Mulai]) --> B[Admin buka menu Kelola Status]
    B --> C[Sistem menampilkan semua pendaftaran<br/>beserta data siswa & lomba]
    C --> D[Admin pilih status baru dari dropdown]
    D --> E[Klik Simpan]
    E --> F{Pendaftaran ditemukan?}
    F -- Tidak --> G[Tampilkan 404] --> Z([Selesai])
    F -- Ya --> H[Update kolom status]
    H --> I[Kembali ke halaman dengan pesan sukses]
    I --> Z
```

### 2.4 Admin Mengelola Mata Lomba (CRUD)

```mermaid
flowchart TD
    A([Mulai]) --> B[Admin buka menu Kelola Lomba]
    B --> C[Sistem tampilkan daftar lomba]
    C --> D{Pilih aksi}
    D -- Tambah --> E[Isi nama & deskripsi lomba] --> F[Simpan lomba baru]
    D -- Edit --> G[Buka form edit] --> H[Ubah data] --> I[Simpan perubahan]
    D -- Hapus --> J[Hapus lomba<br/>pendaftaran terkait ikut terhapus / cascade]
    F --> K[Tampilkan pesan sukses]
    I --> K
    J --> K
    K --> C
```

---

## 3. Class Diagram

```mermaid
classDiagram
    direction LR

    class Authenticatable {
        <<Laravel>>
    }
    class Model {
        <<Laravel>>
    }
    class Controller {
        <<abstract>>
    }

    class User {
        +int id
        +string name
        +string email
        +datetime email_verified_at
        +string password
        +string role
        +string remember_token
        +datetime created_at
        +datetime updated_at
        +pendaftarans() HasMany
    }

    class Lomba {
        +int id
        +string nama_lomba
        +text deskripsi
        +datetime created_at
        +datetime updated_at
        +pendaftarans() HasMany
    }

    class Pendaftaran {
        +int id
        +int user_id
        +int lomba_id
        +string kelas
        +string no_wa
        +string status
        +datetime created_at
        +datetime updated_at
        +user() BelongsTo
        +lomba() BelongsTo
    }

    class AdminController {
        +index() View
        +kelolaLomba() View
        +kelolaSiswa() View
        +kelolaStatus() View
        +storeLomba(Request) RedirectResponse
        +editLomba(id) View
        +updateLomba(Request, id) RedirectResponse
        +destroyLomba(id) RedirectResponse
        +updateStatus(Request, id) RedirectResponse
        +storeSiswa(Request) RedirectResponse
        +editSiswa(id) View
        +updateSiswa(Request, id) RedirectResponse
        +destroySiswa(id) RedirectResponse
    }

    class SiswaController {
        +index() View
        +daftarLomba(Request) RedirectResponse
    }

    class ProfileController {
        +edit(Request) View
        +update(ProfileUpdateRequest) RedirectResponse
        +destroy(Request) RedirectResponse
    }

    class RoleMiddleware {
        +handle(Request, Closure, role) Response
    }

    Authenticatable <|-- User
    Model <|-- Lomba
    Model <|-- Pendaftaran
    Controller <|-- AdminController
    Controller <|-- SiswaController
    Controller <|-- ProfileController

    User "1" --> "0..*" Pendaftaran : memiliki
    Lomba "1" --> "0..*" Pendaftaran : memiliki

    AdminController ..> Lomba : mengelola
    AdminController ..> User : mengelola
    AdminController ..> Pendaftaran : ubah status
    SiswaController ..> Lomba : membaca
    SiswaController ..> Pendaftaran : membuat
    ProfileController ..> User : mengubah
    RoleMiddleware ..> User : cek role
```

---

## 4. Sequence Diagram

### 4.1 Login

```mermaid
sequenceDiagram
    actor U as Pengguna
    participant V as View (login.blade)
    participant AC as AuthenticatedSessionController
    participant Auth as Auth (Laravel)
    participant DB as Database (users)

    U->>V: Buka /login
    U->>V: Isi email & password, submit
    V->>AC: POST /login
    AC->>Auth: attempt(email, password)
    Auth->>DB: SELECT * FROM users WHERE email = ?
    DB-->>Auth: data user
    alt kredensial salah
        Auth-->>AC: false
        AC-->>V: kembali dengan pesan error
        V-->>U: Tampilkan error
    else kredensial benar
        Auth-->>AC: true
        AC->>AC: regenerate session
        AC-->>U: redirect /dashboard
        U->>AC: GET /dashboard
        alt role = admin
            AC-->>U: redirect /admin/dashboard
        else role = siswa
            AC-->>U: redirect /siswa/dashboard
        end
    end
```

### 4.2 Siswa Mendaftar Lomba

```mermaid
sequenceDiagram
    actor S as Siswa
    participant V as View (siswa.dashboard)
    participant MW as RoleMiddleware
    participant SC as SiswaController
    participant P as Pendaftaran (Model)
    participant DB as Database

    S->>V: Isi form (lomba, kelas, no_wa), submit
    V->>MW: POST /siswa/daftar
    MW->>MW: cek Auth & role == 'siswa'
    alt bukan siswa
        MW-->>S: 403 Forbidden
    else siswa
        MW->>SC: daftarLomba(request)
        SC->>SC: validate(lomba_id, kelas, no_wa)
        SC->>P: where(user_id, lomba_id)->exists()
        P->>DB: SELECT ... FROM pendaftarans
        DB-->>P: hasil
        P-->>SC: true / false
        alt sudah terdaftar
            SC-->>V: back() with error
            V-->>S: "Anda sudah terdaftar di mata lomba ini"
        else belum terdaftar
            SC->>P: create(user_id, lomba_id, kelas, no_wa, status)
            P->>DB: INSERT INTO pendaftarans
            DB-->>P: OK
            SC-->>V: back() with success
            V-->>S: "Berhasil mendaftar!"
        end
    end
```

### 4.3 Admin Mengubah Status Pendaftaran

```mermaid
sequenceDiagram
    actor A as Admin
    participant V as View (admin.kelola_status)
    participant MW as RoleMiddleware
    participant AC as AdminController
    participant P as Pendaftaran (Model)
    participant DB as Database

    A->>V: GET /admin/kelola-status
    V->>MW: request
    MW->>AC: kelolaStatus()
    AC->>P: with(['user','lomba'])->get()
    P->>DB: SELECT pendaftarans + users + lombas
    DB-->>P: data
    AC-->>V: tampilkan tabel pendaftaran
    A->>V: Pilih status baru, submit
    V->>MW: POST /admin/pendaftaran/{id}/status
    MW->>AC: updateStatus(request, id)
    AC->>P: findOrFail(id)
    P->>DB: SELECT ... WHERE id = ?
    DB-->>P: pendaftaran
    AC->>P: update(['status' => status])
    P->>DB: UPDATE pendaftarans SET status = ?
    AC-->>V: back() with success
    V-->>A: "Status pendaftaran berhasil diubah!"
```

### 4.4 Admin Menambah Mata Lomba

```mermaid
sequenceDiagram
    actor A as Admin
    participant V as View (admin.kelola_lomba)
    participant MW as RoleMiddleware
    participant AC as AdminController
    participant L as Lomba (Model)
    participant DB as Database

    A->>V: Isi nama_lomba & deskripsi, submit
    V->>MW: POST /admin/lomba
    MW->>MW: cek role == 'admin'
    MW->>AC: storeLomba(request)
    AC->>L: create(request->all())
    L->>DB: INSERT INTO lombas
    DB-->>L: OK
    AC-->>V: back() with success
    V-->>A: "Mata Lomba berhasil ditambahkan!"
```

### 4.5 Admin Menambah Akun Siswa

```mermaid
sequenceDiagram
    actor A as Admin
    participant V as View (admin.kelola_siswa)
    participant AC as AdminController
    participant H as Hash
    participant U as User (Model)
    participant DB as Database

    A->>V: Isi nama, email, password, submit
    V->>AC: POST /admin/siswa
    AC->>H: make(password)
    H-->>AC: password terenkripsi
    AC->>U: create(name, email, password, role='siswa')
    U->>DB: INSERT INTO users
    DB-->>U: OK
    AC-->>V: back() with success
    V-->>A: "Akun siswa berhasil dibuat!"
```

---

## 5. ERD (Entity Relationship Diagram)

```mermaid
erDiagram
    USERS ||--o{ PENDAFTARANS : "mendaftar"
    LOMBAS ||--o{ PENDAFTARANS : "diikuti"
    USERS ||--o{ SESSIONS : "memiliki"

    USERS {
        bigint id PK
        varchar name
        varchar email UK
        timestamp email_verified_at "nullable"
        varchar password
        varchar role "default 'siswa'"
        varchar remember_token "nullable"
        timestamp created_at
        timestamp updated_at
    }

    LOMBAS {
        bigint id PK
        varchar nama_lomba
        text deskripsi "nullable"
        timestamp created_at
        timestamp updated_at
    }

    PENDAFTARANS {
        bigint id PK
        bigint user_id FK "on delete cascade"
        bigint lomba_id FK "on delete cascade"
        varchar kelas
        varchar no_wa
        varchar status "default 'Dokumen dalam Tinjauan'"
        timestamp created_at
        timestamp updated_at
    }

    SESSIONS {
        varchar id PK
        bigint user_id FK "nullable"
        varchar ip_address "nullable"
        text user_agent "nullable"
        longtext payload
        int last_activity
    }

    PASSWORD_RESET_TOKENS {
        varchar email PK
        varchar token
        timestamp created_at "nullable"
    }
```

Kardinalitas:

- **USERS – PENDAFTARANS**: satu siswa dapat memiliki banyak pendaftaran (1 : N).
- **LOMBAS – PENDAFTARANS**: satu lomba dapat diikuti banyak pendaftaran (1 : N).
- Secara konsep, **USERS** dan **LOMBAS** berelasi *many-to-many* melalui tabel **PENDAFTARANS**, dan aplikasi mencegah satu siswa mendaftar lomba yang sama dua kali (dicek di `SiswaController::daftarLomba`).
- `password_reset_tokens` berhubungan dengan `users` lewat kolom `email` (tanpa foreign key).
