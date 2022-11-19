Kamień milowy do: 6.12.2022r

# Założenia
- Rozgrywka indywidualna - każdy przeciw każdemu

# Backend

## Stany gry
Gra może znajdować się w jednym z następujących stanów:
- przygotowanie (domyślny przy tworzeniu)
- oczekiwanie na graczy
- rozgrywka
- zakończona

Na podstawie stanu decydowane jest jakie akcje w endpointach mogą być wykonywane, a jaki e nie. 

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

  
# Frontend

## Start - logowanie
1. Użytkownik odpala apkę
2. Użytkownik wpisuje login i hasło
3. Użytkownik klika loguj
4a. Komunikat (popup/toast) o błędzie w logowaniu
4b. Ekran z dwoma dużymi przyciskami do wyboru GM / gracz
5. Przejście do ekranu [lista gier](#lista-gier)

## [GM] Lista gier
1. Wyświetlają się gry o wszystkich statusach, w których użytkownik jest GMem. Sortowane wg statusu i czasu utworzenia.
2. Można utworzyć nową grę przez kliknięcie przycisku (+), przejście do ekranu [nowa gra](#nowa-gra)
3. Przejście do ekranu [edycja gry](#edycja-gry) po klinięciu na daną grę.

## [GM] Nowa gra / edycja gry
Ekran do edycji i tworzenia gry wyglądają identycznie. Przy tworzeniu nowej gry wszystkie pola są domyślnie puste, a przy edycji zostają automatycznie wypełnione obecnymi informacjami.
1. Wprowadzana jest nazwa gry
2. Wprowadzany jest opis gry
3a. Wybierane jest czy gra ma startować automatycznie o zadanje godzinie (checkbox)
3b. Pokazuje się pole, w którym wprowadzana jest data i godzina startu
4. Wyświetla się lista zadań
4a. Symbolem "+" można dodać nowe zadanie
4b. Klikając na istniejące zadanie można je edytować
5. Użytkownik klika na przycisk "zapisz" (lub z odpowienim symbolem), który przesyła wprowadzone informacje na serwer
6. Użytkownik może kliknąć na przycisk "rozpocznij oczekiwanie na graczy" (albo lepsza nazwa), który prześle do serwera informacje o zmianie stanu gry i przejście na ekran [oczekiwania na graczy](). Przed wykonaniem akcji wyświetla się popup z prośbą o potwierdzenie.

## [GM] Oczekiwanie na graczy
1. Wyświetla się tytuł gry oraz lista graczy, którzy zgłosili się do danej gry
2. Jeżeli gra ma czas automatycznego startu wyświetlane jest data startu
2. Kliknięcie przycisku "start" przesyła do serwera informację o zmianie stanu i przechodzi na ekran [rozgrywka]().

## [GM] Rozgrywka
1. 

## [Gracz] Lista gier
1. Wyświetla się lista gier o stanie "oczekiwanie na graczy"
2. Kliknięcie na grę powoduje przejście do ekranu opis gry
3. Swipem (przeciągnięciem góra/dół) można odświeżyć istę

## [Gracz] Opis gry
1. Wyświetla się tytuł gry, opis, data startu (jeżeli istnieje)
2. 

