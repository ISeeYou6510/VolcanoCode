# Discord Control System

## Opis projektu
Projekt jest **systemem zarządzania serwerem Discord**, składającym się z trzech ściśle rozdzielonych elementów:

1. **Bot Discord** – thin client, wykonawca poleceń  
2. **Aplikacja sterująca (PyQt6)** – główny panel administracyjny  
3. **Warstwa współdzielona (`shared/`)** – jedyne źródło prawdy konfiguracyjnej  

Celem projektu jest zastąpienie wielu botów jednym, spójnym systemem,
zarządzanym centralnie i odpornym na restart oraz zmiany konfiguracji.

---

## Założenia architektoniczne

- Bot **nie podejmuje decyzji** – tylko je wykonuje
- Logika konfiguracyjna znajduje się poza botem
- Zmiany zachowania realizowane są przez edycję konfiguracji
- Jeden styl, jeden system paneli, jedna baza danych
- Projekt przygotowany pod:
  - użycie na wielu serwerach
  - dalszą rozbudowę
  - potencjalną komercjalizację

---

## Struktura projektu

project/
├── bot/ # Bot Discord (thin client)
├── controller_app/ # Aplikacja sterująca (PyQt6)
├── shared/ # Wspólna konfiguracja i schematy
├── data/ # Dane lokalne bota (DB, backupy)
├── docs/ # Dokumentacja techniczna
└── README.md


### bot/
- uruchamia się na serwerze (thin client)
- czyta konfigurację z `shared/`
- zapisuje wyłącznie:
  - logi
  - stan runtime
- nie modyfikuje konfiguracji

### controller_app/
- aplikacja desktopowa (PyQt6)
- edytuje pliki w `shared/`
- waliduje konfigurację
- zarządza modułami, panelami i stylem
- wykonuje commit/push do repozytorium

### shared/
- **jedyna prawda systemu**
- bot ma dostęp tylko do odczytu
- zawiera:
  - schematy paneli
  - konfigurację modułów
  - uprawnienia
  - limity
  - styl UI
- wersjonowana (`version.txt`)

---

## Uprawnienia

- **User** – brak uprawnień administracyjnych
- **Admin** – użytkownik z uprawnieniem `ADMINISTRATOR` na Discordzie

Bot:
- nie karze automatycznie
- nie podejmuje autonomicznych decyzji
- jedynie wykonuje akcje zainicjowane przez admina

---

## Panele

- Jeden panel = jedna funkcja
- Panele są:
  - stałe
  - przypisane do konkretnych kanałów
- Definicje paneli znajdują się w `shared/schemas/panels.json`
- Bot renderuje panele na podstawie schematów

---

## Moduły

- Każda funkcjonalność to osobny moduł
- Moduły:
  - można włączać i wyłączać
  - są definiowane w `shared/schemas/modules.json`
- Każdy moduł:
  - posiada własne panele
  - posiada własne tabele w bazie danych

---

## Dane i trwałość

- Lokalna baza danych: SQLite
- Baza danych:
  - istnieje wyłącznie po stronie bota
  - przechowuje logi i stan techniczny
- Bot MUSI poprawnie działać po restarcie

---

## Reload konfiguracji

- Konfiguracja przeładowywana:
  - wyłącznie przez aplikację sterującą
- Bot:
  - nie obserwuje plików
  - nie reloaduje się samodzielnie
- Niezgodna wersja `shared/` → bot nie uruchamia się

---

## Zasady pracy nad projektem

1. Nie dodajemy logiki do bota bez potrzeby
2. Nie obchodzimy `shared/`
3. Każda zmiana konfiguracji przechodzi przez walidację
4. Brak „szybkich fixów” w runtime
5. Struktura katalogów jest stała i niezmienna

---

## Status
Projekt w aktywnym rozwoju.  
Zmiany w architekturze wymagają uzgodnienia.

