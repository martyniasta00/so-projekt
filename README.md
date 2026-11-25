# Temat 10 – Ośrodek WORD
**Autor:** Martyna Augustyn (nr albumu: 158117)

**Repozytorium GitHub:** https://github.com/martyniasta00/so-projekt

**Opis zadania**

Symulacja przedstawia funkcjonowanie Wojewódzkiego Ośrodka Ruchu Drogowego (WORD), który przeprowadza egzaminy na prawo jazdy. System realizowany jest w architekturze wieloprocesowej (z unikaniem rozwiązań scentralizowanych) w języku C++ i kompilowany w środowisku serwera torus.uck.pk.edu.pl.

**Egzamin składa się z dwóch części:**

Część teoretyczna – przeprowadzana przez dwie niezależne Komisje A w osobnych salach. Każda komisja składa się z 3 osób (przewodniczącego oraz dwóch członków). W ciągu dnia odbywają się dwie tury egzaminu teoretycznego rozpoczynające się o określonych godzinach T_1 oraz T_2.Część praktyczna – przeprowadzana w sposób ciągły przez K Egzaminatorów.

**Zasady ogólne funkcjonowania ośrodka**

Ośrodek WORD jest czynny w godzinach od T_p do T_k. Kandydaci przychodzą na egzamin w losowych momentach czasu i gromadzą się przed budynkiem, oczekując w kolejce na wejście. Warunkiem dopuszczenia do egzaminu jest ukończenie 18. roku życia (około 1% kandydatów nie spełnia tego warunku). O wyznaczonych godzinach T_1 oraz T_2 Dyrektor wpuszcza kandydatów na teren ośrodka, weryfikując ich pełnoletność. Na tej podstawie tworzy listę osób dopuszczonych do egzaminu oraz listę osób niedopuszczonych (zapisując ID procesu). Kandydaci w pierwszej kolejności zdają egzamin teoretyczny. Warunkiem przejścia do części praktycznej jest uzyskanie wyniku >= 84%. Osoby z wynikiem poniżej 84% kończą egzamin. Wśród kandydatów znajdują się osoby powtarzające egzamin, które mają już zaliczoną część teoretyczną (około 5% kandydatów). Osoby te informują Komisję A o posiadaniu zaliczonej teorii i przechodzą bezpośrednio do kolejki na egzamin praktyczny. Dane o wynikach są przesyłane do Dyrektora przez przewodniczących Komisji A oraz Egzaminatorów części praktycznej. Po wyjściu ostatniego kandydata Dyrektor publikuje zbiorczą listę wyników, zawierającą ID kandydata, wyniki z części A i B oraz ocenę końcową. Przebieg symulacji (logi zdarzeń) zapisywany jest w pliku tekstowym (lub plikach tekstowych)

**Przebieg części teoretycznej (Komisja A)**

Kandydaci wchodzą do sal Komisji A zgodnie z listą przekazaną przez Dyrektora. W jednej sali może przebywać jednocześnie maksymalnie 3 kandydatów. Komisja A zadaje łącznie 74 pytania: przewodniczący zadaje 26 pytań, każdy z dwóch członków komisji zadaje po 24 pytania. Pytania są przygotowywane na bieżąco (co losową liczbę sekund) w trakcie trwania egzaminu. W przypadku opóźnienia członka komisji kandydat oczekuje do momentu otrzymania kompletu pytań. Po otrzymaniu wszystkich pytań kandydat ma określony czas T_i na przygotowanie się do odpowiedzi. Następnie kandydat udziela odpowiedzi przed komisją (jeśli miejsce przed komisją jest zajęte przez innego kandydata, czeka na jego zwolnienie). Każda odpowiedź jest oceniana przez członka komisji, który zadał dane pytanie (losowy wynik procentowy z zakresu 0-100%). Przewodniczący ustala końcowy wynik procentowy z części teoretycznej (0=100%).

**Przebieg części praktycznej (Egzaminator K)**

