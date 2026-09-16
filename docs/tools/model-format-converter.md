# Konwerter formatów modeli

`scripts/model_format_converter.py` uruchamia Blendera w trybie bez interfejsu,
importuje obsługiwany plik, sporządza inwentarz sceny, ocenia przewidywane straty i
eksportuje scenę do innego formatu. Narzędzie służy do przygotowania i kontroli
**eksportu pochodnego** przeznaczonego do wymiany danych lub uruchomienia w silniku.
Nie zastępuje ono **mastera DCC**: edytowalnej sceny źródłowej z pełną historią,
rigiem, materiałami i zależnościami. Master DCC pozostaje źródłem prawdy, a wynik
konwersji można zawsze odtworzyć i usunąć.

## Wymagania wstępne

- Python 3 uruchamia skrypt wejściowy; skrypt nie wymaga pakietu `bpy` w tym
  interpreterze, ponieważ sam ponownie uruchamia się w Blenderze.
- Blender musi być dostępny jako `blender` w `PATH`, wskazany zmienną
  `BLENDER_BIN` albo argumentem `--blender`. Na Windows skrypt dodatkowo sprawdza
  standardowe katalogi instalacyjne Blendera 4.2–4.5 i 5.0.
- Importery i eksportery odpowiednich formatów muszą być dostępne w używanej
  wersji Blendera. Zgodność materiałów i USD zależy od wersji Blendera.
- Plik wejściowy i jego zewnętrzne tekstury muszą być czytelne. Katalog wyniku i
  raportu jest tworzony automatycznie.
- Pracuj na kopii eksportu, nigdy na jedynej kopii mastera DCC. Skrypt nie przyjmuje
  plików `.blend`.

## Formaty i granice konwersji

Obsługiwane rozszerzenia wejścia i wyjścia to `.fbx`, `.gltf`, `.glb`, `.usd`,
`.usdz`, `.obj`, `.ply` i `.stl` (wielkość liter nie ma znaczenia). Skrypt dopuszcza
każdą parę tych rozszerzeń, także tę samą na wejściu i wyjściu. Rzeczywista granica
pary jest przecięciem tego, co importer Blendera odczytał ze źródła, i możliwości
formatu docelowego.

Ograniczenie po stronie wejścia jest nieodwracalne: konwerter nie odtwarza danych,
których plik źródłowy nie potrafił zapisać albo których importer Blendera nie
odczytał. Dlatego przed wybraniem celu ustal najpierw górną granicę danych wejścia:

| Format wejściowy | Dane, które mogą wejść do konwersji | Ograniczenie odziedziczone przez każdą parę z tym wejściem |
| --- | --- | --- |
| FBX | Geometria, UV, materiały, tekstury, szkielet, skinning, shape keys i animacje | Semantyka shaderów, jednostek, osi, kości i klipów jest już interpretacją importera FBX Blendera. |
| glTF / GLB | Geometria, UV, materiały glTF, tekstury, skiny, morph targets i animacje | Konwerter nie odzyska węzłów shaderów ani danych DCC spoza modelu glTF; zewnętrzne zasoby `.gltf` muszą być dostępne. |
| USD / USDZ | Geometria, UV, materiały, tekstury, szkielety, deformacje i animacje obsłużone przez importer USD | Warstwy, warianty, instancje i shadery mogą zostać spłaszczone już podczas importu do sceny Blendera. |
| OBJ | Statyczna geometria, UV oraz proste materiały i odwołania do tekstur z MTL | Żaden format wynikowy nie odzyska skeletonu, skinningu, morph targets ani animacji z OBJ. |
| PLY | Statyczna geometria, UV i ewentualne atrybuty kolorów | Żaden format wynikowy nie odzyska materiałów, tekstur ani danych postaci z PLY. |
| STL | Statyczna geometria | Żaden format wynikowy nie odzyska UV, materiałów, tekstur ani danych postaci z STL. |

Następnie zastosuj ograniczenie formatu wyjściowego:

