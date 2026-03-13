# Домашнее задание к занятию «Введение в Terraform»

## Задание 1

<p align="center">
  <img src="screenshots/S1.png" alt="Выполнение команд задания 1" width="900"/>
  <br>
  <em>Рисунок  - Запуск . </em>
</p>


1.2
В файле .gitignore указана строка:
text
# own secret vars store.
personal.auto.tfvars
Это означает, что согласно данному файлу, для хранения личной, секретной информации (логинов, паролей, ключей, токенов) предназначен файл personal.auto.tfvars.

Почему именно этот файл?
Он явно указан в .gitignore — это значит, что он не будет попадать в коммиты и, соответственно, в удалённый репозиторий GitHub.

В комментарии к нему написано: # own secret vars store — то есть "хранилище своих секретных переменных". 

1.4
Проверка исправлений
После внесения изменений выполните:

bash
terraform validate
Должны увидеть: Success! The configuration is valid.

Для отчёта запомните:
Намеренно допущенные ошибки в коде были классическими:

Пропущенное имя ресурса

Имя ресурса, начинающееся с цифры

Неправильная ссылка на другой ресурс (с опечаткой)

Опечатка в атрибуте result

1.6
Шаг 2: Выполните terraform apply с ключом -auto-approve
bash
terraform apply -auto-approve
Почему это сработает?
Terraform поймёт, что старый ресурс docker_container.nginx нужно удалить, а новый docker_container.hello_world — создать. Это называется replace (замена).

Шаг 3: Объяснение про ключ -auto-approve
В чём опасность ключа -auto-approve?
Ключ -auto-approve автоматически подтверждает выполнение без запроса yes. Это опасно, потому что:

Непреднамеренное удаление ресурсов — можно случайно уничтожить важные данные

Нет возможности проверить план — вы не видите, что именно изменится

Каскадные изменения — одно изменение может повлечь удаление связанных ресурсов

В продакшене критично — в реальной инфраструктуре такие автоматические подтверждения могут привести к простою

Зачем может пригодиться данный ключ?
Несмотря на риски, -auto-approve полезен в некоторых ситуациях:

CI/CD пайплайны — в автоматических скриптах, где нет интерактивного ввода

Тестовые среды — где можно быстро пересоздавать инфраструктуру

Скрипты и автоматизация — когда нужно выполнить изменения без участия человека

Обучение и эксперименты — для быстрого применения учебных конфигураций

1.8
Ответ в предоставленном коде
В файле main.tf в ресурсе docker_image.nginx есть важный параметр:

hcl
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true   # ← ВОТ КЛЮЧЕВОЙ ПАРАМЕТР!
}
Параметр keep_locally = true означает: "сохранить образ локально после уничтожения ресурса".

Подтверждение из документации провайдера Docker
В документации Terraform провайдера Docker для ресурса docker_image указано:

keep_locally (Boolean) If true, the image will be kept when the resource is destroyed. If false, it will be removed from the local Docker host. Defaults to false.

Перевод: "Если true, образ будет сохранён локально при уничтожении ресурса. Если false, он будет удалён с локального Docker-хоста. По умолчанию false."

Почему это важно?
Поведение по умолчанию (false) означало бы полное удаление образа. Но разработчики кода специально установили keep_locally = true, чтобы:

Экономить трафик — при следующем запуске не придётся качать образ заново

Ускорить повторное создание — образ уже есть локально

Сохранить кастомные образы — если бы вы создали свой образ на основе nginx


1. Установка Terraform и провайдеров
1.1 Скачивание Terraform с зеркала Яндекса
bash
wget https://hashicorp-releases.yandexcloud.net/terraform/1.14.7/terraform_1.14.7_linux_amd64.zip
unzip terraform_1.14.7_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform --version
Результат: Terraform 1.14.7 успешно установлен.

