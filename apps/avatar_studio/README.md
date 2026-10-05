# Avatar Studio

Desktop companion for the Avatar 3D Self production pipeline.

The application is intentionally local-first. It stores project state in `.avatar-studio/project.sqlite3`, registers artefacts with hashes and metadata, exposes stage dependencies and acts as the future orchestration layer for Blender, COLMAP, Piper and validators.

## Windows

```powershell
python -m pip install -e ".[desktop]"
avatar-studio --workspace D:\Avatar3D\projects\self-avatar
```

A standalone `.exe` is built by the `Desktop package` GitHub Actions workflow. Ad-hoc manual builds are available only as short-lived workflow artifacts. Tags matching `studio-v*` publish durable GitHub Releases containing the Windows and Linux executables plus a SHA-256 manifest; the temporary Actions artifacts are removed after a successful release.

To build the same executable locally, open PowerShell in the repository root:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e ".[desktop,desktop-build]"
pyinstaller --noconfirm --clean --onefile --windowed --name AvatarStudio --collect-all trimesh apps/avatar_studio/launcher.py
.\dist\AvatarStudio.exe --workspace D:\Avatar3D\projects\self-avatar
```

**Oczekiwany wynik:** samodzielny plik `dist\AvatarStudio.exe`. Zweryfikuj go przez uruchomienie z testowym katalogiem roboczym i sprawdzenie, czy lista 21 etapów jest widoczna. Gdy kompilacja nie powiedzie się, usuń katalogi `build` i `dist`, aktywuj ponownie środowisko i powtórz polecenie z opcją `--clean`.

## Linux

```bash
python -m pip install -e ".[desktop]"
avatar-studio --workspace ~/Avatar3D/projects/self-avatar
```

The GUI is not required for deterministic validators; pipeline logic and project storage remain importable without PySide6.

To create the Linux executable locally:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e ".[desktop,desktop-build]"
pyinstaller --noconfirm --clean --onefile --windowed --name AvatarStudio --collect-all trimesh apps/avatar_studio/launcher.py
./dist/AvatarStudio --workspace ~/Avatar3D/projects/self-avatar
```

**Oczekiwany wynik:** wykonywalny plik `dist/AvatarStudio`. Zweryfikuj go z testowym katalogiem roboczym. Jeśli PyInstaller zgłosi brak biblioteki systemowej Qt, zainstaluj pakiety środowiska graficznego właściwe dla dystrybucji, usuń `build` i `dist`, a następnie zbuduj aplikację ponownie.
