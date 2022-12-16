# Запуск

Для запуска: `./script.sh`

# Ход выполнения

1. Для поиска уязвимости я выбрал проект FFmpeg. Нашел в нем [коммит](https://github.com/FFmpeg/FFmpeg/commit/611b35627488a8d0763e75c25ee0875c5b7987dd), который соответствовал требованиям, а именно:
   - Соответствовал типу **CWE-476** - разыменование нулевого указателя

2. Создал докерфайл, в котором:
   1. Выбрал в качестве базового образа Ubuntu 20.04
   2. Уствновил `DEBIAN_FRONTEND=nointeractive` для исключения взаимодействия с командной строкой во время установки пакетов и других команд
   3. Установил необходимые зависимости указанные в официальной [вики](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu)
   4. Подготавливаю рабочее место (папка *workdir*)
   5. Скачиваю, распаковываю и перемещаю коммит с еще не исправленной уязвимостью в рабочее место
   6. Запускаю билд программы с помощью утилиты make
   7. Создаю скрипт *copy_out.sh*, который занимается поиском необходимого объектного файла с ошибкой и копированием его в директорию *workspace/out*
   8. Оставляю инструкцию на выполнение скрипта *copy_out.sh*, которая должна выполниться при запуске контейнера

3. Для автоматизации создаю скрипт *script.sh*, который собирает образ и запускает контейнер из которого затем забирает файл и перемещает его в текущую директорию:
   1. Удаляю текущую папку *out* (если она есть)
   2. Собираю образ с именем *ffmpeg_image*
   3. Запускаю контейнер на основе собранного образа, в котором передаю следующие параметры:
      - `--rm` для автоматического удаления контейнера по завершению
      - `-v $PWD/out:/workspace/out` для монтирования временной директории, через которую обЪектный файл передается на хост
   4. Копирую нужный файл из папки *out*
   5. Удаляю папку *out*

# Анализ уязвимости
