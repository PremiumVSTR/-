# Лабораторные работы по Базам Данных 
Вариант 39. ♻️ Система учета отходов и переработки в городе  
Telegram: @PremiumVSTR

### Постановка задачи

**Условие задачи:**

Имеются контейнерные площадки (адрес, тип контейнеров - стекло, пластик, бумага), операторы вывоза (наименование компании, телефон) и акты вывоза (дата, вес отходов). Накапливаются сведения о вывезенных отходах с указанием типа отходов, даты вывоза и веса.

**🏷️ Сущности:**

1. **Район** (id_района, название_района)
2. **Контейнерная площадка** (id_площадки, адрес, id_района)
3. **Тип отходов** (id_типа, название_типа)
4. **Оператор** (id_оператора, название_компании, телефон)
5. **Акт вывоза** (id_акта, id_площадки, id_оператора, id_типа, дата_вывоза, вес_кг)

**🔄 Процессы:**

Операторы фиксируют акты вывоза отходов определенного типа с контейнерных площадок. Каждая контейнерная площадка привязана к району города и может иметь контейнеры для разных типов отходов.

**📄 Выходные документы:**

1. 📊 Отчет по общему весу вывезенных отходов каждого типа за месяц по районам города, отсортированный по убыванию веса
2. 📅 Для заданной контейнерной площадки выдать график и частоту вывоза каждого типа отходов, отсортированный по дате

---

## Модель базы данных (3 таблицы)

### Структура базы данных

#### Таблица: collection_point (Контейнерные площадки)  
Содержит информацию о площадках и районах.

| Поле       | Тип          | Ограничения           | Описание                  |
|------------|--------------|----------------------|---------------------------|
| point_id   | SERIAL       | PRIMARY KEY          | Уникальный идентификатор площадки |
| address    | VARCHAR(500) | UNIQUE, NOT NULL     | Адрес площадки            |
| district   | VARCHAR(100) | NOT NULL             | Название района           |

#### Таблица: waste_operator (Операторы вывоза)  
Хранит информацию о компаниях-операторах.

| Поле         | Тип          | Ограничения           | Описание                  |
|--------------|--------------|-----------------------|---------------------------|
| operator_id  | SERIAL       | PRIMARY KEY           | Уникальный идентификатор оператора |
| company_name | VARCHAR(200) | UNIQUE, NOT NULL      | Наименование компании      |
| phone        | VARCHAR(20)  | NOT NULL              | Контактный телефон        |

#### Таблица: waste_removal_act (Акты вывоза)  
Основная таблица со всеми данными о вывозе.

| Поле         | Тип            | Ограничения                                | Описание                       |
|--------------|----------------|-------------------------------------------|--------------------------------|
| act_id       | SERIAL         | PRIMARY KEY                              | Уникальный идентификатор акта  |
| point_id     | INTEGER        | FOREIGN KEY REFERENCES collection_point(point_id) | Ссылка на площадку             |
| operator_id  | INTEGER        | FOREIGN KEY REFERENCES waste_operator(operator_id) | Ссылка на оператора            |
| removal_date | DATE           | NOT NULL                                 | Дата вывоза                   |
| waste_type   | VARCHAR(50)    | NOT NULL                                 | Тип отходов (стекло/пластик/бумага) |
| weight_kg   | DECIMAL(10,2)  | NOT NULL CHECK (weight_kg > 0)           | Вес отходов в кг              |

### Связи между таблицами
- `collection_point` → `waste_removal_act`: Один ко многим (одна площадка может иметь много актов вывоза)  
- `waste_operator` → `waste_removal_act`: Один ко многим (один оператор может выполнить много вывозов)  

### Бизнес-правила
- Адрес контейнерной площадки должен быть уникальным  
- Название компании-оператора должно быть уникальным  
- Вес отходов должен быть больше 0  
- Нельзя регистрировать два вывоза одного типа отходов с одной площадки в один день  
- Дата вывоза обязательна для заполнения  

