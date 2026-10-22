# Service / Услугата

> Fill in every TODO. / Попълнете всяко TODO.

## Purpose / Предназначение

TODO: two or three sentences: what the service does and for whom. / две-три изречения: какво прави услугата и за кого.

## Endpoints / Адреси

| Method | Path | What it does / Какво прави |
|---|---|---|
| GET | /health | Reports whether the service is running and able to handle requests. |
| GET | /version | Returns the version of the running service. |
| GET | /notes | Returns the list of all stored notes. |
| POST | /notes | Creates a new note from the request body and returns it. |
| GET | /notes/{id} | Returns the single note with the given id, or an error if it does not exist. |

## Dependencies / Зависимости

TODO: what the service needs in order to run. / какво е нужно на услугата, за да работи.