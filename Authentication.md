# Аутентификация в API сервиса Контур.Страхование
Аутентификация в сервисе производится с использованием протокола [OpenID Connect].
- Для аутентификации на **тестовой** площадке используется сервер, расположенный по адресу https://identity.testkontur.ru.
- Для аутентификации на **боевой** площадке используется сервер, расположенный по адресу https://identity.kontur.ru.
## Получение конфигурации сервера
Для начала работы с OpenID Connect сервером необходимо получить информацию о его конфигурации:
```http
GET /.well-known/openid-configuration HTTP/1.1
Host: identity.testkontur.ru
```
В ответ на данный запрос возвращается документ, содержащий, в частности `token_endpoint`.
```json5
{
    "issuer": "https://identity.testkontur.ru",
    "jwks_uri": "https://identity.testkontur.ru/.well-known/openid-configuration/jwks",
    "authorization_endpoint": "https://identity.testkontur.ru/connect/authorize",
    "token_endpoint": "https://identity.testkontur.ru/connect/token",
    "userinfo_endpoint": "https://identity.testkontur.ru/connect/userinfo",
}
```
## Получение токена
Далее, необходимо обратиться по адресу, содержащемуся в поле `token_endpoint` слудующим образом:
```http
POST /connect/token HTTP/1.1
Host: identity.testkontur.ru
Content-Type: application/x-www-form-urlencoded
Authorization: Basic bWVkaV90ZXN0X2NsaWVudDpiMjlkMGQ5ZS04NzJjLTRlYTUtYTkyYS03MWIzYWJkZWEwOTE=

grant_type=client_credentials&scope=medi_api
```
В теле запроса необходимо передать параметры `grant_type=client_credentials` и `scope=medi_api`. Авторизационный заголовок - закодированная в Base64 строка вида "`clientId`:`clientSecret`".
В ответ вернется аутентификационный токен:
```json5
{
    "access_token": "1cb398cc2207ae709337ccfdacbb96ffd2ab2c759cb21a3de716d11d42d9383e",
    "expires_in": 86400,
    "token_type": "Bearer"
}
```
## Обращение к API Контур.Страхование
Полученный токен необходимо использовать в качестве заголовка аутентификации, при обращении к методам https://medi-api.testkontur.ru/V2:
```http
Authorization: Bearer 1cb398cc2207ae709337ccfdacbb96ffd2ab2c759cb21a3de716d11d42d9383e
```

   [OpenID Connect]: <https://openid.net/connect/>