| Format docelowy | Geometria / UV | Materiały / tekstury | Szkielet / skinning | Morph targets | Animacje | Najważniejsze ograniczenie tej pary |
| --- | --- | --- | --- | --- | --- | --- |
| FBX | tak / tak | tak / tak | tak / tak | tak | tak | Mimo deklarowanej możliwości mapowanie shaderów, osi, jednostek i animacji zależy od importera/eksportera FBX. |
| glTF / GLB | tak / tak | tak / tak | tak / tak | tak | tak | Materiały są redukowane do modelu glTF; `.gltf` może utworzyć pliki zewnętrzne, a `.glb` jest kontenerem binarnym. |
| USD / USDZ | tak / tak | tak / tak | tak / tak | tak | tak | Eksporter może użyć awaryjnego, uboższego wywołania, jeśli bieżący Blender nie zna żądanych opcji; konieczny jest ponowny import. |
| OBJ | tak / tak | tak / tak | nie / nie | nie | nie | Tylko statyczny mesh; wynik otrzymuje towarzyszący `.mtl`. |
| PLY | tak / tak | nie / nie | nie / nie | nie | nie | Tylko statyczny mesh; może zachować atrybuty kolorów, ale raport nie traktuje ich jako materiałów. |
| STL | tak / nie | nie / nie | nie / nie | nie | nie | Wyłącznie geometria statyczna; brak UV i danych postaci. |

Kolumna „tak” opisuje model możliwości używany przez skrypt, a nie gwarancję
bezstratnej konwersji. Przykładowo para OBJ → FBX nie odzyska szkieletu, którego OBJ
nie zawierał, natomiast FBX → OBJ zawsze traci wykryty rig, skinning, morph targets
i animacje. Opcja `--strict` blokuje tylko straty wykryte przez ten model.

## Składnia CLI

```text
python scripts/model_format_converter.py INPUT OUTPUT
    [--textures {auto,embed,copy,skip}]
    [--texture-format {auto,png,jpeg}]
    [--max-texture-size PIXELS]
    [--animations {auto,keep,strip}]
    [--apply-transforms]
    [--report PATH]
    [--blender PATH]
    [--strict]
```

| Argument | Znaczenie |
| --- | --- |
| `INPUT` | Istniejący plik wejściowy w jednym z obsługiwanych formatów. |
| `OUTPUT` | Ścieżka i format wyniku, wybrane przez rozszerzenie. |
| `--textures auto` | Wartość domyślna; zachowuje węzły tekstur i pozwala eksporterowi dobrać sposób zapisu. Dla FBX używa trybu kopiowania bez wymuszenia osadzania. |
| `--textures embed` | Pakuje obrazy w scenie i prosi eksporter FBX o osadzenie. Nie oznacza, że każdy format potrafi osadzić każdy obraz. |
| `--textures copy` | Zachowuje tekstury i dla FBX wybiera kopiowanie. Dla pozostałych formatów nie uruchamia osobnego mechanizmu kopiowania. |
| `--textures skip` | Usuwa węzły obrazów przed eksportem; wykryte tekstury pojawią się w `losses`. |
| `--texture-format auto\|png\|jpeg` | Pozostawia format obrazu albo zapisuje niepakowane, istniejące obrazy jako PNG/JPEG w katalogu `converted_textures`. Nie konwertuje obrazów pakowanych ani brakujących. |
| `--max-texture-size PIXELS` | Proporcjonalnie zmniejsza obrazy, których dłuższy bok przekracza limit. Minimum to 64; nie powiększa obrazów. |
| `--animations auto\|keep\|strip` | `auto` i `keep` zachowują eksport animacji; `strip` usuwa dane animacji i akcje przed eksportem. |
| `--apply-transforms` | Stosuje obrót i skalę wszystkich obiektów przed eksportem; nie stosuje położenia. Używaj dopiero po sprawdzeniu riggu. |
| `--report PATH` | Zastępuje domyślną ścieżkę raportu `<OUTPUT>.conversion.json`. |
| `--blender PATH` | Nazwa lub ścieżka do pliku wykonywalnego Blender, sprawdzana jako pierwsza. Jeśli nie wskazuje istniejącego pliku, skrypt nadal próbuje `BLENDER_BIN`, `PATH` i — na Windows — znanych katalogów instalacyjnych. |
| `--strict` | Przerywa przed eksportem, jeśli analiza wykryje stratę cechy lub jawne `strip`/`skip`. |

Skrypt można również uruchomić bezpośrednio przez Blender; argumenty skryptu muszą
wtedy wystąpić po separatorze `--`:

```text
blender --background --python scripts/model_format_converter.py -- INPUT OUTPUT [OPCJE]
```

### Kody zakończenia

