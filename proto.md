# Wersja protokołu: 1.0.7a (01-12-2022)

# Założenia:
* Wszystkie id są globalnie unikalne.
* Session token jest przekazywany w headerze i na jego podstawie jest określane userId.
* Dla odpowiedzi tasków typu tekst nakładamy limit 500 znaków.
* Dla tasków AUDIO i PHOTO robimy weryfikację formatu odebranych danych od użytkownika.
  
# Logowanie i Sesja
## Logowanie użytkownika
    POST /user/login
Opis:  
Loguje użytkownika o danym loginie i haśle, zwraca mu session token.  
  
Request:
```JSON
{
    "username": "misos",
    "password": "krowa101"
}
```
  
Response:
```JSON
{
    "sessionToken": "abcdefghjijfhrsnfnrnsfwnjfnjeffwfewfw",
    "validUntil": "1997-07-16T19:20:30.45+01:00"
}
```
  
Wyjątki:
* W przypadku złego loginu/hasła lub innego błędu: 400 Bad Request
  
## Rejestracja użytkownika
    POST /user/register
Opis:  
Rejestruje użytkownika o danym loginie i haśle (na razie bez autentyfikacji - każdy może wywołać).  
  
Request:
```JSON
{
    "username": "misos2",
    "password": "1234"
}
```
  
Wyjątki:
* W przypadku błędu (np. istnienia użytkownika o danym loginie): 400 Bad Request
  
## Uwierzytelnianie
W requestach, które wymagają uwierzytelnienia wysyłamy token w headerze postaci:
```
Authorization=Bearer <token>
```
  
# Drużyny i dołączanie do gry
## Lista wszystkich drużyn
    GET /teams
Opis:  
Zwraca listę wszystkich drużyn.  
  
Response:
```JSON
[
    {"id": 1, "gameId": 2, "name": "Mundo's Overlords"},
    {"id": 3, "gameId": 2, "name": "Janna's Owls"},
    {"id": 5, "gameId": 6, "name": "Teemo's Scouts"}
]
```
  
## Lista drużyn dla danej gry
    GET /teams?gameId=<id>
Opis:  
Zwraca listę drużyn dla danej gry.  
  
Response:
```JSON
[
    {"id": 1, "gameId": 2, "name": "Mundo's Overlords"},
    {"id": 3, "gameId": 2, "name": "Janna's Owls"},
]
```
  
Wyjątki:
* W przypadku, gdy gra nie istnieje: 400 Bad Request
  
## Lista drużyn gracza
    GET /teams?filter=joined
Opis:  
Zwraca listę drużyn, których użytkownik jest częścią (użytkownik brany z sesji).  
  
Response:
```JSON
[
    {"id": 1, "gameId": 2, "name": "Mundo's Overlords"},
]
```
  
Uwaga:  
Jeśli gracz nie dołączył do żadnej drużyny, zwracamy listę pustą.  

Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
  
## Tworzenie drużyny
    POST /teams
Opis:  
Tworzy dryżynę.  
  
Request:
```JSON
[
    {"gameId": 2, "name": "Mundo's Overlords"},
]
```
  
Uwaga:  
Serwer generuje nowe id drużyny, wypełnia pole creator i dodaje creatora jako członka. Mogą być 2 drużyny o tej samej nazwie!

Wyjątki:
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku błędu (np. nieistnienia gry): 400 Bad Request

## Szczegóły drużyny
    GET	/teams/<id>
Opis:  
Zwraca szczegóły dotyczące drużyny o danym id.  
  
Response:
```JSON
{
    "id": 1,
    "gameId": 2,
    "name": "Mundo's Overlords",
    "creator": "misos",
    "members": ["misos", "XGregory", "husiek"]
}
```
  
Uwaga:  
W polach creator i members znajdują się loginy uzytkowników.  

Wyjątki:
* W przypadku błędu (np. nieistnienia drużyny): 400 Bad Request  
  
## Modyfikacja drużyny
    PATCH /teams/<id>/
Opis:  
Modyfikuje dane drużyny (tylko creator lub GM może wykonać tego reuqesta).  
  
```JSON
{
    "name": "Mundo's Overlords",
    "members": ["misos", "XGregory", "husiek"]
}
```
  
Uwaga:  
Jedyne pola jakie można na ten moment modyfikować to name i members. Creator nie może zostać usunięty z własnej drużyny!

Wyjątki:
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry/użytkownika niebędącego creatorem drużyny: 403 Forbidden
* W przypadku innego błędu (np. nieistnienia drużyny): 400 Bad Request  

## Dołączanie do gry
    POST /teams/<id>/join
