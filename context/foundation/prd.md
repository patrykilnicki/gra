---
project: TribalWars
version: 1
status: draft
created: 2026-09-24
context_type: greenfield
product_type: web-app
target_scale:
  users: small
  qps: low
  data_volume: small
timeline_budget:
  mvp_weeks: 3
  hard_deadline: 2026-12-06
  after_hours_only: true
---

# TribalWars

## Vision & Problem Statement

Na rynku brakuje strategii browserowej, w której w wolnym czasie (wieczór po pracy, weekend) da się rywalizować planem rozwoju i wojny, a nie ciągłym siedzeniem online. Istniejące gry w stylu Plemion wymagają stałej obecności, żeby coś osiągnąć.

Insight: wynik ma zależeć od wcześniej ustalonej strategii i doktryny oraz od jakości decyzji, a nie od snajpienia czasem ani przewagi z częstszego klikania. Gracz wciela się we władcę wioski; „rada” w MVP to reguły i ustawienia władcy (doktryna, priorytety), nie osobni gracze na stanowiskach. Przy 100× większej liczbie graczy reguła bitwy się nie zmienia; w MVP potyczki są 1:1.

## User & Persona

**Primary persona:** Gracz strategii browserowych / w stylu Plemion. W wolnym czasie chce poczuć się władcą osady — planować rozwój i przebieg wojny — bez obowiązku ciągłego reagowania co do sekundy.

## Success Criteria

### Primary
- Zalogowany gracz widzi swoją wioskę, ma jedną kolejkę (budowa *albo* szkolenie), ustawia doktrynę obronną i przeprowadza atak na hardcodowanego przeciwnika (jedna wioska przeciwnika), rozstrzygnięty w oknie czasowym — z uwzględnieniem doktryny broniącego.

### Secondary
- Prosta mapa.

### Guardrails
- Logowanie, widok wioski i jedna kolejka muszą nadal działać.

## User Stories

### US-01: Atak w oknie czasowym z doktryną obronną

- **Given** dwóch zalogowanych graczy, każdy ze swoją wioską; atakujący ma jednostkę gotową do wysłania (z kolejki szkolenia lub już dostępną); broniący ma ustawioną doktrynę obronną; wioska przeciwnika jest z góry znana (hardcodowany cel)
- **When** atakujący zleca atak na tę wioskę przeciwnika
- **Then** atak nie rozstrzyga się co do sekundy, tylko w oknie czasowym, a wynik uwzględnia doktrynę broniącego

#### Acceptance Criteria
- Rozstrzygnięcie bitwy następuje w zdefiniowanym oknie czasowym, nie na precyzyjnej sekundzie zlecenia
- Doktryna obronna broniącego wpływa na wynik
- Atakujący i broniący widzą skutek bitwy po rozstrzygnięciu
- Cel ataku w MVP to hardcodowana wioska przeciwnika (bez wyboru z listy)

## Functional Requirements

### Authentication
- FR-001: Gracz can zalogować się. Priority: must-have
  > Socrates: Counter-argument considered: local profile / link / no account. Resolution: kept as written.

### Village
- FR-002: Gracz can zobaczyć swoją wioskę. Priority: must-have
  > Socrates: Counter-argument considered: raw numbers panel enough / village view too empty. Resolution: kept as written.

- FR-003: Gracz can dodać pozycję do jednej kolejki (budowa albo szkolenie). Priority: must-have
  > Socrates: Counter-argument considered: skip queue / both queues / only one type. Resolution: kept as written.

### Doctrine & combat
- FR-004: Gracz can ustawić doktrynę obronną. Priority: must-have
  > Socrates: Counter-argument considered: battle first without doctrine / empty without options. Resolution: kept as written.

- FR-005: Gracz can zlecić atak na hardcodowaną wioskę przeciwnika (bez listy celów). Priority: must-have
  > Socrates: Counter-argument considered: "lista celów zbędna — wystarczy hardcodowany jeden przeciwnik." Resolution: revised; target list dropped for MVP.

- FR-006: Gracz can zlecić atak rozstrzygany w oknie czasowym. Priority: must-have
  > Socrates: Counter-argument considered: instant battle first / window rules unclear. Resolution: kept as written.

### Map (secondary)
- FR-007: Gracz can zobaczyć prostą mapę. Priority: nice-to-have
  > Socrates: Counter-argument considered: drop entirely / promote to must-have. Resolution: kept as nice-to-have.

## Non-Functional Requirements

- Skład wojsk i doktryna przeciwnika nie są ujawniane przed bitwą (poza tym, co wynika z raportu po walce, w tym ograniczenie przy braku ocalałych).
- Wynik bitwy nie zależy od precyzyjnej sekundy kliknięcia — rozstrzygnięcie w oknie czasowym.
- Raport wojenny pojawia się po zakończonej bitwie (po domknięciu okna rozstrzygnięcia), w czasie odczuwalnym jako „od razu po walce”, nie jako wielogodzinne opóźnienie.

## Business Logic

Aplikacja rozstrzyga bitwę matematyką i prawdopodobieństwem, biorąc pod uwagę ilość i rodzaj wojsk (różne współczynniki ataku/obrony i kontry, np. pikinierzy vs kawaleria), morale oraz doktrynę określającą zachowanie wojsk w walce.

Wejścia: gracz wybiera ilość wojsk oraz doktrynę.

Wyjście: raport wojenny; m.in. jeśli nie wróci żaden żołnierz, nie widać ile jednostek miał przeciwnik.

Gracz napotyka regułę w sesji multiplayer jako raporty po potyczce.

## Access Control

Login (email + hasło / OAuth / passwordless). Model płaski: każdy zalogowany gracz = władca swojej wioski. Brak osobnych ról admin/gość w MVP.

## Non-Goals

- Plemiona, fronty i walka o regiony — poza MVP; nacisk na mały świat 1:1.
- Pełny rynek / kupcy — poza MVP.
- Lista celów i pełna mapa strategiczna jako must-have — mapa tylko nice-to-have; cel ataku hardcodowany.
- Rada jako osobni gracze na stanowiskach — w MVP tylko ustawienia władcy.
- Walki wielu stron naraz — w MVP tylko potyczki 1:1.
- Snajpienie sekundą / tryb wymagający ciągłego online — sprzeczne z wizją okien czasowych i doktryny.

## Open Questions

1. **Ile typów jednostek i budynków wchodzi w MVP (jedna kolejka: budowa *albo* szkolenie)?** — TBD by user. Owner: user.
2. **Dokładna długość / definicja okna czasowego bitwy.** — TBD by user. Owner: user.
3. **Jakie konkretne opcje ma doktryna w MVP?** — TBD by user. Owner: user.
