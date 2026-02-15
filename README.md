# Домашнее задание к занятию «Введение в Terraform»
Скачайте и установите Terraform версии >=1.12.0 . Приложите скриншот вывода команды terraform --version

![Alt text](images/version.jpg)

## Задача 1
1. Перейдите в каталог src. Скачайте все необходимые зависимости, использованные в проекте.
```
cp .terraformrc ~/
```
```
terrafrom init
```

![Alt text](images/1.1-1.jpg)

![Alt text](images/1.1-2.jpg)

2. Изучите файл .gitignore. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)
```
personal.auto.tfvars
```
3. Выполните код проекта. Найдите в state-файле секретное содержимое созданного ресурса random_password, пришлите в качестве ответа конкретный ключ и его значение.
```
"result": "nDkMwN7njaqfDr4E",
```

![Alt text](images/1.3.jpg)

4. Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла main.tf. Выполните команду terraform validate. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.

Отсутствует имя для ресурса
Все блоки ресурсов должны иметь 2 метки (тип, имя).
Для docker_image надо добавить имя, например: resource "docker_image" "nginx" {

Недопустимое имя ресурса
Имя должно начинаться с буквы или подчеркивания и может содержать только буквы, цифры, подчеркивания и дефисы.
Для "docker_container" "nginx1" надо изменить имя на допустимое, например: "nginx".

random_string_FAKE — неверное имя ресурса (есть только random_string)
resulT — опечатка, должно быть result
Исправление: name  = "example_${random_password.random_string.result}"

5. Выполните код. В качестве ответа приложите: исправленный фрагмент кода и вывод команды docker ps

![Alt text](images/1.5-1.jpg)

```
terrafrom apply
```

![Alt text](images/1.5-2.jpg)

![Alt text](images/1.5-3.jpg)

![Alt text](images/1.5-4.jpg)

6. Замените имя docker-контейнера в блоке кода на hello_world. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду terraform apply -auto-approve. Объясните своими словами, в чём может быть опасность применения ключа -auto-approve. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды docker ps

![Alt text](images/1.6.jpg)

Команда terraform apply -auto-approve автоматически применяет изменения в инфраструктуре без запроса подтверждения.

Опасность ключа -auto-approve
Этот флаг обходит обязательный шаг ручного подтверждения плана изменений, показанного Terraform перед применением. В результате можно случайно уничтожить ресурсы (например, удалить базу данных), внести нежелательные модификации или потерять данные из-за ошибок в конфигурации, не успев их заметить. Особенно рискованно в production-средах, где изменения влияют на реальную инфраструктуру.

Зачем нужен ключ:
Он упрощает автоматизацию: идеален для CI/CD-пайплайнов, скриптов и неинтерактивных окружений (dev/test), где ручное подтверждение замедляет процесс.

7. Уничтожьте созданные ресурсы с помощью terraform. Убедитесь, что все ресурсы удалены. Приложите содержимое файла terraform.tfstate.
```
terraform destroy
```
![Alt text](images/1.7-1.jpg)

![Alt text](images/1.7-2.jpg)

8. Объясните, почему при этом не был удалён docker-образ nginx:latest. Ответ ОБЯЗАТЕЛЬНО НАЙДИТЕ В ПРЕДОСТАВЛЕННОМ КОДЕ, а затем ОБЯЗАТЕЛЬНО ПОДКРЕПИТЕ строчкой из документации terraform провайдера docker. (ищите в классификаторе resource docker_image )

Образ не удален из локального хранилища Docker при операции destroy, т.к. keep_locall = true. 
Следует из документации: https://library.tf/providers/kreuzwerker/docker/latest/docs/resources/image

keep_locally (Boolean) If true, then the Docker image won't be deleted on destroy operation. If this is false, it will delete the image from the docker local storage on destroy operation.

![Alt text](images/1.8.jpg)

## Задача 2
1. Создайте в облаке ВМ. Сделайте это через web-консоль, чтобы не слить по незнанию токен от облака в github(это тема следующей лекции). Если хотите - попробуйте сделать это через terraform, прочитав документацию yandex cloud. Используйте файл personal.auto.tfvars и гитигнор или иной, безопасный способ передачи токена!

![Alt text](images/2.1.jpg)

2. Подключитесь к ВМ по ssh и установите стек docker.

![Alt text](images/2.2.jpg)

3. Найдите в документации docker provider способ настроить подключение terraform на вашей рабочей станции к remote docker context вашей ВМ через ssh.

https://docs.comcloud.xyz/providers/kreuzwerker/docker/latest/docs#remote-hosts

Файл ~/.ssh/conf 
```
Host vm-yandex
  HostName Имя ВМ в облаке
  User alex
  Port 22
  IdentityFile ~/.ssh/id_ed25519
  StrictHostKeyChecking no
```

4. Используя terraform и remote docker context, скачайте и запустите на вашей ВМ контейнер mysql:8 на порту 127.0.0.1:3306, передайте ENV-переменные. Сгенерируйте разные пароли через random_password и передайте их в контейнер, используя интерполяцию из примера с nginx.(name  = "example_${random_password.random_string.result}" , двойные кавычки и фигурные скобки обязательны!)
    environment:
      - "MYSQL_ROOT_PASSWORD=${...}"
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - "MYSQL_PASSWORD=${...}"
      - MYSQL_ROOT_HOST="%"
  
![Alt text](images/2.4.jpg)
    
5. Зайдите на вашу ВМ , подключитесь к контейнеру и проверьте наличие секретных env-переменных с помощью команды env. Запишите ваш финальный код в репозиторий.

![Alt text](images/2.5.jpg)

[main.tf](./main.tf)
```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
    }
    random = {
      source  = "hashicorp/random"
    }
  }
}

provider "docker" {
  host = "ssh://vm-yandex"
}

resource "random_password" "mysql_root" {
  length  = 16
  special = true
}

resource "random_password" "mysql_user" {
  length  = 16
  special = true
}

resource "docker_image" "mysql" {
  name = "mysql:8"
}

resource "docker_container" "mysql" {
  image = docker_image.mysql.image_id
  name  = "mysql-wordpress"

  ports {
    internal = 3306
    external = 3306
    ip     = "127.0.0.1"
  }

  env = [
    "MYSQL_ROOT_PASSWORD=${random_password.mysql_root.result}",
    "MYSQL_DATABASE=wordpress",
    "MYSQL_USER=wordpress", 
    "MYSQL_PASSWORD=${random_password.mysql_user.result}",
    "MYSQL_ROOT_HOST=%"
  ]

  restart = "always"
}
```



