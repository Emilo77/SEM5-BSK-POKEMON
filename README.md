# 2 Inżynieria wsteczna - Pokemon

### Dołączone pliki
Do projektu zostały dołączone [następujące pliki](task_files):
<details><summary>:pushpin: Rozwiń </summary>
<p>
    
- [assets/](task_files/assets)
- [SDL2.dll](task_files/SDL2.dll)
- [SDL2_image.dll](task_files/SDL2_image.dll)
- [retask.exe](task_files/retask.exe)
- [retask.pdb](task_files/retask.pdb)

</p>
</details>

## Treść zadania

Poniżej znajduje się opis zadania zaliczeniowego dla modułu **RE** (inżynieria wsteczna). Załączony plik stanowi prototyp gry na platformę Windows, którą należy przeanalizować i zmodyfikować tak aby wykonać poniższe zadania:

- `[1pkt]` Znajdź flagę. Flaga jest w formacie `"FLAG{...}"`. Opisz, w jaki sposób udało się uzyskać flagę.
- `[3pkt]` Flagę można również wyświetlić w grze. Opisz, jak sprawić, by niezmodyfikowana gra wyświetliła tę flagę.
- `[2pkt]` Zmodyfikuj grę, aby można było chodzić przez ściany i inne obiekty, które normalnie blokują gracza.
- `[4pkt]` Spraw, aby chodzić przez ściany dało się tylko trzymając klawisz `Shift` (wystarczy obsłużyć tylko jeden czyli lewy albo prawy).
- `[1pkt]` (**trudne**) Dodaj NPC policjanta na północy mapy, który wytłumaczy graczowi, że przez szczelinę w czasoprzestrzeni, Viridian City przestało istnieć.
- `[1pkt]` (**trudne**) Do rozmowy z policjantem dodaj również kwestie dialogowe wypowiadane przez gracza, w których ten rozpacza nad losem znajdującego się w tym mieście Garyego.

W dwóch ostatnich (**trudnych**) podzadaniach możecie wykazać się kreatywnością, a pliki gry zawierają dodatkowy asset dla policjanta. Pamiętajcie, że tak jak w poprzednim zadaniu zaliczeniowym, za częściowe rozwiązania również można otrzymać punkty!

Wysyłając rozwiązanie podeślijcie:

- sprawozdanie, w którym dokładnie opisany będzie sposób rozwiązania poszczególnych zadań. Sprawozdanie najlepiej w formacie plaintekstowym - plik `.txt` lub Markdown (`.md`).
- zmodyfikowany plik `.exe` z działającymi zmianami. Wystarczy podesłać jeden plik `.exe` ze wszystkimi zmianami. Binarka pozwalająca chodzić przez ściany z klawiszem `Shift` zalicza poprzednie zadanie, gdzie `Shift` nie był wymagany. Opis jak dokonać modyfikacji działającej bez Shifta jest dalej obowiązkowy.

Zadanie można zrealizować zarówno za pomocą oprogramowania używanego w ramach laboratorium (wymienionego w mailu wprowadzającym do modułu `RE`), jak i jego odpowiedników. Nie ma więc problemu, jeśli zamiast dekompilatora wbudowanego w `IDA Free` użyjecie np. oprogramowania `Ghidra`.

Dodatkowe porady odnośnie rozwiązywania zadania:

- Środowisko Windows powinno mieć zainstalowany pakiet `Microsoft Visual C++ Redistributable` dla `Visual Studio 2019 x64` (plik instalacyjny: [https://aka.ms/vs/17/release/vc_redist.x64.exe](https://aka.ms/vs/17/release/vc_redist.x64.exe)). Aplikacja była testowana na Windows 10 i Windows 11. W razie problemów z uruchomieniem, prosimy o maila z wersją systemu operacyjnego i komunikatem błędu (może być dodatkowo screenshot).
- Pamiętajcie, by wypakować wszystkie pliki z archiwum `.zip` zanim uruchomicie aplikację i przystąpicie do analizy.
- Zadanie zawiera plik `.pdb` (symbole). Upewnijcie się, że są one dla was dostępne podczas analizy tj. widzicie nazwy funkcji i kliknęliście `Tak`/`Yes` w oknie `IDA` *"Do you want to look for this file at the specified path and the Microsoft Symbol Server?"*
- Biblioteki `SDL` zawierają tzw. `TLS callbacks`, na które `x64dbg` nakłada breakpointy. W `Options -> Preferences` pozostawcie włączony tylko `Entry Breakpoint` i wyłączcie `System Breakpoint` i `TLS Callbacks`.
- W `x64dbg` można nakładać patche przez `File -> Patch file (CTRL+P)`. Instrukcje można zmieniać za pomocą asemblera wbudowanego w `x64dbg` (spacją) lub wcześniej zasemblować w `nasm` i dodać binarnie przez `PPM -> Edit -> Binary Edit (CTRL+E)`.
- Pamiętajcie, że nie całe miejsce w sekcji jest odzwierciedlone w pliku (wyrównanie pliku to `0x200`, a strony to `0x1000`). Jeśli nie wszystkie patche udało przełożyć na plik i dostaniecie komunikat np. `8/14 patches applied` (a nie `14/14`), oznacza to, że część patchy wykroczyła poza zakres sekcji dostępny w pliku.
- Adresy pokazywane w `IDA` mają inny adres bazowy niż w `x64dbg`. Adresy względem początku podmapowanego pliku będą jednak takie same. Warto się tym posłużyć, gdy szukacie instrukcji tej samej instrukcji w `IDA` i w `x64dbg`. Okienko `Go to` wspiera tzw. `RVA`, więc chcąc znaleźć `0x140005D47` wystarczy, że podacie w `x64dbg` :`$0x5D47`

<p align=center>
<img src="md_assets/coding.png" alt="x64dbg_view" width="100%"/>
</p>


<p align=center>
<img src="md_assets/game.png" alt="game_view" width="100%"/>
</p>

### Rozwiązanie

<details><summary>:pushpin: Rozwiń </summary>
<p>

Zaimplementowane rozwiązanie znajduje się [tutaj](src/solution.md)

</p>
</details>