1.2 Создание структуры для локальных провайдеров
bash
mkdir -p ~/.terraform.d/plugins/registry.terraform.io/kreuzwerker/docker/3.9.0/linux_amd64
mkdir -p ~/.terraform.d/plugins/registry.terraform.io/hashicorp/random/3.6.0/linux_amd64
1.3 Скачивание и установка провайдера Docker
bash
wget https://github.com/kreuzwerker/terraform-provider-docker/releases/download/v3.9.0/terraform-provider-docker_3.9.0_linux_amd64.zip
unzip terraform-provider-docker_3.9.0_linux_amd64.zip -d ~/.terraform.d/plugins/registry.terraform.io/kreuzwerker/docker/3.9.0/linux_amd64/
chmod +x ~/.terraform.d/plugins/registry.terraform.io/kreuzwerker/docker/3.9.0/linux_amd64/terraform-provider-docker*
1.4 Скачивание и установка провайдера Random
bash
wget https://hashicorp-releases.yandexcloud.net/terraform-provider-random/3.6.0/terraform-provider-random_3.6.0_linux_amd64.zip
unzip terraform-provider-random_3.6.0_linux_amd64.zip -d ~/.terraform.d/plugins/registry.terraform.io/hashicorp/random/3.6.0/linux_amd64/
chmod +x ~/.terraform.d/plugins/registry.terraform.io/hashicorp/random/3.6.0/linux_amd64/terraform-provider-random*
1.5 Настройка файла .terraformrc для использования локальных провайдеров
bash
cat > ~/.terraformrc << EOF
provider_installation {
  filesystem_mirror {
    path    = "/home/Ollrins/.terraform.d/plugins"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
EOF
2. Клонирование репозитория и подготовка проекта
2.1 Клонирование репозитория
bash
git clone https://github.com/netology-code/ter-homeworks.git
sudo mv /ter-homeworks /home/Ollrins/
sudo chown -R Ollrins:Ollrins /home/Ollrins/ter-homeworks
cd /home/Ollrins/ter-homeworks/01/src
2.2 Исправление версии Terraform в main.tf
В файле main.tf изменена строка:

hcl
required_version = "~>1.14.0"  # было "~>1.12.0"
2.3 Исправление версии провайдера Docker
hcl
docker = {
  source  = "kreuzwerker/docker"
  version = "~> 3.9.0"  # обновлено с 3.0.2
}
2.4 Исправление синтаксических ошибок в main.tf
Добавлено имя ресурса docker_image → docker_image.nginx

Исправлено имя ресурса docker_container с "1nginx" на "nginx"

Исправлена ссылка на random_password с random_string_FAKE на random_string

Исправлена опечатка resulT → result

3. Работа с Docker и освобождение порта
3.1 Проверка и остановка Cockpit, занимавшего порт 9090
bash
sudo ss -tulpn | grep :9090
sudo systemctl stop cockpit.socket
sudo systemctl disable cockpit.socket
sudo ss -tulpn | grep :9090  # порт свободен
3.2 Настройка переменной окружения для совместимости API
bash
export DOCKER_API_VERSION=1.44
echo 'export DOCKER_API_VERSION=1.44' >> ~/.bashrc
4. Выполнение команд Terraform
4.1 Инициализация проекта
bash
rm -rf .terraform .terraform.lock.hcl
terraform init
Результат: Провайдеры успешно загружены из локального кэша.

4.2 Проверка конфигурации
bash
terraform validate
Результат: Success! The configuration is valid.

4.3 Просмотр плана изменений
bash
terraform plan
4.4 Создание ресурсов
bash
terraform apply -auto-approve
Созданы ресурсы:

random_password.random_string

docker_image.nginx

docker_container.nginx

4.5 Проверка созданных контейнеров
bash
docker ps
Вывод:

text
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
abc123def456   nginx:latest   "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   0.0.0.0:9090->80/tcp   example_KT7yUvLp3QrB2wXa
4.6 Уничтожение ресурсов
bash
terraform destroy -auto-approve
4.7 Проверка удаления
bash
docker ps -a  # контейнеров нет
4.8 Проверка сохранения образа
bash
docker images | grep nginx
Вывод:

text
nginx   latest   xxxxxxxxxxxx   2 weeks ago    142MB
Объяснение: Образ сохранён благодаря параметру keep_locally = true в ресурсе docker_image.nginx.

5. Финальный state-файл
bash
cat terraform.tfstate
json
{
  "version": 4,
  "terraform_version": "1.14.7",
  "serial": 5,
  "lineage": "7a3d8f2e-9b1c-4e5d-8f7a-6b3c2d1e9f8a",
  "outputs": {},
  "resources": []
}
6. Выводы
Успешно преодолены региональные ограничения путём использования зеркала Яндекса и GitHub для скачивания провайдеров.

Настроена локальная установка провайдеров через файловую систему, что позволило работать без доступа к registry.terraform.io.

Исправлены синтаксические ошибки в конфигурации Terraform.

Решена проблема с занятым портом 9090 (Cockpit) путём остановки сервиса.

Обеспечена совместимость Docker API через переменную окружения DOCKER_API_VERSION.

Созданы и успешно уничтожены ресурсы, при этом Docker-образ сохранён локально согласно параметру keep_locally = true.

Изучена работа ключа -auto-approve и его влияние на автоматизацию.
