# vpn_infra

Ansible-плейбук, который готовит Ubuntu-сервер под VPN-выход: SSH только по ключу, ufw (22 и 443), автообновления, BBR, панель 3x-ui с ежедневным бэкапом базы.

В репозитории только код. IP серверов, пользователи и пароли в git не хранятся: локально они лежат в файлах из `.gitignore`, для GitHub Actions — в Secrets.

## Что нужно

- Сервер на Ubuntu, пользователь с sudo, вход по SSH-ключу.
- Для запуска с компьютера: Ansible (`brew install ansible` или `pipx install --include-deps ansible`).

## Запуск через GitHub Actions

1. Сделать форк репозитория.
2. Создать отдельный SSH-ключ и добавить его на сервер:
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/vpn_deploy -N "" -C "github-actions"
   ssh-copy-id -i ~/.ssh/vpn_deploy.pub <пользователь>@<IP>
   ```
3. Settings → Secrets and variables → Actions → New repository secret:

   | Секрет | Что положить |
   |---|---|
   | `SSH_PRIVATE_KEY` | содержимое `~/.ssh/vpn_deploy` |
   | `KNOWN_HOSTS` | вывод `ssh-keyscan -t ed25519 <IP>` |
   | `INVENTORY` | содержимое по образцу `ansible/inventory.example.ini` |
   | `SECRET_VARS` | содержимое по образцу `ansible/vault.example.yml` |
   | `BECOME_PASSWORD` | пароль sudo пользователя на сервере |

4. Actions → deploy → Run workflow.

## Запуск с компьютера

```bash
cd ansible
cp inventory.example.ini inventory.ini                # вписать свой сервер
cp vault.example.yml group_vars/all/vault.yml         # вписать значения
ansible-vault encrypt group_vars/all/vault.yml
ansible all -m ping -J
ansible-playbook site.yml -K -J
```

Чтобы не вводить пароль vault каждый раз, положите его в файл с правами 600 и укажите путь в переменной окружения:
`export ANSIBLE_VAULT_PASSWORD_FILE=~/.config/vpn-infra/vault_pass`

## Панель 3x-ui

Порт панели снаружи закрыт. Вход через SSH-туннель:
```bash
ssh -L 2053:127.0.0.1:2053 <пользователь>@<IP>
```
Затем в браузере: `http://localhost:2053/<vault_xui_web_base_path>/`.

