### diplom-ansible

Ansible-конфигурация настройки Ubuntu VM и первоначального
развёртывания приложения дипломного проекта.

## Назначение

- Установка и настройка Docker.
- Подготовка Docker Compose-проекта.
- Запуск веб-приложения.
- Проверка результата развёртывания.

Управляющий хост: Debian 13.
Целевая ВМ: Ubuntu.
SSH-пользователь: `ubuntu`.

## Основные файлы

- `ansible.cfg` — настройки Ansible.
- `requirements.yml` — необходимые коллекции.
- `inventory/hosts.ini.example` — пример inventory.
- `inventory/hosts.ini` — локальный рабочий inventory, не публикуется.
- `inventory/group_vars/web.yml` — параметры группы web.
- `templates/compose.yaml.j2` — шаблон Compose-конфигурации.
- `playbooks/docker.yml` — настройка Docker.
- `playbooks/deploy.yml` — развёртывание приложения.
- `playbooks/site.yml` — общий playbook.

## Подготовка

Команды выполняются из корня репозитория:

```bash
cd /root/diplom/ansible
ansible-galaxy collection install -r requirements.yml
```

При использовании Terraform-сценария рабочий inventory
формируется автоматически из output `vm_external_ip`.
Для самостоятельного запуска необходимо скопировать:

```bash
cp inventory/hosts.ini.example inventory/hosts.ini
```

Далее указываем актуальный IP:

```ini
[web]
diplom-vm ansible_host=<актуальный-IP-ВМ> ansible_user=ubuntu
```

До запуска должна быть завершена первоначальная настройка cloud-init,
и должна быть проверена подлинность SSH host key должна быть проверена.

## Запуск

```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml
```

## Результат

- Приложение размещается в `/home/ubuntu/diplom`.
- Используется образ `zooombooom/diplom:latest`.
- Compose-сервис называется `web`.
- Контейнер называется `diplom-web`.
- Приложение доступно через порт 8080 целевой ВМ.

При успешном выполнении в PLAY RECAP:

```text
unreachable=0
failed=0
```

## Повторное применение

На текущем стенде повторный запуск завершился с результатом:

```text
ok=14 changed=0 unreachable=0 failed=0
```

## Разделение ответственности

Terraform создаёт инфраструктуру.
Ansible настраивает сервер и разворачивает приложение.
GitHub Actions доставляет последующие версии приложения.

Ansible и CI/CD используют один каталог Compose-проекта.
Их нельзя запускать одновременно во время изменения приложения.

## Безопасность

Реальный inventory хранится локально и исключён из Git.
В репозиторий добавляется только пример inventory.

Приватные SSH-ключи, токены и credentials не публикуются.
