# Wersja protokołu: 1.0.1-CANDIDATE (09-11-2022)

Założenia:
- Wszystkie id są globalnie unikalne.
- Session token jest przekazywany w headerze i na jego podstawie jest określane userId.
- Dla odpowiedzi tasków typu tekst nakładamy limit 500 znaków.
- Dla tasków AUDIO i PHOTOT robimy weryfikacje formatu odebranych danych od użytkownika.
- WAŻNA ZMIANA: Z tasków wysyłanych przy listowaniu usuwamy pole response - bo obrazki za dużo by zajmowały.

    GET /tasks?gameId=1 - zwraca listę wszystkich tasków dla gry o id 1
Przykład:
```JSON
[
    {
        "taskId": 1,
        "name": "Example Task 1",
        "description": "Example description 1",
        "subtasks": [{"type": "TEXT"}],
        "dependencies":[]
    },
    {
        "taskId": 2,
        "name": "Example Task 2",
        "description": "Example description 2",
        "subtasks": [{"type": "PHOTO"}],
        "dependencies":[]
    },
    {
        "taskId": 3,
        "name": "Example Task 3",
        "description": "Example description 3",
        "subtasks": [{"type": "QR_CODE"}],
        "dependencies":[]
    },
    {
        "taskId": 4,
        "name": "Example Task 4",
        "description": "Example description 4",
        "subtasks": [{"type": "NAV_POS"}],
        "dependencies":[]
    },
    {
        "taskId": 5,
        "name": "Example Task 5",
        "description": "Example description 5",
        "subtasks": [{"type": "AUDIO"}],
        "dependencies":[]
    },
    {
        "taskId": 1337,
        "name": "Multi task example",
        "description": "Multi task description",
        "subtasks": [{"type": "AUDIO"}, {"type": "NAV_POS"}, {"type": "PHOTO"}],
        "dependencies":[1, 4]
    }
]
```
type: TEXT, PHOTO, QR_CODE, NAV_POS, AUDIO, MULTI
W przypadku braku gameId response/innego błędu: 400 Bad Request.

    POST /answer - dodaj odpowiedź
Przykłady:
```JSON
{"taskId": 1, "response":["tutaj tekst"]}
{"taskId": 2, "response":["Base64EncodedJPEGByteArray"]}
{"taskId": 3, "response":["QR_CODE_TEXT"]}
{"taskId": 4, "response":[{"lat":51.2137, "lon":17.2137}]}
{"taskId": 5, "response":["Bade64EncodedMP3"]}
{"taskId": 1337, "response":["Bade64EncodedMP3", {"lat":51.2137, "lon":17.2137}, ["Base64EncodedJPEGByteArray"]]}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku próby dodania odpowiedzi do taska, którego odpowiedzi do dependencies nie zostały zatwierdzone przez GM'a: 403 Forbidden
W przypadku błędnych danych w polu response/innego błędu: 400 Bad Request

    POST /task?gameId=1 - dodaje taska dla gry o id 1
Przykład:
```JSON
{
    "name": "Example Task 6",
    "description": "Example description 6",
    "subtasks": [{"type": "PHOTO"}]
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
W przypadku innego błędu: 400 Bad Request

Zwraca:
```JSON
{
    "taskId": 6,
    "name": "Example Task 6",
    "description": "Example description 6",
    "subtasks": [{"type": "PHOTO"}]
}
```

    PATCH /task?taskId=6 - edytujemy istniejący task o danym taskId
Uwaga: Nie przesyłamy subtasks - po utworzeniu taska nie można zmienić jego typu.
Przykład:
```JSON
{
    "name": "New Example Task 6",
    "description": "New Example description 6",
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
W przypadku innego błędu: 400 Bad Request

    [GM] GET /answers?gameId=1&filter=unchecked - zwraca listę wszystkich niesprawdzonych odpowiedzi w danej grze
Zwraca:
```JSON
{
    "answers": [
        {"taskId": 1, "answerId": 2138420, "userId": 1, "approved": false, "checked": false},
        {"taskId": 2, "answerId": 13231232, "userId": 2, "approved": false, "checked": false},
        {"taskId": 3, "answerId": 13893212, "userId": 3, "approved": false, "checked": false}
    ]
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
W przypadku innego błędu: 400 Bad Request

    [GM] GET /answers?gameId=1&filter=all - zwraca listę wszystkich odpowiedzi w danej grze
Zwraca:
```JSON
{
    "answers": [
        {"taskId": 1, "answerId": 2138420, "userId": 1, "approved": false, "checked": false},
        {"taskId": 2, "answerId": 13231232, "userId": 2, "approved": false, "checked": false},
        {"taskId": 3, "answerId": 13893212, "userId": 3, "approved": false, "checked": false},
        {"taskId": 1, "answerId": 721387420, "userId": 5, "approved": false, "checked": true},
        {"taskId": 3, "answerId": 521387420, "userId": 5, "approved": true, "checked": true},
    ]
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku użytkownika niebędącego GM'em danej gry: 403 Forbidden
W przypadku innego błędu: 400 Bad Request

    [USER] GET /answers?gameId=1&filter=unchecked - zwraca listę wszystkich niesprawdzonych odpowiedzi w danej grze dla użytkownika identyfikującego się session tokenem
Zwraca:
```JSON
{
    "answers": [
        {"taskId": 1, "answerId": 2138420, "userId": 1, "approved": false, "checked": false},
        {"taskId": 2, "answerId": 2138421, "userId": 1, "approved": false, "checked": false},
        {"taskId": 3, "answerId": 2138422, "userId": 1, "approved": false, "checked": false}
    ]
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku innego błędu: 400 Bad Request

    [USER] GET /answers?gameId=1&filter=all - zwraca listę wszystkich odpowiedzi w danej grze dla użytkownika identyfikującego się session tokenem
Zwraca:
```JSON
{
    "answers": [
        {"taskId": 1, "answerId": 2138420, "userId": 1, "approved": false, "checked": false},
        {"taskId": 2, "answerId": 2138421, "userId": 1, "approved": false, "checked": false},
        {"taskId": 3, "answerId": 2138422, "userId": 1, "approved": false, "checked": false},
        {"taskId": 5, "answerId": 2138423, "userId": 1, "approved": true, "checked": true},
        {"taskId": 6, "answerId": 2138424, "userId": 1, "approved": true, "checked": true},
        {"taskId": 7, "answerId": 2138425, "userId": 1, "approved": false, "checked": true}
    ]
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku innego błędu: 400 Bad Request

    GET /answers/<answerId> - zwraca całą odpowiedź (włącznie z polem response)
Zwraca:
```JSON
{
    "answers": [
        {"taskId": 1, "answerId": 2138420, "userId": 1, "response":["fhshjjsdkjfdsnjfnj fssnfknsndkfnskfsd sfnsnf" "approved"]: false, "checked": false}
    ]
}
```
W przypadku braku/błędnego session tokenu: 401 Unauthorized
W przypadku użytkownika niebędącego GM'em danej gry/użytkownika niebędącego autorem odpowiedzi: 403 Forbidden
W przypadku innego błędu: 400 Bad Request
