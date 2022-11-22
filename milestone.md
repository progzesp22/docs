Kamień milowy do: 6.12.2022r

# Założenia
- Drużyny jednoosobowe, czyli każdy gra przeciw każdemu

## Stany gry
Gra może znajdować się w jednym z następujących stanów:
- przygotowanie (domyślny przy tworzeniu)
- oczekiwanie na graczy
- rozgrywka
- zakończona

Na podstawie stanu decydowane jest jakie akcje w endpointach mogą być wykonywane, a jakie nie. 

Wpływ stanów na uprawnienia:
- przygotowanie

  GM tworzy i edytuje zadania

  Gracz nie widzi gry na liście

- oczekiwanie na graczy

  Gracz widzi grę i jej opis

  Gracz nie widzi zadań

  GM nie może edytować zadań

- rozgrywka

  Gracz widzi grę i jej opis (możliwość dołączania do gry w trakcie jej trwania)

  Gracz widzi zadania i może przesyłać odpowiedzi

  GM ocenia odpowiedzi do zadań

  GM nie może edytować zadań

- zakończona

  GM widzi grę na liście

  Gracz nie widzi gry na liście

  Nie można nic edytować

## Zakończenie gry
Możliwy jest wybór sposobu zakończenia gry:
- manualny
    
    GM według uznania kończy grę. Niezależnie czy został wybrany którykolwiek z pozostałych trybów, możliwe jest manualne zakończenie gry.

- czasowy

    Gra kończy się o ustalonej dacie i godzinie.

- zdobycia liczby punktów

    Ustalany jest próg punktowy. Jeżeli którakolwiek drużyna zdobędzie taką lub większą liczbę punktów, gra kończy się.

- wyczerpanie zadań

    Jeżeli istnieje drużyna, która nie ma dostępnych zadań bez zgłoszonej zatwierdzonej odpowiedzi, gra kończy się.
    


## Konstrukcja zadań
Zadanie składa się z:
    - tytułu
    - opisu tekstowego
    - listy subtasków, czyli typów załączników jakie użytkownik musi przesłać w jednej odpowiedzi
    - liczby punktów za wykonanie zadania
    - list prerequistes, czyli zadań, które muszą zostać wykonane, zanim to zadanie stanie się widoczne dla drużyny
  
# Frontend

