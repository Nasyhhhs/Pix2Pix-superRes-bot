# Pix2Pix + SuperResolution Telegram Bot 


Данный проект выполнен в качестве итогового проекта обучения на курсе “Deep Learning -Продвинутый поток” от МФТИ. 
Выполнение его включало три этапа:
- Генерация собственного датасета
- Выбор архитектуры модели и ее обучение
- Упаковка телеграм бота и настройка функционала для пользователя
- Деплой бота на сервер

## Демонстрация



https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/5fee7345-0d9d-44eb-bab5-9b5aafdc0285




## Сборка датасета
Моя идея состояла в том, чтобы экспериментировать с нестандартным подходом к раскраске изображений, а не просто переводить черно-белые снимки в цветные. Я хотела, чтобы обычные фотографии заиграли сочными неоновыми переливами как в киберпанк-играх. Именно они послужили источником вдохновения для моей цветовой гаммы.
Для создания датасета я самостоятельно собрала около 3000 изображений размером 256x256 путем парсинга сочных неоновых скриншотов с сайта flickr. 
![ds](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/f69c4e76-64b3-4de5-bed4-1a3e12cd560d)

Код, который я использовала находится в папке dataset. 

## Архитектура Pix2Pix
![2023-07-05_150708](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/c7b0f2ac-c2d5-429f-b8a3-b07f81746127)

 
За основу была взята модель Pix2Pix из [оригинальной статьи](https://arxiv.org/abs/1611.07004v3)

Pix2Pix - это архитектура глубокого обучения, которая используется для задачи условной генерации изображений. Она позволяет преобразовывать изображения из одного домена в изображения другого домена с помощью сопоставления пар изображений вход-выход, иными словами, генерировать новые изображения, обучившись на парах входных и выходных изображений.
В моем случае я использовала пары изображений из вышеописанного датасета, сформировав два даталоадера – цветных и черно-белых изображений, убрав цветовую информацию из основного датасета.
Pix2Pix широко используется в различных задачах, включая перевод стиля изображений, синтез текстур, цветизацию черно-белых изображений, создание фотореалистичных изображений на основе набросков и другие. Она также находит применение в области компьютерного зрения, где может быть использована для сегментации объектов, восстановления изображений или улучшения разрешения изображений.

Архитектура Pix2Pix основана на генеративно-состязательных сетях (GAN) и состоит из двух моделей : 
Генератора и Дискриминатора
Генератор преобразует входное изображение в выходное изображение, а дискриминатор оценивает, насколько хорошо генератор выполнил свою задачу. Обучение модели происходит чередованием обновления генератора и дискриминатора, чтобы достичь более качественного преобразования изображений.

Структура генератора основана на модифицированной версии UNet. 

UNet - это сверточная нейронная сеть, изначально разработанная для задач сегментации изображений. 
![c8b56ae4-47ef-42d7-b651-9745c90911ba](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/29146a6d-9a3d-4ca9-abe4-d3079135accf)


Генератор в Pix2Pix состоит из энкодера, декодера и пропускных соединений (skip connections), что является ключевым аспектом UNet. Энкодер принимает входное изображение и постепенно уменьшает его размерность, извлекая информацию о содержимом изображения на разных уровнях абстракции. Затем, декодер последовательно увеличивает размерность и генерирует выходное изображение на основе этих абстрактных представлений. Пропускные соединения позволяют передавать информацию между энкодером и декодером на разных уровнях, что помогает сохранить детали и структуру изображения.
![Без названия](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/d827be31-3030-4dae-b468-01a50177b798)

 
Помимо стандартной архитектуры UNet, генератор Pix2Pix также включает в себя операции свертки, обратной свертки и нормализации. 

Оптимизация генератора происходит путем минимизации разницы между сгенерированными изображениями и целевыми изображениями с использованием функции потерь (BCEWithLogitsLoss() и L1Loss()).
Таким образом, генератор на основе UNet в архитектуре Pix2Pix способен преобразовывать входные изображения в соответствующие выходные изображения с сохранением деталей и структуры, что отлично подходит для переноса цветовой схемы.


![2023-07-05_165337](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/007340f5-0d92-4803-ac1d-66038e9c90eb)

![2023-07-05_165219](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/b522e942-d751-45b8-b12e-8b9c9032afcb)

![2023-07-05_165316](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/4db07c82-6709-45bd-9a51-5e013ccd10c5)

Результаты, полученные на разных эпохах:


![epochs](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/fb6ad192-539c-4ac5-b422-28e036754a15)
![learning_curve](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/0e33b293-2f49-48d6-baef-56b7d5c9b4bd)

Обучение модели длилось 54 эпохи(*изначально было 100, но результаты не сохранились) . Полный код процесса обучения модели находится в корневой папке репозитория в файле .ipynb.

## Super-Resolution
Поскольку обучение производилось на довольно небольших изображениях я решила подключить данную модель к своему проекту, чтобы изображения не сильно теряли в размере и качестве и вывод картинок более привлекательным. Так как это не основная модель проекта, я решила не уделять много времени ее обучению с нуля и использовала предобученную модель из библиотеки super-image: [Источник]https://github.com/eugenesiow/super-image

## Telegram-bot

Логику работы бота хотелось сделать интуитивной и немного игровой, чтобы она соответствовала стилю проекта:
- Пользователь заходит и видит кнопку START, нажимает и видит приветствие. 
- Есть кнопка меню в левом нижнем углу, где прописаны основные команды.
- После отправки картинки пользователю предлагаются два варианта – Neon (собственно применение цветовой схемы) и Upscaling, то есть увеличение размера изображения. Таким образом он может не только красиво раскрасить картинку, но и бонусом немного поднять ее качество ::)
- Размер картинки остается исходным после преобразования(под капотом еще апскейлинг и ресайз).
- Функции бота прописаны асинхронно, чтобы несколько пользователей могли пользоваться ботом параллельно.
- Файлы находятся в папке bot в текущем репозитории. Файл исполнения main.py



