## Authorization

### POST `http://localhost:3000/api/users/register` - Create a new user
Отримує body у форматі (поля email, password обов'язкові з валідацією):
```json
{
  "email": "example@example.com",
  "password": "examplepassword"
}
```
При помилці валідації повертає <Помилка від Joi або іншої бібліотеки валідації> і статусом 400 Bad Request.
Якщо пошта вже використовується кимось іншим, повертає json з ключем {"message": "Email in use"} і статусом 409 Conflict.
За результатом роботи повертає об'єкт і статус 201 Created:
```json
 {
  "user": {
    "email": "example@example.com",
    "subscription": "starter"
  }
}
```

### GET `http://localhost:3000/api/users/verify/:verificationToken` - Email verification
По параметру verificationToken шукається користувач в моделі User.
Якщо користувач з таким токеном не знайдений, повертається json з ключем {"message": "User not found"} і статусом 404 Not Found.
Якщо користувач знайдений, встановлюється { verify: true, verificationToken: " " } в документі користувача і повертається json з ключем {"message": "Verification successful"} і статусом 200 OK.

### POST `http://localhost:3000/api/users/verify` - Resend an email for verification
Отримує body у форматі (поле email обов'язкове з валідацією):
```json
{
  "email": "example@example.com"
}
```
При помилці валідації повертає <Помилка від Joi або іншої бібліотеки валідації> і статусом 400 Bad Request.
Якщо в body немає обов'язкового поля email, повертається json з ключем {"message":"missing required field email"} і статусом 400 Bad Request.
Якщо з body все добре, виконується повторна відправка листа з verificationToken на вказаний email, але тільки якщо користувач не верифікований.
Якщо користувач вже пройшов верифікацію, повертається json з ключем {"message":"Verification has already been passed"} зі статусом 400 Bad Request.
В іншому випадку повертається json з ключем {"message":"Verification email sent"} зі статусом 200 OK.


### POST `http://localhost:3000/api/users/login` - Login user
Отримує body у форматі (поля email, password обов'язкові з валідацією):
```json
{
  "email": "example@example.com",
  "password": "examplepassword"
}
```
При помилці валідації повертає <Помилка від Joi або іншої бібліотеки валідації> і статусом 400 Bad Request.
Якщо пароль або імейл невірний, повертає json з ключем {"message": "Email or password is wrong"} і статусом 401 Unauthorized.
Якщо імейл не верифікувано, повертає json з ключем {"message": "Email not verified"} і статусом 401 Unauthorized.
В іншому випадку, порівнюється пароль для знайденого користувача; якщо паролі збігаються, створюється токен, зберігається в поточного юзера і повертається об'єкт і статус 200 OK:
```json
{
  "token": "exampletoken",
  "user": {
    "email": "example@example.com",
    "subscription": "starter"
  }
}
```

### POST `http://localhost:3000/api/users/logout` - Logout user
Не отримує body.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Шукає у моделі User користувача за _id.
Якщо користувача не існує, повертає json з ключем {"message": "Not authorized"} і статусом 401 Unauthorized.
В іншому випадку, видаляється токен у поточного юзера і повертається відповідь зі статусом 204 No Content.

### GET `http://localhost:3000/api/users/current` - Get user data by token
Не отримує body.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Якщо користувача не існує, повертається повертає json з ключем {"message": "Not authorized"} і статусом 401 Unauthorized.
В іншому випадку повертається об'єкт і статус 200 OK:
```json
{
  "email": "example@example.com",
  "subscription": "starter"
}
```

### PATCH `http://localhost:3000/api/users` - Update subscription field
Отримує body у форматі:
```json
{
    "subscription": "business"
}
```
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Якщо користувача не існує, повертається повертає json з ключем {"message": "Not authorized"} і статусом 401 Unauthorized.
В іншому випадку повертається об'єкт і статус 200 OK:
```json
{
  "email": "example@example.com",
  "subscription": "business"
}
```

### PATCH `http://localhost:3000/api/users/avatars` - Update user's avatar 
Отримує завантажений файл у body.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Якщо користувача не існує, повертається повертає json з ключем {"message": "Not authorized"} і статусом 401 Unauthorized.
В іншому випадку повертається об'єкт і статус 200 OK:
```json
{
  "avatarURL": "тут буде посилання на зображення"
}
```


## Contacts

### GET `http://localhost:3000/api/contacts` - Get all user contacts
Не отримує body.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Повертає масив всіх контактів в json-форматі зі статусом 200 OK.

### GET `http://localhost:3000/api/contacts/:contactId` - Get user contact by id
Не отримує body.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Отримує параметр contactId.
Якщо такий id є, повертає об'єкт контакту в json-форматі зі статусом 200 OK.
Якщо такого id немає, повертає json з ключем "message": "Contact with id not found" і статусом 404 Not Found.

### POST `http://localhost:3000/api/contacts` - Add new contact
Отримує body у форматі (усі поля обов'язкові):
```json
{
  "name": "User Name",
  "email": "username@mail.com",
  "phone": "(XXX) XXX-XXXX"
}
```
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Якщо в body немає якихось обов'язкових полів, повертає json з ключем {"message": "missing required name field"} і статусом 400 Bad Request.
Якщо з body все добре, додає унікальний ідентифікатор в об'єкт контакту і повертає об'єкт з доданим id та статусом 201 Created:
```json
{
  "name": "User Name",
  "email": "username@mail.com",
  "phone": "(XXX) XXX-XXXX",
  "_id": "...",
  "favorite": false,
}
```

### DELETE `http://localhost:3000/api/contacts/:contactId` - Delete contact
Не отримує body.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Отримує параметр contactId.
Якщо такий id є, повертає json формату {"message": "contact deleted"} і статусом 200 OK.
Якщо такого id немає, повертає json з ключем "message": "Not found" і статусом 404 Not Found.

### PUT `http://localhost:3000/api/contacts/:contactId` - Update contact by id
Отримує параметр contactId.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Отримує body в json-форматі c оновленням будь-яких полів name, email и phone.
Якщо body немає, повертає json з ключем {"message": "missing fields"} і статусом 400 Bad Request.
За результатом роботи повертає оновлений об'єкт контакту і статусом 200 OK. 
В іншому випадку, повертає json з ключем "message": "Not found" і статусом 404 Not Found.

### PATCH `http://localhost:3000/api/contacts/:contactId/favorite` - Update favorite field by id
Отримує параметр contactId.
Обов'язковий заголовок Authorization: "Bearer {{token}}".
Отримує body в json-форматі c оновленням поля favorite:
```json
{
  "favorite": true,
}
```
Якщо body немає, повертає json з ключем {"message": "missing field favorite"} і статусом 400 Bad Request.
За результатом роботи повертає оновлений об'єкт контакту і статусом 200 OK. 
В іншому випадку, повертає json з ключем " message ":" Not found " і статусом 404 Not Found.

### Команди:

- `npm start` &mdash; старт сервера в режимі production
- `npm run start:dev` &mdash; старт сервера в режимі розробки (development)

