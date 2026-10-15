# Zasady dla współtwórców

## Zakres projektu

**Avatar Studio** jest profesjonalnym środowiskiem i edytowalnym pipeline'em tworzenia fotorealistycznego cyfrowego człowieka czasu rzeczywistego. Repozytorium zawiera narzędzia Python, dokumentację MkDocs i aplikację desktopową `avatar_studio`. Rekonstrukcja, Blender, COLMAP, Unreal Engine oraz Piper działają lokalnie i nie są wykonywane w standardowym CI.

Priorytety jakości: podobieństwo do materiałów referencyjnych, anatomia i deformacje, naturalna mimika, realistyczne oczy i skóra, wiarygodny lip-sync oraz wydajność czasu rzeczywistego.

## Zasady obowiązkowe

- Stosuj konwencje z `docs/project/conventions.md`, nazewnictwo z `docs/project/naming.md` oraz architekturę z `docs/project/architecture.md`.
- Kod, identyfikatory i docstringi zapisuj po angielsku. Dokumentację użytkową w Markdown zapisuj po polsku albo, dla stabilnych nazw standardów i API, zachowuj terminologię angielską.
- Procedury instalacyjne, polecenia CLI i ścieżki systemowe dokumentuj osobno dla Windows i Linux.
- Nie wprowadzaj frameworków ani warstw abstrakcji bez konkretnej potrzeby.
- Nowy lub zmieniony kod Python musi mieć czytelne nazwy, jawne importy i docstringi dla publicznych modułów, klas oraz funkcji.
- Dla zmian w Pythonie uruchom `python -m pytest -q` oraz `ruff check scripts apps tests`.
- Dla zmian dokumentacji uruchom `mkdocs build --strict`.

## Dane i artefakty

- Zdjęcia, skany, nagrania głosu i adnotacje biometryczne są danymi wrażliwymi. Nie umieszczaj ich w publicznym repozytorium bez wyraźnej zgody.
- Duże, celowo wersjonowane zasoby używają Git LFS. Prywatne źródła pozostają poza publicznym repozytorium.
- Zachowuj osobno źródłową scenę DCC, retopologię, tekstury, rig i animacje. Eksport FBX, GLB albo USD nie może być jedynym źródłem modelu.
- GitHub Pages zawiera wyłącznie publiczną dokumentację. Stan projektu i prywatne artefakty są obsługiwane lokalnie przez Avatar Studio.

## Zmiany

- Nie wypychaj zmian bezpośrednio do `main`.
- Zmiana granicy architektonicznej, formatu kanonicznego, zasad prywatności lub środowiska wykonawczego wymaga ADR.
- Formatów, nazw kości i plików wymaganych przez narzędzia nie zmieniaj bez migracji wszystkich referencji.

## GitHub Actions policy for AI agents

- Manual-first: heavy GPU/3D/avatar generation, exports, simulations, packaging, validation and releases use `workflow_dispatch` only.
- Automatic events are allowed only for explicitly approved low-cost publishing/deployment on `push` to `main`; never use `pull_request`, `pull_request_target`, `merge_group` or non-main branches.
- All jobs must have an effective `github.ref == 'refs/heads/main'` condition, including manual jobs. Preserve existing manual inputs, approval flags, disabled job gates, secret handling, permissions and runners.
- Agents should test locally on feature branches, never dispatch Actions from those branches, and verify actual workflow results before claiming success.
- Backups and essential scheduled monitoring can remain automatic only after explicit review; do not silently disable them. Document all permitted automated exceptions.
- Old branch versions can still permit non-main manual dispatch; run any manual workflow on `main` only. Removing PR checks may require adjustments to required branch checks.
