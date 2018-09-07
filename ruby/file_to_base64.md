# Сериализуем файл в base64 и обратно

### Файл в base64

* Читаем бинарные данные

    ```ruby
    bin_data = File.binread 'photo.jpg'
    # uJlu
    File.open 'photo.jpg', 'br' do |file|
      bin_data = file.read
    end
    # uJlu
    bin_data = user.photo.image.read
    ```
    
* Кодируем в base64

    ```ruby
    base64_data = Base64.encode64 bin_data # => [String]
    ```

### Из base64 в файл 

* Пишем в файл в `бинарном` режиме

    ```ruby
    File.open('photo.jpg', 'wb') do |file|
      file.write Base64.decode64(base64_data)
    end
    # uJlu
    File.binwrite 'photo.jpg', Base64.decode64(base64_data)
    ```

### Из base64 во временный файл

* Создаём временный файл `Tempfile`

    ```ruby
    tmp = Tempfile.new %w(document .jpg)
    ```

* Переводим его в `бинарный` режим

    ```ruby
    tmp.binmode
    ```

* Пишем в файл раскодированные `бинарные` данные

    ```ruby
    tmp.write Base64.decode64(base64_data)
    ```

* Добавляем документ в модель пользователя

    ```ruby
    user.create_photo! image: tmp
    ```

* Закрываем и удаляем файл

    ```ruby
    tmp.close
    tmp.unlink
    ```
    
> Также можно использовать `strict_encode64` и `strict_decode64`.

### Применение

Применялось для формирования экспортирования пользователя в json-объект вместе с файлами и обратно.
