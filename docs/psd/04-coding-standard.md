# 04 - Coding Standard

**Versi Dokumen:** 1.0.0
**Status:** Active
**Kategori:** Standard
**Terakhir Diperbarui:** [TANGGAL]
**Pemilik Dokumen:** [NAMA TIM / ROLE]

---

## Dokumen Terkait

| Dokumen | Path | Keterangan |
| --- | --- | --- |
| Project Overview | `docs/psd/01-project-overview.md` | Konteks project dan tech stack yang digunakan |
| Setup Guide | `docs/psd/02-setup-guide.md` | Tools linter dan formatter yang digunakan project ini |
| System Architecture | `docs/psd/03-system-architecture.md` | Struktur lapisan yang menjadi dasar standar ini |

---

## Cara Membaca Dokumen Ini

Dokumen ini menjelaskan standar yang berlaku di **project ini**. Ini bukan panduan clean code secara umum dan bukan tutorial framework. Setiap aturan dalam dokumen ini berlaku karena ada keputusan spesifik yang diambil tim untuk project ini.

Pengecualian dari standar ini hanya diperbolehkan dengan persetujuan Tech Lead dan harus didokumentasikan dalam komentar kode yang bersangkutan.

---

## Daftar Isi

1. [Struktur Folder Project](about:blank#1-struktur-folder-project)
2. [Layer Responsibility](about:blank#2-layer-responsibility)
3. [Naming Convention](about:blank#3-naming-convention)
4. [Database Naming Convention](about:blank#4-database-naming-convention)
5. [API Naming Convention](about:blank#5-api-naming-convention)
6. [Validation Placement](about:blank#6-validation-placement)
7. [Business Logic Placement](about:blank#7-business-logic-placement)
8. [Repository Usage](about:blank#8-repository-usage)
9. [Error Handling Standard](about:blank#9-error-handling-standard)
10. [Logging Standard](about:blank#10-logging-standard)
11. [Configuration Standard](about:blank#11-configuration-standard)
12. [Environment Variable Standard](about:blank#12-environment-variable-standard)
13. [Reusable Code Standard](about:blank#13-reusable-code-standard)
14. [Documentation in Code Convention](about:blank#14-documentation-in-code-convention)
15. [Dependency Management Convention](about:blank#15-dependency-management-convention)
16. [Git Workflow](about:blank#16-git-workflow)
17. [Branching Convention](about:blank#17-branching-convention)
18. [Commit Message Convention](about:blank#18-commit-message-convention)
19. [Pull Request Convention](about:blank#19-pull-request-convention)
20. [Forbidden Practice](about:blank#20-forbidden-practice)

---

## 1. Struktur Folder Project

### 1.1 Struktur yang Wajib Diikuti

Setiap kode baru yang ditambahkan ke project harus ditempatkan sesuai struktur berikut. Tidak ada direktori baru yang boleh dibuat di luar struktur ini tanpa persetujuan Tech Lead.

```
src/
├── main.[ext]                      # Entry point aplikasi; tidak berisi business logic
├── app.module.[ext]                # Mendaftarkan semua Module; tidak berisi logic
│
├── modules/                        # Semua domain bisnis ada di sini
│   └── [nama-module]/
│       ├── [nama-module].module.[ext]
│       ├── [nama-module].controller.[ext]
│       ├── [nama-module].service.[ext]
│       ├── [nama-module].repository.[ext]
│       ├── [nama-module].model.[ext]
│       ├── dto/
│       │   ├── create-[nama-module].dto.[ext]
│       │   ├── update-[nama-module].dto.[ext]
│       │   └── query-[nama-module].dto.[ext]
│       └── tests/
│           ├── [nama-module].service.spec.[ext]
│           └── [nama-module].controller.spec.[ext]
│
├── shared/                         # Komponen yang dipakai lebih dari satu Module
│   ├── middleware/
│   ├── guards/
│   ├── filters/
│   ├── interceptors/
│   ├── decorators/
│   ├── utils/
│   └── constants/
│
└── config/                         # Konfigurasi koneksi dan environment
    ├── database.config.[ext]
    ├── cache.config.[ext]
    └── app.config.[ext]
```

### 1.2 Aturan Penempatan Kode

| Situasi | Tempat yang Benar |
| --- | --- |
| Kode hanya digunakan oleh satu Module | Di dalam direktori Module itu sendiri |
| Kode digunakan oleh dua atau lebih Module | Pindahkan ke `src/shared/` |
| Konfigurasi koneksi ke service eksternal | `src/config/` |
| Fungsi utilitas tanpa side effect | `src/shared/utils/` |
| Nilai yang dipakai di banyak tempat | `src/shared/constants/` |
| Integrasi dengan layanan eksternal | `src/modules/[nama-module]/adapters/` |
| Job Queue | `src/modules/[nama-module]/jobs/` |

---

## 2. Layer Responsibility

### 2.1 Apa yang Boleh Ada di Setiap Layer

Ini adalah aturan yang tidak boleh dilanggar di project ini. Pelanggaran menyebabkan kode sulit di-test dan sulit dilacak saat debugging.

**Controller** — Hanya boleh:
- Mendefinisikan route (URL dan HTTP method)
- Mengekstrak data dari request (body, params, query, headers)
- Memanggil DTO untuk validasi format input
- Memanggil satu method Service
- Membentuk response

```tsx
// BENAR — Controller yang bersih di project ini
@Post()
async createUser(@Body() dto: CreateUserDto) {
  const result = await this.userService.createUser(dto);
  return { status: 'success', data: result };
}

// SALAH — Ada kondisi bisnis di Controller
@Post()
async createUser(@Body() dto: CreateUserDto) {
  // SALAH: cek bisnis bukan urusan Controller
  const existing = await this.userRepository.findByEmail(dto.email);
  if (existing) throw new ConflictException('Email sudah ada');
  // ...
}
```

**Service** — Hanya boleh:
- Menjalankan Business Flow dan memeriksa aturan bisnis
- Memanggil Repository
- Memanggil Service lain (lintas Module jika diperlukan)
- Mengelola transaksi Database
- Mempublikasikan job ke Queue

```tsx
// BENAR — Business Flow ada di Service
async createUser(dto: CreateUserDto): Promise<User> {
  // Cek aturan bisnis
  const existing = await this.userRepository.findByEmail(dto.email);
  if (existing) throw new ConflictException({ code: 'EMAIL_ALREADY_REGISTERED' });

  const hashed = await hashPassword(dto.password);
  const user = await this.userRepository.create({ ...dto, password: hashed });

  // Side effect asinkron
  await this.emailQueue.add('SEND_WELCOME', { userId: user.id });
  return user;
}
```

**Repository** — Hanya boleh:
- Menulis query Database
- Mengembalikan Model atau data mentah
- Menerima parameter filter dari Service

```tsx
// BENAR — Repository hanya urusan query
async findByEmail(email: string): Promise<User | null> {
  return this.userModel.findOne({ where: { email } });
}

// SALAH — Ada business logic di Repository
async findByEmail(email: string): Promise<User | null> {
  const user = this.userModel.findOne({ where: { email } });
  // SALAH: ini adalah aturan bisnis, bukan query
  if (user?.role === 'BANNED') throw new ForbiddenException();
  return user;
}
```

**Model** — Hanya boleh:
- Mendefinisikan properti entitas
- Mendefinisikan mapping ke tabel Database via dekorator ORM
- Mendefinisikan relasi antar entitas

Model tidak boleh memiliki method yang berisi logic.

---

## 3. Naming Convention

### 3.1 Format Nama per Tipe

| Tipe | Format | Contoh di Project Ini |
| --- | --- | --- |
| Class | PascalCase | `UserService`, `CreateUserDto`, `UserRole` |
| Interface / Type | PascalCase | `PaginatedResult`, `JwtPayload` |
| Enum | PascalCase | `UserRole`, `OrderStatus` |
| Method / Function | camelCase | `createUser()`, `findByEmail()`, `hashPassword()` |
| Variable | camelCase | `activeUsers`, `requestedUser`, `isActive` |
| Konstanta global | SCREAMING_SNAKE_CASE | `MAX_LOGIN_ATTEMPTS`, `DEFAULT_PAGE_SIZE` |
| Enum value | SCREAMING_SNAKE_CASE | `UserRole.SUPER_ADMIN`, `OrderStatus.PENDING` |
| Environment variable | SCREAMING_SNAKE_CASE | `DATABASE_HOST`, `JWT_SECRET` |

### 3.2 File Naming Convention

Semua file menggunakan format `[nama-entitas].[tipe].[ekstensi]` dengan `kebab-case`.

| Tipe File | Format | Contoh |
| --- | --- | --- |
| Module | `[nama].module.[ext]` | `user.module.ts` |
| Controller | `[nama].controller.[ext]` | `user.controller.ts` |
| Service | `[nama].service.[ext]` | `user.service.ts` |
| Repository | `[nama].repository.[ext]` | `user.repository.ts` |
| Model / Entity | `[nama].model.[ext]` | `user.model.ts` |
| DTO | `[aksi]-[nama].dto.[ext]` | `create-user.dto.ts`, `update-user.dto.ts` |
| Guard | `[nama].guard.[ext]` | `roles.guard.ts` |
| Middleware | `[nama].middleware.[ext]` | `auth.middleware.ts` |
| Filter | `[nama].filter.[ext]` | `http-exception.filter.ts` |
| Adapter | `[nama-layanan].adapter.[ext]` | `email.adapter.ts` |
| Job | `[nama-job].job.[ext]` | `send-welcome-email.job.ts` |
| Util | `[nama].util.[ext]` | `password.util.ts`, `date.util.ts` |
| Constant | `[nama].constant.[ext]` | `error-code.constant.ts` |
| Test | `[nama-file].spec.[ext]` | `user.service.spec.ts` |

### 3.3 Class Naming Convention

```tsx
// Module — suffix "Module"
export class UserModule {}

// Controller — suffix "Controller"
export class UserController {}
export class AuthController {}

// Service — suffix "Service"
export class UserService {}
export class AuthService {}

// Repository — suffix "Repository"
export class UserRepository {}

// Model/Entity — nama entitas bisnis saja, tanpa suffix
export class User {}
export class Order {}

// DTO — aksi + nama entitas + suffix "Dto"
export class CreateUserDto {}
export class UpdateUserProfileDto {}
export class QueryUserDto {}

// Guard — deskripsi + suffix "Guard"
export class RolesGuard {}
export class AuthGuard {}

// Exception — deskripsi + suffix "Exception"
export class UserNotFoundException extends NotFoundException {}
export class EmailAlreadyRegisteredException extends ConflictException {}

// Enum — PascalCase
export enum UserRole {
  SUPER_ADMIN = 'SUPER_ADMIN',
  ADMIN = 'ADMIN',
  USER = 'USER',
}
```

### 3.4 Method Naming Convention

```tsx
// Pattern: [kata kerja] + [objek/konteks]

// Untuk query (Repository)
findById(id: string)
findByEmail(email: string)
findAll(filters: QueryUserDto)
findAllActive()

// Untuk operasi tulis (Repository)
create(data: CreateUserData)
update(id: string, data: UpdateUserData)
softDelete(id: string)

// Untuk Business Flow (Service)
createUser(dto: CreateUserDto)
deactivateUser(id: string, requesterId: string)
changeUserRole(id: string, role: UserRole)

// Untuk query booleans
isEmailRegistered(email: string): Promise<boolean>
hasActiveSession(userId: string): Promise<boolean>

// DILARANG: nama yang tidak jelas
processData()    // Proses data apa?
handleUser()     // Handle apa?
doUpdate()       // Update apa?
```

### 3.5 Variable Naming Convention

```tsx
// Boolean — awali dengan is / has / can / should
const isActive: boolean
const hasPermission: boolean
const canProcess: boolean
const shouldRetry: boolean

// Array — bentuk jamak
const users: User[]
const orderIds: string[]
const activeProducts: Product[]

// Deskriptif, bukan singkatan
// SALAH
const u = await this.userRepository.findById(id);
const d = new Date();
const arr = users.filter(u => u.isActive);

// BENAR
const requestedUser = await this.userRepository.findById(id);
const currentDate = new Date();
const activeUsers = users.filter(user => user.isActive);

// Konstanta dalam scope lokal — SCREAMING_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const DEFAULT_PAGE_LIMIT = 20;
```

---

## 4. Database Naming Convention

### 4.1 Konvensi Nama Objek Database

| Elemen | Konvensi | Contoh |
| --- | --- | --- |
| Nama tabel | `snake_case`, bentuk jamak | `users`, `order_items`, `audit_logs` |
| Nama kolom | `snake_case`, bentuk tunggal | `user_id`, `is_active`, `created_at` |
| Primary key | Selalu `id` | `id UUID` |
| Foreign key | `[tabel_singular]_id` | `user_id`, `product_id`, `order_id` |
| Kolom timestamp | `[event]_at` | `created_at`, `updated_at`, `deleted_at` |
| Kolom boolean | Awali `is_` atau `has_` | `is_active`, `has_verified`, `is_deleted` |
| Index | `idx_[tabel]_[kolom]` | `idx_users_email`, `idx_orders_user_id` |
| Unique constraint | `uq_[tabel]_[kolom]` | `uq_users_email` |
| Foreign key constraint | `fk_[tabel_anak]_[kolom]` | `fk_sessions_user_id` |
| Check constraint | `chk_[tabel]_[deskripsi]` | `chk_users_role` |

### 4.2 Tipe Data Standar

| Data | Tipe yang Digunakan | Alasan |
| --- | --- | --- |
| Primary key | `UUID` | Lebih aman dari serial integer; tidak mengekspos jumlah record |
| Foreign key | `UUID` | Konsisten dengan primary key |
| Teks pendek | `VARCHAR(N)` dengan N yang realistis | Batas yang eksplisit lebih baik dari TEXT untuk kolom terstruktur |
| Teks panjang | `TEXT` | Deskripsi, notes, konten bebas |
| Nilai enum | `VARCHAR(50)` + CHECK constraint | Lebih mudah dimigrasikan daripada tipe ENUM native |
| Uang / harga | `DECIMAL(15,2)` | FLOAT tidak presisi untuk nilai moneter |
| Boolean | `BOOLEAN` | Bukan `SMALLINT` atau `VARCHAR('Y','N')` |
| Timestamp | `TIMESTAMPTZ` | Selalu timezone-aware; disimpan dalam UTC |

---

## 5. API Naming Convention

### 5.1 Konvensi URL

| Aturan | Benar | Salah |
| --- | --- | --- |
| Gunakan kebab-case | `/api/v1/user-profiles` | `/api/v1/userProfiles` |
| Kata benda jamak untuk resource | `/api/v1/users` | `/api/v1/user` |
| Jangan gunakan kata kerja | `/api/v1/users/[id]` | `/api/v1/getUser/[id]` |
| Nested untuk relasi | `/api/v1/users/[id]/orders` | `/api/v1/getUserOrders/[id]` |
| Selalu diawali `/api/v[N]` | `/api/v1/users` | `/users` |

### 5.2 HTTP Method dan Status Code

| Operasi | Method | Status Sukses |
| --- | --- | --- |
| Ambil koleksi | `GET /[resource]` | `200 OK` |
| Ambil satu item | `GET /[resource]/[id]` | `200 OK` |
| Buat baru | `POST /[resource]` | `201 Created` |
| Update penuh | `PUT /[resource]/[id]` | `200 OK` |
| Update sebagian | `PATCH /[resource]/[id]` | `200 OK` |
| Hapus | `DELETE /[resource]/[id]` | `200 OK` atau `204 No Content` |

### 5.3 Format Response

Seluruh response API project ini menggunakan format ini tanpa terkecuali:

```json
// Response sukses — satu item
{
  "status": "success",
  "data": { "id": "uuid", "name": "...", "createdAt": "..." }
}

// Response sukses — koleksi
{
  "status": "success",
  "data": [ ... ],
  "meta": {
    "total": 150,
    "page": 1,
    "limit": 20,
    "totalPages": 8
  }
}

// Response error
{
  "status": "error",
  "code": "KODE_ERROR_BISNIS",
  "message": "Pesan yang dapat dibaca manusia",
  "timestamp": "2025-01-15T08:30:00.000Z"
}
```

---

## 6. Validation Placement

### 6.1 Di Mana Validasi Ditulis

Project ini menggunakan validasi di tiga lapisan yang berbeda untuk tujuan yang berbeda:

| Lapisan | Yang Divalidasi | Contoh |
| --- | --- | --- |
| DTO (Controller) | Format dan tipe data input | Email valid, field wajib ada, string maksimal 255 karakter |
| Service | Aturan bisnis yang butuh data dari Database | Email belum terdaftar, status order boleh dipindah ke status target |
| Database | Constraint sebagai lapisan terakhir | UNIQUE constraint, NOT NULL, FK constraint |

### 6.2 Contoh Validasi di DTO

```tsx
// dto/create-user.dto.ts
export class CreateUserDto {
  @IsNotEmpty({ message: 'Nama tidak boleh kosong' })
  @IsString()
  @MaxLength(255, { message: 'Nama maksimal 255 karakter' })
  name: string;

  @IsNotEmpty({ message: 'Email tidak boleh kosong' })
  @IsEmail({}, { message: 'Format email tidak valid' })
  @Transform(({ value }) => value.toLowerCase().trim())
  email: string;

  @IsNotEmpty({ message: 'Password tidak boleh kosong' })
  @MinLength(8, { message: 'Password minimal 8 karakter' })
  @Matches(/^(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password harus mengandung huruf kapital dan angka',
  })
  password: string;

  @IsOptional()
  @IsEnum(UserRole, { message: 'Role tidak valid' })
  role?: UserRole;
}
```

### 6.3 Validasi Bisnis di Service

```tsx
// user.service.ts
async createUser(dto: CreateUserDto): Promise<User> {
  // Validasi bisnis: cek keunikan email
  const existing = await this.userRepository.findByEmail(dto.email);
  if (existing) {
    throw new ConflictException({ code: 'EMAIL_ALREADY_REGISTERED' });
  }

  // Validasi bisnis lainnya di sini
  // ...
}
```

---

## 7. Business Logic Placement

### 7.1 Aturan

Seluruh Business Flow dan aturan bisnis project ini ditulis **hanya** di Service. Tidak ada pengecualian.

Business Flow yang dimaksud:
- Pemeriksaan kondisi bisnis sebelum operasi
- Penghitungan atau transformasi data berdasarkan aturan bisnis
- Penentuan kapan harus mengirim notifikasi atau job ke Queue
- Pengelolaan urutan pemanggilan Repository dalam satu transaksi

### 7.2 Contoh Business Flow di Service

```tsx
// user.service.ts — contoh business flow yang benar
async deactivateUser(
  targetUserId: string,
  requestingUserId: string,
): Promise<void> {
  // Aturan bisnis 1: target user harus ada
  const target = await this.userRepository.findById(targetUserId);
  if (!target) throw new NotFoundException({ code: 'USER_NOT_FOUND' });

  // Aturan bisnis 2: admin tidak bisa hapus diri sendiri
  if (targetUserId === requestingUserId) {
    throw new UnprocessableEntityException({ code: 'CANNOT_DELETE_SELF' });
  }

  // Aturan bisnis 3: jalankan dalam satu transaksi
  await this.dataSource.transaction(async (em) => {
    await em.update(User, targetUserId, {
      isActive: false,
      deletedAt: new Date(),
    });
    // Hapus semua sesi aktif target user
    await em.delete(Session, { userId: targetUserId });
  });

  // Side effect: catat ke audit log
  await this.auditService.log('USER_DEACTIVATED', {
    targetId: targetUserId,
    performedBy: requestingUserId,
  });
}
```

---

## 8. Repository Usage

### 8.1 Aturan Penggunaan Repository

Semua query Database di project ini ditulis di Repository. Service tidak boleh menulis query langsung.

```tsx
// BENAR — query ada di Repository
// user.repository.ts
async findAllActive(filters: QueryUserDto): Promise<[User[], number]> {
  const qb = this.userModel
    .createQueryBuilder('user')
    .where('user.deletedAt IS NULL')
    .andWhere('user.isActive = :isActive', { isActive: true });

  if (filters.search) {
    qb.andWhere(
      '(user.name ILIKE :search OR user.email ILIKE :search)',
      { search: `%${filters.search}%` },
    );
  }

  return qb
    .orderBy('user.createdAt', 'DESC')
    .take(filters.limit)
    .skip((filters.page - 1) * filters.limit)
    .getManyAndCount();
}
```

### 8.2 Naming Method Repository

Gunakan pola yang konsisten untuk semua Repository di project ini:

```tsx
// Query
findById(id: string): Promise<User | null>
findByEmail(email: string): Promise<User | null>
findAll(filters: QueryUserDto): Promise<[User[], number]>
findAllByUserId(userId: string): Promise<Order[]>
existsByEmail(email: string): Promise<boolean>

// Mutasi
create(data: CreateUserData): Promise<User>
update(id: string, data: Partial<User>): Promise<User>
softDelete(id: string): Promise<void>
hardDelete(id: string): Promise<void>
```

### 8.3 Dependency Flow: Siapa Memanggil Siapa

```
HTTP Request
     |
     v
Controller.createUser(dto)
     |
     | Memanggil
     v
UserService.createUser(dto)
     |
     +-- Memanggil UserRepository.findByEmail(email)
     |         |
     |         v
     |   Database (SELECT FROM users WHERE email = ?)
     |
     +-- Memanggil UserRepository.create(data)
               |
               v
         Database (INSERT INTO users ...)
               |
               v
         Return user
```

---

## 9. Error Handling Standard

### 9.1 Hierarchy Exception di Project Ini

```tsx
// 400 — Request tidak dapat diparsing
throw new BadRequestException('JSON tidak valid');

// 401 — Tidak terautentikasi
throw new UnauthorizedException({ code: 'TOKEN_EXPIRED' });

// 403 — Terautentikasi tapi tidak punya izin
throw new ForbiddenException({ code: 'INSUFFICIENT_PERMISSION' });

// 404 — Resource tidak ditemukan
throw new NotFoundException({ code: 'USER_NOT_FOUND' });

// 409 — Konflik dengan data yang sudah ada
throw new ConflictException({ code: 'EMAIL_ALREADY_REGISTERED' });

// 422 — Data valid tapi melanggar aturan bisnis
throw new UnprocessableEntityException({ code: 'CANNOT_DELETE_SELF' });

// 500 — Jangan pernah throw manual; biarkan Global Filter menangkap
```

### 9.2 Custom Exception

Buat custom exception untuk error bisnis yang spesifik project ini:

```tsx
// shared/exceptions/user-not-found.exception.ts
export class UserNotFoundException extends NotFoundException {
  constructor(id?: string) {
    super({
      code: 'USER_NOT_FOUND',
      message: id
        ? `User dengan ID '${id}' tidak ditemukan`
        : 'User tidak ditemukan',
    });
  }
}

// Penggunaan di Service
const user = await this.userRepository.findById(id);
if (!user) throw new UserNotFoundException(id);
```

### 9.3 Global Exception Filter

Project ini memiliki satu Global Exception Filter yang menangkap semua exception dan memformat response secara konsisten. Tidak perlu try-catch di Controller atau Service kecuali untuk kasus yang sangat spesifik.

```tsx
// shared/filters/http-exception.filter.ts
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse();

    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const body = exception.getResponse();
      return response.status(status).json({
        status: 'error',
        code: (body as any).code ?? 'HTTP_ERROR',
        message: (body as any).message ?? exception.message,
        timestamp: new Date().toISOString(),
      });
    }

    // Unexpected error — log dan kembalikan response generic
    console.error('[UNHANDLED]', exception);
    return response.status(500).json({
      status: 'error',
      code: 'INTERNAL_SERVER_ERROR',
      message: 'Terjadi kesalahan pada server',
      timestamp: new Date().toISOString(),
    });
  }
}
```

### 9.4 Kode Error Bisnis

Semua kode error bisnis project ini terdefinisi di satu file konstanta:

```tsx
// shared/constants/error-code.constant.ts
export const ErrorCode = {
  // Authentication
  TOKEN_INVALID: 'TOKEN_INVALID',
  TOKEN_EXPIRED: 'TOKEN_EXPIRED',
  INVALID_CREDENTIALS: 'INVALID_CREDENTIALS',
  ACCOUNT_INACTIVE: 'ACCOUNT_INACTIVE',

  // User
  USER_NOT_FOUND: 'USER_NOT_FOUND',
  EMAIL_ALREADY_REGISTERED: 'EMAIL_ALREADY_REGISTERED',
  CANNOT_DELETE_SELF: 'CANNOT_DELETE_SELF',

  // [Tambahkan kode error bisnis lain di sini]
} as const;
```

---

## 10. Logging Standard

### 10.1 Cara Logging di Project Ini

Gunakan logger yang dikonfigurasi project, bukan `console.log`. Logger ini menulis dalam format JSON yang dapat dibaca oleh log aggregator.

```tsx
// Di Service atau komponen lain
@Injectable()
export class UserService {
  private readonly logger = new Logger(UserService.name);

  async createUser(dto: CreateUserDto): Promise<User> {
    this.logger.log({
      message: 'Memulai pembuatan user baru',
      email: dto.email,
    });

    const user = await this.userRepository.create({ ... });

    this.logger.log({
      message: 'User berhasil dibuat',
      userId: user.id,
      email: user.email,
      role: user.role,
    });

    return user;
  }
}
```

### 10.2 Level Log yang Digunakan

| Level | Kapan Digunakan | Contoh |
| --- | --- | --- |
| `log` / `info` | Operasi normal yang penting | User dibuat, login berhasil, job Queue selesai |
| `warn` | Kondisi tidak normal tapi tidak merusak | Token akan expire dalam 1 menit, retry ke-2 gagal |
| `error` | Error yang tidak terduga | Koneksi Database putus, exception tidak tertangkap |
| `debug` | Detail teknis yang hanya berguna saat debugging | Nilai variable saat proses kalkulasi |

`debug` hanya aktif di environment `local`. Level `debug` dinonaktifkan di staging dan production.

### 10.3 Yang Tidak Boleh Dicatat

```tsx
// DILARANG — jangan log data sensitif
this.logger.log({ password: dto.password });         // password
this.logger.log({ token: accessToken });              // token
this.logger.log({ secret: process.env.JWT_SECRET }); // secret
this.logger.log({ cardNumber: payment.cardNumber });  // data kartu
```

---

## 11. Configuration Standard

### 11.1 Cara Mengakses Konfigurasi

Di project ini, konfigurasi **tidak** diakses langsung lewat `process.env` di dalam kode bisnis. Semua konfigurasi harus mengalir melalui `config/` terlebih dahulu.

```tsx
// config/app.config.ts — definisi konfigurasi
export default () => ({
  app: {
    name: process.env.APP_NAME ?? '[NAMA PROJECT]',
    port: parseInt(process.env.APP_PORT ?? '3000', 10),
    env: process.env.APP_ENV ?? 'local',
    debug: process.env.APP_DEBUG === 'true',
  },
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT ?? '5432', 10),
    name: process.env.DATABASE_NAME,
    user: process.env.DATABASE_USER,
    password: process.env.DATABASE_PASSWORD,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN ?? '15m',
  },
});
```

```tsx
// Di Service — akses konfigurasi via ConfigService
@Injectable()
export class AuthService {
  constructor(private readonly configService: ConfigService) {}

  private getJwtSecret(): string {
    return this.configService.get<string>('jwt.secret');
  }
}

// DILARANG — akses process.env langsung di luar config/
@Injectable()
export class AuthService {
  private readonly secret = process.env.JWT_SECRET; // DILARANG
}
```

---

## 12. Environment Variable Standard

### 12.1 Konvensi Penamaan

Semua environment variable menggunakan `SCREAMING_SNAKE_CASE` dengan prefix yang menunjukkan kategori:

```bash
# Aplikasi — prefix APP_
APP_NAME=
APP_PORT=
APP_ENV=
APP_DEBUG=
APP_KEY=

# Database — prefix DATABASE_
DATABASE_HOST=
DATABASE_PORT=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_SSL=

# Cache — prefix REDIS_
REDIS_HOST=
REDIS_PORT=
REDIS_PASSWORD=
REDIS_DB=

# Auth — prefix JWT_
JWT_SECRET=
JWT_EXPIRES_IN=
JWT_REFRESH_EXPIRES_IN=

# Integrasi eksternal — prefix [NAMA_LAYANAN]_
EMAIL_SERVICE_API_KEY=
EMAIL_SERVICE_BASE_URL=
PAYMENT_GATEWAY_API_KEY=
PAYMENT_GATEWAY_BASE_URL=
```

### 12.2 Aturan

- Setiap variable wajib ada di `.env.example` dengan komentar yang menjelaskan tujuannya
- Nilai secret tidak boleh ada di `.env.example` — gunakan placeholder `[MINTA KE TECH LEAD]`
- Aplikasi wajib gagal saat startup (fail fast) jika variable wajib tidak tersedia
- Tidak ada nilai hardcode di kode untuk hal yang bisa berubah antar environment

---

## 13. Reusable Code Standard

### 13.1 Kapan Sebuah Kode Harus Dipindah ke Shared

Aturan sederhana: jika kode yang sama muncul di **tiga atau lebih** tempat berbeda, ekstrak ke `shared/`.

| Tipe Kode | Diekstrak Ke | Contoh |
| --- | --- | --- |
| Pure function tanpa side effect | `src/shared/utils/` | `hashPassword()`, `formatCurrency()`, `slugify()` |
| Nilai yang dipakai di banyak tempat | `src/shared/constants/` | `MAX_LOGIN_ATTEMPTS`, `ErrorCode` |
| Service yang dibutuhkan banyak Module | `src/shared/` atau Module baru | `AuditService`, `NotificationService` |
| Middleware, Guard, Filter | `src/shared/` | `AuthMiddleware`, `RolesGuard` |

### 13.2 Contoh Utilitas yang Sudah Ada di Project

```tsx
// src/shared/utils/password.util.ts
export async function hashPassword(plain: string): Promise<string> { ... }
export async function verifyPassword(plain: string, hash: string): Promise<boolean> { ... }

// src/shared/utils/pagination.util.ts
export function buildPaginationMeta(total: number, page: number, limit: number) {
  return {
    total,
    page,
    limit,
    totalPages: Math.ceil(total / limit),
    hasNextPage: page < Math.ceil(total / limit),
    hasPreviousPage: page > 1,
  };
}

// src/shared/constants/error-code.constant.ts
export const ErrorCode = { ... };
```

---

## 14. Documentation in Code Convention

### 14.1 Apa yang Harus Didokumentasikan dalam Kode

Komentar dalam kode harus menjelaskan **mengapa**, bukan **apa**. Kode yang sudah jelas tidak perlu komentar tambahan.

```tsx
// SALAH — menjelaskan apa yang sudah jelas dari kode
// Cari user berdasarkan email
const user = await this.userRepository.findByEmail(email);

// BENAR — menjelaskan mengapa keputusan ini dibuat
// Kedua kondisi menghasilkan error yang sama untuk mencegah user enumeration.
// Penyerang tidak boleh tahu apakah email terdaftar atau tidak.
if (!user || !(await verifyPassword(dto.password, user.password))) {
  throw new UnauthorizedException({ code: ErrorCode.INVALID_CREDENTIALS });
}
```

### 14.2 JSDoc untuk Method Publik di Service

Method publik di Service wajib memiliki JSDoc yang menjelaskan tujuan, parameter, return value, dan exception yang dilempar.

```tsx
/**
 * Membuat user baru dalam sistem.
 *
 * Memvalidasi keunikan email, meng-hash password, menyimpan user,
 * dan mempublikasikan job email selamat datang ke Queue.
 *
 *@paramdto - Data user yang sudah divalidasi oleh CreateUserDto
 *@returns User yang baru dibuat (tanpa field password)
 *@throws{ConflictException} Jika email sudah terdaftar — kode: EMAIL_ALREADY_REGISTERED
 */
async createUser(dto: CreateUserDto): Promise<User> {
  // ...
}
```

### 14.3 TODO dan FIXME

```tsx
//TODO [PROJ-123]: Tambahkan validasi nomor telepon setelah integrasi OTP selesai
//FIXME [PROJ-456]: Bug saat user memiliki lebih dari 10 sesi aktif — sedang diselidiki
//HACK: Sementara menggunakan polling karena webhook [LAYANAN] belum stabil.
//       Akan diganti saat vendor merilis versi API yang baru (estimasi Q3 2025)
```

Komentar `TODO` tanpa kode tiket atau tanpa alasan tidak diperbolehkan dan akan ditolak saat code review.

---

## 15. Dependency Management Convention

### 15.1 Aturan Penambahan Library

- Library baru untuk production build (`dependencies`) wajib didiskusikan dengan Tech Lead sebelum ditambahkan
- Library development (`devDependencies`) boleh ditambahkan sendiri
- Cek apakah kebutuhan bisa dipenuhi library yang sudah ada sebelum menambahkan yang baru
- Library yang tidak aktif dirawat, memiliki kerentanan tidak di-patch, atau lisensi tidak kompatibel tidak diperbolehkan

### 15.2 Pemisahan Dependencies

```json
// package.json
{
  "dependencies": {
    // HANYA yang dibutuhkan saat runtime production
    "[framework]": "[versi]",
    "[orm]": "[versi]",
    "[library-bisnis]": "[versi]"
  },
  "devDependencies": {
    // Yang hanya dibutuhkan saat development
    "[linter]": "[versi]",
    "[formatter]": "[versi]",
    "[testing-framework]": "[versi]",
    "@types/[nama]": "[versi]"
  }
}
```

### 15.3 Pin Versi untuk Library Kritikal

```json
// Library yang bila berubah memengaruhi seluruh codebase: pin exact version
"[nama-library]": "[EXACT_VERSION]"

// Library biasa: allow patch updates
"[nama-library]": "~[MAJOR].[MINOR].[PATCH]"

// DILARANG
"[nama-library]": "latest"
"[nama-library]": "*"
```

Selalu commit file lock (`pnpm-lock.yaml` / `package-lock.json` / `yarn.lock`) ke repository.

---

## 16. Git Workflow

### 16.1 Alur Kerja Harian

```
Setiap hari mulai kerja:

1. git checkout [BRANCH-UTAMA — contoh: develop]
   git pull origin develop

2. Buat branch baru untuk task yang dikerjakan
   git checkout -b feature/[nama-task]

3. Kerjakan task; commit secara incremental
   (setiap commit adalah satu unit perubahan yang koheren)

4. Sebelum push, pastikan:
   - Linter lulus
   - Type check lulus
   - Test lulus

5. Push branch ke remote
   git push origin feature/[nama-task]

6. Buat Pull Request ke [BRANCH-UTAMA]
```

### 16.2 Aturan Git

- Tidak ada commit langsung ke `main` atau `develop`; semua perubahan melalui Pull Request
- Setiap commit harus bisa di-build dan lulus test secara mandiri
- Tidak ada file `.env` atau credential yang boleh di-commit dalam kondisi apapun
- Tidak ada file binary besar, file log, atau file debug yang boleh di-commit

---

## 17. Branching Convention

### 17.1 Nama Branch

```
Format: [tipe]/[deskripsi-dalam-kebab-case]

# Feature baru
feature/user-profile-endpoint
feature/email-verification-flow
feature/export-attendance-report

# Perbaikan bug
fix/login-token-expiry-bug
fix/pagination-count-incorrect

# Hotfix di production
hotfix/sql-injection-in-search
hotfix/sessions-not-cleared-on-logout

# Task teknis (update deps, refactor, konfigurasi)
chore/update-typescript-5-3
chore/refactor-auth-middleware

# Update dokumentasi
docs/update-setup-guide
docs/add-api-error-codes
```

### 17.2 Branch Utama

| Branch | Tujuan | Deploy Ke | Siapa yang Push |
| --- | --- | --- | --- |
| `main` | Kode production | Production | Hanya via PR dari staging/hotfix |
| `develop` | Integrasi fitur aktif | Development | Hanya via PR |
| `feature/*` | Satu fitur atau task | Tidak ada | Developer pemilik task |
| `fix/*` | Perbaikan bug | Tidak ada | Developer pemilik task |
| `hotfix/*` | Fix darurat production | Production + develop | Senior dev + Tech Lead |

---

## 18. Commit Message Convention

### 18.1 Format

Project ini menggunakan **Conventional Commits**:

```
[tipe]([scope-opsional]): [deskripsi singkat]

[body opsional — penjelasan mengapa, bukan apa]

[footer opsional — referensi tiket atau breaking change]
```

### 18.2 Tipe Commit

| Tipe | Kapan Digunakan |
| --- | --- |
| `feat` | Fitur baru yang terlihat oleh pengguna |
| `fix` | Perbaikan bug |
| `refactor` | Restrukturisasi kode tanpa perubahan perilaku |
| `test` | Penambahan atau perbaikan test |
| `docs` | Update dokumentasi |
| `chore` | Update dependency, konfigurasi build, tooling |
| `style` | Perubahan formatting saja (tidak ada perubahan logic) |
| `perf` | Perbaikan performa |
| `revert` | Membatalkan commit sebelumnya |

### 18.3 Contoh Commit yang Benar

```bash
# Feature baru
feat(auth): tambah endpoint refresh token

# Perbaikan bug — dengan body yang menjelaskan mengapa
fix(user): perbaiki email tidak di-lowercase saat registrasi

Email yang mengandung huruf kapital (contoh: John@Domain.COM) dapat
membuat user terdaftar ganda karena pencarian email bersifat case-sensitive.
Perbaikan: transform email ke lowercase di DTO sebelum disimpan.

# Chore
chore(deps): update TypeScript ke versi 5.3.3

# Docs
docs(psd): perbarui setup guide dengan langkah verifikasi Docker

# Test
test(auth): tambah unit test untuk kasus token expire

# Dengan referensi tiket
feat(attendance): tambah flow check-in via QR code

Refs: PROJ-234
```

### 18.4 Aturan Penulisan

- Deskripsi singkat maksimal **72 karakter**
- Gunakan **huruf kecil** untuk deskripsi singkat
- Gunakan **kata kerja perintah**: “tambah”, “perbaiki”, “hapus” (bukan “menambahkan”)
- Satu commit = satu unit perubahan yang koheren
- Jangan gabungkan perbaikan bug dan fitur baru dalam satu commit

---

## 19. Pull Request Convention

### 19.1 Judul Pull Request

Mengikuti format yang sama dengan commit message:

```
feat(user): tambah endpoint update foto profil
fix(auth): perbaiki refresh token tidak ter-revoke saat logout
chore(deps): update seluruh dependency ke versi terbaru
```

### 19.2 Template Deskripsi

Setiap Pull Request wajib mengisi template berikut:

```markdown
## Apa yang Berubah

[Jelaskan dalam 2-3 kalimat apa yang ditambahkan, diubah, atau dihapus]

## Mengapa

[Jelaskan masalah yang diselesaikan atau alasan perubahan ini diperlukan]
[Sertakan link ke tiket jika ada: Refs: PROJ-XXX]

## Cara Menguji

1.[Langkah untuk memverifikasi perubahan ini bekerja]
2.[Contoh request curl atau langkah di UI]
3.[Hasil yang diharapkan]

## Checklist

-[ ] Mengikuti standar di 04-coding-standard.md
-[ ] Linter lulus
-[ ] Type check lulus
-[ ] Unit test ditulis untuk kode baru
-[ ] Seluruh test lulus
-[ ] Dokumentasi diperbarui jika ada perubahan kontrak API atau flow
-[ ] Tidak ada secret atau file .env yang ikut di-commit
-[ ] Self-review sudah dilakukan
```

### 19.3 Aturan Review

- Pull Request tidak boleh di-merge oleh pembuatnya sendiri; minimal satu reviewer lain
- Seluruh CI check harus lulus sebelum review diminta
- Semua komentar reviewer wajib ditanggapi sebelum merge
- PR yang mengubah kontrak API atau Business Flow harus disertai update dokumentasi di PR yang sama

### 19.4 Cara Memberikan Komentar Review

Gunakan prefiks berikut agar pembuat PR tahu urgensi setiap komentar:

```
[WAJIB] Harus diperbaiki sebelum merge
[SARAN] Boleh diabaikan, tapi dipertimbangkan
[TANYA] Perlu klarifikasi atau penjelasan
[NITPICK] Detail kecil, tidak urgent
```

---

## 20. Forbidden Practice

Praktik berikut **tidak diperbolehkan** di project ini. Pull Request yang mengandung praktik ini akan ditolak.

| Praktik | Alasan |
| --- | --- |
| `any` type tanpa komentar justifikasi | Menghilangkan manfaat type safety seluruh codebase |
| `process.env` langsung di luar `config/` | Konfigurasi tersebar dan tidak terpusat |
| `console.log` di kode production | Mengotori log dengan output tidak terstruktur |
| Query Database di Controller atau Service langsung | Melanggar aturan lapisan; sulit di-test |
| Raw SQL dengan string concatenation | Rentan SQL injection |
| Menelan exception dengan `catch {}` kosong | Menyembunyikan error nyata |
| Circular dependency antar Module | Menyebabkan runtime error yang sulit dilacak |
| Magic number atau magic string tanpa konstanta | Kode sulit dipahami dan diubah |
| Hardcode URL, credential, atau API key | Risiko keamanan dan tidak bisa dikonfigurasi per environment |
| Commit file `.env` | Mengekspos credential ke seluruh anggota yang punya akses repo |
| Nested `if` lebih dari 3 level | Kompleksitas yang tidak perlu; gunakan guard clause |
| `TODO` tanpa nomor tiket atau alasan | Technical debt tidak terlacak |
| Import Repository dari Module lain langsung | Melanggar prinsip kepemilikan data antar Module |
| Side effect di DTO atau Model | Logika tidak ada di lapisan yang tepat |