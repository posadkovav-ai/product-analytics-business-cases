# product-analytics-business-cases

# 📈 Проект №1: Мониторинг роста пользователей и курьеров

> Анализ баланса между двумя сторонами платформы: покупателями и службой доставки

## 🎯 Бизнес-задача

В интернет-магазине с доставкой критически важно отслеживать баланс между двумя сторонами платформы: **покупателями** и **курьерами**.  

- Если пользователей становится больше, а курьеров нет → падает скорость доставки и качество сервиса  
- Если курьеров слишком много → растут издержки без увеличения выручки  

**Цель проекта:** создать систему мониторинга, которая ежедневно показывает:
- ✅ сколько новых пользователей приходит в сервис
- ✅ сколько новых курьеров подключается
- ✅ как меняется общая база тех и других с течением времени

**Главный бизнес-вопрос:** *не отстаёт ли найм курьеров от роста аудитории?*

---

## 🛠 Что сделано

### 1. SQL-запрос

```sql
-- Ежедневная динамика новых пользователей и курьеров с накоплением
WITH all_dates AS (
    SELECT DISTINCT DATE(time) AS date FROM user_actions
    UNION
    SELECT DISTINCT DATE(time) AS date FROM courier_actions
),
first_user_date AS (
    SELECT user_id, MIN(DATE(time)) AS first_date
    FROM user_actions
    GROUP BY user_id
),
new_users_daily AS (
    SELECT first_date AS date, COUNT(*) AS new_users
    FROM first_user_date
    GROUP BY first_date
),
first_courier_date AS (
    SELECT courier_id, MIN(DATE(time)) AS first_date
    FROM courier_actions
    GROUP BY courier_id
),
new_couriers_daily AS (
    SELECT first_date AS date, COUNT(*) AS new_couriers
    FROM first_courier_date
    GROUP BY first_date
)
SELECT 
    d.date,
    COALESCE(nu.new_users, 0)::INTEGER AS new_users,
    COALESCE(nc.new_couriers, 0)::INTEGER AS new_couriers,
    SUM(COALESCE(nu.new_users, 0)) OVER (ORDER BY d.date)::INTEGER AS total_users,
    SUM(COALESCE(nc.new_couriers, 0)) OVER (ORDER BY d.date)::INTEGER AS total_couriers
FROM all_dates d
LEFT JOIN new_users_daily nu ON d.date = nu.date
LEFT JOIN new_couriers_daily nc ON d.date = nc.date
ORDER BY d.date;
```
## 🛠 Ключевые технические решения

- 🧩 CTE для декомпозиции сложной логики
- 🔗 UNION для объединения дат из двух таблиц
- 📊 Оконная функция SUM() OVER() для накопительных итогов
- 🛡️ COALESCE для обработки дней без новых регистраций

---

## 📊 Дашборд

Дашборд включает:

- 📈 Линейный график `new_users` и `new_couriers` по дням
- 📊 График накопленных `total_users` и `total_couriers`
- 🎛️ Фильтр по периодам (месяц, квартал)
- 🛡️ Отношение всех пользователей к новым курьерам

<img width="1771" height="806" alt="image" src="https://github.com/user-attachments/assets/e7859740-9c91-4cd5-92a2-5340962f19e3" />


---

## 📈 Ключевые инсайты

| Инсайт | Бизнес-интерпретация |
|--------|----------------------|
| 🔴 После рекламных кампаний `new_users` резко растёт, а `new_couriers` остаётся на прежнем уровне | Мы привлекаем клиентов быстрее, чем расширяем службу доставки → риск снижения качества сервиса |
| 📈 Разрыв между `total_users` и `total_couriers` увеличивается с каждым месяцем | Требуется пересмотр стратегии найма курьеров |
| ⚠️ Дни с нулевым `new_couriers` повторяются регулярно | Система найма не автоматизирована → зона для оптимизации |

---

## 💡 Рекомендации для бизнеса

| № | Рекомендация | Ожидаемый эффект |
|---|--------------|------------------|
| 1️⃣ | Привязать найм курьеров к прогнозу новых пользователей | Заблаговременное расширение доставки |
| 2️⃣ | Внедрить алерты в дашборд при превышении соотношения `total_users / total_couriers` > 50 | Быстрая реакция на дисбаланс |
| 3️⃣ | Добавить анализ retention курьеров (как долго работают) | Снижение текучести кадров |

---

## 🧠 Демонстрируемые навыки

| Навык | Как проявлен |
|-------|---------------|
| 🐘 Продвинутый SQL | CTE, оконные функции, UNION, работа с датами |
| 🎯 Продуктовое мышление | Понимание метрик two-sided platform |
| 📊 Визуализация данных | Дашборд с бизнес-выводами |
| 💬 Бизнес-коммуникация | Инсайты + рекомендации, а не просто "цифры" |
