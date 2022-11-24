# Wersja protokołu: 1.0.6 (24-11-2022)
<br />
Założenia:
- Wszystkie id są globalnie unikalne.
- Session token jest przekazywany w headerze i na jego podstawie jest określane userId.
- Dla odpowiedzi tasków typu tekst nakładamy limit 500 znaków.
- Dla tasków AUDIO i PHOTO robimy weryfikację formatu odebranych danych od użytkownika.


## Logowanie i Sesja:
    POST /user/login - loguje użytkownika o danym loginie i haśle i zwraca mu session token
Request:
```JSON
{
    "login": "misos",
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

    POST /user/register - rejestruje użytkownika o danym loginie i haśle (na razie bez autentyfikacji - każdy może wywołać)
```JSON
{
    "login": "misos2",
    "password": "1234"
}
```

W requestach, które wymagają uwierzytelnienia wysyłamy <token> w headerze postaci:
```
Authorization=Bearer <token>
```


## Tworzenie team'ów i dołączanie do gry
    GET /teams - zwraca listę wszystkich drużyn
Response:
```JSON
[
    {"id": 1, "gameId": 2, "name": "Mundo's Overlords"},
    {"id": 3, "gameId": 2, "name": "Janna's Owls"},
    {"id": 5, "gameId": 6, "name": "Teemo's Scouts"}
]
```

    GET /teams?gameId=<id> - zwraca listę drużyn dla danej gry
Response:
```JSON
[
    {"id": 1, "gameId": 2, "name": "Mundo's Overlords"},
    {"id": 3, "gameId": 2, "name": "Janna's Owls"},
]
```

    GET /teams?filter=joined - zwraca listę drużyn, których użytkownik jest częścią (użytkownik brany z sesji)
Response:
```JSON
[
    {"id": 1, "gameId": 2, "name": "Mundo's Overlords"},
]
```

    POST /teams - tworzy dryżynę
Request:
```JSON
[
    {"gameId": 2, "name": "Mundo's Overlords"},
]
```

Uwaga: Serwer generuje nowe id drużyny, wypełnia pole creator i dodaje creatora jako członka.

    GET	/teams/<id> - zwraca szczegóły dotyczące drużyny o danym id
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

Uwaga: W polach creator i members znajdują się loginy uzytkowników.

    PATCH /teams/<id>/ - modyfikuje dane drużyny (tylko creator lub GM może wykonać tego reuqesta)
```JSON
{
    "name": "Mundo's Overlords",
    "members": ["misos", "XGregory", "husiek"]
}
```

Uwaga: Jedyne pola jakie można na ten moment modyfikować to name i members. Creator nie może usunąć siebie z własnej drużyny!

    POST /teams/<id>/join - dołącza użytkownika o danym session tokenie do drużyny


## Tworzenie i edycja gier
Notka edytorska - możliwe stany gry
```
CREATED -> PENDING -> STARTED -> FINISHED
```

    GET /games - Zwraca listę gier
```JSON
[{"id": 1234, "name": "Bring back season 4", "gameMaster": "misos", "state": "CREATED"}]
```

    POST /games - Tworzy nową grę - użytkownik z session token'u staje się GM'em
Request:
```JSON
{   
    "name": "Bring back season 4",
    "startTime": "1997-07-16T19:20:30.45+01:00",
    "endCondition": "TIME",
    "endTime": "1997-07-18T19:20:30.45+01:00"
}
```

    GET	/games/<id> - zwraca grę o danym id
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

Uwaga: Jeśli pojawia się endContion: TIME, to musi pojawić się pole endTime, w p.p. nie ma tego pola.
Uwaga: Jeśli pojawia się endContition: SCORE, to musi pojawić się pole endScore, w p.p. nie ma tego pola.
End conditions: MANUAL, TIME, SCORE, TASKS

    PATCH /games/<id> - zmienia grę o danym id. Użytkownik musi być jej gameMasterem. Jeśli status !- "CREATED" to jego zmiana jest błędem.
```JSON
{
    "name": "Season 4 was the best",
    "startTime": "1997-07-16T19:20:30.45+01:00",
    "endCondition": "TIME",
    "endTime": "1997-07-18T19:20:30.45+01:00",
    "state": "STARTED"
}
```


## Tworzenie i edycja zadań
    GET /tasks - zwraca listę wszystkich tasków
    GET /tasks?gameId=1 - zwraca listę wszystkich tasków dla gry o id 1
Przykład:
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
        "gameId": 1,
        "type": "NAV_POS",
        "prerequisiteTasks":[]
    },
    {
        "id": 5,
        "name": "Example Task 5",
        "description": "Example description 5",
        "gameId": 1,
        "type": "AUDIO",
        "prerequisiteTasks":[]
    }
]
```