## Start - logowanie
1. Użytkownik uruchamia aplikację
2. Użytkownik wpisuje login i hasło
3. Użytkownik klika loguj <br>
4a. Komunikat (popup/toast) o błędzie w logowaniu <br>
4b. Ekran z dwoma dużymi przyciskami do wyboru GM / gracz 
5. Przejście do ekranu [lista gier](#lista-gier)

## [GM] Lista gier
1. Wyświetlają się gry o wszystkich statusach, w których użytkownik jest GMem. Sortowane wg statusu i czasu utworzenia.
2. Można utworzyć nową grę przez kliknięcie przycisku (+), przejście do ekranu [nowa gra](#nowa-gra)
3. Przejście do ekranu [edycja gry](#edycja-gry) po klinięciu na daną grę.

## [GM] Nowa gra / edycja gry
Ekran do edycji i tworzenia gry wyglądają identycznie. Przy tworzeniu nowej gry wszystkie pola są domyślnie puste, a przy edycji zostają automatycznie wypełnione obecnymi informacjami.
1. Wprowadzana jest nazwa gry
2. Wprowadzany jest opis gry <br>
3a. Wybierane jest czy gra ma startować automatycznie o zadanje godzinie (checkbox) <br>
3b. Pokazuje się pole, w którym wprowadzana jest data i godzina startu
4. Wybierane jest czy gra ma mieć automatyczne zakończenie. Jeżeli to potrzebne, pojawiają się pola z miejscem na wpisanie czasu końca lub wymaganej liczby punktów.
5. Wyświetla się lista zadań <br>
6a. Symbolem "+" można dodać nowe zadanie <br>
6b. Klikając na istniejące zadanie można je edytować
7. Użytkownik klika na przycisk "zapisz" (lub z odpowienim symbolem), który przesyła wprowadzone informacje na serwer
8. Użytkownik może kliknąć na przycisk "rozpocznij oczekiwanie na graczy" (albo lepsza nazwa), który prześle do serwera informacje o zmianie stanu gry i przejście na ekran [oczekiwania na graczy](). Przed wykonaniem akcji wyświetla się popup z prośbą o potwierdzenie.

## [GM] Nowe zadanie
1. Pola tekstowe na tytuł i treść zadania
2. Pole na wpisanie liczby punktów, może być wprowadzone za pomocą klawiatury lub zmienione przy pomocy przycisków + / -.
3. Pole, które umożliwia zaznaczenie checkboxami jakie zadania należy wykonać zanim to zadanie stanie się dostępne.
4. Przycisk (+) / "Dodaj pole odpowiedzi", po jego kliknięciu z listy rozwijanej wybierany jest typ TEXT, PHOTO, QR lub AUDIO.
5. Przycisk "Zapisz"

## [GM] Oczekiwanie na gracz
1. Wyświetla się tytuł gry oraz lista graczy, którzy zgłosili się do danej gry
2. Jeżeli gra ma czas automatycznego startu wyświetlane jest data startu
3. Kliknięcie przycisku "start" przesyła do serwera informację o zmianie stanu i przechodzi na ekran [gra]().

## [GM] Gra
1. Wyświetla się nazwa gry
2. Jeżeli jest ustalony czas zakończenia, wyświetla się pasek postępu
3. Wyświetla się pasek postępu ile całkowicie zadań zostało wykonanych
4. Przycisk "koniec gry", który kończy grę. Wymaga potwierdzenia przez popup. Po zakończeniu przechodzi do ekranu ranking.
5. Wyświetla się liczba nieocenionych zadań, wraz z przyciskiem "oceniaj", który otwiera ekran [ocenianie](). 

## [GM] Ocenianie 
1. Co 5 sekund w tle pobierane są nowe odpowiedzi.
2. Jeżeli wszystkie ocenione odpowiedzi są ocenione, wyświetla się stosowny komunikat.
3. Jeżeli są nieocenione zadania, wybierane jest to najstarsze.
4. Wyświetla się nazwa zadania, jego treść oraz przesłane subtaski.
5. Zadanie można ocenić przy pomocy dwóch przycisków czerwony (odrzucenie) i zielony (akceptacja), bądź poprzez swipe jak na Tinderze.
6. Po ocenianiu zadania, wyświetlane jest kolejne.

## [GM] Wiadomości
1. Pole tekstowe do którego można wpisać wiadomość
2. Przycisk "Wyślij wiadomość do wszystkich"

## [GM] Hamburger menu
Podczas trwania gry można przemieszczać się między ekranami:
- gra
- podgląd zadań
- odpowiedzi
- wiadomości
- ranking

## [Gracz] Lista gier
1. Wyświetla się lista gier o stanie "oczekiwanie na graczy"
2. Kliknięcie na grę powoduje przejście do ekranu opis gry
3. Swipem (przeciągnięciem góra/dół) można odświeżyć istę

## [Gracz] Opis gry
1. Wyświetla się tytuł gry, opis, data startu (jeżeli istnieje)
2. Wyświetla się nazwa drużyny, w której obecnie znajduje się gracz
3. Jeżeli gra jest w stanie "rozgrywka", pokazuje się przycisk "Graj" albo "Przejdź do zadań"
4. Kliknięcie przycisku powoduje przejście do ekranu [lista zadań]() i uruchomienie w tle procesu wykrywania końca gry.

## [Gracz] Lista zadań
1. Zadania wyświetlają się w kolejności: z przesłanym błędnym rozwiązaniem, bez przesłanych rozwiązań, zaliczone
2. Przy każdym zadaniu wyświetla się jego nazwa, liczba punktów i status
3. Po kliknięciu na zadanie, przejście do ekranu [zadanie]().

## [Gracz] Hamburger menu
W czasie trwania gry można przechodzić między ekranami:
- lista zadań
- ranking
- wiadomości

## [Gracz] Zadanie
1. Wyświetla się tytuł zadania, jego treść oraz lista załączników do treści (np. zdjęcie).
2. Wyświetla się lista odpowiedzi, które użytkownik zgłosił do tego zadania.
3. Przy każdej odpowiedzi wyświetla się jej status: nieoceniona / zatwierdzona / odrzucona.
4. Jeżeli nie ma odpowiedzi, bądź żadna nie została zatwierdzona, pokazuje się przycisk "Dodaj odpowiedź", który powoduje przejście do ekranu [udzielania odpowiedzi]().

## [Gracz] Odpowiedź
1. Wyświetla się tytuł zadania.
2. Dla każdego subtaska (typu załącznika) pojawia się pole na wprowadzenie jego treści. Jedno pod drugim
    - TEXT, zwykłe duże pole tekstowe
    - PHOTO, początkowo wyświetla się ikonka aparatu. Kliknięcie na ikonkę otwiera ekran aparatu. Po poprawnym wykonaniu zdjęcia, ikonka zastępowana jest miniaturką. Zdjęcie można wykonać ponownie poprzez kliknięcie na miniaturkę.
    - QR\_CODE, przcisk "Skanuj", informacja o pomyślnym zeskanowaniu
    - AUDIO, przycisk "Nagrywaj", "Stop" i "Odtwórz" oraz pasek odtwarzania nagrania. 
3. Na dole znajduje się przycisk "Prześlij". Domyślnie wyszarzony. Aktywuje się, gdy wszystkie powyższe pola zostaną wypełnione.



## [Gracz] Wiadomości
1. Wyświetla się lista broadcastów otrzymanych od GMa

## [Gracz] Procesy w tle
Proces uruchamia się w momencie dołączenia do gry. Co stały interwał (np. 15 sekund) sprawdzane jest:
- czy gra jest w stanie "rozgrywka" czy jest już "zakończona". Jeżeli gra się zakończy, otwiera się ekran [podsumowanie](). Jeżeli aplikacja jest zamknięta, wysyłane jest powiadomienie.
- czy pojawiły się nowe broadcasty, jeżeli tak, pojawia się nowe powiadomienie.

## [GM] Procesy w tle
Proces uruchamia się w momencie startu gry. Co stały interwał (np. 15 sekund) sprawdzane jest:
- czy pojawiły się nowe odpowiedzi, każda odpowiedź wysyła krótki sygnał dźwiękowy


## Ranking
1. Wyświetla się nazwa gry, opis.
2. Wyświetla się lista drużyn, w kolejności liczby zdobytych punktów.
3. Drużyna, w której znajduje się gracz jest wyróżniona graficznie.
4. Klikając na drużynę, można rozwinąć listę jej członków.