## Физическая модель (DDL)
```sql
-- 1. Таблица контейнерных площадок
CREATE TABLE collection_point (
    point_id SERIAL PRIMARY KEY,
    address VARCHAR(500) UNIQUE NOT NULL,
    district VARCHAR(100) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE
);

-- 2. Таблица операторов вывоза
CREATE TABLE waste_operator (
    operator_id SERIAL PRIMARY KEY,
    company_name VARCHAR(200) UNIQUE NOT NULL,
    phone VARCHAR(20) NOT NULL
);

-- 3. Таблица актов вывоза
CREATE TABLE waste_removal_act (
    act_id SERIAL PRIMARY KEY,
    point_id INTEGER NOT NULL,
    operator_id INTEGER NOT NULL,
    removal_date DATE NOT NULL,
    waste_type VARCHAR(50) NOT NULL,
    weight_kg DECIMAL(10,2) NOT NULL CHECK (weight_kg > 0),
    
    FOREIGN KEY (point_id) REFERENCES collection_point(point_id) ON DELETE CASCADE,
    FOREIGN KEY (operator_id) REFERENCES waste_operator(operator_id) ON DELETE RESTRICT,
    
    -- Уникальное ограничение: нельзя дважды зарегистрировать вывоз одного типа с одной площадки в один день
    UNIQUE (point_id, waste_type, removal_date)
);

-- Индексы для оптимизации
CREATE INDEX idx_removal_date ON waste_removal_act(removal_date);
CREATE INDEX idx_point_date ON waste_removal_act(point_id, removal_date);
CREATE INDEX idx_waste_type ON waste_removal_act(waste_type);
```
## Анализ нормальных форм

### 1. Первая нормальная форма (1NF)
**Требования:**  
- Все атрибуты атомарны  
- Отсутствуют повторяющиеся группы  
- Определен первичный ключ  

**Анализ:**  
- **collection_point:** Все атрибуты атомарны, первичный ключ `point_id`, нет повторяющихся групп. ✅  
- **waste_operator:** Все атрибуты атомарны, первичный ключ `operator_id`, нет повторяющихся групп. ✅  
- **waste_removal_act:** Все атрибуты атомарны, первичный ключ `act_id`, нет повторяющихся групп. ✅  

**Вывод:** Все таблицы соответствуют 1NF.

---

### 2. Вторая нормальная форма (2NF)
**Требования:**  
- Соответствие 1NF  
- Все неключевые атрибуты полностью зависят от всего первичного ключа  

**Анализ:**  
- **collection_point:** Все атрибуты зависят от `point_id`. ✅  
- **waste_operator:** Все атрибуты зависят от `operator_id`. ✅  
- **waste_removal_act:** Все атрибуты зависят от `act_id`. ✅  

**Вывод:** Все таблицы соответствуют 2NF.

---

### 3. Третья нормальная форма (3NF)
**Требования:**  
- Соответствие 2NF  
- Отсутствие транзитивных зависимостей  

**Анализ:**  
- **collection_point:** Нет транзитивных зависимостей. ✅  
- **waste_operator:** Нет транзитивных зависимостей. ✅  
- **waste_removal_act:** Нет транзитивных зависимостей, но поле `waste_type` содержит строковые значения, которые могут дублироваться. Идеально — выделить отдельную таблицу типов отходов, но для минимализма допускается так. ⚠️  

**Вывод:** Таблицы в основном соответствуют 3NF с допустимым компромиссом для `waste_type`.

---

### 4. Нормальная форма Бойса-Кодда (BCNF)
**Требования:**  
- Каждая детерминанта является потенциальным ключом  

**Анализ:**  
- **collection_point:** Потенциальные ключи `point_id`, `address`. ✅  
- **waste_operator:** Потенциальные ключи `operator_id`, `company_name`. ✅  
- **waste_removal_act:** Потенциальный ключ `act_id`. ✅  

**Вывод:** Все таблицы соответствуют BCNF.

---

