-- PostgreSQL
-- Тематика: банковские счета клиентов

'''sql
## Задание 1

Найдите клиентов, у которых счет выглядит интересным для менеджера.

Счет можно считать таким, если он активен, город клиента известен, а баланс выше среднего баланса по всем активным счетам.

Выведите основную информацию о клиенте и счете.

Подсказка: пригодится подзапрос для расчета среднего баланса.

'''sql

## Решение

```sql
SELECT client_name,
  city,
  balance,
  status
FROM bank_accounts
WHERE city IS NOT NULL
AND status = 'active'
AND balance > (SELECT AVG(balance)
  FROM bank_accounts
  WHERE status = 'active')
ORDER BY balance DESC;
---

## Задание 2

Покажите города, где у банка есть не меньше трех счетов.

Для каждого такого города нужно примерно оценить клиентскую базу:

- сколько всего счетов;
- какая общая сумма балансов;
- какой средний баланс;
- сколько счетов активны.

Города без названия не учитывайте.

Отсортируйте результат так, чтобы города с большим количеством счетов были выше.

## Решение
```sql
SELECT
  city,
  COUNT(account_type) AS total_accounttype,
  COUNT(CASE WHEN status = 'active'THEN 1 END) AS active_status,
  SUM(COALESCE(balance, 0)) AS total_balance,
  AVG(COALESCE(balance, 0)) AS avg_balance
FROM bank_accounts
WHERE city IS NOT NULL
GROUP BY city
HAVING COUNT(account_type) > 3
ORDER BY total_accounttype DESC;
--

## Задание 3

Найдите счета, по которым давно не было операций.

Счет стоит проверить, если последняя операция отсутствует или была раньше `2024-03-01`.

Закрытые счета можно исключить.

В результат добавьте понятный текстовый признак причины, например:

- `нет операций`;
- `операция была давно`.

Подсказка: используйте `CASE`.

---
## Решение
```sql
SELECT
  account_type,
  status,
  last_transaction,
  CASE
    WHEN last_transaction > DATE '2024-03-01'THEN 'Недавняя операция'
    WHEN last_transaction < DATE '2024-03-01'THEN 'Операция была давно'
    ELSE 'Статус неизвестен'
  END AS transaction
FROM bank_accounts
WHERE status ='active'
AND last_transaction IS NOT NULL
ORDER BY transaction;

## Задание 4

Оцените клиентский портфель каждого менеджера.

Для каждого менеджера нужно получить краткую сводку:

- сколько всего счетов он ведет;
- сколько среди них активных;
- какая общая сумма балансов;
- какой средний баланс;
- сколько клиентов без указанного города.

Если менеджер не указан, покажите такие счета отдельной группой, например `Без менеджера`.

Подсказка: используйте `COALESCE` и условную агрегацию.

## Решение
```sql

SELECT
  COALESCE(manager_name,'Без менеджера') AS manager_total,
  COUNT(*) AS total_accounttype,
  COUNT(CASE WHEN status = 'active'THEN 1 END) AS active_status,
  SUM(COALESCE(balance, 0)) AS total_balance,
  AVG(COALESCE(balance, 0)) AS avg_balance,
  COUNT(CASE WHEN city IS NULL THEN 1 END) AS client_with_nocity
FROM bank_accounts
GROUP BY manager_name
ORDER BY manager_total;

---

## Задание 5

Найдите кредитные счета с повышенным риском.

Счет можно считать рискованным, если:

- это кредитный счет;
- кредитный лимит указан;
- баланс отрицательный;
- использовано больше 60% кредитного лимита.

Добавьте расчет процента использования лимита.

Также можно добавить уровень риска:

- больше 90% — `высокий`;
- от 75% до 90% — `средний`;
- от 60% до 75% — `умеренный`.

Отсортируйте результат по проценту использования лимита по убыванию.

## Решение
```sql
SELECT
  account_id,
  credit_limit,
  balance,
  ROUND((ABS(balance) * 100.0 / credit_limit), 2) AS credit_percent,
  CASE
    WHEN (ABS(balance) * 100.0 / credit_limit) > 90 THEN 'высокий'
    WHEN (ABS(balance) * 100.0 / credit_limit) BETWEEN 75 AND 90 THEN 'средний'
    WHEN (ABS(balance) * 100.0 / credit_limit) BETWEEN 60 AND 75 THEN 'умеренный'
    ELSE 'низкий'
  END AS risk_level
FROM bank_accounts
WHERE
  account_type = 'credit'
  AND credit_limit IS NOT NULL
  AND credit_limit > 0
  AND balance < 0
  AND (ABS(balance) * 100.0 / credit_limit) > 60
ORDER BY credit_percent DESC;# -