Kandydat po zaliczeniu teorii (lub zwolniony z niej) zgłasza prośbę o przydzielenie wolnego egzaminatora. W przypadku braku dostępnych egzaminatorów oczekuje w kolejce. Egzamin praktyczny składa się z dwóch etapów:
- Zadania na placu manewrowym (egzaminator wydaje polecenie, kandydat odpowiada/wykonuje zadanie, losowana ocena tak/nie).
- Jazda w ruchu miejskim (analogiczny przebieg po zaliczeniu placu).

**Zasada dwóch błędów:** w przypadku popełnienia błędu przy zadaniu egzaminator daje drugą szansę na poprawne wykonanie tego samego manewru. Dwukrotny błąd w tym samym zadaniu skutkuje wynikiem negatywnym.

**Natychmiastowe przerwanie egzaminu:** stworzenie bezpośredniego zagrożenia dla życia lub zdrowia powoduje natychmiastowe przerwanie egzaminu z wynikiem negatywnym.

Zaliczenie części praktycznej wymaga uzyskania pozytywnych ocen z obu etapów (tak/tak)

**Obsługa sygnałów sterujących**

**Sygnał 1 (Ewakuacja)** – wysyłany przez Dyrektora. Kandydaci przebywający w budynku natychmiast przerywają egzamin teoretyczny i opuszczają ośrodek WORD. Kandydaci będący w trakcie egzaminu praktycznego kontynuują go do zakończenia. Po ustaniu symulacji Dyrektor publikuje listę wyników kandydatów, którzy wzięli udział w egzaminie.

**Sygnał 2 (Zamknięcie jednej z Komisji A)** – wysyłany przez Dyrektora do wybranej Komisji A. Komisja przestaje przyjmować nowych kandydatów i kończy pracę po przeegzaminowaniu osób, które znajdowały się już w sali w momencie otrzymania sygnału

**Architektura procesów**

Program zostanie podzielony na niezależne moduły/procesy uruchamiane za pomocą fork() i exec():

**Dyrektor** – proces nadzorujący godziny pracy ośrodka, wpuszczanie tur (T_1, T_2), weryfikację wieku, obsługę sygnałów, zbieranie wyników oraz publikację raportów końcowych i sprzątanie zasobów.

**Komisja A** – 2 niezależne procesy, z których każdy uruchamia 3 wątki reprezentujące przewodniczącego oraz dwóch członków komisji, odpowiedzialne za współbieżne generowanie pytań i ocenianie odpowiedzi.

**Egzaminator K** – K procesów realizujących egzaminy praktyczne (plac manewrowy i ruch miejski) oraz raportujących wyniki do Dyrektora.

**Kandydat** – procesy generowane w losowych odstępach czasu, przechodzące przez kolejkę wejściową, część teoretyczną (z uwzględnieniem ~5% osób powtarzających oraz ~1% niepełnoletnich) i praktyczną.

Do komunikacji i synchronizacji wykorzystane zostaną systemowe mechanizmy IPC (m.in. pamięć dzielona, semafory, kolejki komunikatów) oraz mechanizmy synchronizacji wątków (muteksy, zmienne warunkowe) z minimalnymi wymaganymi prawami dostępu, bez stosowania aktywnego oczekiwania (busy waiting), wraz z pełną obsługą błędów (perror(), errno).

**Opis planowanych testów**

Testy mają na celu weryfikację poprawności mechanizmów komunikacji międzyprocesowej, synchronizacji, obsługi sygnałów oraz braku zakleszczeń, zagłodzenia czy wycieków zasobów systemowych.

**Test 1:**

Przeciążenie kolejki wejściowej i limit miejsc w Komisji A

**Cel:** Sprawdzenie poprawności synchronizacji przy dużej liczbie kandydatów oraz weryfikacja limitu maksymalnie 3 osób w sali jednej komisji.

