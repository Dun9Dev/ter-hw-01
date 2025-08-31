# Домашнее задание к занятию «Введение в Terraform»

### Цели задания

1. Установить и настроить Terrafrom.
2. Научиться использовать готовый код.

------

### Чек-лист готовности к домашнему заданию

1. Скачайте и установите **Terraform** версии >=1.8.4 . Приложите скриншот вывода команды ```terraform --version```.
   ![PNG](https://github.com/Dun9Dev/ter-hw-01/blob/main/img/Screenshot_2.png)
3. Скачайте на свой ПК этот git-репозиторий. Исходный код для выполнения задания расположен в директории **01/src**.
4. Убедитесь, что в вашей ОС установлен docker.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Репозиторий с ссылкой на зеркало для установки и настройки Terraform: [ссылка](https://github.com/netology-code/devops-materials).
2. Установка docker: [ссылка](https://docs.docker.com/engine/install/ubuntu/). 
------
### Внимание!! Обязательно предоставляем на проверку получившийся код в виде ссылки на ваш github-репозиторий!
------

### Задание 1

1. Перейдите в каталог [**src**](https://github.com/netology-code/ter-homeworks/tree/main/01/src). Скачайте все необходимые зависимости, использованные в проекте. 
2. Изучите файл **.gitignore**. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)
   - Согласно .gitignore, секреты можно хранить в файле personal.auto.tfvars, так как он не попадает в репозиторий:
     # own secret vars store.
     personal.auto.tfvars
4. Выполните код проекта. Найдите  в state-файле секретное содержимое созданного ресурса **random_password**, пришлите в качестве ответа конкретный ключ и его значение.
    - "result": "gwypvAOKOxqXo27j"
6. Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла **main.tf**.
Выполните команду ```terraform validate```. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.
  - В закомментированном блоке кода были намеренно допущены следующие ошибки:

 1. У ресурса `docker_image` отсутствовало имя, из-за чего невозможно сослаться на него в `docker_container`.  
    **Исправление:** добавлено имя `nginx`: `resource "docker_image" "nginx"`

 2. Имя ресурса `docker_container "1nginx"` начиналось с цифры, что недопустимо в Terraform.  
   **Исправление:** заменено на `nginx_container`.

 3. В выражении `random_password.random_string_FAKE.resulT` использовалось несуществующее имя ресурса (`_FAKE`) и неправильное название атрибута (`resulT` вместо `result`).  
    **Исправление:** заменено на `random_password.random_string.result`.
 После исправлений `terraform validate` прошёл успешно.

8. Выполните код. В качестве ответа приложите: исправленный фрагмент кода и вывод команды ```docker ps```.
   ![PNG](https://github.com/Dun9Dev/ter-hw-01/blob/main/img/Screenshot_3.png)
   ```
   resource "docker_image" "nginx" {
    name         = "nginx:latest"
    keep_locally = true
    }

    resource "docker_container" "nginx_container" {
     name  = "hello_world"
    image = docker_image.nginx.image_id

    ports {
      internal = 80
      external = 9090
    }
    }
   ```
10. Замените имя docker-контейнера в блоке кода на ```hello_world```. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду ```terraform apply -auto-approve```.
Объясните своими словами, в чём может быть опасность применения ключа  ```-auto-approve```. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды ```docker ps```.
![PNG](https://github.com/Dun9Dev/ter-hw-01/blob/main/img/Screenshot_8.png)
Ключ -auto-approve пропускает интерактивное подтверждение при выполнении terraform apply.
Вместо того чтобы ждать, пока пользователь введёт yes, Terraform автоматически применяет все изменения.

Почему это опасно? 

Может привести к случайному удалению или изменению критических ресурсов (например, ВМ с базой данных, S3-бакеты с данными).
В продакшене (production) такое поведение может вызвать простои, потерю данных или финансовые потери.
Нет возможности "передумать" — команда выполняется мгновенно.
Пример:
Если terraform plan показывает удаление 5 ВМ, а ты не заметил — с -auto-approve они будут удалены без предупреждения.

12. Уничтожьте созданные ресурсы с помощью **terraform**. Убедитесь, что все ресурсы удалены. Приложите содержимое файла **terraform.tfstate**.
    ![PNG](https://github.com/Dun9Dev/ter-hw-01/blob/main/img/Screenshot_9.png)
14. Объясните, почему при этом не был удалён docker-образ **nginx:latest**. Ответ **ОБЯЗАТЕЛЬНО НАЙДИТЕ В ПРЕДОСТАВЛЕННОМ КОДЕ**, а затем **ОБЯЗАТЕЛЬНО ПОДКРЕПИТЕ** строчкой из документации [**terraform провайдера docker**](https://docs.comcloud.xyz/providers/kreuzwerker/docker/latest/docs).  (ищите в классификаторе resource docker_image )
    -браз nginx:latest не был удалён при выполнении terraform destroy, потому что в ресурсе docker_image установлено:
```
keep_locally = true
```
Это свойство предотвращает удаление образа с локальной машины, даже если ресурс уничтожается.

Согласно документации провайдера Docker для Terraform:

keep_locally (Boolean) If true, then the Docker image won't be deleted on destroy operation. 

🔗 https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs/resources/image#keep_locally 


------
