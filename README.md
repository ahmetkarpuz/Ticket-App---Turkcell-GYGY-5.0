# 🎟️ Ticket App — Proje Dokümantasyonu

> **Kotlin + Jetpack Compose** ile geliştirilmiş Android etkinlik bileti uygulaması.  
> Proje multi-modül mimarisine, Koin DI'ya ve Retrofit ile REST API entegrasyonuna sahiptir.

---

## 📋 İçindekiler

1. [Proje Genel Bakış](#1-proje-genel-bakış)
2. [Mimari — Multi-Modül Yapısı](#2-mimari--multi-modül-yapısı)
3. [Modüller ve Dosya Yapısı](#3-modüller-ve-dosya-yapısı)
4. [Kullanılan Teknolojiler ve Kütüphaneler](#4-kullanılan-teknolojiler-ve-kütüphaneler)
5. [Katmanlar Arası İlişki (Dependency Graph)](#5-katmanlar-arası-i̇lişki-dependency-graph)
6. [Koin Dependency Injection — Nasıl Çalışır?](#6-koin-dependency-injection--nasıl-çalışır)
7. [Dosya Bazlı Açıklamalar](#7-dosya-bazlı-açıklamalar)
8. [Veri Akışı — Login Senaryosu](#8-veri-akışı--login-senaryosu)
9. [API Bağlantısı](#9-api-bağlantısı)
10. [Geliştirmeye Nereden Başlarım?](#10-geliştirmeye-nereden-başlarım)

---

## 1. Proje Genel Bakış

Bu uygulama, kullanıcıların etkinlikleri görebileceği, bilet satın alabileceği ve QR kod ile kapıda check-in yapabileceği bir mobil platformdur. Uygulama backend'e REST API üzerinden bağlanır.

### Roller

| Rol | Yetkiler |
|---|---|
| `USER` | Etkinlik listesi, bilet satın alma, biletlerimi görme |
| `STAFF` | Atandığı etkinliklerde QR tarama / check-in |
| `ADMIN` | Etkinlik yönetimi, bilet türü oluşturma, staff atama |

---

## 2. Mimari — Multi-Modül Yapısı

Proje **3 ayrı modülden** oluşur. Her modülün belirli bir sorumluluğu vardır:

```
kotlin-ticket-app/
├── app/        ← UI katmanı (Ekranlar, ViewModel, Navigation)
├── core/       ← Paylaşılan sözleşmeler (interface, model, tema)
├── data/       ← Veri katmanı (API, Repository implementasyonu, DTO)
```

### Neden Multi-Modül?

- **Sorumlulukları ayırmak (Separation of Concerns):** Her modül sadece kendi işini yapar.
- **Bağımlılık yönü tek taraflıdır:** `app` → `core`'u ve `data`'yı bilir, ama `data` asla `app`'i bilmez.
- **`core` bağımsızdır:** Ne `app`'e ne de `data`'ya bağlıdır. Sadece sözleşmeleri tanımlar.

```
app  ──depends on──►  core
app  ──depends on──►  data
data ──depends on──►  core
core ──(kimseye bağlı değil)
```

---

## 3. Modüller ve Dosya Yapısı

### 📦 `app` modülü — UI Katmanı

```
app/src/main/java/com/turkcell/ticketapp/
│
├── MainActivity.kt               ← Uygulamanın tek Activity'si
├── TicketAppApplication.kt       ← Koin'i başlatan Application sınıfı
│
├── di/
│   └── AppModule.kt              ← ViewModel bağımlılıklarının tanımlandığı Koin modülü
│
├── navigation/
│   ├── AppDestinations.kt        ← Ekran rotaları (Login, Register, Home)
│   └── AppNavHost.kt             ← NavController ile ekranları birbirine bağlar
│
├── screen/
│   └── LoginScreen.kt            ← Giriş ekranı (Jetpack Compose UI)
│
└── viewmodel/
    └── LoginViewModel.kt         ← Login iş mantığı + UI state yönetimi
```

### 📦 `core` modülü — Paylaşılan Sözleşmeler

```
core/src/main/java/com/turkcell/core/
│
├── domain/
│   ├── AuthRepository.kt         ← Kimlik doğrulama sözleşmesi (interface)
│   ├── AuthSession.kt            ← Giriş sonrası oturum bilgisi (data class)
│   ├── User.kt                   ← Kullanıcı modeli
│   └── UserRole.kt               ← USER / STAFF / ADMIN rolleri (enum)
│
├── ui/theme/
│   ├── Color.kt                  ← Renk paleti
│   ├── Theme.kt                  ← Material3 tema tanımı
│   └── Type.kt                   ← Yazı tipi stilleri
│
└── util/
    └── DateFormatter.kt          ← Türkçe tarih formatlayıcı
```

### 📦 `data` modülü — Veri Katmanı

```
data/src/main/java/com/turkcell/data/
│
├── di/
│   └── DataModule.kt             ← Retrofit, OkHttp, Repository Koin tanımları
│
├── dto/
│   ├── CredentialsDto.kt         ← Login/Register isteği gövdesi
│   ├── RefreshRequestDto.kt      ← Token yenileme isteği
│   ├── TokenPairDto.kt           ← API'den dönen token + user verisi
│   └── UserDto.kt                ← API'den dönen kullanıcı verisi
│
├── network/
│   └── NetworkException.kt       ← NetworkException ve ApiException hata sınıfları
│
├── remote/
│   └── AuthApi.kt                ← Retrofit interface (login, register, refresh)
│
├── repository/
│   └── AuthRepositoryImpl.kt     ← AuthRepository'nin gerçek implementasyonu
│
└── util/
    ├── ApiResult.kt              ← Sealed interface: Success / Error sarmalayıcı
    └── RunCatchingApi.kt         ← API çağrılarını try-catch ile saran yardımcı fonksiyon
```

---

## 4. Kullanılan Teknolojiler ve Kütüphaneler

| Kütüphane | Amaç |
|---|---|
| **Jetpack Compose** | Deklaratif UI framework (XML yerine Kotlin kodu ile UI) |
| **ViewModel** | UI state yönetimi, ekran döndürmede veriyi korur |
| **Navigation Compose** | Ekranlar arası geçiş yönetimi |
| **Koin** | Dependency Injection (bağımlılıkları otomatik enjekte eder) |
| **Retrofit** | HTTP istekleri için type-safe REST client |
| **OkHttp** | Retrofit'in altında çalışan HTTP engine + logging |
| **Kotlinx Serialization** | JSON ↔ Kotlin data class dönüşümü |
| **Kotlin Coroutines + Flow** | Asenkron işlemler ve reaktif veri akışı |
| **DataStore Preferences** | Token'ları cihazda saklamak için (henüz implement edilmemiş) |

---

## 5. Katmanlar Arası İlişki (Dependency Graph)

```
┌─────────────────────────────────────────────────────┐
│                    app modülü                       │
│                                                     │
│  MainActivity → AppNavHost → LoginScreen            │
│                                    ↓                │
│                            LoginViewModel           │
│                                    ↓                │
│                         AuthRepository (interface)  │
└──────────────────────────────┬──────────────────────┘
                               │ core modülünden gelir
                               ▼
┌─────────────────────────────────────────────────────┐
│                   core modülü                       │
│                                                     │
│  AuthRepository (interface)  ← Sözleşme            │
│  AuthSession, User, UserRole ← Domain modeller      │
│  TicketAppTheme              ← Paylaşılan tema      │
└─────────────────────────────────────────────────────┘
                               ▲
                               │ core'u implement eder
┌─────────────────────────────────────────────────────┐
│                   data modülü                       │
│                                                     │
│  AuthRepositoryImpl → AuthApi (Retrofit)            │
│  DataModule (Koin) → tüm bağımlılıkları sağlar     │
│  DTO'lar → API'den gelen JSON'ı temsil eder         │
└─────────────────────────────────────────────────────┘
```

---

## 6. Koin Dependency Injection — Nasıl Çalışır?

**Dependency Injection (DI)** nedir? Bir sınıfın ihtiyaç duyduğu nesneleri (bağımlılıkları) kendisi oluşturmak yerine dışarıdan almasıdır.

### Adım 1 — Uygulama başladığında Koin ayağa kalkar

```kotlin
// TicketAppApplication.kt
class TicketAppApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@TicketAppApplication)
            modules(
                dataModule,  // data katmanının bağımlılıkları
                appModule    // UI katmanının bağımlılıkları
            )
        }
    }
}
```

### Adım 2 — `dataModule` ağ altyapısını ve Repository'yi tanımlar

```kotlin
// DataModule.kt (sadeleştirilmiş)
val dataModule = module {
    single { Json { ignoreUnknownKeys = true } }           // JSON parser (tek instance)
    single { OkHttpClient.Builder()... .build() }          // HTTP client (tek instance)
    single { Retrofit.Builder()... .build() }              // Retrofit (tek instance)
    single { get<Retrofit>().create(AuthApi::class.java) } // API interface
    single<AuthRepository> { AuthRepositoryImpl(get()) }   // Repository
}
```

### Adım 3 — `appModule` ViewModel'i tanımlar

```kotlin
// AppModule.kt
val appModule = module {
    viewModelOf(::LoginViewModel)
    // Koin, LoginViewModel'in AuthRepository'ye ihtiyacı olduğunu otomatik anlar
    // ve dataModule'den sağlar
}
```

### Adım 4 — Ekran ViewModel'i otomatik alır

```kotlin
// LoginScreen.kt
@Composable
fun LoginScreen(
    viewModel: LoginViewModel = koinViewModel(), // Koin otomatik inject eder
    ...
)
```

---

## 7. Dosya Bazlı Açıklamalar

### `TicketAppApplication.kt`
Uygulamanın giriş noktasıdır. Android'de `Activity`'lerden önce oluşturulur ve uygulama kapanana kadar bellekte tek instance olarak kalır (Singleton). Koin'i burada başlatmak, bağımlılıkların tüm uygulama boyunca erişilebilir olmasını sağlar.

---

### `AppDestinations.kt`
Tip-güvenli (type-safe) navigasyon için ekran rotalarını tanımlar. `@Serializable` anotasyonu, Jetpack Navigation'ın bu objeleri rota olarak kullanabilmesi için gereklidir.

```kotlin
@Serializable object Login    // Giriş ekranı rotası
@Serializable object Register // Kayıt ekranı rotası
@Serializable object Home     // Ana sayfa rotası
```

---

### `AppNavHost.kt`
Hangi rota hangi ekrana gidecek? Bu dosya bunu belirler. `NavHost` composable'ı, `startDestination` parametresi ile hangi ekranın ilk açılacağını ayarlar.

---

### `LoginViewModel.kt`
**MVVM mimarisinin** ViewModel katmanıdır. UI'dan gelen olayları (email değişti, butona basıldı) işler ve `StateFlow` aracılığıyla UI'ya güncel state'i iletir.

```kotlin
data class LoginUiState(
    val email: String = "",
    val password: String = "",
    val isLoading: Boolean = false,
    val errorMessage: String? = null,
    val isLoggedIn: Boolean = false
) {
    // Buton aktif mi? Email boş değil + şifre 8 karakter + yükleme yok
    val canSubmit: Boolean get() = email.isNotBlank() && password.length >= 8 && !isLoading
}
```

`submit()` fonksiyonu:
1. `isLoading = true` → UI'da spinner gösterilir
2. `authRepository.login()` coroutine ile çalışır (thread bloke etmez)
3. Başarılı → `isLoggedIn = true` → `LaunchedEffect` tetiklenir → ekran geçişi
4. Başarısız → hata mesajı `errorMessage`'a yazılır

---

### `AuthRepository.kt` (core)
**Interface (Sözleşme):** Ne yapılacağını tanımlar, nasıl yapılacağını değil. Bu sayede `data` modülü değişse bile `app` modülü etkilenmez.

```kotlin
interface AuthRepository {
    val isLoggedIn: Flow<Boolean>
    suspend fun login(email: String, password: String): Result<AuthSession>
    suspend fun register(email: String, password: String): Result<AuthSession>
    suspend fun logout(): Result<Unit>
}
```

---

### `AuthRepositoryImpl.kt` (data)
Interface'in gerçek implementasyonu. `AuthApi` üzerinden API çağrısı yapar, gelen DTO'yu domain modeline (`AuthSession`) dönüştürür.

---

### `AuthApi.kt` (data)
Retrofit interface'i. Her fonksiyon bir API endpoint'ini temsil eder. Retrofit, bu interface'i runtime'da dinamik olarak implement eder.

```kotlin
interface AuthApi {
    @POST("/auth/login")
    suspend fun login(@Body body: CredentialsDto): TokenPairDto

    @POST("/auth/register")
    suspend fun register(@Body body: CredentialsDto): TokenPairDto

    @POST("/auth/refresh")
    suspend fun refresh(@Body body: RefreshRequestDto): TokenPairDto
}
```

---

### `RunCatchingApi.kt` (data)
Tüm API çağrılarını tek bir try-catch bloğunda saran yardımcı fonksiyon. HTTP hatalarını `ApiException`'a, ağ hatalarını `NetworkException`'a dönüştürür. Bu sayede her yerde aynı hata yönetimi uygulanır.

```kotlin
suspend inline fun <T> runCatchingApi(block: suspend () -> T): Result<T> = try {
    Result.success(block())
} catch(e: HttpException) {
    Result.failure(ApiException(code = e.code(), ...))
} catch(e: IOException) {
    Result.failure(NetworkException(e))
}
```

---

### `UserRole.kt` (core)
API'den gelen string değerini (`"ADMIN"`, `"STAFF"`, `"USER"`) Kotlin enum'a dönüştürür. `fromApi()` fonksiyonu bilinmeyen bir değer gelirse `USER` olarak kabul eder (güvenli varsayılan).

---

### DTO'lar nedir? (data/dto/)
**DTO (Data Transfer Object):** API'den gelen JSON verisini temsil eden sınıflardır. Domain modelden (örn. `User`) ayrı tutulur; böylece API değişse bile domain modeli etkilenmez.

| DTO | Açıklama |
|---|---|
| `CredentialsDto` | Login/Register isteği: `{ email, password }` |
| `TokenPairDto` | API yanıtı: `{ user, accessToken, refreshToken }` |
| `UserDto` | API'deki kullanıcı bilgisi: `{ id, email, role }` |
| `RefreshRequestDto` | Token yenileme isteği: `{ refreshToken }` |

---

## 8. Veri Akışı — Login Senaryosu

Kullanıcı "Giriş Yap" butonuna bastığında neler olur?

```
1. LoginScreen (UI)
   └── Butona basıldı → viewModel.submit() çağrılır

2. LoginViewModel
   └── state güncellenir: isLoading = true
   └── viewModelScope.launch { ... } → coroutine başlar
   └── authRepository.login(email, password) çağrılır

3. AuthRepositoryImpl (data katmanı)
   └── runCatchingApi { authApi.login(CredentialsDto(...)) }
   └── Retrofit HTTP isteği gönderir: POST /auth/login
   └── Cevap gelir: TokenPairDto (JSON → Kotlin)
   └── TokenPairDto → AuthSession dönüşümü yapılır

4. LoginViewModel (devam)
   └── .onSuccess → state: isLoggedIn = true
   └── .onFailure → state: errorMessage = "Email veya şifre hatalı"

5. LoginScreen (UI)
   └── LaunchedEffect(state.isLoggedIn) → true oldu
   └── onLoginSuccess() çağrılır → NavController sonraki ekrana gider
```

---

## 9. API Bağlantısı

- **Base URL:** `https://tickets-api.halitkalayci.com/`
- **Auth:** `Authorization: Bearer <accessToken>` header'ı
- **Token süresi:** Access token 15 dk, Refresh token 7 gün
- **Swagger UI:** `GET /docs` → API'yi tarayıcıdan test edebilirsin

### Geliştirme test kullanıcıları

| Email | Şifre | Rol |
|---|---|---|
| `admin@example.com` | `admin123` | ADMIN |
| `staff@example.com` | `staff123` | STAFF |
| `user@example.com` | `user123` | USER |

---

## 10. Geliştirmeye Nereden Başlarım?

### Projeyi açmak
1. Android Studio'yu aç
2. `File → Open` → `kotlin-ticket-app-gygy5-master` klasörünü seç
3. Gradle sync beklenir (ilk açılışta biraz sürer)
4. Emülatör veya fiziksel cihaz bağla → `Run` (▶)

### Yeni bir ekran eklemek için adımlar
1. `AppDestinations.kt`'e yeni rota nesnesi ekle (`@Serializable object MyScreen`)
2. `app/screen/` altına yeni `MyScreen.kt` Composable'ı oluştur
3. `AppNavHost.kt`'e yeni `composable<MyScreen>` bloğu ekle
4. Gerekirse `AppModule.kt`'e ViewModel'i kaydet

### Henüz implement edilmemiş özellikler (`TODO`)
- `AuthRepositoryImpl.isLoggedIn` → DataStore'dan oturum bilgisi okunacak
- `AuthRepositoryImpl.register()` → Kayıt işlemi
- `AuthRepositoryImpl.logout()` → Token iptal + local silme
- Token'ların `EncryptedSharedPreferences` / `DataStore`'a kaydedilmesi
- Register ekranı UI'ı (`RegisterScreen.kt` yok)
- Home ekranı (`Home` rotası boş)
- Etkinlik listesi, bilet satın alma ve QR check-in ekranları

---

## Mimari Özet — MVVM + Clean Architecture

```
UI Layer (app)          →  Ne gösteriliyor?
   LoginScreen               Compose UI
   LoginViewModel            State + İş mantığı

Domain Layer (core)     →  Ne yapılabilir?
   AuthRepository            Sözleşme (interface)
   User, AuthSession         Domain modeller

Data Layer (data)       →  Nasıl yapılır?
   AuthRepositoryImpl        Gerçek implementasyon
   AuthApi (Retrofit)        HTTP istekleri
   DTO'lar                   JSON ↔ Kotlin dönüşümü
```

> Bu mimari sayesinde; backend değişse sadece `data` katmanı, UI değişse sadece `app` katmanı güncellenir. `core` her zaman sabit kalır.
