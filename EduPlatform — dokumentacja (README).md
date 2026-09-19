# EduPlatform

Aplikacja do zarządzania platformą kursów online. Projekt szkolny — kod jest pisany prosto, z komentarzami po polsku.

- Backend: ASP.NET Core Web API, .NET 10, EF Core (SQLite domyślnie, SQL Server opcjonalnie), JWT + role
- Frontend: React + TypeScript (Vite), podział na models / views / controllers

## Jak uruchomić

### Backend (Rider lub terminal)

```bash
cd src/EduPlatform.Api
dotnet run
```

- API: http://localhost:5000
- Swagger (dokumentacja endpointów): http://localhost:5000/swagger
- Baza `eduplatform.db` (SQLite) tworzy się automatycznie z migracji, wraz z przykładowymi danymi.

W Riderze: otwórz `EduPlatform.sln`, wybierz profil `EduPlatform.Api` i kliknij Run.

### Frontend (VS Code lub terminal)

```bash
cd frontend
npm install
npm run dev
```

- Aplikacja: http://localhost:5173
- Adres API ustawia plik `.env` (`VITE_API_URL`).

### Konta testowe (dane startowe)

| Rola | Email | Hasło |
|---|---|---|
| Admin | admin@edu.pl | Admin123! |
| Instructor | teacher@edu.pl | Teacher123! |
| Student | student@edu.pl | Student123! |

## Testy

```bash
dotnet test          # backend: 12 testów (xUnit, SQLite in-memory)
cd frontend && npm test   # frontend: 8 testów (Vitest + React Testing Library)
```

## Struktura projektu

```
EduPlatform.sln
src/
  EduPlatform.Domain/          encje: User, Course, Lesson, Enrollment, Comment, ResourceFile
  EduPlatform.Application/     DTO, interfejsy, serwisy (logika biznesowa), PagedResult, AppException
  EduPlatform.Infrastructure/  EF Core (AppDbContext, migracje), PBKDF2, JWT, zapis plików, seed
  EduPlatform.Api/             kontrolery, Program.cs (DI, JWT, CORS, rate limit), middleware błędów
tests/
  EduPlatform.Tests/           testy jednostkowe serwisów
frontend/
  src/models/                  typy TS + klient axios + serwisy API (authService, courseService)
  src/views/                   komponenty prezentacyjne / strony
  src/controllers/             hooki z logiką (useCourses, useCourseDetails)
  src/store/                   stan globalny (zustand): zalogowany użytkownik
  src/__tests__/               testy komponentów i hooków
```

Zależności idą w jedną stronę: `Api → Infrastructure → Application → Domain`. Warstwa Application nie zna EF-owej implementacji — korzysta z interfejsu `IAppDbContext`.

## Endpointy API (wszystkie pod `/api/v1`)

Autoryzacja: nagłówek `Authorization: Bearer <token>`.

### Auth
| Metoda | Ścieżka | Opis | Dostęp |
|---|---|---|---|
| POST | `/auth/register` | rejestracja | wszyscy |
| POST | `/auth/login` | logowanie (access + refresh token) | wszyscy |
| POST | `/auth/refresh` | odświeżenie tokenu | zalogowany |
| GET | `/auth/me` | dane zalogowanego | zalogowany |

### Users
| Metoda | Ścieżka | Opis | Dostęp |
|---|---|---|---|
| GET | `/users/{id}` | szczegóły użytkownika | Admin lub właściciel |
| GET | `/users/{id}/enrollments` | zapisy na kursy | Admin lub właściciel |

### Courses
| Metoda | Ścieżka | Opis | Dostęp |
|---|---|---|---|
| GET | `/courses` | lista z filtrami i paginacją | wszyscy |
| GET | `/courses/{id}` | szczegóły z lekcjami | wszyscy |
| POST | `/courses` | utworzenie kursu (z lekcjami, w transakcji) | Instructor/Admin |
| PUT | `/courses/{id}` | edycja | właściciel/Admin |
| DELETE | `/courses/{id}` | usunięcie | właściciel/Admin |

Parametry listy: `category`, `tags`, `instructorId`, `priceMin`, `priceMax`, `published`, `q`, `page`, `pageSize`, `sort` (`createdAt_desc`, `title_asc`, `price_asc`, `price_desc`).

Przykład: `GET /api/v1/courses?q=react&category=Frontend&priceMax=200&page=1&pageSize=6&sort=price_asc`

