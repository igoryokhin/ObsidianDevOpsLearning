---
aliases:
  - "# Topic: Linux Permissions & Users"
  - "#linux #security #permissions"
tags:
  - "#linux"
  - "#security"
  - "#permissions"
---
[[Таблица прав доступа Linux]]
## Определение

**Права доступа** в Linux — это атрибуты файла, определяющие полномочия для владельца (Owner), группы (Group) и остальных (Others). Обозначаются как `rwx`.

### Таблица весов (Numeric mode)
- `4` — Чтение (Read)
- `2` — Запись (Write)
- `1` — Выполнение (Execute)
- *Пример: 7 (4+2+1), 5 (4+0+1)*

> [!example] Работа с правами
> - `chmod 600 key.pem` — доступ только владельцу (чтение/запись).
> - `chown -R www-data:www-data /var/www` — рекурсивная смена владельца.
> - `sudo !!` — повторить последнюю команду с правами root.

