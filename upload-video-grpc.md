# Загрузка видео

## Получение доступа к Cloud Video
Для получения доступа к Cloud Video отправьте [заявку](https://video.cloud.yandex.ru/) и дождесь ее рассморения.

## [Опционально] Выдача доступов
Для выдачи прав по работе с видео членам организации или сервисным аккаунтам перейдите в [сервис организации](https://org.cloud.yandex.ru/acl) и выдайте роль `video.admin` или `video.editor`.
Подробнее про роли можно почитать вот [здесь](https://cloud.yandex.ru/ru/docs/video/security/).

## Получение токена
Для получения IAM-токена возмользуйтесь [инструкцией](https://cloud.yandex.ru/ru/docs/iam/operations/iam-token/create).

## Получение organization_id
Для получения идентификатора организации перейдите в [сервис организации](https://org.cloud.yandex.ru/) и скопируйте идентификатор в верхней панели.

## Создание канала
Если канал еще не создан в [интерфейсе Cloud Video](https://video.cloud.yandex.ru/), то его можно создать через апи:
```
grpcurl -rpc-header "authorization:Bearer $TOKEN" \
-d '{"organization_id": "$ORGANIZATION_ID", "title": "first_channel"}' \
video.api.cloud.yandex.net:443 yandex.cloud.video.v1.ChannelService/Create
```
В результате мы получим channel_id (поле `response.id`) который будем использовать далее.

## Получение размера исходного файла
Для дальнейшей загрузки файла нужно получить его размер. Пример:
```
$ ls -l ~/Downloads/test-video.mp4
... 6899618 ... /Users/diman/Downloads/test-video.mp4
```

## Создание видео
Создаем видeo аналогично созданию канала:
```
grpcurl -rpc-header "authorization:Bearer $TOKEN" \
-d '{"channel_id": "$CHANNEL_ID", "title": "first_video", "tusd": {"file_size": $VIDEO_FILE_SIZE}, "public_access": {}}' \
video.api.cloud.yandex.net:443 yandex.cloud.video.v1.VideoService/Create
```
В результате получаем идентификатор видео в поле `response.id` и ссылку на загрузку видео в поле `response.tusd.url` в формате `https://tusd.video.cloud.yandex.net/files/...`.

## Загрузка видeo
Загружаем видео через curl:
```
curl --location --request PATCH 'https://tusd.video.cloud.yandex.net/files/...' \
--header 'Content-Type: application/offset+octet-stream' \
--header 'Upload-Offset: 0' \
--header 'Tus-Resumable: 1.0.0' \
--data-binary '@/Users/diman/Downloads/test-video.mp4'
```

Можно воспользоваться и более продвинутыми способами загрузки файла по протоколу [tus](https://tus.io/).

## Получение ссылки на плеер
```
grpcurl -rpc-header "authorization:Bearer $TOKEN" \ 
-d '{"video_id": "$VIDEO_ID"}' \
video.api.cloud.yandex.net:443 yandex.cloud.video.v1.VideoService/GetPlayerURL
```
Итоговая ссылка лежит в поле `playerUrl`.
Пример: https://runtime.video.cloud.yandex.net/player/video/vplvq3iqqqwlrm3aldwq