### Lessons / Files
| Metoda | Ścieżka | Opis | Dostęp |
|---|---|---|---|
| POST | `/courses/{id}/lessons` | dodanie lekcji | Instructor/Admin |
| PUT | `/courses/{id}/lessons/{lessonId}` | edycja lekcji (też `order`) | Instructor/Admin |
| DELETE | `/courses/{id}/lessons/{lessonId}` | usunięcie lekcji | Instructor/Admin |
| POST | `/courses/{id}/upload` | upload materiału (multipart) | Instructor/Admin |

### Enrollments / Comments / Admin
| Metoda | Ścieżka | Opis | Dostęp |
|---|---|---|---|
| POST | `/courses/{id}/enroll` | zapis na kurs (transakcja + powiadomienie) | zalogowany |
| PUT | `/enrollments/{id}/progress?percent=50` | postęp w kursie | właściciel/Admin |
| GET | `/courses/{id}/comments` | komentarze w wątkach | wszyscy |
| POST | `/courses/{id}/comments` | dodanie komentarza / odpowiedzi | zalogowany |
| GET | `/admin/statistics` | statystyki platformy | Admin |

## Format odpowiedzi

Sukces:

```json
{ "success": true, "data": { }, "error": null }
```

Błąd:

```json
{ "success": false, "data": null, "error": "Kurs nie zostal znaleziony." }
```

Kody: `400` błędne dane, `401` brak/zły token, `403` brak uprawnień, `404` nie znaleziono, `409` konflikt (np. zajęty email, drugi zapis na ten sam kurs), `429` przekroczony limit zapytań, `500` błąd serwera.

## Jak zrealizowane są wymagania

| Wymaganie | Gdzie w kodzie |
|---|---|
| Hashowanie haseł (PBKDF2, 100 tys. iteracji) | `Infrastructure/Security/PasswordHasher.cs` |
| JWT + role | `Infrastructure/Security/TokenService.cs`, `Program.cs`, atrybuty `[Authorize(Roles = ...)]` |
| Transakcja: kurs + lekcje | `Application/Services/CourseService.CreateAsync` |
| Transakcja: zapis + powiadomienie | `Application/Services/EnrollmentService.EnrollAsync` |
| Optimistic concurrency | `Course.RowVersion` + `IsRowVersion()` w `AppDbContext`, obsługa `DbUpdateConcurrencyException` |
| Zapobieganie N+1 | `Include` / `ThenInclude` i projekcje w `Select` (jedno zapytanie SQL na listę) |
| Wyszukiwanie i filtry | `CourseService.GetListAsync` (LIKE + indeksy na `Title`, `Category`, `Slug`) |
| Paginacja offset-based | `PagedResult<T>` + `Skip`/`Take`; zmiana na cursor-based wymaga tylko podmiany tych dwóch linii |
| Logowanie (Serilog do pliku) | `Program.cs`, katalog `logs/` |
| Middleware błędów | `Api/Middleware/ExceptionMiddleware.cs` |
| Walidacja wejścia | atrybuty DataAnnotations w DTO + `[ApiController]`; na frontendzie zod |
| Limit i typy uploadu | `CoursesController.Upload` + sekcja `FileStorage` w `appsettings.json` |
| Rate limiting | `Program.cs` — 100 zapytań na minutę |
| SQL Injection | wyłącznie LINQ/EF Core (zapytania parametryzowane) |
| Wersjonowanie API | prefiks `api/v1` na wszystkich kontrolerach |
| Swagger z JWT | `Program.cs` (`AddSwaggerGen` + security definition) |
| SOLID / DI | serwisy przez konstruktor, interfejsy w `Application/Interfaces` |
| DTO (bez encji EF w API) | katalog `Application/Dtos` |

## Przejście na SQL Server

W `appsettings.json` ustaw `"Database": { "Provider": "SqlServer" }` i poprawny `ConnectionStrings:SqlServer`, potem:

```bash
dotnet ef migrations add InitialSqlServer -p src/EduPlatform.Infrastructure -s src/EduPlatform.Api
dotnet run --project src/EduPlatform.Api
```

## Uwagi

- Refresh token jest uproszczony (nie jest zapisywany w bazie) — wystarczająco na potrzeby projektu, w produkcji trzymałoby się go w tabeli z datą wygaśnięcia.
- Powiadomienie o zapisie na kurs to wpis w logu (`LogNotificationService`), zamiast maila.
- Pliki zapisywane są lokalnie w `src/EduPlatform.Api/wwwroot/uploads` i serwowane jako `/uploads/...`.
