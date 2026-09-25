# Kryteria odbiorcze

Odbiór jest powtarzalną sesją porównującą artefakt z zapisanym wynikiem oczekiwanym. Ocena „wygląda dobrze” może być uwagą ekspercką, lecz nie zastępuje identyfikatora testu, dowodu i decyzji. Sesję wykonuje się na kopii zatwierdzonego kandydata, bez poprawiania artefaktu w trakcie testów.

## Przygotowanie sesji

1. Nadaj sesji niezmienny `session_id`, wskaż wersję kandydata, commit narzędzi, profil eksportu i środowisko docelowe.
2. Zapisz system operacyjny, CPU, GPU, sterownik, wersje DCC/silnika, rozdzielczość, jakość renderowania i limit klatek na sekundę.
3. Ustal osobę wykonującą test i osobę zatwierdzającą wyjątki. Jedna osoba nie zatwierdza samodzielnie własnego wyjątku.
4. Skopiuj pozy, animacje, kamerę, oświetlenie i ustawienia testowe z wersjonowanego zestawu. Wyłącz losowość albo zapisz ziarno.
5. Zarezerwuj katalog dowodów, sprawdź wolne miejsce i uruchom krótki test nagrywania. Materiał biometryczny pozostaje poza publicznym repozytorium.

## Wymagane artefakty

- edytowalna scena wzorcowa oraz eksport-kandydat z sumami SHA-256;
- manifest zależności: wersja siatki, szkieletu, materiałów, tekstur, animacji i profil eksportu;
- zatwierdzony zestaw pomiarów oraz neutralne materiały referencyjne;
- wersjonowany zestaw póz, klipów mowy i animacji testowych;
- budowa środowiska docelowego i konfiguracja pomiaru wydajności;
- raport geometrii, log importu/eksportu i rozpoczęty raport odbiorczy;
- katalog dowodów z prawem zapisu, lecz bez poprzednich wyników pod tym samym `session_id`.

Brak któregokolwiek wejścia wymaganego przez dany test oznacza, że test nie został wykonany; nie zapisuj wtedy `passed` ani `failed`.

## Uruchamianie testów

W każdym teście najpierw wczytaj kandydata bez ręcznych poprawek, zastosuj wskazaną scenę/klip, wykonaj pełny zakres ruchu w obie strony, a potem zapisz wynik liczbowy i dowód. Dodatkowe kontrole siatki opisuje [walidacja geometrii](geometry.md).

| ID | Sposób uruchomienia | Oczekiwany wynik i dowód |
| --- | --- | --- |
| `GEO-001` | W widoku ortograficznym porównaj obwiednię kandydata z zatwierdzonym wymiarem i jednostkami sceny. | Różnica skali nie przekracza ±0,5%; raport wymiarów i zrzut miarki. |
| `GEO-002` | Uruchom kontrolę manifold, powierzchni zerowych i normalnych na każdym obiekcie renderowanym. | Brak błędów blokujących; log kontroli i zrzuty oznaczonych elementów. |
| `BODY-001` | Odtwórz z profilu zgięcie każdego łokcia od 0° do 130° i zatrzymaj 0°, 45°, 90° i 130°. | Brak utraty objętości i ostrych fałd; klip oraz cztery kadry na stronę. |
| `BODY-002` | Unieś każde ramię w płaszczyźnie bocznej od 0° do zakresu rigu, obserwując przód, bok i tył. | Zachowana objętość barku, brak zapadnięcia pachy; klip z trzech kamer. |
| `HAND-001` | Z neutralnej dłoni animuj pełną pięść osobno po obu stronach i sprawdź ją w zwolnieniu. | Brak krytycznego przenikania i załamania stawów; klip i kadry wnętrza dłoni. |
| `HAND-002` | Uruchom chwyt szczypcowy kciuka z palcem wskazującym dla obu dłoni. | Opuszki osiągają kontakt bez widocznego przenikania; zbliżenie z boku i od przodu. |
| `FACE-001` | Wykonaj powolne mrugnięcie lewe, prawe i obustronne, z kamerą frontalną i boczną. | Pełne domknięcie bez penetracji gałki ocznej; klip i klatka maksymalnego domknięcia. |
| `FACE-002` | Odtwórz otwarcie żuchwy od neutralu do maksimum i z powrotem. | Żuchwa, dolne zęby i język zachowują relację; klip przekroju/półprofilu. |
| `FACE-003` | Uruchom uśmiech kolejno po lewej i prawej stronie, potem obustronnie. | Strony są sterowane niezależnie bez niezamierzonego ruchu strony przeciwnej; klip frontalny. |
| `EYE-001` | Skieruj każde oko osobno na znaczniki skrajne, a następnie wykonaj wspólny ruch spojrzenia. | Oczy reagują niezależnie, a powieki poprawnie podążają; klip frontalny i boczny. |
| `SPEECH-001` | Odtwórz nagranie `/pa ta ka/` z widocznym przebiegiem fali dźwiękowej i markerami fonemów. | Zwarcia są rozróżnialne i zsynchronizowane; klip z dźwiękiem i zrzut osi czasu. |
| `SPEECH-002` | Odtwórz zatwierdzone zdanie w tempie nominalnym oraz 0,5×. | Brak skokowych przejść między kształtami ust; oba klipy i wykres krzywych. |
| `HAIR-001` | Obróć głowę do granic rigu przy włączonej kolizji włosów. | Brak trwałej penetracji włosów w głowę; klip ze znacznikiem klatek błędnych. |
| `CLOTH-001` | Odtwórz wersjonowane klipy przysiadu i chodu od przodu, boku i tyłu. | Brak krytycznych penetracji lub niestabilności; klipy oraz kadry najgorszego przypadku. |
| `RUNTIME-001` | Uruchom build docelowy, rozgrzej scenę przez 60 s, a następnie rejestruj co najmniej 120 s tą samą kamerą. | Mediana i 1% low mieszczą się w zatwierdzonym budżecie; surowy ślad profilera i podsumowanie. |

