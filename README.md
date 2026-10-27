# Trasy biegowe i rowerowe

Aplikacja na Androida typu **lista–szczegóły** napisana w **Kotlinie** z interfejsem w **Jetpack Compose** (Material 3). Jest to spis tras biegowych i rowerowych z opisem, ilustracją, stoperem do mierzenia czasu przejścia trasy oraz historią zapisanych wyników.

Projekt powstał na zajęciach z przedmiotu **Aplikacje mobilne** i był rozwijany etapami na kolejnych laboratoriach.

---

## Spis treści

1. [Funkcje w skrócie](#funkcje-w-skrócie)
2. [Uruchomienie](#uruchomienie)
3. [Struktura projektu](#struktura-projektu)
4. [Zadanie 2 – aplikacja typu lista–szczegóły](#zadanie-2--aplikacja-typu-listaszczegóły)
5. [Zadanie 3 – wersja na tablety](#zadanie-3--wersja-na-tablety)
6. [Zadanie 4 – dynamiczny komponent (stoper)](#zadanie-4--dynamiczny-komponent-stoper)
7. [Laboratoria 5, 6, 7 – elementy biblioteki wsparcia wzornictwa](#laboratoria-5-6-7--elementy-biblioteki-wsparcia-wzornictwa)
8. [Laboratorium 8 – animacje (splash screen)](#laboratorium-8--animacje-splash-screen)
9. [Obsługa zmiany orientacji – podsumowanie](#obsługa-zmiany-orientacji--podsumowanie)

---

## Funkcje w skrócie

- animowany ekran powitalny (splash screen) reagujący na przechylenie telefonu,
- trzy karty: **Start** (karta główna), **Biegowe** i **Rowerowe** (karty kategorii) z przełączaniem gestem przesunięcia,
- trasy w kategoriach prezentowane w siatce (obrazek + nazwa),
- ekran szczegółów z dużym obrazkiem, opisem, stoperem i listą wyników,
- stoper z przyciskami ikonowymi: start, stop, przerwij, międzyczas, zapisz wynik,
- przycisk FAB **„Wyślij czas”** udostępniający zmierzony czas innym aplikacjom,
- zapisywanie wyników (trasa + czas + data) w bazie SQLite i ekran **„Moje wyniki”**,
- szuflada nawigacyjna, motyw jasny i ciemny,
- układ dwupanelowy na tabletach,
- poprawne działanie po obrocie ekranu.

---

## Uruchomienie

**Wymagania:** Android Studio (aktualna wersja stabilna) z Android SDK. Dodatkowe biblioteki nie są potrzebne: baza danych korzysta z wbudowanego w Androida SQLite, a ikony są zapisane jako zasoby wektorowe projektu.

1. Otwórz projekt w Android Studio (`File → Open`).
2. Poczekaj na synchronizację Gradle.
3. Wybierz emulator lub telefon z włączonym debugowaniem USB i uruchom konfigurację **app** (▶).

Aby zainstalować aplikację bez Android Studio: `Build → Generate App Bundles or APKs → Generate APKs`, a następnie skopiuj na telefon plik `app/build/outputs/apk/debug/app-debug.apk`.

> Po zmianie zawartości `routes.json` trzeba odinstalować aplikację, bo baza danych tworzy się tylko przy pierwszym uruchomieniu.

---

## Struktura projektu

```
app/src/main/
├── AndroidManifest.xml
├── assets/
│   └── routes.json               # dane tras (źródło danych wczytywane do bazy)
├── java/com/example/apl_mob/
│   ├── MainActivity.kt           # aktywność główna: splash, karty, siatka, szuflada, układ tabletowy
│   ├── DetailActivity.kt         # aktywność szczegółów + wspólna treść szczegółów i przycisk FAB
│   ├── HistoryActivity.kt        # aktywność „Moje wyniki”
│   ├── Stopwatch.kt              # stan stopera i komponent stopera
│   ├── SplashAnimation.kt        # animowany splash screen + obsługa czujnika
│   ├── AppTheme.kt               # motywy jasny/ciemny i zapamiętywanie wyboru
│   ├── Route.kt                  # model trasy i repozytorium
│   └── TrasyDatabase.kt          # baza SQLite (trasy i wyniki)
└── res/
    ├── drawable/                 # ikony wektorowe (styl Material)
    └── drawable-nodpi/           # ilustracje tras route_1 … route_8
```

---

## Zadanie 2 – aplikacja typu lista–szczegóły

### Treść zadania
Aplikacja z interfejsem w Jetpack Compose, będąca spisem tras biegowych i rowerowych. Składa się z dwóch aktywności: głównej z listą tras oraz aktywności szczegółów uruchamianej po kliknięciu trasy, pokazującej jej nazwę i opis. Aplikacja ma się dopasowywać do urządzenia i działać poprawnie po zmianie orientacji.
**Wymaganie dodatkowe:** źródło danych inne niż tablica, np. baza danych lub usługa internetowa.

### Realizacja
- **Dwie aktywności.** `MainActivity` wyświetla trasy, a `DetailActivity` pokazuje szczegóły wybranej trasy. Przejście odbywa się przez jawny `Intent`, do którego przekazywany jest tylko identyfikator trasy (`putExtra(EXTRA_ROUTE_ID, route.id)`). Aktywność szczegółów odczytuje go w `onCreate` i pobiera trasę z repozytorium.
- **Model danych.** Klasa `Route` (id, nazwa, typ, długość, trudność, opis, nazwa obrazka) oraz `enum RouteType` (biegowa / rowerowa).
- **Dopasowanie do urządzenia.** Układy Compose są elastyczne (`fillMaxWidth`, `weight`, siatka adaptacyjna), a treść szczegółów jest przewijana, więc nie ucina się na małych ekranach ani w orientacji poziomej.

### Wymaganie dodatkowe – baza danych
Dane tras **nie są zapisane w tablicy w kodzie**:
1. Trasy znajdują się w pliku `assets/routes.json`.
2. Przy pierwszym uruchomieniu `TrasyDatabase` (`SQLiteOpenHelper`) tworzy bazę `trasy.db` z tabelą `routes` i wypełnia ją danymi z pliku JSON (`org.json.JSONArray`).
3. Przy każdym kolejnym uruchomieniu `RouteRepository.init()` odczytuje trasy z bazy zapytaniem SQL.

Ta sama baza ma też tabelę `results`, w której zapisywane są wyniki ze stopera (zadanie 4).

---

## Zadanie 3 – wersja na tablety

### Treść zadania
Aplikacja ma się dopasowywać do wielkości i orientacji ekranu. Na tabletach ma działać w postaci **jednej aktywności**: z lewej strony lista, z prawej szczegóły wybranej trasy. Zmiana orientacji ma zachowywać stan ekranu.

### Realizacja
- **Rozpoznanie tabletu.** W funkcji `RoutesApp` sprawdzany jest warunek `LocalConfiguration.current.smallestScreenWidthDp >= 600` (standardowa granica między telefonem a tabletem w Androidzie). Wartość ta **nie zmienia się przy obrocie**, dlatego telefon w poziomie dalej działa jak telefon, a tablet pokazuje dwa panele w obu orientacjach.
- **Telefon:** lista na cały ekran, szczegóły w osobnej aktywności `DetailActivity`.
- **Tablet:** wszystko w `MainActivity`. W `Row` z lewej znajdują się karty z siatką tras, z prawej (`RouteDetailPane`) szczegóły wybranej trasy. Przed wybraniem trasy wyświetla się komunikat „Wybierz trasę z listy”, a wybrana trasa jest wyróżniona kolorem w siatce.
- **Zachowanie stanu.** Identyfikator wybranej trasy jest przechowywany w `rememberSaveable`, więc po obrocie tabletu w prawym panelu nadal widać tę samą trasę.
- **Wspólny kod.** Treść szczegółów (`RouteDetailContent`) jest jedną funkcją Compose używaną zarówno w `DetailActivity`, jak i w prawym panelu tabletu.

---

## Zadanie 4 – dynamiczny komponent (stoper)

### Treść zadania
W aktywności szczegółów należy dodać stoper, który pokazuje czas od uruchomienia z dokładnością do sekundy, działa na smartfonach i tabletach oraz po zmianie orientacji. Wymagane przyciski: **start**, **stop**, **przerwij**.
**Wymagania dodatkowe:** ikony zamiast napisów na przyciskach, dodatkowe funkcje i przyciski, zapamiętanie czasu powiązanego z trasą i datą.

### Realizacja
Stoper jest zaimplementowany w pliku `Stopwatch.kt` i składa się z dwóch części:

**1. Stan – klasa `StopwatchState`**
- pamięta, czy stoper chodzi (`running`), czas zebrany przed ostatnim startem (`accumulatedMs`), moment ostatniego startu (`startedAt`) oraz listę międzyczasów (`laps`);
- czas liczony jest z `SystemClock.elapsedRealtime()`, a nie przez dodawanie sekund w pętli. Dzięki temu pomiar jest dokładny i nie gubi czasu podczas odtwarzania aktywności ani gdy aplikacja jest w tle;
- stan zapisywany jest do `Bundle` przez własny `Saver` (`listSaver`) i `rememberSaveable`, więc **stoper przetrwa obrót ekranu i dalej odlicza**;
- każda trasa ma własny stoper (`rememberStopwatchState(route.id)`), więc po wybraniu innej trasy na tablecie stoper zaczyna od zera.

**2. Widok – funkcja `Stopwatch`**
- czas w formacie `GG:MM:SS`, odświeżany przez `LaunchedEffect` tylko wtedy, gdy stoper chodzi;
- okrągła tarcza (`Canvas` + `drawArc`), której pierścień zapełnia się w ciągu każdej minuty.

**Przyciski (z ikonami zamiast napisów):**

| Ikona | Przycisk | Działanie |
|---|---|---|
| ▶ | Start | uruchamia odliczanie (lub wznawia je po zatrzymaniu) |
| ■ | Stop | zatrzymuje odliczanie, zmierzony czas zostaje na wyświetlaczu |
| ✕ | Przerwij | w dowolnym momencie wyłącza odliczanie i kasuje pomiar |
| 🚩 | Międzyczas | *(dodatkowy)* zapisuje międzyczas okrążenia, z różnicą względem poprzedniego |
| 💾 | Zapisz | *(dodatkowy)* zapisuje zatrzymany wynik w bazie danych |

Przyciski są aktywne tylko wtedy, gdy ich użycie ma sens, np. Stop tylko podczas odliczania, Zapisz tylko po zatrzymaniu i przy czasie co najmniej 1 s.

### Zapamiętywanie czasu z trasą i datą
- Po naciśnięciu „Zapisz” do tabeli `results` trafia rekord: `route_id`, `time_ms`, `date` (znacznik czasu).
- Pod stoperem wyświetla się lista wyników dla danej trasy (data i czas).
- Wszystkie wyniki są dostępne w osobnej aktywności **„Moje wyniki”** (`HistoryActivity`), otwieranej z szuflady nawigacyjnej. Przy każdym wyniku widać trasę, datę, tempo (min/km dla biegu) lub średnią prędkość (km/h dla roweru), a wynik można usunąć.
- Operacje na bazie wykonują się w tle (`Dispatchers.IO`), żeby nie blokować interfejsu.

---

## Laboratoria 5, 6, 7 – elementy biblioteki wsparcia wzornictwa

### Treść zadania
Zmiana strony wizualnej i części mechanizmów nawigacji. Ekran listy ma się składać z co najmniej trzech kart: karty głównej informującej o przeznaczeniu aplikacji i kart kategorii.

### Realizacja wymagań

**1. Karty kategorii z siatką**
- Ekran główny ma trzy karty (`PrimaryTabRow`): **Start**, **Biegowe**, **Rowerowe**, każda z ikoną i podpisem.
- **Karta główna** (`HomePage`) zawiera ilustrację, opis przeznaczenia aplikacji, statystyki kategorii (liczba tras i łączny dystans) oraz przyciski przejścia do kategorii.
- **Karty kategorii** dzielą trasy według rodzaju aktywności. Każda wyświetla trasy w siatce `LazyVerticalGrid` z `GridCells.Adaptive(160.dp)`, więc liczba kolumn dopasowuje się do szerokości ekranu. Pozycja siatki to karta z obrazkiem i nazwą trasy (oraz długością i trudnością).
- Kliknięcie pozycji otwiera szczegóły: **nazwę, większy obrazek i opis** trasy, a także informacje o typie, długości i trudności.
- Ilustracje tras znajdują się w `res/drawable-nodpi`, a ich nazwy są zapisane w bazie danych razem z resztą danych trasy.

**2. Przycisk FAB**
- Na ekranie szczegółów znajduje się `ExtendedFloatingActionButton` **„Wyślij czas”**.
- Wysyła zmierzony czas przez systemowe okno udostępniania (`Intent.ACTION_SEND` + `Intent.createChooser`), czyli SMS-em, e-mailem lub komunikatorem. Wiadomość zawiera nazwę trasy, dystans, czas oraz tempo albo średnią prędkość.
- Jeśli czas nie został jeszcze zmierzony, pojawia się komunikat zamiast pustej wiadomości.
- Na tablecie FAB jest umieszczony w prawym dolnym rogu panelu szczegółów.

**3. Motywy jasny i ciemny**
- W `AppTheme.kt` zdefiniowane są dwa schematy kolorów Material 3 (`lightColorScheme` i `darkColorScheme`) w zielonej, „outdoorowej” kolorystyce.
- Domyślnie aplikacja stosuje motyw systemowy. W szufladzie nawigacyjnej można przełączać tryb: **systemowy → jasny → ciemny**. Wybór jest zapisywany w `SharedPreferences`.
- Kolor ikon paska stanu dopasowuje się do motywu.

**4. Zmiana orientacji urządzenia**
Opisana zbiorczo w sekcji [Obsługa zmiany orientacji](#obsługa-zmiany-orientacji--podsumowanie).

**5. Pasek aplikacji w każdej aktywności**
Wszystkie trzy aktywności (`MainActivity`, `DetailActivity`, `HistoryActivity`) mają `TopAppBar`. W głównej jest przycisk menu otwierający szufladę, a w pozostałych przycisk powrotu.

**6. Przewijanie szczegółów razem z paskiem aplikacji**
Ekran szczegółów używa `TopAppBarDefaults.enterAlwaysScrollBehavior()` połączonego z `Modifier.nestedScroll(...)`. Przy przewijaniu treści w dół pasek aplikacji chowa się razem z nią, a przy przewijaniu w górę wraca. Tak samo działa ekran „Moje wyniki”.

**7. Przechodzenie między kartami gestem przeciągnięcia**
Zawartość kart jest umieszczona w `HorizontalPager`, który dzieli stan (`PagerState`) z paskiem kart. Na karty można przechodzić przesunięciem palca w bok, kliknięciem karty w pasku albo z szuflady, a wskaźnik aktywnej karty zawsze jest zsynchronizowany.

**8. Szuflada nawigacyjna**
`ModalNavigationDrawer` z pozycjami:
- Strona główna, Trasy biegowe, Trasy rowerowe (przejście do odpowiedniej karty),
- Moje wyniki (otwiera `HistoryActivity`),
- Motyw (przełączanie motywu).

Szufladę otwiera się przyciskiem menu na pasku aplikacji. Gest otwierania szuflady jest wyłączony, gdy jest zamknięta, żeby nie kolidował z przesuwaniem kart.

---

## Laboratorium 8 – animacje (splash screen)

### Treść zadania
Krótka animacja wyświetlana podczas uruchamiania aplikacji (splash screen), związana tematycznie z aplikacją.
**Elementy dodatkowe:** animacja reagująca na czujnik, ruchy zgodne z regułami Material Design, ruchy zgodne z prawami fizyki.

### Realizacja
Animacja znajduje się w pliku `SplashAnimation.kt` i jest wyświetlana przez `MainActivity` przed ekranem głównym. Przejście do ekranu głównego odbywa się płynnym przenikaniem (`Crossfade`).

**Przebieg animacji (ok. 2,5 s):**
1. Na tle nieba ze słońcem i wzgórzami **od dołu ekranu w stronę horyzontu rysuje się droga**: `PathMeasure.getSegment` rysuje coraz dłuższy fragment krzywej Béziera.
2. Na czubku rysowanej drogi przesuwa się **znacznik biegacza/rowerzysty** z pulsującym pierścieniem.
3. Pojawia się napis **„Trasy – biegowe i rowerowe”**, który wskakuje ze sprężystym odbiciem.
4. Na końcu drogi **spada chorągiewka mety i odbija się od ziemi**.

**Elementy dodatkowe:**

| Element | Realizacja |
|---|---|
| **Reakcja na czujnik** | Akcelerometr (`SensorManager`, `TYPE_ACCELEROMETER`). Przechylenie telefonu przesuwa słońce i dwie warstwy wzgórz z różną prędkością (efekt paralaksy). Nasłuchiwanie czujnika jest wyrejestrowywane w `DisposableEffect` po zniknięciu ekranu. |
| **Ruch zgodny z Material Design** | Rysowanie drogi i ruch znacznika korzystają z krzywej `FastOutSlowInEasing`, czyli standardowej krzywej przyspieszenia z wytycznych Material Motion (szybki start, łagodne hamowanie). |
| **Ruch zgodny z prawami fizyki** | Animacje sprężynowe (`spring`) oparte na modelu fizycznym sprężyny z tłumieniem: odbicie napisu, chorągiewka spadająca i odbijająca się od podłoża jak piłka oraz ruch elementów sceny za przechyleniem z bezwładnością. |
| **Inne** | Wersja dzienna i nocna sceny (w ciemnym motywie księżyc i gwiazdy). Dotknięcie ekranu pomija animację. |

Splash screen jest pokazywany tylko raz: flaga przechowywana w `rememberSaveable` sprawia, że **nie wraca po obrocie ekranu**.

> Na Androidzie 12 i nowszych system sam wyświetla krótki ekran startowy z ikoną aplikacji, a zaraz po nim zaczyna się opisana animacja.

> W emulatorze reakcję na czujnik można sprawdzić w `⋮ → Extended controls → Virtual sensors`, obracając wirtualne urządzenie.

---

## Obsługa zmiany orientacji – podsumowanie

Przy obrocie ekranu Android odtwarza aktywność od nowa, dlatego stan, który ma przetrwać obrót, jest zapisywany przez `rememberSaveable` (do `Bundle`) albo przekazywany w `Intent`:

| Co jest zachowywane | Jak |
|---|---|
| Wybrana trasa na ekranie szczegółów (telefon) | identyfikator trasy w `Intent` aktywności |
| Wybrana trasa w prawym panelu (tablet) | `rememberSaveable` |
| Aktywna karta | `rememberPagerState` (zapisuje stan automatycznie) |
| Stan stopera i międzyczasy | `rememberSaveable` z własnym `Saver` + `SystemClock.elapsedRealtime()` |
| Informacja, że splash już się wyświetlił | `rememberSaveable` |
| Wybrany motyw | `SharedPreferences` |
| Wyniki i dane tras | baza danych SQLite |

Układ dopasowuje się do nowej orientacji: siatka zmienia liczbę kolumn, a treść szczegółów i karty głównej jest przewijana.

---

## Wykorzystane technologie

- Kotlin, Jetpack Compose, Material 3
- `HorizontalPager`, `LazyVerticalGrid`, `ModalNavigationDrawer`, `PrimaryTabRow`, `ExtendedFloatingActionButton`, `TopAppBar` ze `scrollBehavior`
- Animacje Compose: `Animatable`, `spring`, `tween`, `rememberInfiniteTransition`, `Crossfade`, `Canvas`
- SQLite (`SQLiteOpenHelper`), `org.json`, `SharedPreferences`
- `SensorManager` (akcelerometr)
- Coroutines (`LaunchedEffect`, `produceState`, `Dispatchers.IO`)
