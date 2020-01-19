### 约束(CONSTRAINT)



#### check

使用boolean 表达式对某列进行约束

demo

```plsql
CREATE TABLE employees (
   id serial PRIMARY KEY,
   first_name VARCHAR (50),
   last_name VARCHAR (50),
   birth_date DATE CHECK (birth_date > '1900-01-01'),
   joined_date DATE CHECK (joined_date > birth_date),
   salary numeric CHECK(salary > 0)
);

CREATE TABLE employees
(
    id          serial PRIMARY KEY,
    first_name  VARCHAR(50),
    last_name   VARCHAR(50),
    birth_date  DATE,
    CHECK (birth_date > '1900-01-01'),
    joined_date DATE,
    salary      numeric,
    CHECK (joined_date > birth_date AND salary > 0)
);
```

默认情况下,pgsql check 的约束名格式为: `{table}_{column}_check`

如果想使用自定义的名字 ,格式为:`column_name data_type CONSTRAINT constraint_name CHECK(...)`



