### FreeIPA
Сложить файлы в одну папку и запустить
```Bash
vagrant up
```

Добавлены сторонние репозитории для установки сервера и клиента
```Bash
cd /vagrant/ansible
```
```Bash
ansible-playbook -i hosts provision.yml
```
<img width="1680" height="323" alt="image" src="https://github.com/user-attachments/assets/97912c2d-631d-44d2-8be9-78879566d27f" />

Создан пользователь *otus-user*
