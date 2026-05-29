# Ansible Playbook: ClickHouse + Vector
## Самостоятельное практическое задание "Работа с Playbook"

## Описание

Данный Ansible playbook устанавливает и конфигурирует следующее:

* ClickHouse server
* Vector log collector
* ClickHouse database `logs`
* Vector configuration template

Скрипт предназначен для систем на базе Rocky Linux / RHEL с использованием `dnf`.

---

## Project Structure

```text
.
├── inventory/
│   └── prod.yml
├── group_vars/
│   └── clickhouse/
│       └── vars.yml
├── templates/
│   └── vector.yaml
├── playbook/
│   └── site.yml
└── README.md
```

---

## Requirements

* Ansible >= 2.15
* Rocky Linux / CentOS / RHEL 9
* SSH access to managed hosts
* Sudo privileges

---

## Inventory Example

```yaml
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 192.168.1.207
      ansible_user: dops
      ansible_port: 22
      ansible_ssh_private_key_file: ~/.ssh/id_ed25519
```

---

## Variables

| Variable              | Description                                 | Default         |
| --------------------- | ------------------------------------------- | --------------- |
| `clickhouse_version`  | Версия пакета ClickHouse                    | `22.3.3.44`     |
| `clickhouse_packages` | Список пакетов ClickHouse                   | смотри vars     |
| `vector_arch`         | Vector архитектура пакета                   | `x86_64`        |
| `user_pass`           | Пароль Sudo, сохраненный в in Ansible Vault | vault encrypted |

Пример:

```yaml
clickhouse_version: "22.3.3.44"

clickhouse_packages:
  - clickhouse-client
  - clickhouse-server
  - clickhouse-common-static

vector_arch: "x86_64"
```

---

## Features

### ClickHouse

Playbook выполняет:

* установку `yum-utils`
* настройку репозитория
* установку пакета ClickHouse
* перезапуск и включение службы
* ожидание открытия TCP-порта `9000`
* создание базы данных `logs`

### Vector

Playbook выполняет:

* загрузку RPM-пакета Vector
* установку Vector
* создание каталога конфигурации
* развертывание шаблона конфигурации Vector
* Перезапуск векторной службы

---

## Выполнение проекта

Run playbook:

```bash
ansible-playbook -i inventory/prod.yml playbook/site.yml --ask-vault-pass
```

---

## Ansible Vault

Переменная `user_pass` зашифрована с помощью Ansible Vault.

Edit vault:

```bash
ansible-vault edit group_vars/clickhouse/vars.yml
```

Encrypt file:

```bash
ansible-vault encrypt group_vars/clickhouse/vars.yml
```

---

## Handlers

### ClickHouse Handler

Restarts and enables:

* `clickhouse-server`

### Vector Handler

Restarts and enables:

* `vector`

---

## Notes

* Vector configuration is deployed to:

```text
/etc/vector/vector.yaml
```

* ClickHouse database created by playbook:

```text
logs
```

* Services are managed via `systemd`.
