# Założenia wstępne
1. Zakładamy na razie 1 lobby - wspólne dla wszystkich graczy (nie robimy na razie kontroli uprawnień)
2. Na razie nie implementujemy stanów gry - chcemy jedynie stworzyć szkielet komunikacji klient-serwer
3. GM na tym etapie projektowania będzie mieć dwa widoki - jeden z edycją zadań -> Frontend GM (edycja zadań), drugi z akceptacją -> Frontend GM (akceptacja zadań)
4. Frontend ma handlować przyciskk back
5. Dane przesyłamy JSON-em
6. Jak znajdujecie typo, to możecie poprawiać

# Frontend
1. Użytkownik odpala apkę
2. Użytkownik wpisuje nick i wybiera rolę(GM/zwykły gracz) - GM wybiera odpowiedni tryb (punkt 3 założeń wstępnych - TLDR: dwa radio buttony)

## Frontend zwykły gracz
3. Aplikacja wyświetla listę zadań pobraną z serwera (można odświeżyć listę zadań "swipem" w górę/dół)
4. Przy każdym zadaniu wyświetla się nazwa zadania i przycisk "odpowiedz"
    1. Użytkownik klika w przycisk odpowiedz
    2. Wyświetla się ekran zadania z polem opisu, odpowiedzi (pole tekstowe) i przyciskiem wyślij
    3. Użytkownik dostaje komunikat o powodzeniu/błędzie wysłania
        1. W przypadku sukcesu użytkownik powraca do punktu 3.
        2. W przypadku błędu użytkownik ma możliwość ponownej próby lub anulowania operacji (back do listy)

## Frontend GM (edycja zadań)
3. Aplikacja wyświetla listę zadań pobraną z serwera (można odświeżyć listę zadań "swipem" w górę/dół)
4. Przy każdym zadaniu wyświetla się nazwa zadania i przycisk "edytuj", dodatkowo mamy "floating button" dodania zadania
    1. GM klika w przycisk edytuj/dodaj (w przypadku edycji aktualy opis zadania jest pobierany z serwera/w przypadu nowego zadania pola są puste)
    2. GM wprowadza/edytuje dane i zatwierdza zmiany przyciskiem (proste pole z nazwą zadania, opisem)
    3. GM dostaje komunikat o powodzeniu/błędzie wysłania
        1. W przypadku sukcesu GM powraca do punktu 3.
        2. W przypadku błędu GM ma możliwość ponownej próby lub anulowania operacji (back do listy)

## Frontend GM (akceptacja zadań)
3. Aplikacja wyświetla listę zgłoszeń użytkowników pobraną z serwera (można odświeżyć listę zadań "swipem" w górę/dół)
4. Przy każdym zgłoszeniu wyświetla się nazwa zgłoszenia, nazwa przesyłającego i przycisk "wyświetl"
    1. GM klika w przycisk "wyświetl"
    2. Wyświetla się nowy widok ze szczegółowymi danymi zgłoszenia (nazwa użytkownika przesyłającego, nazwa zadania, opis zadania, odpowiedź użytkownika, timestamp itp.)
    3. GM wybiera opcję zatwierdź/odrzuć (trzeba wymyślić rozwiązanie, które zapobiega misclickom)
    4. GM powraca do punktu 3. Jeśli wysłanie zatwierdzenia/odrzucenia na serwer nie powiodło się, to wyskakuje powiadomienie Toast

# Comms/Proto
Mamy takie endpointy:
1. GET /tasks/list
Example response:
```JSON
{
    "tasks": [
        {"task_id": 1, "name": "Zbierz szyszunie", "description": "Dobre szyszunie", "type":"text"},
        {"task_id": 2, "name": "Zagraj URFa", "description": "Ligusia", "type":"text"},
        {"task_id": 4, "name": "Kup bananka", "description": "Przynieś rachunek i CCV do Guresza", "type":"text"}
        ]
}
```

2. POST /answears/submit
Example payload:
```JSON
{"task_id": 1, "username": "misos", "response":"tutaj tekst, ale w ogólności byte array np. - na razie styka text"}
```

3. POST /tasks/create
Example payload:
```JSON
{"name": "Zbierz szyszunie", "description": "Dobre szyszunie", "type":"text"}
```

returns:
```JSON
{"task_id":123}
```

4. POST /tasks/edit
Example payload:
```JSON
{"task_id": 1, "name": "Zbierz szyszunie", "description": "Dobre szyszunie", "type":"text"}
```

5. GET /answears/list_unchecked
Example response:
```JSON
{
    "answears": [
        {"task_id": 1, "answear_id": 2138420, "username": "misos", "response":"tutaj tekst, ale w ogólności byte array np. - na razie styka text"},
        {"task_id": 2, "answear_id": 13231232, "username": "burak", "response":"Better nerf irelia"},
        {"task_id": 3, "answear_id": 13893212, "username": "bbb", "response":"I'm ummm"}
    ]
}
```

6. POST /answears/approve
Example payload 1:
```JSON
{"answear_id": 2138420, "approve":true}
```

Example payload 2:
```JSON
{"answear_id": 13231232, "approve":false}
```

# Backend
Implementacja serwera realizującego zapytania GET/POST z Comms/Proto -> ma przechowywać dane przynajmniej do restartu (w RAMie). Jak jesteście ambitni, to dogadajcie się z zespołem DB odnośnie zapisu. Jak mamy requesta POST /answears/approve to zapisujemy, które answeary dostały odpowiedzi, a które nie i potem nie wysyłamy ich w GET /answears/list_unchecked.

# Grafika
Jakieś szkice węglem byłyby miło widziane.