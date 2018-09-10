# Запускаем dev сервер puma с поддержкой SSL.

* Генерируем самоподписанный ssl сертификат:

  ```bash
  openssl req -x509 -sha256 -nodes -newkey rsa:2048 -days 365 -keyout localhost.key -out localhost.crt
  ```

* Запускаем puma с самоподписанным сертификатом, указав пути до сгенерированных файлов:

  ```bash
  rails s -b 'ssl://localhost:3000?key=/path/to/localhost.key&cert=/path/to/localhost.crt'
  ```