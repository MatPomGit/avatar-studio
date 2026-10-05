# Pakowanie i wydania Avatar Studio

Pakowanie aplikacji oznacza przygotowanie samodzielnego artefaktu uruchomieniowego dla użytkownika końcowego. Nie jest to zwykłe skopiowanie środowiska Python. Wydanie musi zawierać aplikację, wymagane biblioteki, metadane wersji oraz instrukcję zależności zewnętrznych, a jednocześnie nie może zawierać prywatnych danych projektu.

## Docelowe artefakty

Dla Windows docelowym artefaktem jest `AvatarStudio.exe` lub katalog dystrybucyjny zawierający program i wymagane biblioteki. Dla Linux docelowy jest autonomiczny build `AvatarStudio` lub pakiet dystrybucyjny zatwierdzony dla konkretnej dystrybucji.

Blender, COLMAP, FFmpeg i Piper pozostają narzędziami zewnętrznymi. Avatar Studio wykrywa je przez adaptery i nie powinien bez potrzeby kopiować ich binariów do własnego pakietu.

## Artefakty CI i trwałe wydania

Workflow **Desktop package** uruchamia testy GUI i buduje natywne pakiety Windows oraz Linux. Ręczne uruchomienia udostępniają krótkotrwałe artefakty CI do diagnostyki. Dla tagów `studio-v*` workflow publikuje natomiast trwały **GitHub Release** z plikami `AvatarStudio-Windows.exe`, `AvatarStudio-Linux` oraz `SHA256SUMS.txt`, a przejściowe artefakty Actions usuwa po poprawnym utworzeniu wydania.

Release jest trwałym punktem odniesienia dla wersji aplikacji; artefakt CI służy wyłącznie do krótkotrwałej walidacji. Kod w `apps/avatar_studio` pozostaje kanonicznym, edytowalnym źródłem.

## PyInstaller

Projekt używa PyInstaller jako podstawowej warstwy budowania aplikacji desktopowej. Budowanie wykonujemy osobno na każdym docelowym systemie operacyjnym. Nie należy traktować builda Windows wykonanego na Linux jako równoważnego natywnemu wydaniu Windows.

## Wersja aplikacji

Wersja wydania powinna być zgodna z `pyproject.toml`. Każdy artefakt publikowany użytkownikowi powinien pozwalać ustalić:

- wersję Avatar Studio;
- commit Git;
- system operacyjny i architekturę;
- wersję Python używaną podczas budowania;
- wersję PyInstaller;
- sumę kontrolną SHA-256 artefaktu.

## Windows

Minimalny proces:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e ".[desktop,desktop-build]"
python -m pytest -q tests/test_avatar_studio.py tests/test_gui_smoke.py
pyinstaller --noconfirm --clean --onefile --windowed --name AvatarStudio --collect-all trimesh apps\avatar_studio\launcher.py
.\dist\AvatarStudio.exe --workspace "$env:TEMP\avatar-studio-smoke"
```

Po buildzie należy uruchomić program na czystym profilu użytkownika i sprawdzić co najmniej: start aplikacji, otwarcie workspace, bazę SQLite, inspekcję artefaktu oraz diagnostykę narzędzi.

## Linux

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e ".[desktop,desktop-build]"
python -m pytest -q tests/test_avatar_studio.py tests/test_gui_smoke.py
pyinstaller --noconfirm --clean --onefile --windowed --name AvatarStudio --collect-all trimesh apps/avatar_studio/launcher.py
./dist/AvatarStudio --workspace /tmp/avatar-studio-smoke
```

Build należy testować na docelowej rodzinie dystrybucji. Różnice bibliotek systemowych mogą powodować błędy mimo poprawnego uruchomienia na maszynie budującej.

## Dane, których nie wolno pakować

Do publicznego wydania nie trafiają:

- prywatne fotografie referencyjne;
- modele głosu i dane treningowe Piper;
- prywatne workspace użytkownika;
- pliki `.avatar-studio/project.sqlite3` konkretnego projektu;
- sekrety, tokeny i pliki `.env`;
- niezatwierdzone modele 3D osoby referencyjnej.

## Podpis i integralność

Dla wydania Windows docelowo zalecane jest podpisanie kodu. Niezależnie od podpisu publikujemy SHA-256 każdego artefaktu. Suma kontrolna wykrywa zmianę pliku po zbudowaniu i ułatwia weryfikację pobranego wydania.

## Test wydania

Wydanie przechodzi test typu smoke test, czyli krótki test potwierdzający, że najważniejsze funkcje uruchamiają się w środowisku zbliżonym do użytkownika końcowego.

Minimalny zestaw:

1. start aplikacji;
2. utworzenie nowego workspace;
3. ponowne otwarcie workspace;
4. rejestracja prostego artefaktu;
5. zapis i odczyt bazy SQLite;
6. diagnostyka adapterów narzędzi;
7. otwarcie dokumentacji z poprawnego adresu GitHub Pages;
8. zamknięcie bez utraty danych.

## Definition of Done

Wydanie jest gotowe, gdy:

- powstaje natywny artefakt dla docelowego systemu;
- smoke test kończy się powodzeniem;
- artefakt nie zawiera prywatnych danych;
- wersja i commit są identyfikowalne;
- opublikowano SHA-256;
- dokumentacja instalacji odpowiada rzeczywistemu pakietowi;
- zależności zewnętrzne są wykrywane, a ich brak daje czytelny komunikat zamiast awarii aplikacji.
