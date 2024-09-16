# Создание канала
Для загрузки видео сначала необходимо создать канал, для этого выполним следующий запрос, он создаст канал с названием `Test channel`

```
curl --request POST \
  --url https://video.api.cloud.yandex.net/video/v1/channels \
  --header 'Authorization: Bearer <IAM token>' \
  --header 'Content-Type: application/json' \
  --data '{
    "organization_id": "<organization_id>",
    "title": "Test channel"
  }'
```
Проверим то что канал создался выполнив запрос на листинг каналов
```
curl --request GET \
  --url 'https://video.api.cloud.yandex.net/video/v1/channels?organization_id=<organization_id>' \
  --header 'Authorization: Bearer <IAM token>'
```
# Создание и загрузка видео
Предварительно необходимо знать размер загружаемого файла в байтах, сделать это можно, например, с помощью команды `ls -l`. Затем выполним следующий запрос, channel_Id - идентификатор канала, созданного на предыдущем шаге
```
curl --request POST \
  --url https://video.api.cloud.yandex.net/video/v1/videos \
  --header 'Authorization: <IAM token>' \
  --header 'Content-Type: application/json' \
  --data '{
    "channel_id": "<channel_id>",
    "title": "test video",
    "tusd": {
      "file_size": <file_size>,
      "file_name": "test.mp4"
    },
    "public_access": {}
  }'
```
После выполнения запроса в ответе будет ссылка на загрузку видео (\<TUSD URL\>). 
```
{
  "done": true,
  "metadata": {
    ...
  },
  "response": {
    ...
    "tusd": {
      "url": <TUSD URL>
    },
    ...
  },
  ...
}
```
### Загрузка видео
Загрузка осуществляется с помощью протокола tus. Готовые клиенты поддерживающие протокол можно найти здесь: https://tus.io/implementations
Загрузить файл при помощи curl можно так:
```
curl --location --request PATCH <TUSD_URL> \                                     
--header 'Content-Type: application/offset+octet-stream' \
--header 'Upload-Offset: 0' \    
--header 'Tus-Resumable: 1.0.0' \                  
--data-binary '@<FILE_PATH>'
```
### Получение ссылки на плеер
После загрузки и обработки видео, можно выполнить запрос для получения ссылки на видеоплеер 
```
curl --request GET \
  --url https://video.api.cloud.yandex.net/video/v1/videos/<VIDEO ID>:getPlayerURL \
  --header 'Authorization: Bearer <IAM TOKEN>'
```
