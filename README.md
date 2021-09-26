# API для Yatube

## Описание:

API Yatube - социальной сети для публикации личных дневников.
Это API сайта, на котором можно создать свою страницу. Если на нее зайти, то можно посмотреть все записи автора.
Пользователи могут заходить на чужие страницы, подписываться на авторов и комментировать их записи.
Автор может выбрать имя и уникальный адрес для своей страницы.

## Как запустить проект:

Клонировать репозиторий и перейти в него в командной строке:

```
git clone https://github.com/noteasycode/api_final_yatube.git
```

```
cd api_final_yatube
```

Cоздать и активировать виртуальное окружение:

```
python3 -m venv env
```

```
source env/bin/activate
```

```
python3 -m pip install --upgrade pip
```

Установить зависимости из файла requirements.txt:

```
pip install -r requirements.txt
```

Выполнить миграции:

```
python3 manage.py migrate
```

Запустить проект:

```
python3 manage.py runserver
```

Для начала работы создадим нового пользователя через API. Для этого отправим POST-запрос на ```http://127.0.0.1:8000/auth/users/```

```
curl \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"username": "leo", "password": "change_me"}' \
  http://127.0.0.1:8000/auth/users/
```

Если вы всё сделали правильно, вам вернётся HTTP-ответ со статус-кодом 201 Created:

Анонимные пользователи имеют доступ к API в режиме чтения. Для полноценного взаимодействия с API используется аутентификация на основе токена. Что-бы получить токен: отправьте POST-запрос на эндпоинт /auth/jwt/create/, передав действующий логин и пароль в полях ```username``` и ```password```.

```
curl \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"username": "leo", "password": "change_me"}' \
  http://127.0.0.1:8000/api/jwt/create/
```

API вернёт JWT-токен:

```javascript
{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIsImV4cCI6MTYyMDk0MTQ3NywianRpIjoiODUzYzE5MTg5NzMwNDQwNTk1ZjI3ZTBmOTAzZDcxZDEiLCJ1c2VyX2lkIjoxfQ.0vJBPIUZG4MjeU_Q-mhr5Gqjx7sFlO6AShlfeINK8nA",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjIwODU1Mzc3LCJqdGkiOiJkY2EwNmRiYTEzNWQ0ZjNiODdiZmQ3YzU2Y2ZjNGE0YiIsInVzZXJfaWQiOjF9.eZfkpeNVfKLzBY7U0h5gMdTwUnGP3LjRn5g8EIvWlVg"
}
```
Токен вернётся в поле ```access```, а данные из поля ```refresh``` пригодятся для обновления токена.

Этот токен также надо будет передавать в заголовке каждого запроса, в поле Authorization. Перед самим токеном должно стоять ключевое слово Bearer и пробел.

## Примеры запросов к API:

Получить список всех публикаций. При указании параметров limit и offset выдача работает с пагинацией.

```
curl \
  -X GET \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjIwODU1Mzc3LCJqdGkiOiJkY2EwNmRiYTEzNWQ0ZjNiODdiZmQ3YzU2Y2ZjNGE0YiIsInVzZXJfaWQiOjF9.eZfkpeNVfKLzBY7U0h5gMdTwUnGP3LjRn5g8EIvWlVg" \
  http://127.0.0.1:8000/api/v1/posts/
```
Получение публикации по id.

```
curl \
  -X GET \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjIwODU1Mzc3LCJqdGkiOiJkY2EwNmRiYTEzNWQ0ZjNiODdiZmQ3YzU2Y2ZjNGE0YiIsInVzZXJfaWQiOjF9.eZfkpeNVfKLzBY7U0h5gMdTwUnGP3LjRn5g8EIvWlVg" \
  http://127.0.0.1:8000/api/v1/posts/1/
```

Добавление нового комментария к публикации. Анонимные запросы запрещены.

```
curl \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjIwODU1Mzc3LCJqdGkiOiJkY2EwNmRiYTEzNWQ0ZjNiODdiZmQ3YzU2Y2ZjNGE0YiIsInVzZXJfaWQiOjF9.eZfkpeNVfKLzBY7U0h5gMdTwUnGP3LjRn5g8EIvWlVg" \
  -d '{"text": "string"}' \
  http://127.0.0.1:8000/api/v1/posts/{post_id}/comments/
```

После запуска проекта, по адресу http://127.0.0.1:8000/redoc/ будет доступна документация для API Yatube.