| Kod | Znaczenie |
| --- | --- |
| `0` | Proces Blendera zwrócił powodzenie. Potwierdź je istnieniem wyniku i raportu. |
| `2` | `argparse` odrzucił składnię lub wartość wyboru (gdy błąd jest obsługiwany bezpośrednio przez interpreter uruchamiający skrypt). |
| inny niż `0` | Interpreter, Blender albo skrypt uruchomiony poza Blenderem zgłosił błąd. Przy uruchomieniu przez Python zwracany jest kod procesu Blendera. |

Polecenie uruchamiające Blender nie dodaje `--python-exit-code`. Zależnie od wersji
Blendera nieobsłużony wyjątek skryptu, np. brak pliku, niedostępny operator, limit
poniżej 64 lub strata przy `--strict`, może nie zostać wiarygodnie odwzorowany na
niezerowy kod procesu. Dlatego sukces zawsze potwierdzaj trzema warunkami: kodem
`0`, istnieniem `OUTPUT` i istnieniem poprawnego raportu JSON. Komunikat
`Gotowe:` jest dodatkowym potwierdzeniem dojścia do końca skryptu.

### Generowane pliki

- `OUTPUT` — główny wynik konwersji;
- `<OUTPUT>.conversion.json` — raport domyślny, np.
  `avatar.glb.conversion.json`, albo plik wskazany przez `--report`;
- przy OBJ — plik `.mtl` obok OBJ, wpisany także do `companion_files`;
- przy oddzielnym glTF — zasoby tworzone przez eksporter Blendera obok `.gltf`, w
  tym dane binarne i ewentualne obrazy; nie są one wpisywane do `companion_files`;
- przy zmianie formatu tekstur — katalog `converted_textures` względem katalogu
  bazowego Blendera; dokładne ścieżki zapisuje `processed_textures[].filepath`.

## Kompletna procedura — Windows

**Dane wejściowe:** `exports\avatar_v012.fbx` oraz wszystkie używane przez niego
tekstury. **Oczekiwany wynik:** GLB i raport w `exports\review`.

1. Utwórz kopię wejścia i katalog wyniku:

    ```powershell
    New-Item -ItemType Directory -Force work\conversion, exports\review | Out-Null
    Copy-Item exports\avatar_v012.fbx work\conversion\avatar_v012_source.fbx
    ```