Opis:  
Dołącza użytkownika do drużyny.  

Wyjątki:
* W przypadku braku/błędnego session tokenu: 401 Unauthorized

# Tworzenie i edycja gier
## Możliwe stany gry
```
CREATED -> PENDING -> STARTED -> FINISHED
```
  
## Lista gier
    GET /games
Opis:  
Zwraca listę gier.  

Response:  
```JSON
[{"id": 1234, "name": "Bring back season 4", "gameMaster": "misos", "state": "CREATED"}]
```

## Tworzenie gry
    POST /games
Opis:  
Tworzy nową grę - użytkownik z session token'u staje się GM'em.  

Request:  
```JSON
{   
    "name": "Bring back season 4",
    "startTime": "1997-07-16T19:20:30.45+01:00",
    "endCondition": "TIME",
    "endTime": "1997-07-18T19:20:30.45+01:00"
}
```
  
Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku innego błędu (np. błędnych danych): 400 Bad Request  
  
## Szczegóły gry
    GET	/games/<id>
Opis:  
Zwraca grę o danym id.  
  
Response (example 1):  
```JSON
{
    "id": 1234,
    "name": "Bring back season 4",
    "gameMaster": "misos",
    "startTime": "1997-07-16T19:20:30.45+01:00",
    "endCondition": "TIME",
    "endTime": "1997-07-18T19:20:30.45+01:00",
    "state": "CREATED"
}
```
  
Response (example 2):  
```JSON
{
    "id": 12345,
    "name": "Bring back season 4 v2",
    "gameMaster": "misos",
    "startTime": "1997-07-26T19:20:30.45+01:00",
    "endCondition": "SCORE",
    "endScore": 1337
    "state": "CREATED"
}
```
  
Uwaga:  
Jeśli pojawia się endContion: TIME, to musi pojawić się pole endTime, w p.p. nie ma tego pola.
Uwaga:  
Jeśli pojawia się endContition: SCORE, to musi pojawić się pole endScore, w p.p. nie ma tego pola.  
  
End conditions:  
    MANUAL, TIME, SCORE, TASKS
  
Wyjątki:  
* W przypadku braku gry o takim id/innego błędu: 400 Bad Request 

## Modyfikowanie gry
    PATCH /games/<id>
Opis:  
Zmienia grę o danym id. Użytkownik musi być jej GM'em. Jeśli status != "CREATED", to jego zmiana jest błędem.  
  
Request:  
```JSON
{
    "name": "Season 4 was the best",
    "startTime": "1997-07-16T19:20:30.45+01:00",
    "endCondition": "TIME",
    "endTime": "1997-07-18T19:20:30.45+01:00",
    "state": "STARTED"
}
```

Wyjątki:
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
* W przypadku innego błędu (np. nieistnienia drużyny): 400 Bad Request  
  
# Tworzenie i edycja zadań
## Lista wszystkich zadań
    GET /tasks
Opis:  
Zwraca listę wszystkich tasków.
  
Response:
```JSON
[
    {
        "id": 1,
        "name": "Example Task 1",
        "description": "Example description 1",
        "gameId": 1,
        "type": "TEXT",
        "prerequisiteTasks":[]
    },
    {
        "id": 2,
        "name": "Example Task 2",
        "description": "Example description 2",
        "gameId": 1,
        "type": "PHOTO",
        "prerequisiteTasks":[]
    },
    {
        "id": 3,
        "name": "Example Task 3",
        "description": "Example description 3",
        "gameId": 1,
        "type": "QR_CODE",
        "prerequisiteTasks":[]
    },
    {
        "id": 4,
        "name": "Example Task 4",
        "description": "Example description 4",
        "gameId": 2,
        "type": "NAV_POS",
        "prerequisiteTasks":[]
    },
    {
        "id": 5,
        "name": "Example Task 5",
        "description": "Example description 5",
        "gameId": 2,
        "type": "AUDIO",
        "prerequisiteTasks":[]
    }
]
```

types:  
    TEXT, PHOTO, QR_CODE, NAV_POS, AUDIO  
  
## Lista zadań dla danej gry
    GET /tasks?gameId=<id>
Zwraca listę wszystkich tasków dla gry o danym id.  

Response:
```JSON
[
    {
        "id": 1,
        "name": "Example Task 1",
        "description": "Example description 1",
        "gameId": 1,
        "type": "TEXT",
        "prerequisiteTasks":[],
        "maxScore": 12,
        "correct_answer": null,
    },
    {
        "id": 2,
        "name": "Example Task 2",
        "description": "Example description 2",
        "gameId": 1,
        "type": "PHOTO",
        "prerequisiteTasks":[],
        "correct_answer": null,
        "maxScore": 12
    },
    {
        "id": 3,
        "name": "Example Task 3",
        "description": "Example description 3",
        "gameId": 1,
        "type": "QR_CODE",
        "prerequisiteTasks":[],
        "maxScore": 12,
        "correct_answer": "1234567" 
    }
]
```
  