# Лабораторная работа 1
## ER-диаграмма 
![Иллюстрация к проекту](https://github.com/PremiumVSTR/-/blob/main/othodi.png)

---

# Лабораторная работа 2
## DDL-запросы
Таблица collection_point
<img width="896" height="540" alt="image" src="https://github.com/user-attachments/assets/8db6790a-a62b-4e40-9b4e-3a04e3c1f07d" />

Таблица waste_operator
<img width="890" height="548" alt="image" src="https://github.com/user-attachments/assets/41c9a8cc-29f1-4f05-bfb3-8aa84b3f0542" />

Таблица waste_removal_act
<img width="899" height="543" alt="image" src="https://github.com/user-attachments/assets/f71642e4-a868-46d4-b574-38959e004476" />

## Заполнение таблиц
<img width="687" height="737" alt="image" src="https://github.com/user-attachments/assets/b69aaee9-83be-4919-aa08-be0db60c983a" />
<img width="701" height="352" alt="image" src="https://github.com/user-attachments/assets/3996c84d-6604-4b19-824d-ff7c0f2c8cea" />
<img width="862" height="758" alt="image" src="https://github.com/user-attachments/assets/c1fbdb7d-bfab-40cf-a362-db7ce669661e" />
<img width="859" height="743" alt="image" src="https://github.com/user-attachments/assets/0ba6b0a9-0408-457e-90c4-77831c354f3b" />

## Проверка что данные были введены

<img width="575" height="692" alt="image" src="https://github.com/user-attachments/assets/5b1759d3-8ff6-46e2-bb6f-7b97731033f3" />
<img width="513" height="428" alt="image" src="https://github.com/user-attachments/assets/26572917-cab9-46b3-a015-64ee80650850" />
<img width="1115" height="740" alt="image" src="https://github.com/user-attachments/assets/5b26ba9d-4efa-466d-b364-80e0bd23b073" />

# SELECT-запросы с JOIN
## Запрос 1: Отчет по общему весу вывезенных отходов каждого типа за месяц по районам (отсортированный по убыванию веса)
<img width="1260" height="752" alt="image" src="https://github.com/user-attachments/assets/c78c6f88-1560-4a1a-ac66-5e117ed868ef" />

## Запрос 2: График и частота вывоза для заданной контейнерной площадки
<img width="989" height="756" alt="image" src="https://github.com/user-attachments/assets/39d731c9-151e-413b-aa28-a48d61020ec9" />

# Лабораторная работа 3
## Представления для выходных документов
## ПРЕДСТАВЛЕНИЕ 1: Отчет по весу отходов за месяц
<img width="627" height="320" alt="image" src="https://github.com/user-attachments/assets/21ecb943-cc2c-42c4-bd5a-ae72f07f0fb7" />

## ПРЕДСТАВЛЕНИЕ 2: График вывоза для площадок 
<img width="645" height="429" alt="image" src="https://github.com/user-attachments/assets/a2a1c379-ed70-46dc-a9d3-94d11ad575c2" />

## ПРОЦЕДУРА 1: Добавить новый акт вывоза
<img width="706" height="715" alt="image" src="https://github.com/user-attachments/assets/929161f2-f2ab-4796-9065-46b8d307e4a8" />
```
CREATE OR REPLACE VIEW "Martynovich2261".vw_point_schedule AS
SELECT 
    cp.point_id AS "ID площадки",
    cp.address AS "Адрес",
    cp.district AS "Район",
    wra.removal_date AS "Дата вывоза",
    wra.waste_type AS "Тип отходов",
    wra.weight_kg AS "Вес (кг)",
    wo.company_name AS "Оператор"
FROM "Martynovich2261".waste_removal_act wra
JOIN "Martynovich2261".collection_point cp ON wra.point_id = cp.point_id
JOIN "Martynovich2261".waste_operator wo ON wra.operator_id = wo.operator_id
ORDER BY cp.point_id, wra.removal_date DESC;

-- Комментарий
COMMENT ON VIEW "Martynovich2261".vw_point_schedule 
IS 'График вывоза отходов по всем контейнерным площадкам';

-- ============================================
-- ПРОЦЕДУРА 1: Добавить новый акт вывоза (ИЗМЕНЯЕТ БД)
-- ============================================
CREATE OR REPLACE PROCEDURE "Martynovich2261".sp_add_removal(
    p_point_id INTEGER,
    p_operator_id INTEGER,
    p_removal_date DATE,
    p_waste_type VARCHAR(50),
    p_weight_kg DECIMAL(10,2),
    OUT p_result_message VARCHAR(200)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_point_exists BOOLEAN;
    v_operator_exists BOOLEAN;
    v_new_act_id INTEGER;
BEGIN
    -- Проверяем существование площадки
    SELECT EXISTS(
        SELECT 1 FROM "Martynovich2261".collection_point 
        WHERE point_id = p_point_id
    ) INTO v_point_exists;
    
    IF NOT v_point_exists THEN
        p_result_message := 'ОШИБКА: Площадка с ID=' || p_point_id || ' не найдена';
        RETURN;
    END IF;
    
    -- Проверяем существование оператора
    SELECT EXISTS(
        SELECT 1 FROM "Martynovich2261".waste_operator 
        WHERE operator_id = p_operator_id
    ) INTO v_operator_exists;
    
    IF NOT v_operator_exists THEN
        p_result_message := 'ОШИБКА: Оператор с ID=' || p_operator_id || ' не найден';
        RETURN;
    END IF;
    
    -- Проверяем вес
    IF p_weight_kg <= 0 THEN
        p_result_message := 'ОШИБКА: Вес должен быть больше 0';
        RETURN;
    END IF;
    
    -- Проверяем дату (не может быть в будущем)
    IF p_removal_date > CURRENT_DATE THEN
        p_result_message := 'ОШИБКА: Дата вывоза не может быть в будущем';
        RETURN;
    END IF;
    
    -- Пробуем вставить запись
    BEGIN
        INSERT INTO "Martynovich2261".waste_removal_act 
            (point_id, operator_id, removal_date, waste_type, weight_kg)
        VALUES 
            (p_point_id, p_operator_id, p_removal_date, p_waste_type, p_weight_kg)
        RETURNING act_id INTO v_new_act_id;
        
        p_result_message := 'УСПЕХ: Акт вывоза №' || v_new_act_id || ' добавлен';
        
    EXCEPTION 
        WHEN unique_violation THEN
            p_result_message := 'ОШИБКА: Вывоз этого типа отходов уже зарегистрирован на эту дату для этой площадки';
        WHEN OTHERS THEN
            p_result_message := 'ОШИБКА: ' || SQLERRM;
    END;
END;
$$;

-- Комментарий
COMMENT ON PROCEDURE "Martynovich2261".sp_add_removal 
IS 'Добавляет новый акт вывоза в базу данных с проверкой данных';
```
## ПРОЦЕДУРА 2: Обновить вес акта вывоза
<img width="674" height="337" alt="image" src="https://github.com/user-attachments/assets/ded1b267-0578-41e1-8d46-f0822f9173fe" />

```
CREATE OR REPLACE PROCEDURE "Martynovich2261".sp_update_removal_weight(
    p_act_id INTEGER,
    p_new_weight_kg DECIMAL(10,2),
    OUT p_result_message VARCHAR(200)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_old_weight DECIMAL(10,2);
    v_rows_updated INTEGER;
BEGIN
    -- Проверяем существование акта
    IF NOT EXISTS(
        SELECT 1 FROM "Martynovich2261".waste_removal_act 
        WHERE act_id = p_act_id
    ) THEN
        p_result_message := 'ОШИБКА: Акт вывоза с ID=' || p_act_id || ' не найден';
        RETURN;
    END IF;
    
    -- Проверяем новый вес
    IF p_new_weight_kg <= 0 THEN
        p_result_message := 'ОШИБКА: Новый вес должен быть больше 0';
        RETURN;
    END IF;
    
    -- Получаем старый вес для информации
    SELECT weight_kg INTO v_old_weight 
    FROM "Martynovich2261".waste_removal_act 
    WHERE act_id = p_act_id;
    
    -- Обновляем вес
    UPDATE "Martynovich2261".waste_removal_act 
    SET weight_kg = p_new_weight_kg
    WHERE act_id = p_act_id;
    
    -- Проверяем, обновилась ли запись
    GET DIAGNOSTICS v_rows_updated = ROW_COUNT;
    
    IF v_rows_updated = 0 THEN
        p_result_message := 'ОШИБКА: Не удалось обновить акт вывоза';
    ELSE
        p_result_message := 'УСПЕХ: Вес акта №' || p_act_id || ' изменен с ' || 
                           v_old_weight || ' кг на ' || p_new_weight_kg || ' кг';
    END IF;
END;
$$;

-- Комментарий
COMMENT ON PROCEDURE "Martynovich2261".sp_update_removal_weight 
IS 'Обновляет вес существующего акта вывоза';
```
# ПРОВЕРКА РАБОТЫ:
## 1. Проверка представлений:
<img width="861" height="622" alt="image" src="https://github.com/user-attachments/assets/db9c5046-1416-4315-9080-28838b8a86c1" />
<img width="1014" height="584" alt="image" src="https://github.com/user-attachments/assets/0b183855-32f2-4c6d-b4d1-340e7dfcb110" />
## 2. Проверка процедур:
<img width="625" height="495" alt="image" src="https://github.com/user-attachments/assets/64543218-997b-4b92-9e11-68ed4c8ce7d8" />
<img width="719" height="451" alt="image" src="https://github.com/user-attachments/assets/39d106eb-14d5-4659-97fb-6958583dc714" />
<img width="486" height="552" alt="image" src="https://github.com/user-attachments/assets/170ce07b-9473-457e-b926-6159343575af" />
<img width="594" height="378" alt="image" src="https://github.com/user-attachments/assets/631c3265-0173-4c73-ba0f-121c24d96ea8" />