2. Jeżeli `blender.exe` jest w `PATH`, wykonaj konwersję:

    ```powershell
    python scripts\model_format_converter.py `
      work\conversion\avatar_v012_source.fbx exports\review\avatar_v012.glb `
      --textures embed --animations keep --strict
    ```

   Jeżeli Blender nie jest w `PATH`, podaj pełną ścieżkę (cudzysłowy są wymagane
   ze względu na spacje):

    ```powershell
    python scripts\model_format_converter.py `
      work\conversion\avatar_v012_source.fbx exports\review\avatar_v012.glb `
      --textures embed --animations keep --strict `
      --blender "C:\Program Files\Blender Foundation\Blender 4.5\blender.exe"
    ```

3. Odczytaj raport i sprawdź wynik procesu oraz oba pliki:

    ```powershell
    if ($LASTEXITCODE -ne 0) { throw "Konwersja nie powiodła się: $LASTEXITCODE" }
    if (-not (Test-Path exports\review\avatar_v012.glb)) { throw "Brak wyniku" }
    if (-not (Test-Path exports\review\avatar_v012.glb.conversion.json)) { throw "Brak raportu" }
    Get-Content exports\review\avatar_v012.glb.conversion.json -Raw |
      ConvertFrom-Json | Format-List
    ```

4. W Blenderze wybierz **File → Import → glTF 2.0**, zaimportuj
   `exports\review\avatar_v012.glb` do pustej sceny i wykonaj checklistę akceptacyjną.
   W razie błędu zachowaj master, usuń pochodny wynik, popraw eksport źródłowy i
   powtórz procedurę na nowej kopii.

## Kompletna procedura — Linux

**Dane wejściowe:** `exports/avatar_v012.fbx` i jego tekstury. **Oczekiwany wynik:**
GLB i raport w `exports/review`.

1. Utwórz kopię wejścia i katalog wyniku:

    ```bash
    mkdir -p work/conversion exports/review
    cp exports/avatar_v012.fbx work/conversion/avatar_v012_source.fbx
    ```

2. Gdy `blender` jest w `PATH`, wykonaj:

    ```bash
    python scripts/model_format_converter.py \
      work/conversion/avatar_v012_source.fbx exports/review/avatar_v012.glb \
      --textures embed --animations keep --strict
    ```

   W przeciwnym razie wskaż pełną ścieżkę:

    ```bash
    python scripts/model_format_converter.py \
      work/conversion/avatar_v012_source.fbx exports/review/avatar_v012.glb \
      --textures embed --animations keep --strict \
      --blender /opt/blender/blender
    ```

3. Sprawdź kod, oba pliki i raport:

    ```bash
    conversion_status=$?
    test "$conversion_status" -eq 0
    test -f exports/review/avatar_v012.glb
    test -f exports/review/avatar_v012.glb.conversion.json
    python -m json.tool exports/review/avatar_v012.glb.conversion.json
    ```

4. W Blenderze wybierz **File → Import → glTF 2.0**, zaimportuj wynik do pustej
   sceny i wykonaj checklistę. W razie błędu wróć do mastera, usuń eksport pochodny
   i ponów pracę na świeżej kopii.

## Jak czytać raport

Inwentarz `scene` jest mierzony **po imporcie, lecz przed** `--apply-transforms`,
usuwaniem animacji i przetwarzaniem tekstur. Dlatego opisuje dane wykryte w źródle,
a nie ponownie zaimportowany wynik.

| Pole JSON | Interpretacja |
| --- | --- |
| `producer`, `input`, `output`, `input_format`, `output_format` | Producent raportu, bezwzględne ścieżki i rozpoznane rozszerzenia. |
| `scene.objects`, `meshes`, `vertices`, `polygons`, `uv_layers`, `color_attributes` | Liczniki sceny i geometrii odczytanej przez Blender. |
| `scene.armatures` | Liczba obiektów szkieletu (armature); raport nie podaje liczby ani nazw kości i nie porównuje hierarchii. |
| `scene.skinned_meshes` | Meshe z modyfikatorem armature **lub dowolnymi** grupami wierzchołków; jest to wskaźnik, nie walidacja wag skinningu. |
| `scene.shape_keys` | Liczba kluczy kształtu bez klucza bazowego; odpowiada kontrolowanemu tu pojęciu morph targets / blend shapes. Nazwy i wartości nie są raportowane. |
| `scene.actions` | Liczba akcji Blendera, używana jako wskaźnik animacji; raport nie podaje klipów, zakresów ani krzywych. |
| `scene.materials`, `scene.textures` | Liczba materiałów przypiętych do meshy i unikalnych obrazów znalezionych w ich węzłach `TEX_IMAGE`. Proceduralne shadery nie są teksturami w tym liczniku. |
| `output_capabilities` | Deklarowane możliwości formatu wyniku: m.in. `armature`, `skinning`, `shape_keys`, `animation`, `materials` i `textures`. Nie jest to wynik kontroli eksportu. |
| `requested` | Efektywne wartości opcji tekstur, animacji i transformacji. |
| `processed_textures` | Nazwa, rozmiar `[szerokość, wysokość]` przed/po oraz ścieżka każdego przetwarzanego obrazu. |
| `companion_files` | Pliki towarzyszące jawnie zarejestrowane przez skrypt; obecnie `.mtl` dla OBJ. |
| `losses`, `lossless_for_detected_features` | Nazwy przewidywanych strat i ich zbiorcza negacja. `true` nie dowodzi identyczności wyniku. |

**Jednostki i osie nie mają pól w obecnej wersji raportu.** Skrypt nie ustawia
jednostek, `up axis` ani `forward axis` i nie mierzy wymiarów. Należy zanotować
konwencję źródła poza raportem, a po ponownym imporcie ręcznie sprawdzić wymiary,
orientację przodu, kierunek osi pionowej oraz transformacje obiektów i kości. Nie
interpretuj `--apply-transforms` jako konwersji jednostek lub osi.

## Typowe straty według formatu wyniku

| Wynik | Typowe straty lub zmiany wymagające kontroli |
| --- | --- |
| FBX | Przepisane materiały, ścieżki tekstur, bake animacji, orientacja kości, skala i osie; model skryptu może mimo to nie zgłosić straty. |
| GLB | Redukcja shaderów do materiałów glTF, zmiana kompresji/formatu tekstur, interpretacja klipów, morph targets i jointów; zasoby są zwykle w jednym kontenerze. |
| OBJ | Utrata szkieletu, skinningu, morph targets i animacji; materiały są uproszczone do MTL, a tekstury pozostają zależnościami zewnętrznymi. |
| PLY | Utrata materiałów, tekstur, szkieletu, skinningu, morph targets i animacji; pozostaje statyczna geometria, UV i ewentualnie kolory wierzchołków. |
| USD / USDZ | Możliwe spłaszczenie materiałów i deformerów, różnice w skeletonie, shape keys, animacji, jednostkach i osiach; zachowanie zależy od operatora Blendera. |

## Diagnostyka

| Objaw | Przyczyna i działanie naprawcze |
| --- | --- |
| `Nie znaleziono Blendera` | Dodaj Blender do `PATH`, ustaw `BLENDER_BIN` albo podaj istniejący plik przez `--blender`. Na Windows ujmij ścieżkę ze spacjami w cudzysłowy; na Linux sprawdź wykonywalność pliku. |
| `Nieobsługiwany format ...` | Popraw rozszerzenie `INPUT` lub `OUTPUT` na jedno z ośmiu obsługiwanych. `.blend` nie jest wejściem tego CLI. Samo przemianowanie pliku nie zmienia formatu. |
| Model ma błędną skalę | Raport nie zawiera jednostek. Porównaj znany wymiar w masterze i po imporcie; ustaw poprawne jednostki w źródle i eksportuj ponownie. `--apply-transforms` stosuje skalę obiektu, ale nie konwertuje jednostek. |
| Model leży, jest obrócony lub patrzy w złą stronę | Raport nie zawiera osi. Sprawdź `up`/`forward`, rest pose i osie kości w aplikacji docelowej. Popraw ustawienia eksportu źródłowego; `--apply-transforms` może utrwalić obrót, lecz nie wybiera konwencji osi. |
| Brakuje tekstur | Sprawdź `scene.textures`, `processed_textures[].filepath`, istnienie plików oraz czy nie użyto `--textures skip`. Dla FBX spróbuj `embed` lub `copy`; dla `.gltf` przenieś komplet zasobów; ponownie zaimportuj wynik. |
| Brakuje morph targets | Sprawdź `scene.shape_keys`, `output_capabilities.shape_keys` i `losses`. OBJ/PLY/STL ich nie obsługują. Dla bogatego formatu sprawdź nazwy i liczbę po ponownym imporcie, ponieważ raport bada źródło, nie wynik. |
| `--strict` przerywa eksport | Odczytaj listę po `UWAGA`, wybierz format zdolny przenieść cechę albo świadomie usuń `--strict` i udokumentuj zaakceptowaną stratę. Przy przerwaniu raport nie powstaje. |

## Checklista akceptacyjna

Zgodnie z procedurą kontroli konwersji wynik można zatwierdzić dopiero po ponownym
imporcie do pustej sceny lub aplikacji docelowej:

- [ ] Master DCC pozostał osobnym, edytowalnym źródłem; wejście było kopią, a
      eksport jest oznaczony jako artefakt pochodny.
- [ ] Proces zakończył się kodem `0`; istnieją wynik, raport i wszystkie wymagane
      pliki towarzyszące.
- [ ] `input_format`, `output_format`, `requested`, `losses` i
      `lossless_for_detected_features` odpowiadają zamierzonej konwersji; każda
      zaakceptowana strata jest zapisana w dokumentacji zadania.
- [ ] Po ponownym imporcie liczba obiektów/meshy oraz wygląd geometrii są zgodne;
      wymiary, skala, `up axis`, `forward axis` i transformacje są poprawne.
- [ ] Rest pose, hierarchia szkieletu, orientacje kości i skinning nie wykazują
      deformacji; sprawdzono ruch tułowia oraz palców.
- [ ] Odtworzono co najmniej jedną animację ciała i zweryfikowano jej zakres,
      tempo oraz pozycję root.
- [ ] Działają `jawOpen`, mrugnięcie i wybrane morph targets; ich nazwy, liczba i
      skrajne wartości są zgodne ze źródłem.
- [ ] Materiały mają poprawne przypisania, a wszystkie tekstury są odnalezione,
      mają oczekiwany format, rozdzielczość i wygląd.
- [ ] Wynik został sprawdzony w rzeczywistym środowisku docelowym, nie tylko w
      sesji Blendera użytej do konwersji.

Pozytywny `lossless_for_detected_features` jest tylko wstępnym sygnałem. Nie
zastępuje powyższej kontroli rest pose, animacji, palców, `jawOpen`, mrugnięcia,
morph targets, materiałów, tekstur, jednostek i osi.
