#!/bin/bash

# Проверка прав администратора
if [ "$(whoami)" != "root" ]; then
    echo "Ошибка: скрипт должен запускаться от root или через sudo!"
    exit 1
fi

# Проверка подключения к FreeIPA
if ! ipa user-show admin &>/dev/null; then
    echo "Ошибка: не удалось подключиться к FreeIPA. Проверьте аутентификацию (kinit admin)."
    exit 1
fi

# Пароль для новых пользователей
USER_PASSWORD="Secret123!"

# Создание 3 групп
for i in {1..3}; do
    if ! ipa group-show "group$i" &>/dev/null; then
        ipa group-add "group$i" --desc="Группа номер $i"
        echo "Создана группа group$i"
    else
        echo "Группа group$i уже существует, пропускаем"
    fi
done

# Создание 30 пользователей и добавление в группы
for i in {1..30}; do
    username="user$i"
    
    # Проверяем, существует ли пользователь
    if ! ipa user-show "$username" &>/dev/null; then
        echo "$USER_PASSWORD" | ipa user-add "$username" --first="$username" --last="User" --password
        echo "Создан пользователь $username"
    else
        echo "Пользователь $username уже существует, пропускаем"
        continue
    fi

    # Определяем группу (1, 2 или 3)
    group_num=$(( (i - 1) / 10 + 1 ))
    ipa group-add-member "group$group_num" --users="$username"
    echo "Пользователь $username добавлен в group$group_num"
done

echo "Готово! Проверьте пользователей и группы:"
echo "  ipa user-find"
echo "  ipa group-show group1"
