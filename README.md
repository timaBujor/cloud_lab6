# Лабораторная работа №6  
# Балансирование нагрузки в облаке и авто-масштабирование (AWS)

---

## Цель работы

Цель работы — закрепить навыки работы с AWS EC2, Elastic Load Balancer, Auto Scaling и CloudWatch и построить отказоустойчивую и масштабируемую архитектуру.

---

## Итоговая архитектура

В рамках лабораторной работы развернуто:

- VPC с публичными и приватными подсетями  
- EC2 инстанс с nginx  
- Application Load Balancer (ALB)  
- Auto Scaling Group  
- CloudWatch мониторинг и алерты  
- нагрузочное тестирование  

---

# Шаг 1. Создание VPC и подсетей

Использовал VPC созданный в предыдущей лабораторной работе

---

# Шаг 2. Создание EC2 и веб-сервера

Развернута виртуальная машина:

| Параметр | Значение |
|---|---|
| AMI | Amazon Linux 2 |
| Instance Type | t3.micro |
| Public IP | Enabled |

---

## Security Group

### Входящие правила

| Тип | Порт | Источник |
|---|---|---|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0 |

### Исходящие правила

| Тип | Порт | Назначение |
|---|---|---|
| ALL | ALL | 0.0.0.0/0 |

---

## CloudWatch Monitoring

Включено:

- Detailed CloudWatch Monitoring = Enabled  

---

## Проверка

После запуска EC2 веб-сервер nginx доступен по публичному IP.

<p align="center">
  <img src="https://github.com/user-attachments/assets/384cdd1d-ea77-4e05-9cf5-c26f21f4cd96" width="900"/>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/77b7a716-11a8-4ffb-afd2-6199093227eb" width="900"/>
</p>

---

## Контрольный вопрос

### Что такое UserData и зачем он используется?

### Ответ

UserData — это скрипт, который выполняется при первом запуске EC2-инстанса.

Используется для:
- автоматической установки ПО;
- настройки окружения;
- запуска сервисов (например nginx);
- ускорения деплоя инфраструктуры.

---

# Шаг 3. Создание AMI

Создан образ:

`project-web-server-ami`

<p align="center">
  <img src="https://github.com/user-attachments/assets/4d2bbf13-e435-4d13-9be5-0ff386a1a0ec" width="900"/>
</p>

---

## Контрольный вопрос

### Что такое AMI и чем она отличается от snapshot? Какие есть варианты использования AMI?

### Ответ

AMI (Amazon Machine Image) — это готовый образ виртуальной машины.

Отличие:
- Snapshot = диск
- AMI = полный образ сервера

Использование:
- масштабирование  
- backup  
- автоматизация инфраструктуры  

---

# Шаг 4. Launch Template

Создан шаблон:

`project-launch-template`

<p align="center">
  <img src="https://github.com/user-attachments/assets/730dc9be-a4ea-4473-b40d-2c452772c148" width="900"/>
</p>

---

## Контрольный вопрос

### Что такое Launch Template и чем он отличается от Launch Configuration?

### Ответ

Launch Template — современный способ описания конфигурации EC2.

Отличия:
- поддерживает версии  
- более гибкий  
- используется в Auto Scaling  

---

# Шаг 5. Target Group

Создана Target Group:

`project-target-group`

<p align="center">
  <img src="https://github.com/user-attachments/assets/7ba6bcde-54dd-49ba-a744-6b5e2672aac3" width="900"/>
</p>

---

## Контрольный вопрос

### Зачем нужна Target Group?

### Ответ

Target Group — это группа серверов, на которые балансировщик распределяет трафик.

---

# Шаг 6. Application Load Balancer

Создан Load Balancer:

`project-alb`

<p align="center">
  <img src="https://github.com/user-attachments/assets/a9023704-f04f-4cd5-b8ba-ec63e5fb72bf" width="900"/>
</p>

---

## Контрольный вопрос

### Разница Internet-facing и Internal?

### Ответ

- Internet-facing → доступен из интернета  
- Internal → доступен только внутри VPC  

---

## Default Action

### Ответ

Default Action — действие ALB, если не подошло правило маршрутизации.

Варианты:
- forward  
- redirect  
- fixed-response  

---

# Шаг 7. Auto Scaling Group

Создана группа:

`project-auto-scaling-group`

<p align="center">
  <img src="https://github.com/user-attachments/assets/8d6d35bf-a506-48ff-8b8e-d83d9576b672" width="900"/>
</p>

---

## Конфигурация

| Параметр | Значение |
|---|---|
| Min | 2 |
| Max | 4 |
| Desired | 2 |

---

## Контрольный вопрос

### Почему используются приватные подсети?

### Ответ

Для безопасности:
- нет прямого доступа из интернета  
- доступ только через ALB  

---

## Availability Zone distribution

### Ответ

Распределяет инстансы по AZ для отказоустойчивости.

---

## Instance Warm-up Period

### Ответ

Время, необходимое новому инстансу для стабилизации перед учётом метрик.

---

# Шаг 8. Тестирование Load Balancer

<p align="center">
  <img src="https://github.com/user-attachments/assets/7b49291e-6da5-4a10-a9ab-631acd5a3e72" width="900"/>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/c4dc1838-c6eb-41e6-a8e3-8b0d63dad25e" width="900"/>
</p>

---

## Контрольный вопрос

### Какие IP-адреса отображаются и почему?

### Ответ

Разные IP EC2-инстансов, потому что ALB распределяет нагрузку.

---

# Шаг 9. Auto Scaling тест

<p align="center">
  <img src="https://github.com/user-attachments/assets/d426410d-7432-4723-a21e-6542426bc529" width="900"/>
</p>

---

## Контрольный вопрос

### Роль Auto Scaling?

### Ответ

Auto Scaling автоматически:
- добавляет инстансы при нагрузке  
- удаляет при снижении нагрузки  
- балансирует систему  

---

# Шаг 10. Очистка ресурсов

Удалено:
- ALB  
- Target Group  
- Auto Scaling Group  
- EC2  
- AMI  
- Launch Template  
- VPC  

<p align="center">
  <img src="https://github.com/user-attachments/assets/57dd6fce-8c61-4fc0-9b67-69593f4328f6" width="900"/>
</p>

---

# Вывод

Реализована масштабируемая архитектура AWS:

- Load Balancing  
- Auto Scaling  
- Cloud Monitoring  
- EC2 infrastructure  

Система автоматически адаптируется под нагрузку и обеспечивает отказоустойчивость.
