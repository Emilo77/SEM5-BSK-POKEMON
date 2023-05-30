# 2 Inżynieria wsteczna - Pokemon

### Dołączone pliki
Do projektu zostały dołączone następujące pliki:
<details><summary>:pushpin: Rozwiń </summary>
<p>



</p>
</details>


### Rozwiązanie

<details><summary>:pushpin: Rozwiń </summary>
<p>

- [atomic-volatile.cpp](atomic-volatile.cpp)

</p>
</details>


## 1. Znalezienie flagi poza grą:

Uruchomiłem aplikację w programie `IDA`.

Wykonałem komendę `Alt + t`, ręcznie to polecenie z zakładki `Search > Text ...` .

Wpisałem do wyszukania frazę `Flag{`. W programie zostałem przeniesiony do fragmentu kodu:

<img src="md_assets/flag_1.png" alt="drawing" width="50%"/>

Z innych zdań zrozumiałem, że `27h` we fladze jest zamiennikiem na pojedynczy apostrof `'`.

Zatem flagą jest **`FLAG{gr3pp!ng-thr0ugh-str1ngs?-isn't-th4t-t0o-ez?}`**.

## 2. Znalezienie flagi w grze:

W edytorze  `IDA` znalazłem wystąpienie nazwy `aFlagGr3ppNgThr` poprzez najechanie nazwy
kursorem i naciśnięcie `X` (`IDA` pokazuje tak wszystkie wystąpienia frazy w kodzie):

<img src="md_assets/flag_2.png" alt="drawing" width="50%"/>

Przed analizą kodu trochę potestowałem grę. Zauważyłem, że tekst flagi w kodzie znajduje się obok
tekstów występujących na zewnątrz w świecie gry. Postanowiłem sprawdzić `player_mailbox`:

<img src="md_assets/flag_3.png" alt="drawing" width="50%"/>

<img src="md_assets/flag_4.png" alt="drawing" width="50%"/>

Zaciekawiła mnie funkcja `check`, która w programie przed kompilacją jest zapewne jakąś funkcją
zwracającą `bool`.
Na początku postanowiłem spatchować grę ze zmienioną instrukcją

```angular2html
jnz     short loc_140004B6F
```

na

```angular2html
jz     short loc_140004B6F
```

i sprawdzić w grze, co wyświetli się w skrzynce.

<img src="md_assets/flag_5.png" alt="drawing" width="50%"/>

Dzięki temu wiedziałem już dokładnie, gdzie powinna normalnie wyświetlić się flaga. Pozostaje
znaleźć sposób w grze, aby instrukcja `jnz     short loc_140004B6F` wykonała skok.

Z analizy kodu założyłem, że funkcje `check`, `mark` i `clear` wywołują się po pewnych postępach
gry. Zrozumiałem, że aby flaga pokazała się w skrzynce, musi wywołać się funkcja `mark` z
rejestrem `cl` o wartości 5.

Sprawdziłem wystąpienia funkcji `mark`, gdzie rejestr `cl` będzie przyjmował wartość 5.
Powodowała to funkcja `overworld_keypress`:

<img src="md_assets/flag_6.png" alt="drawing" width="50%"/>

Aby doszło do wykonania `mark`, wartość `edx` powinna być równa `0Eh`. Wtedy instrukcja `jnz`
nie wykona skoku i później dojdzie do skoku do funkcji `mark`.

Skoro funkcja zawiera `keypress` w nazwie, zapewne chodzi o naciśnięcie jakiejś sekwencji
klawiszy na klawiaturze.

Aby znaleźć sekwencję tych klawiszy, znalazłem dwie opcje:

### Brute

Postanowiłem uruchomić program w `IDA`, ustawiając breakpoint na `inc`. Następnie po kolei
naciskałem każdy klawisz na klawiaturze, aż w końcu któryś zatrzymywał aplikację na breakpoincie.
Na bieżąco zapisywałem klawisze, które powodowały zatrzymanie.

<img src="md_assets/flag_7.png" alt="drawing" width="50%"/>

Otrzymałem sekwencję `c-a-n-i-h-a-z-f-l-a-g-p-l-x`.

### Odszyfrowanie sekwencji przez analizę kodu

Zrozumiałem, że `dword_7FF7BF8C1A44` jest licznikiem, natomiast pod adresem `asc_7FF7BF8BA100`
znajduje się ciąg zaszyfrowanych liter. Przy naciśnięciu klawisza zostaje wykonane porównanie
wartości klawisza z elementem w tablicy o danym indeksie. W przypadku naciśnięcia
odszyfrowanego pierwszego elementu tablicy licznik zwiększa się o 1, tak iterujemy się po całej
tablicy.