types:  
    TEXT, PHOTO, QR_CODE, NAV_POS, AUDIO  
  
Wyjątki:  
* W przypadku błędnego gameId: 400 Bad Request  
  
## Tworzenie zadań
    POST /tasks
Dodaje taska. Pole `correct_answer` jest domyślnie inicjalizowane jako null, jest wykorzystywane tylko przy sprawdzaniu kodów QR.
  
Request:  
```JSON
{
    "name": "Example Task 6",
    "description": "Example description 6",
    "gameId": 1,
    "type": "PHOTO",
    "prerequisiteTasks":[],
    "maxScore": 12
}

```

  
```JSON
{
    "id": 3,
    "name": "Example Task 3",
    "description": "Example description 3",
    "gameId": 1,
    "type": "QR_CODE",
    "prerequisiteTasks":[],
    "maxScore": 12,
    "correct_answer": "1234567" 
}

```
  
Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
* W przypadku innego błędu: 400 Bad Request
  
## Modyfikacja zadań
    PATCH /tasks/<id>
Opis:  
Edytujemy istniejący task o danym id.  
  
Request:
```JSON
{
    "name": "New Example Task 6",
    "description": "New Example description 6",
    "maxScore": 12
}
```
  
Uwaga:  
Nie przesyłamy gameId, prerequisiteTasks - po utworzeniu taska nie można tego zmienić (przynajmnej na razie).  
  
Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
* W przypadku innego błędu: 400 Bad Request

# Tworzenie i edycja odpowiedzi
## Dodanie odpowiedzi
    POST /answers
Opis:  
Dodaj odpowiedź.  
  
Requests (multiple examples):
```JSON
{"taskId": 1, "response":"tutaj tekst"}
{"taskId": 2, "response":"Base64EncodedJPEGByteArray"}
{"taskId": 3, "response":"QR_CODE_TEXT"}
{"taskId": 4, "response":{"lat":51.2137, "lon":17.2137}}
{"taskId": 5, "response":"Base64EncodedMP3"}
```

Uwagi:
Pole score jest inicjalizowane 0.  

Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku próby dodania odpowiedzi do taska, którego odpowiedzi do prerequisites nie zostały zatwierdzone przez GM'a: 403 Forbidden
* W przypadku błędnych danych w polu response/innego błędu: 400 Bad Request
  
## Lista odpowiedzi [GM]
    GET /answers?gameId=1&filter=unchecked
Opis:  
Zwraca listę wszystkich niesprawdzonych odpowiedzi w grze o danym gameId.  

Response:
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false, "score":0},
    {"id": 13231232, "taskId": 2, "userId": 2, "approved": false, "checked": false, "score":0},
    {"id": 13893212, "taskId": 3, "userId": 3, "approved": false, "checked": false, "score":0}
]
```
  
Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
* W przypadku innego błędu: 400 Bad Request

    GET /answers?gameId=1&filter=all

Opis:  
Zwraca listę wszystkich odpowiedzi w danej grze.  
  
Response:
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false, "score":0},
    {"id": 13231232, "taskId": 2, "userId": 2, "approved": false, "checked": false, "score":0},
    {"id": 13893212, "taskId": 3, "userId": 3, "approved": false, "checked": false, "score":0},
    {"id": 721387420, "taskId": 1, "userId": 5, "approved": false, "checked": true, "score":0},
    {"id": 521387420, "taskId": 3, "userId": 5, "approved": true, "checked": true, "score":9},
]
```

Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
* W przypadku innego błędu: 400 Bad Request
  
## Lista odpowiedzi [USER]
    GET /answers?gameId=1&filter=unchecked
Opis:  
Zwraca listę wszystkich niesprawdzonych odpowiedzi w danej grze dla użytkownika identyfikującego się session tokenem.  
  