## Упаковка Docker контейнера

### Создание образа

-Создание Dockerfile и docker compose (requirements.txt если еще не создан, а также файл .dockerignore)
PS: текущий репозиторий не содержит файлов для сборки контейнера,  репозиторий с внесенными изменениями для упаковки контейнера можно найти также выложенном у меня на гитхабе *fantastic-octopus-bot*

-Сборка образа:
перейдите в директорию проекта в терминале и выполните следующую команду:
`docker build -t <имя_образа> .`


![2023-07-08_121557](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/0c27f6f9-b4b4-41fe-835a-9cd2f1bb9ecc)


Проверить, появился ли образ можно командой `docker images`
-Запуск Docker-контейнера: 
`docker run -d --name your-container-name your-image-name`

![eeee](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/93dc9f73-7485-41b1-8301-376abb35d47b)


## Проверка работы контейнера и тп

Чтобы убедиться, что контейнер успешно запущен и работает, вы можете выполнить команду 
docker ps            покажет запущенные контейнеры
docker ps –a        посмотреть все контейнеры
docker logs <id контейнера>  получение информации о возможных ошибках или проблемах при запуске контейнера
docker start <id контейнера>   запустить собранный контейнер
docker stop <id контейнера>  остановить контейнер
docker stop your-container-name     остановить контейнер





# Деплой на сервер

## Создание ВМ

![2023-07-08_181848](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/6666cd4d-fa85-48b1-beb2-c4809f84d998)

В качестве виртуальной машины был выбран облачный сервис VK Cloud. 
Для начала работы необходимо выполнить регистрацию и создать инстанс Виртуальной машины, выполнив все необходимые настройки.
- Создать подключение по ssh ключу  (/.ssh/id_rsa и /.ssh/id_rsa.pub)
- Чтобы перейти к инстансу ВМ из локального терминала выполнить команду:
ssh -i <путь к id_rsa> name@<ip адрес ВМ>
Так вы попадете в окно терминала, но уже в ВМ

## Переселение на виртуальную машину

1.	Сохранение архива
После успешного создания образа, можно сохранить его в виде файлового архива с расширением .tar:

`docker save -o your-image-name.tar <your_image_name>`
Здесь your-image-name.tar - это имя файла архива, в который будет сохранен ваш Docker образ.

-	Перемещение файла архива (your-image-name.tar) на виртуальную машину, используя инструменты передачи файлов, такие как scp. 
В локальном терминале запустите команду:


`scp <полный путь к файлу .tar> name@<ip адрес ВМ >:<путь на ВМ>`
 Например, я использовала команду:
`scp /c/Users/PC/cyber_bot.tar ubuntu@89.208.196.136:cyber_bot`

2.	Установка docker на ВМ:

`sudo snap install docker`
*Перед этим мне понадобилось выполнить команду sudo snap refresh, чтобы обновить snapd


3. Импорт Docker образа из файла архива на ВМ:
*возникла проблема Permission denied, надо запустить с правами суперпользователя от root:
`sudo bash`
`docker load -i <name_of_your_archive.tar>`
Эта команда загрузит Docker образ из файла архива на виртуальную машину.


4.Запуск контейнера на ВМ
`docker run -d --name <your-container-name>  -e BOT_TOKEN=<your-token-value>  <image_name>`
Здесь your-container-name - это имя, которое вы хотите назначить вашему контейнеру.

Теперь готовый Docker образ был скачан на виртуальную машину и запущен в контейнере, УРА!

PS Я считаю, что мой алгоритм действий был неверный и можно сразу развернуть докер контейнер из образа на Виртуальной машине, минуя архивирование, разархивирование и двойную сборку контейнеров. В итоге мне потребовался объем аж 50 ГБ на Виртуальной машине, в оптимальном варианте хватило бы 12ГБ (сам докер контейнер весит 8.6 ГБ + установка докера и обновлений). Но для первого опыта сойдет ::)




## Улучшение и развитие бота
Данный бот не является конечным и будет дорабатываться по мере моих сил и энтузиазма. Возможно в скором времени будет обучен на большем количестве эпох и датасете лучшего качества. Также возможно будут добавлены новые функции и возможности работы с изображениями. 

## Примеры
![2](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/ad8eadc3-7547-42a3-a9ef-729620b6569d)
![2](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/688ccec7-ac8c-44a3-9780-fc888c26ddb0)



![7](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/a80040da-affd-4fe1-a45c-9fc9459ab944)

![7](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/b523d3f7-d97a-46f6-80cb-4effa538b825)

![images (2)](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/e4bfa30b-34b1-457c-af92-bfa0ae2871b9)

![photo_5274098373974150072_x](https://github.com/Nasyhhhs/Pix2Pix-superRes-bot/assets/109277211/9e703f76-7bfd-4c55-b9f7-244e5d3d5a13)


Больше примеров можно найти в папке examples текущего репозитория ::)