type: TEXT, PHOTO, QR_CODE, NAV_POS, AUDIO \
W przypadku braku gameId zwracamy wszystkie taski \
W przypadku błędnego gameId: 400 Bad Request.


    PATCH /tasks/<id> - edytujemy istniejący task o danym id \
Uwaga: Nie przesyłamy gameId, prerequisiteTasks - po utworzeniu taska nie można tego zmienić (przynajmnej na razie) \
Przykład:
```JSON
{
    "name": "New Example Task 6",
    "description": "New Example description 6",
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden \
W przypadku innego błędu: 400 Bad Request

## Tworzenie i edycja odpowiedzi
    POST /answers - dodaj odpowiedź
Przykłady:
```JSON
{"taskId": 1, "response":"tutaj tekst"}
{"taskId": 2, "response":"Base64EncodedJPEGByteArray"}
{"taskId": 3, "response":"QR_CODE_TEXT"}
{"taskId": 4, "response":{"lat":51.2137, "lon":17.2137}}
{"taskId": 5, "response":"Base64EncodedMP3"}
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku próby dodania odpowiedzi do taska, którego odpowiedzi do prerequisites nie zostały zatwierdzone przez GM'a: 403 Forbidden \
W przypadku błędnych danych w polu response/innego błędu: 400 Bad Request

    POST /tasks - dodaje taska
Przykład:
```JSON
{
    "name": "Example Task 6",
    "description": "Example description 6",
    "gameId": 1,
    "type": "PHOTO",
    "prerequisiteTasks":[]
}
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden \
W przypadku innego błędu: 400 Bad Request

    [GM] GET /answers?gameId=1&filter=unchecked - zwraca listę wszystkich niesprawdzonych odpowiedzi w grze o danym gameId \
Zwraca:
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false},
    {"id": 13231232, "taskId": 2, "userId": 2, "approved": false, "checked": false},
    {"id": 13893212, "taskId": 3, "userId": 3, "approved": false, "checked": false}
]
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden \
W przypadku innego błędu: 400 Bad Request

    [GM] GET /answers?gameId=1&filter=all - zwraca listę wszystkich odpowiedzi w danej grze
Zwraca:
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false},
    {"id": 13231232, "taskId": 2, "userId": 2, "approved": false, "checked": false},
    {"id": 13893212, "taskId": 3, "userId": 3, "approved": false, "checked": false},
    {"id": 721387420, "taskId": 1, "userId": 5, "approved": false, "checked": true},
    {"id": 521387420, "taskId": 3, "userId": 5, "approved": true, "checked": true},
]
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden \
W przypadku innego błędu: 400 Bad Request

    [USER] GET /answers?gameId=1&filter=unchecked - zwraca listę wszystkich niesprawdzonych odpowiedzi w danej grze dla użytkownika identyfikującego się session tokenem
Zwraca:
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false},
    {"id": 2138421, "taskId": 2, "userId": 1, "approved": false, "checked": false},
    {"id": 2138422, "taskId": 3, "userId": 1, "approved": false, "checked": false}
]
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku innego błędu: 400 Bad Request

    [USER] GET /answers?gameId=1&filter=all - zwraca listę wszystkich odpowiedzi w danej grze dla użytkownika identyfikującego się session tokenem
Zwraca:
```JSON
[
    {"id": 2138420, "taskId": 1, "userId": 1, "approved": false, "checked": false},
    {"id": 2138421, "taskId": 2, "userId": 1, "approved": false, "checked": false},
    {"id": 2138422, "taskId": 3, "userId": 1, "approved": false, "checked": false},
    {"id": 2138423, "taskId": 5, "userId": 1, "approved": true, "checked": true},
    {"id": 2138424, "taskId": 6, "userId": 1, "approved": true, "checked": true},
    {"id": 2138425, "taskId": 7, "userId": 1, "approved": false, "checked": true}
]
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku innego błędu: 400 Bad Request

    GET /answers/<id> - zwraca całą odpowiedź (włącznie z polem response)
Zwraca:
```JSON
{"id": 2138420, "taskId": 1, "userId": 1, "response":"fhshjjsdkjfdsnjfnj fssnfknsndkfnskfsd sfnsnf", "approved":false, "checked": false}
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku użytkownika niebędącego GM'em danej gry/użytkownika niebędącego autorem odpowiedzi: 403 Forbidden \
W przypadku innego błędu: 400 Bad Request


    [GM] PATCH /answers/{id} - zmieniamy parametry answer, czyli status approved i checked 
Example payload 1:
```JSON
{ "approved":true,
"checked": true}
```

Example payload 2:
```JSON
{ "approved":false,
"checked": true}
```

W przypadku braku/błędnego session tokenu: 401 Unauthorized \
W przypadku użytkownika niebędącego GM'em danej gry/użytkownika niebędącego autorem odpowiedzi: 403 Forbidden \
W przypadku innego błędu: 400 Bad Request

