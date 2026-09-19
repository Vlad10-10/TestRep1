EduPlatform
Projekt i implementacja kompletnej aplikacji do zarządzania platformą kursów online EduPlatform.
Architektura
Backend
ASP.NET Core (C#) — Web API + MVC controllers
EF Core + SQL Server
JWT auth + role-based access
Warstwy:
Api (Controllers)
Application (serwisy, DTOs)
Domain (encje)
Infrastructure (EF Core, Repositories, FileStorage)
Frontend
React + TypeScript
Architektura MVC
Routing, formularze, walidacja, paginacja, filtrowanie
Upload plików (materiały kursów)
Obsługa błędów
Wymagania funkcjonalne
Modele domenowe
User
Id (GUID)
Email
PasswordHash
FullName
Role (Admin/Instructor/Student)
CreatedAt
IsActive
Course
Id
Title
Slug
Description
Category
Tags (lista)
Price
Published
InstructorId (FK User)
CreatedAt
UpdatedAt
Lesson
Id
CourseId (FK)
Title
Content (HTML/MD)
Order
VideoUrl (opcjonalne)
ResourceFiles (lista plików)
DurationSeconds
Enrollment
Id
UserId
CourseId
EnrolledAt
ProgressPercent
IsCompleted
Comment
Id
UserId
CourseId
Text
CreatedAt
ParentCommentId (wątkowanie)
Funkcjonalności kluczowe
Autentykacja i autoryzacja
Rejestracja/logowanie (hasła hashowane PBKDF2/Bcrypt/Argon2).
JWT access token (opcjonalnie refresh token).
Role:
Admin: pełne prawa
Instructor: zarządzanie kursami/lekcjami, publikacja
Student: zapis na kursy, dostęp do treści, komentarze
Kursy i lekcje
CRUD kursów i lekcji
Publikowanie / proste wersjonowanie
Upload plików (PDF/zip/obrazy) — backend zapisuje i zwraca URL
Zarządzanie kolejnością lekcji (Order)
Zapobieganie N+1 zapytaniom
Transakcje i spójność
Tworzenie kursu z lekcjami i zasobami w jednej transakcji
Obsługa współbieżności (optimistic concurrency: row version/timestamp)
Wyszukiwanie i filtrowanie
Filtry: kategorie, tagi, cena, instruktor, status publikacji
Pełnotekstowe wyszukiwanie (SQL Server Full-Text / LIKE z indeksami)
Paginacja i sortowanie
Wsparcie dla paginacji offset-based (z możliwością zmiany na cursor-based)
Logowanie i monitorowanie
Serilog (logowanie do pliku)
Middleware obsługi wyjątków → ustandaryzowane błędy
Bezpieczeństwo
Walidacja wejścia
Limit rozmiaru uploadu + sprawdzanie typów plików
Rate-limiting (uproszczony)
Zabezpieczenie przed SQL Injection (EF Core)
API versioning
Wszystkie endpointy pod /api/v1/...
Testy
Backend:
testy jednostkowe (serwisy)
testy integracyjne (endpointy, InMemory DB/test DB)
Frontend:
testy jednostkowe (komponenty)
integracyjne (React Testing Library)
Dokumentacja
Swagger/OpenAPI — opis endpointów z przykładami
README.md — dokumentacja projektu
Internationalization (opcjonalnie)
i18n w React (PL/EN)
Wymagania niefunkcjonalne
Backend
EF Core + migracje (SQL Server / SQLite)
Automapper (opcjonalnie)
SOLID + Dependency Injection
DTO do komunikacji API (bez eksponowania encji EF)
Frontend
React + TypeScript
Architektura:
models (typy, serwisy API)
views (komponenty prezentacyjne)
controllers/containers (logika, fetchowanie danych, stan)
Routing: react-router
State management: Redux Toolkit lub Zustand
Formularze: react-hook-form + Yup/Zod
HTTP: axios (interceptory: token, refresh)
Testy: Jest + React Testing Library
Stylowanie: Tailwind / CSS Modules / styled-components
API — przykładowe endpointy
Autoryzacja: Bearer JWT
Auth
POST /api/v1/auth/register — rejestracja użytkownika
POST /api/v1/auth/login — logowanie (zwraca access/refresh token)
POST /api/v1/auth/refresh — odświeżenie tokenu
Users
GET /api/v1/users/{id} — szczegóły użytkownika (Admin / właściciel)
Courses
GET /api/v1/courses — lista kursów (filtry: category, tags, instructorId, priceMin, priceMax, published, q=search, page, pageSize, sort)
GET /api/v1/courses/{id} — szczegóły kursu (z lekcjami)
POST /api/v1/courses — utwórz kurs (Instructor/Admin)
PUT /api/v1/courses/{id} — aktualizuj kurs (Owner/Admin)
DELETE /api/v1/courses/{id} — usuń kurs (Owner/Admin)
Lessons
POST /api/v1/courses/{id}/lessons — dodaj lekcję
PUT /api/v1/courses/{id}/lessons/{lessonId} — edytuj lekcję
Files
POST /api/v1/courses/{id}/upload — upload materiałów (multipart/form-data)
Enrollments
POST /api/v1/courses/{id}/enroll — zapisanie studenta (transakcja: Enrollment + powiadomienie)
GET /api/v1/users/{id}/enrollments — lista zapisów użytkownika
Comments
POST /api/v1/courses/{id}/comments — dodaj komentarz
Admin
GET /api/v1/admin/statistics — statystyki platformy (kursy, aktywni użytkownicy)
Standardy odpowiedzi API
Format odpowiedzi: ustandaryzowany JSON
Kody błędów:
400 — Bad Request
401 — Unauthorized
403 — Forbidden
404 — Not Found
409 — Conflict
500 — Internal Server Error

Create Backend in .NET 10 and in Rider, FrontEnd create in VS Code using simple syntaxis and language, don't add too much of nessecary things, it's a project for school and i need to know how it work, simply
