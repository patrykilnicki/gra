---
project: TribalWars
context_type: greenfield
created: 2026-09-24
updated: 2026-09-24
product_type: web-app
target_scale:
  users: small
  qps: low
  data_volume: small
timeline_budget:
  mvp_weeks: 3
  hard_deadline: 2026-12-06
  after_hours_only: true
seed: >
  Gra przeglądarkowa w stylu Plemion bez snajpienia czasem: rozwój wioski,
  surowce, budynki, walka w oknach czasowych i doktryna obronna offline.
  Multiplayer od startu, ale MVP to mały świat 2+ graczy bez
  plemion/frontów/regionów/pełnego rynku. Cel: mały projekt pod podstawy,
  termin 6.12.2026. Otwarte: ile jednostek/budynków w MVP i czy rynek w ogóle
  wchodzi.
checkpoint:
  current_phase: 8
  phases_completed: [1, 2, 3, 4, 5, 6, 7]
  gray_areas_resolved:
    - topic: pain category
      decision: missing market capability — no browser strategy where outcome depends on plan, not constant online presence
    - topic: insight
      decision: pre-set strategy and doctrine beat time-sniping; activity gives more decisions, not click-frequency advantage
    - topic: primary persona
      decision: hobbyist browser-strategy / Tribal-Wars-style players
    - topic: council in MVP
      decision: council = ruler settings (doctrine, priorities), not separate players on roles
    - topic: auth strategy
      decision: login (email + password / OAuth / passwordless); flat model — each logged-in player is ruler of their own village
    - topic: MVP first flow
      decision: >-
        login → village → one queue (build OR train) → defense doctrine →
        hardcoded single opponent village (no target list) → attack resolved in time window;
        secondary = simple map; guardrails = login + village view + one queue stay working;
        3 weeks after-hours for first flow; hard deadline 2026-12-06 after-hours
    - topic: FR-005 target selection
      decision: Socrates 5a — drop target list; hardcoded one opponent for MVP
    - topic: business logic & NFRs
      decision: >-
        battle resolved by math+probability from troop counts/types (counters), morale, doctrine;
        inputs = troop counts + doctrine; output = war report with fog if no survivors return;
        NFRs = privacy of enemy composition/doctrine pre-battle, no second-sniping, report after battle ends
    - topic: product framing
      decision: web-app; small scale (handful); at 100x domain rule unchanged; MVP skirmishes 1:1; deadline 2026-12-06; after-hours
    - topic: non-goals
      decision: all listed scope avoids accepted
    - topic: project name
      decision: TribalWars
  frs_drafted: 7
  quality_check_status: accepted
---

# Shape notes

## Seed

Gra przeglądarkowa w stylu Plemion bez snajpienia czasem: rozwój wioski, surowce, budynki, walka w oknach czasowych i doktryna obronna offline. Multiplayer od startu, ale MVP to mały świat 2+ graczy bez plemion/frontów/regionów/pełnego rynku. Cel: mały projekt pod podstawy, termin 6.12.2026. Otwarte: ile jednostek/budynków w MVP i czy rynek w ogóle wchodzi.

## Vision & Problem Statement

Na rynku brakuje strategii browserowej, w której w wolnym czasie (wieczór po pracy, weekend) da się rywalizować planem rozwoju i wojny, a nie ciągłym siedzeniem online. Istniejące gry w stylu Plemion wymagają stałej obecności, żeby coś osiągnąć.

Insight: wynik ma zależeć od wcześniej ustalonej strategii i doktryny oraz od jakości decyzji, a nie od snajpienia czasem ani przewagi z częstszego klikania. Gracz wciela się we władcę wioski; „rada” w MVP to reguły i ustawienia władcy (doktryna, priorytety), nie osobni gracze na stanowiskach.

Skala: przy 100× większej liczbie graczy reguła bitwy się nie zmienia; w MVP potyczki są 1:1.

## User & Persona

**Primary persona:** Gracz strategii browserowych / w stylu Plemion. W wolnym czasie chce poczuć się władcą osady — planować rozwój i przebieg wojny — bez obowiązku ciągłego reagowania co do sekundy.

## Access Control

Login (email + hasło / OAuth / passwordless). Model płaski: każdy zalogowany gracz = władca swojej wioski. Brak osobnych ról admin/gość w MVP.

## Success Criteria

### Primary
- Zalogowany gracz widzi swoją wioskę, ma jedną kolejkę (budowa *albo* szkolenie), ustawia doktrynę obronną i przeprowadza atak na hardcodowanego przeciwnika (jedna wioska przeciwnika), rozstrzygnięty w oknie czasowym — z uwzględnieniem doktryny broniącego.

### Secondary
- Prosta mapa.

### Guardrails
- Logowanie, widok wioski i jedna kolejka muszą nadal działać.

### Timeline note
- Pierwszy przepływ: ~3 tygodnie po godzinach (`mvp_weeks: 3`).
- Twardy deadline oddania: 2026-12-06; praca po godzinach.

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

## Business Logic

Aplikacja rozstrzyga bitwę matematyką i prawdopodobieństwem, biorąc pod uwagę ilość i rodzaj wojsk (różne współczynniki ataku/obrony i kontry, np. pikinierzy vs kawaleria), morale oraz doktrynę określającą zachowanie wojsk w walce.

Wejścia: gracz wybiera ilość wojsk oraz doktrynę.

Wyjście: raport wojenny; m.in. jeśli nie wróci żaden żołnierz, nie widać ile jednostek miał przeciwnik.

Gracz napotyka regułę w sesji multiplayer jako raporty po potyczce.

## Non-Functional Requirements

- Skład wojsk i doktryna przeciwnika nie są ujawniane przed bitwą (poza tym, co wynika z raportu po walce, w tym ograniczenie przy braku ocalałych).
- Wynik bitwy nie zależy od precyzyjnej sekundy kliknięcia — rozstrzygnięcie w oknie czasowym.
- Raport wojenny pojawia się po zakończonej bitwie (po domknięciu okna rozstrzygnięcia), w czasie odczuwalnym jako „od razu po walce”, nie jako wielogodzinne opóźnienie.

## Non-Goals

- Plemiona, fronty i walka o regiony — poza MVP; nacisk na mały świat 1:1.
- Pełny rynek / kupcy — poza MVP.
- Lista celów i pełna mapa strategiczna jako must-have — mapa tylko nice-to-have; cel ataku hardcodowany.
- Rada jako osobni gracze na stanowiskach — w MVP tylko ustawienia władcy.
- Walki wielu stron naraz — w MVP tylko potyczki 1:1.
- Snajpienie sekundą / tryb wymagający ciągłego online — sprzeczne z wizją okien czasowych i doktryny.

## Open Questions

- Ile typów jednostek i budynków wchodzi w MVP (jedna kolejka: budowa *albo* szkolenie)?
- Dokładna długość / definicja okna czasowego bitwy.
- Jakie konkretne opcje ma doktryna w MVP?

## Quality cross-check

- Access Control: present
- Business Logic: present (one-sentence rule)
- Project artifacts: present (`shape-notes.md` + checkpoint)
- Timeline-cost ack: present (`mvp_weeks: 3`)
- Non-Goals: present
- Preserved behavior: n/a (greenfield)
- Result: accepted (no checklist gaps). Project name set to TribalWars.