**Scenariusz:** Uruchomienie symulacji z dużą liczbą kandydatów przybywających przed turą T_1 (z uwzględnieniem kandydatów niepełnoletnich) oraz wprowadzenie wydłużonych, losowych opóźnień po stronie jednego z członków Komisji A przy zadawaniu pytań.

**Oczekiwany rezultat:** Osoby niepełnoletnie trafiają wyłącznie na listę niedopuszczonych. W żadnej z sal Komisji A nie przebywa jednocześnie więcej niż 3 kandydatów. Kandydaci poprawnie oczekują na komplet pytań oraz na zwolnienie miejsca przed komisją bez zakleszczeń i bez przekroczenia systemowego limitu procesów.

**Test 2:**

Rywalizacja o ograniczoną liczbę egzaminatorów praktycznych (K)

**Cel:** Weryfikacja mechanizmu kolejkowania do części praktycznej oraz ścieżki dla kandydatów z wcześniej zaliczoną teorią.

**Scenariusz:** Konfiguracja symulacji z małą liczbą egzaminatorów (np. K = 2) oraz wysokim odsetkiem kandydatów zdających teorię (oraz grupą ok. 5% osób z zaliczoną już wcześniej teorią).

**Oczekiwany rezultat:** Kandydaci z zaliczoną wcześniej teorią zgłaszają ten fakt Komisji A i trafiają bezpośrednio do kolejki na praktykę. Egzaminatorzy obsługują po jednym kandydacie naraz, poprawnie realizują zasadę dwóch błędów oraz przerwania awaryjnego, a po zakończeniu egzaminu zwalniają zasób dla kolejnej oczekującej osoby bez zagłodzenia procesów.

**Test 3:** 

Asynchroniczna ewakuacja ośrodka (Sygnał 1)

**Cel:** Sprawdzenie reakcji wszystkich procesów na sygnał ewakuacji wysłany w krytycznym momencie symulacji.

**Scenariusz:** Wysłanie sygnału 1 przez Dyrektora w chwili, gdy część kandydatów znajduje się w trakcie odpowiadania przed Komisjami A, część oczekuje w kolejce, a część odbywa egzamin praktyczny.

**Oczekiwany rezultat:** Procesy kandydatów na teorii i w kolejce natychmiast przerywają działanie i opuszczają budynek, nie blokując semaforów ani kolejek komunikatów. Egzaminy praktyczne będące w toku kończą się normalnie. Po ich zakończeniu Dyrektor poprawnie zbiera dotychczasowe dane i publikuje listę wyników.

**Test 4:** 

Zamknięcie jednej z Komisji A w trakcie trwania tury (Sygnał 2)

**Cel:** Weryfikacja kontrolowanego wyłączania pojedynczej komisji bez blokowania pozostałych kandydatów w kolejce.

**Scenariusz:** Wysłanie sygnału 2 do jednej z dwóch Komisji A w momencie, gdy w jej sali przebywają 3 osoby, a przed salą oczekują kolejni kandydaci.

**Oczekiwany rezultat:** Kandydaci przebywający już w zamykanej sali dokańczają egzamin teoretyczny, po czym komisja bezpiecznie kończy działanie. Pozostali oczekujący kandydaci zostają obsłużeni przez drugą, wciąż aktywną Komisję A, bez zawieszenia symulacji.

**Test 5:** 

Zakończenie dnia pracy i czyszczenie struktur IPC

**Cel:** Sprawdzenie poprawnego zamykania symulacji po upływie czasu T_k oraz weryfikacja zwalniania zasobów systemowych.

**Scenariusz:** Przeprowadzenie pełnego cyklu obu tur (T_1, T_2) aż do godziny zamknięcia T_k i wyjścia ostatniego kandydata, a następnie sprawdzenie stanu systemu.

**Oczekiwany rezultat:** Wszystkie procesy potomne kończą się bez pozostawiania procesów zombie. W systemie nie pozostają żadne nieusunięte segmenty pamięci dzielonej, semafory ani kolejki komunikatów, a plik z logami oraz listy wyników od Dyrektora są kompletne.