Próg wydajności musi być wpisany do profilu przed testem. Nie wolno dobierać go po obejrzeniu wyniku.

## Struktura dowodów i raportu

Każdy plik ma stabilną ścieżkę względną i nie jest zastępowany po podpisaniu raportu:

```text
acceptance/<session_id>/
├── report.json
├── environment.json
├── manifests/
├── logs/
└── evidence/<test_id>/
    ├── result.json
    ├── overview.mp4
    └── frame-000123.png
```

Rekord testu zawiera co najmniej: `test_id`, `status`, wersję procedury, operatora, czas rozpoczęcia i zakończenia, identyfikator artefaktu oraz jego SHA-256, parametry wejściowe, obserwację, oczekiwany wynik, metryki z jednostkami, ścieżki dowodów i ich SHA-256. Dla wyjątku wymagane są dodatkowo właściciel ryzyka, uzasadnienie, zakres, termin ważności i zatwierdzający.

## Znaczenie statusów

- `passed` — procedurę wykonano w całości na wskazanym artefakcie, wynik spełnia wszystkie progi, a kompletne dowody są czytelne.
- `failed` — procedurę wykonano, lecz co najmniej jeden wynik nie spełnia progu albo dowód ujawnia błąd blokujący. Brak dowodu także uniemożliwia zaliczenie i musi zostać opisany.
- `accepted_exception` — znane odstępstwo od progu zostało świadomie zaakceptowane przez uprawnioną osobę dla określonej wersji, platformy i czasu. Nie oznacza ono `passed`, nie ukrywa błędu i wygasa przy zmianie zależnego artefaktu lub w zapisanym terminie.

Status sesji jest `failed`, jeśli którykolwiek wymagany test ma `failed`, niezatwierdzony wyjątek albo nie został wykonany. Sesja może zostać wydana z `accepted_exception` wyłącznie wtedy, gdy polityka wydania dopuszcza wyjątki i wszystkie takie rekordy mają kompletne zatwierdzenie.

## Ponowny test

Po naprawie utwórz nowy `session_id`; nie edytuj historycznego wyniku. Powtórz test, który nie przeszedł, oraz wszystkie testy zależne od zmienionego artefaktu. Zmiana topologii wymaga co najmniej ponowienia geometrii, deformacji, twarzy, włosów, odzieży i wydajności; zmiana wyłącznie progu wymaga pełnej nowej sesji według nowej wersji procedury. Nowy rekord wskazuje `supersedes` i identyfikator poprawki, ale zachowuje stare dowody. Wyjątek nie przechodzi automatycznie na ponowny test.

## Minimalny kompletny raport

Poniższy skrócony przykład przedstawia kompletną sesję obejmującą jeden wymagany test; rzeczywisty raport zawiera rekord dla każdego testu wymaganego przez profil:

```json
{
  "schema": "avatar-studio-acceptance-report-v1",
  "session_id": "acceptance-2026-09-25-001",
  "candidate": {
    "id": "avatar-runtime-v042",
    "sha256": "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8"
  },
  "procedure_version": "v001",
  "environment": "environment.json",
  "started_at": "2026-09-25T10:00:00Z",
  "completed_at": "2026-09-25T10:04:12Z",
  "operator": "reviewer-02",
  "required_tests": ["GEO-001"],
  "results": [
    {
      "test_id": "GEO-001",
      "status": "passed",
      "artifact_sha256": "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8",
      "started_at": "2026-09-25T10:01:00Z",
      "completed_at": "2026-09-25T10:02:00Z",
      "operator": "reviewer-02",
      "inputs": {"reference_height_mm": 1750.0},
      "expected": "absolute_scale_error_percent <= 0.5",
      "observed": "Height 1746.5 mm; absolute error 0.2%.",
      "metrics": {"absolute_scale_error_percent": 0.2},
      "evidence": [
        {
          "path": "evidence/GEO-001/frame-000001.png",
          "sha256": "3a7bd3e2360a3d80d4f859f704e4b8f9f1a6bbad2fcbd7c601f46f601b5f1f00"
        }
      ]
    }
  ],
  "overall_status": "passed",
  "approved_by": "lead-reviewer-01"
}
```
