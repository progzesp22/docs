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
1. GET /tasks - get all tasks
Example response:
```JSON
[
    {
        "id": 2,
        "name": "Example Task",
        "description": "Example description",
        "taskType": "PHOTO",
        "gameId": 1
    },
    {
        "id": 4,
        "name": "Example Task 2 ",
        "description": "Example description 2 ",
        "taskType": "QR_CODE",
        "gameId": 1
    }
]
```

2. POST /answers - dodaj answer
Example payload:
```JSON
{"taskId": 1, "userId": 1, "response":"tutaj tekst, ale w ogólności byte array np. - na razie styka text"}
```

3. POST /tasks - dodaj task
Example payload:
```JSON
{
    "name": "Example Task 2 ",
    "description": "Example description 2 ",
    "gameId" : 1,
    "taskType": "PHOTO"
}
```

returns:
```JSON
{
    "id": 5,
    "name": "Example Task 2 ",
    "description": "Example description 2 ",
    "taskType": "PHOTO",
    "gameId": 1
}
```

4. PUT /tasks/{id} - edytujemy istniejący task
Example payload:
```JSON
{
    "name": "Example Task 2 ",
    "description": "Example description 2 ",
    "gameId" : 1,
    "taskType": "PHOTO"
}
```

5. GET /answers/unchecked - get all unchecked answers
Example response:
```JSON
{
    "answers": [
        {"taskId": 1, "answerId": 2138420, "userId": 1, "response":"tutaj tekst, ale w ogólności byte array np. - na razie styka text"},
        {"taskId": 2, "answerId": 13231232, "userId": 2, "response":"Better nerf irelia"},
        {"taskId": 3, "answerId": 13893212, "userId": 3, "response":"I'm ummm"}
    ]
}
```

6. PATCH /answers/{id} - zmieniamy pojedyńczy parametr answer, czyli status approved 
Example payload 1:
```JSON
{ "approve":true}
```

Example payload 2:
```JSON
{ "approve":false}
```

# Backend
Implementacja serwera realizującego zapytania GET/POST z Comms/Proto -> ma przechowywać dane przynajmniej do restartu (w RAMie). Jak jesteście ambitni, to dogadajcie się z zespołem DB odnośnie zapisu. Jak mamy requesta POST /answers/approve to zapisujemy, które answery dostały odpowiedzi, a które nie i potem nie wysyłamy ich w GET /answers/list_unchecked.