Tablica `asc_7FF7BF8BA100` zawiera wartości (w systemie 10):

| 9   | 11  | 4   | 3   | 2   | 11  | 16  | 13  | 6   | 11  | 13  | 26  | 6   | 18  |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|

Każdy element jest xorowany z wartością `6A` (`106` w systemie 10). Ponieważ `xor` jest operacją
odwracalną, postanowiłem zastosować go na tej tablicy:

| Value         | 9   | 11  | 4   | 3   | 2   | 11  | 16  | 13  | 6   | 11  | 13  | 26  | 6   | 18  |
|---------------|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| `xor` applied | 99  | 97  | 110 | 105 | 104 | 97  | 16  | 13  | 108 | 97  | 103 | 112 | 108 | 120 |
| `ASCII` sign  | c   | a   | n   | i   | h   | a   | z   | f   | l   | a   | g   | p   | l   | x   |

Po naciśnięciu sekwencji sprawdziłem skrzynkę pocztową na zewnątrz:

<img src="md_assets/flag_8.png" alt="drawing" width="50%"/>

Flaga pojawiła się bez modyfikacji gry.

## Chodzenie przez obiekty

Postanowiłem sprawdzić funkcję do obsługi poruszania się strzałkami, czyli
`handle_movement_input` (dla wygody pozamieniałem później labele na bardziej zrozumiałe):

<img src="md_assets/image_1.png" alt="drawing" width="50%"/>

Sprawdziłem `player_step`.

<img src="md_assets/image_2.png" alt="drawing" width="50%"/>

Zrozumiałem, że funkcja `object_can_move` sprawdza, czy wykonanie
ruchu jest legalne. Funkcja `object_can_move` ustawia wartość `al` na 1, jeżeli ruch jest
legalny, wpp. ustawia `al` na 0.

Nastepnie wykonuje się instrukcja skoku warunkowego. Możemy zmodyfikować tę instrukcję, aby
nigdy nie skakała do `cant_move`, tylko aby zawsze przechodziła do `can_move`. W Idzie dokonałem
zmiany instrukcji

```asm
jz      short cant_move
```

na instrukcję

```asm
jz      short can_move
```

Uruchomiłem program z naniesionym patchem, można było przechodzić przez obiekty.

<img src="md_assets/image_3.png" alt="drawing" width="50%"/>

<img src="md_assets/image_4.png" alt="drawing" width="50%"/>

## Chodzenie przez obiekty z naciśniętym klawiszem shift

Wykonujemy analogiczną metodę jak bez shifta, tylko w `player_step`, w instrukcji `jz      short
cant_move` skaczemy do funkcji sprawdzającej, czy został naciśnięty `LShift`. Jeżeli został
naciśnięty, zezwalamy na wykonanie ruchu i skaczemy do `can_move`, wpp. skaczemy do `cant_move`.

Tym razem dokonywałem zmian w `x64dbg`, ponieważ łatwiej tam było zmieniać instrukcje i z
jakiegoś powodu obszar wolnej pamięci programu był bardziej widoczny niż w `IDA`.

Dokonałem następujących modyfikacji:

<img src="md_assets/image_5.png" alt="drawing" width="50%"/>

Skok warunkowy `jz      short cant_move` zmodyfikowałem tak, aby skakał parę adresów dalej, do
kolejnego skoku na koniec programu.

Na końcu programu dodałem funkcję sprawdzającą, czy został naciśnięty `LShift`. Napisałem ją
analogicznie jak w funkcji `handle_movement_input`, która sprawdzała naciśnięcie klawiszy strzałek.

<img src="md_assets/image_6.png" alt="drawing" width="50%"/>

```asm
; --- Funkcja sprawdzająca LShift na końcu programu ---
handle_shift_input:
    sub     rsp, 0x28
    xor     ecx, ecx
    call    qword ptr ds:[<&SDL_GetKeyboardState>]
    cmp     byte ptr ds:[rax + 0xE1], 0x0
    jz      shift_not_pressed
    add     rsp, 0x28
    jmp     can_move
shift_not_pressed:
    add     rsp, 0x28
    jmp     cant_move

```

Naniosłem patcha na aplikację, gracz od teraz może podczas trzymania `LShift` przechodzić przez
ściany.