Response:  
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false, "score":0},
    {"id": 2138421, "taskId": 2, "userId": 1, "approved": false, "checked": false, "score":0},
    {"id": 2138422, "taskId": 3, "userId": 1, "approved": false, "checked": false, "score":0}
]
```
  
Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku innego błędu: 400 Bad Request
  
    GET /answers?gameId=1&filter=all
Opis:  
Zwraca listę wszystkich odpowiedzi w danej grze dla użytkownika identyfikującego się session tokenem.  
  
Response:  
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false, "score": 0},
    {"id": 2138421, "taskId": 2, "userId": 1, "approved": false, "checked": false, "score": 0},
    {"id": 2138422, "taskId": 3, "userId": 1, "approved": false, "checked": false, "score": 0},
    {"id": 2138423, "taskId": 5, "userId": 1, "approved": true, "checked": true, "score": 8},
    {"id": 2138424, "taskId": 6, "userId": 1, "approved": true, "checked": true, "score": 4},
    {"id": 2138425, "taskId": 7, "userId": 1, "approved": false, "checked": true, "score": 0}
]
```

Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku innego błędu: 400 Bad Request
  
## Szczegóły odpowiedzi
    GET /answers/<id>
Opis:  
Zwraca całą odpowiedź (włącznie z polem response).  
  
Response:  
```JSON
{"id": 2138420, "taskId": 1, "userId": 1, "response":"fhshjjsdkjfdsnjfnj fssnfknsndkfnskfsd sfnsnf", "approved":false, "checked": false, "score":0}
```
  
Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry/użytkownika niebędącego autorem odpowiedzi: 403 Forbidden
* W przypadku innego błędu: 400 Bad Request
  
## Zmienianie odpowiedzi
    [GM] PATCH /answers/{id}
Opis:  
Zmieniamy parametry answer, czyli status approved, checked oraz score.
  
Request (example 1):  
```JSON
{ "approved":true,
"checked": true,
"score": 5}
```

Request (example 2):  
```JSON
{ "approved":false,
"checked": true,
"score":0}
```
  
Wyjątki:  
* W przypadku braku/błędnego session tokenu: 401 Unauthorized
* W przypadku użytkownika niebędącego GM'em danej gry/użytkownika niebędącego autorem odpowiedzi: 403 Forbidden
* W przypadku innego błędu: 400 Bad Request

# Wiadomości broadcast od GMa do graczy

## Wysłanie nowej wiadomości
    [GM] POST /messages
Opis: Tworzy nową wiadomość. Czynność może zostać wykonana tylko przez GMa danej gry. Należy zagwarantować by przypisywane `id` były rosnące, ponieważ jest wykorzystywane do filtrowania nowych wiadomości w zapytaniu GET.

Request:
```JSON
{
    "gameId": 1,
    "content": "Powodzenia misiaczki!"
}
```

## Sprawdzanie wiadomości
    GET /messages?gameId=1
Opis: 
Zwraca listę wszystkich wiadomości wysłanych w danej grze. Timestampy podawane w ISO8601.

Response:
```JSON
[
    {
        "id": 124,
        "content": "Powodzenia!",
        "timestamp": "1997-07-16T19:20:30.45+01:00",
    },
    {
        "id": 325,
        "content": "Zaraz kończymy. Pośpieszcie się z 5 zadaniem",
        "timestamp": "1997-07-16T19:20:30.45+01:00",
    }
]
```

Wyjątki:
* W przypadku, gdy gra nie istnieje: 400 Bad Request

    GET /messages?gameId=1?newerThan=124

Opis: Zwraca listę wszystkich wiadomości wysłanych w danej grze, których id jest większy niż pole `newerThan`.

Response:
```JSON
[
    {
        "id": 325,
        "content": "Zaraz kończymy. Pośpieszcie się z 5 zadaniem",
        "timestamp": "1997-07-16T19:20:30.45+01:00",
    }
]
```

Wyjątki:
* W przypadku, gdy gra nie istnieje: 400 Bad Request


# Rankingi i statystyka
## Ranking drużyn
    GET /stat/leaderboard?gameId=1 

Opis: Zwraca listę drużyn sortowaną według malejącej liczby punktów.
Response:
```JSON
[
  {
    "id":5,
    "name":"Drużyna A",
    "members":["Ala", "Hela", "Ola"],
    "score": 230
  },
  {
    "id":15,
    "name":"Drużyna B",
    "members":["Adam", "Henryk", "Oleksy"],
    "score": 120
  },
]
```

## Statystyka zadań
    GET /stat/tasks?gameId=1
Opis: Zwraca listę zadań w danej grze z wyliczonymi statystkami. 

Response:
```JSON
[
  {
    "id":5,
    "name":"Szyszunie",
    "maxPossibleScore": 20,
    "averageScore": 5.43,
    "teamsAttempted": 3,
    "teamsApproved": 2,
    "averageSolvingTime": 1024,
  }
]
```

Uwaga: averageSolvingTime - średni czas (po drużynach, które wykonały zadanie) od kiedy zadanie stało się dostępne do zatwierdzenia, jednostka: sekunda.


Wyjątki:
* W przypadku, gdy gra nie istnieje: 400 Bad Request


