# PostgreSQL-da JOIN

## 1. Jadvallarni yaratish va ma'lumot kiritish

```sql
-- Bo'limlar jadvalini yaratamiz
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL,
    location VARCHAR(100)
);

-- Xodimlar jadvalini yaratamiz
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    department_id INTEGER REFERENCES departments(department_id),
    salary DECIMAL(10, 2)
);

-- Bo'limlarga ma'lumot qo'shamiz
INSERT INTO departments (department_name, location) VALUES
('IT', '1-qavat'),
('Marketing', '2-qavat'),
('Finance', '3-qavat'),
('HR', '4-qavat'),
('Operations', NULL);

-- Xodimlarga ma'lumot qo'shamiz
INSERT INTO employees (first_name, last_name, department_id, salary) VALUES
('Ali', 'Valiyev', 1, 5000.00),
('Guli', 'Nosirova', 2, 4500.00),
('Hasan', 'Husanov', 1, 5500.00),
('Dilfuza', 'Rahimova', 3, 6000.00),
('Sarvar', 'Qodirov', NULL, 4000.00),
('Lola', 'Karimova', 4, 4800.00),
('Javlon', 'Ergashev', NULL, 5200.00);
```

## 2. INNER JOIN Misoli

```sql
-- Faqat department_id si mavjud bo'lgan xodimlarni ko'rsatadi
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    d.location
FROM 
    employees e
INNER JOIN 
    departments d ON e.department_id = d.department_id;
```

Natija:
```
employee_id | first_name | last_name | department_name | location
------------+------------+-----------+-----------------+----------
          1 | Ali        | Valiyev   | IT              | 1-qavat
          2 | Guli       | Nosirova  | Marketing       | 2-qavat
          3 | Hasan      | Husanov   | IT              | 1-qavat
          4 | Dilfuza    | Rahimova  | Finance         | 3-qavat
          6 | Lola       | Karimova  | HR              | 4-qavat
```

## 3. LEFT JOIN Misoli

```sql
-- Barcha xodimlarni, hatto department_id si bo'lmaganlarni ham ko'rsatadi
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    d.location
FROM 
    employees e
LEFT JOIN 
    departments d ON e.department_id = d.department_id;
```

Natija:
```
employee_id | first_name | last_name | department_name | location
------------+------------+-----------+-----------------+----------
          1 | Ali        | Valiyev   | IT              | 1-qavat
          2 | Guli       | Nosirova  | Marketing       | 2-qavat
          3 | Hasan      | Husanov   | IT              | 1-qavat
          4 | Dilfuza    | Rahimova  | Finance         | 3-qavat
          5 | Sarvar     | Qodirov   | NULL            | NULL
          6 | Lola       | Karimova  | HR              | 4-qavat
          7 | Javlon     | Ergashev  | NULL            | NULL
```

## 4. RIGHT JOIN Misoli

```sql
-- Barcha bo'limlarni, hatto xodimlari bo'lmaganlarni ham ko'rsatadi
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    d.location
FROM 
    employees e
RIGHT JOIN 
    departments d ON e.department_id = d.department_id;
```

Natija:
```
employee_id | first_name | last_name | department_name | location
------------+------------+-----------+-----------------+----------
          1 | Ali        | Valiyev   | IT              | 1-qavat
          3 | Hasan      | Husanov   | IT              | 1-qavat
          2 | Guli       | Nosirova  | Marketing       | 2-qavat
          4 | Dilfuza    | Rahimova  | Finance         | 3-qavat
          6 | Lola       | Karimova  | HR              | 4-qavat
        NULL | NULL       | NULL      | Operations      | NULL
```

## 5. FULL JOIN Misoli

```sql
-- Barcha xodimlar va barcha bo'limlarni ko'rsatadi
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    d.location
FROM 
    employees e
FULL JOIN 
    departments d ON e.department_id = d.department_id;
```

Natija:
```
employee_id | first_name | last_name | department_name | location
------------+------------+-----------+-----------------+----------
          1 | Ali        | Valiyev   | IT              | 1-qavat
          2 | Guli       | Nosirova  | Marketing       | 2-qavat
          3 | Hasan      | Husanov   | IT              | 1-qavat
          4 | Dilfuza    | Rahimova  | Finance         | 3-qavat
          5 | Sarvar     | Qodirov   | NULL            | NULL
          6 | Lola       | Karimova  | HR              | 4-qavat
          7 | Javlon     | Ergashev  | NULL            | NULL
        NULL | NULL       | NULL      | Operations      | NULL
```

## 6. Qo'shimcha Misol: WHERE bilan JOIN

```sql
-- IT bo'limida ishlaydigan xodimlarni topamiz
SELECT 
    e.employee_id,
    e.first_name || ' ' || e.last_name AS full_name,
    e.salary,
    d.department_name
FROM 
    employees e
INNER JOIN 
    departments d ON e.department_id = d.department_id
WHERE 
    d.department_name = 'IT';
```

Natija:
```
employee_id | full_name      | salary | department_name
------------+----------------+--------+-----------------
          1 | Ali Valiyev    | 5000.00| IT
          3 | Hasan Husanov  | 5500.00| IT
```
 
