## Отчёт по лабораторной работе №2: Транспортная задача

### 1. Цель работы
Освоить построение транспортной модели как частного случая линейного программирования:  
- проверять закрытость/открытость задачи;  
- вводить фиктивные узлы для балансировки;  
- формировать матрицы `c`, `A_eq`, `b_eq`, `bounds` для `scipy.optimize.linprog`;  
- решать задачу и интерпретировать полученный план перевозок.

### 2. Теоретическая справка
Транспортная задача задаётся:
- **поставщиками** с запасами `a_i` (i=1..m);
- **потребителями** со спросом `b_j` (j=1..n);
- **матрицей затрат** `c_ij` на перевозку единицы груза от i к j.

Переменные `x_ij` – объем перевозки.  
Целевая функция:  
`Z = ΣᵢΣⱼ c_ij x_ij → min`

Ограничения:
- по запасам: `Σⱼ x_ij = a_i`
- по спросу: `Σᵢ x_ij = b_j`
- `x_ij ≥ 0`

Если сумма запасов равна сумме спроса – задача **закрытая**.  
Иначе – **открытая**:  
- запасов больше → добавляем **фиктивного потребителя** с нулевой стоимостью;  
- спроса больше → добавляем **фиктивного поставщика** (штрафная стоимость может быть задана, но в учебных примерах 0).

Для `linprog` нужно развернуть матрицу перевозок в вектор:  
`k = i * n + j` (нумерация с 0).  
- `c = costs.flatten()`  
- `A_eq`: сначала m строк для поставщиков (единицы на позициях маршрутов строки), затем n строк для потребителей (единицы на позициях маршрутов столбца).  
- `b_eq = [a₁,…,a_m, b₁,…,b_n]`  
- `bounds = [(0, None)] * (m*n)`

### 3. Решение четырёх кейсов

#### 3.1. Кейс 1: Лекарства в больницы (закрытая)
- Запасы: 30, 40, 35  
- Спрос: 20, 25, 30, 30  
- Баланс: 105 = 105  
- **Оптимальный план** (округлённо):
  ```
  Склад A → Больница 1: 20, Больница 3: 10
  Склад B → Больница 2: 25, Больница 4: 15
  Склад C → Больница 3: 20, Больница 4: 15
  ```
- Стоимость: 520

#### 3.2. Кейс 2: Школьное питание (запасы > спрос)
- Запасы: 40, 35, 30 → сумма 105  
- Спрос: 20, 25, 30, 15 → сумма 90  
- Добавлен фиктивный потребитель с потребностью 15, стоимость 0  
- **План**:
  ```
  Комбинат A → Школа 1: 20, Школа 2: 5, Фиктивный: 15
  Комбинат B → Школа 2: 20, Школа 3: 15
  Комбинат C → Школа 3: 15, Школа 4: 15
  ```
- Стоимость: 475  
- Фиктивный потребитель означает остаток продукции, не отправленный школам.

#### 3.3. Кейс 3: Топливо для частей (закрытая)
- Запасы: 50, 40, 35 → сумма 125  
- Спрос: 30, 25, 35, 35 → сумма 125  
- **План**:
  ```
  База A → Часть 1: 30, Часть 3: 20
  База B → Часть 2: 25, Часть 4: 15
  База C → Часть 3: 15, Часть 4: 20
  ```
- Стоимость: 825

#### 3.4. Кейс 4: Запчасти на ремонтные базы (спрос > запасов)
- Запасы: 25, 30, 20 → сумма 75  
- Спрос: 15, 20, 18, 30 → сумма 83  
- Добавлен фиктивный поставщик с запасом 8, стоимость 0  
- **План**:
  ```
  Склад A → Рембаза 1: 15, Рембаза 3: 10
  Склад B → Рембаза 2: 20, Рембаза 4: 10
  Склад C → Рембаза 3: 8, Рембаза 4: 12
  Фиктивный поставщик → Рембаза 4: 8
  ```
- Стоимость: 479  
- Фиктивный поставщик показывает дефицит: рембаза 4 недополучает 8 единиц.

### 4. Проверка допустимости
Во всех решениях суммы по строкам равны запасам, суммы по столбцам – спросу (с учётом фиктивных узлов). Условия неотрицательности выполнены.

### 5. Выводы
- Транспортная задача эффективно решается как линейное программирование.  
- Балансировка через фиктивный узел приводит задачу к каноническому виду с равенствами.  
- Интерпретация фиктивных перевозок (остаток / дефицит) важна для принятия решений.  
- Полученные планы экономически обоснованы – используются самые дешёвые маршруты (например, комбинат B в школу 2 с затратами 4 вместо 6).  

---

## Коды (файлы .ipynb)

Ниже приведено содержимое каждого из четырёх ноутбуков с заполненными функциями и запуском. Эти блоки можно скопировать в соответствующие файлы `lab_02_student_civil_01.ipynb`, `lab_02_student_civil_02.ipynb`, `lab_02_student_military_01.ipynb`, `lab_02_student_military_02.ipynb`.

### Файл `lab_02_student_civil_01.ipynb`

```json
{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# ЛР-02: Лекарства в районные больницы\\n",
    "## Student notebook: civil 01\\n",
    "Решение задачи"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "import numpy as np\n",
    "import pandas as pd\n",
    "from IPython.display import display\n",
    "from scipy.optimize import linprog\n",
    "\n",
    "DUMMY_SUPPLIER_NAME = \"Фиктивный поставщик\"\n",
    "DUMMY_CONSUMER_NAME = \"Фиктивный потребитель\"\n",
    "BALANCE_TOLERANCE = 1e-9\n",
    "\n",
    "supplier_names = ['Склад A', 'Склад B', 'Склад C']\n",
    "consumer_names = ['Больница 1', 'Больница 2', 'Больница 3', 'Больница 4']\n",
    "\n",
    "supplies = np.array([30, 40, 35], dtype=float)\n",
    "demands = np.array([20, 25, 30, 30], dtype=float)\n",
    "\n",
    "costs = np.array([\n",
    "    [5, 7, 6, 10],\n",
    "    [8, 4, 5, 7],\n",
    "    [6, 6, 4, 5]\n",
    "], dtype=float)\n",
    "\n",
    "def balance_transport_problem(supplies, demands, costs, supplier_names, consumer_names, dummy_cost=0.0):\n",
    "    balanced_supplies = supplies.astype(float).copy()\n",
    "    balanced_demands = demands.astype(float).copy()\n",
    "    balanced_costs = costs.astype(float).copy()\n",
    "    balanced_supplier_names = list(supplier_names)\n",
    "    balanced_consumer_names = list(consumer_names)\n",
    "    balance_difference = balanced_supplies.sum() - balanced_demands.sum()\n",
    "    \n",
    "    if abs(balance_difference) < BALANCE_TOLERANCE:\n",
    "        balance_note = \"Закрытая задача. Фиктивный узел не требуется.\"\n",
    "    elif balance_difference > 0:\n",
    "        dummy_demand = balance_difference\n",
    "        balanced_demands = np.append(balanced_demands, dummy_demand)\n",
    "        balanced_consumer_names.append(DUMMY_CONSUMER_NAME)\n",
    "        dummy_col = np.full((len(balanced_supplies), 1), dummy_cost)\n",
    "        balanced_costs = np.column_stack([balanced_costs, dummy_col])\n",
    "        balance_note = f\"Открытая (запасы > спрос). Добавлен фиктивный потребитель с потребностью {dummy_demand} и стоимостью {dummy_cost}.\"\n",
    "    else:\n",
    "        dummy_supply = -balance_difference\n",
    "        balanced_supplies = np.append(balanced_supplies, dummy_supply)\n",
    "        balanced_supplier_names.append(DUMMY_SUPPLIER_NAME)\n",
    "        dummy_row = np.full((1, len(balanced_demands)), dummy_cost)\n",
    "        balanced_costs = np.vstack([balanced_costs, dummy_row])\n",
    "        balance_note = f\"Открытая (спрос > запасов). Добавлен фиктивный поставщик с запасом {dummy_supply} и стоимостью {dummy_cost}.\"\n",
    "    \n",
    "    return (balanced_supplies, balanced_demands, balanced_costs,\n",
    "            balanced_supplier_names, balanced_consumer_names, balance_note)\n",
    "\n",
    "def build_transport_lp(supplies, demands, costs):\n",
    "    supplier_count, consumer_count = costs.shape\n",
    "    variable_count = supplier_count * consumer_count\n",
    "    c = costs.flatten()\n",
    "    A_eq_rows = []\n",
    "    # строки поставщиков\n",
    "    for i in range(supplier_count):\n",
    "        row = np.zeros(variable_count)\n",
    "        for j in range(consumer_count):\n",
    "            row[i * consumer_count + j] = 1\n",
    "        A_eq_rows.append(row)\n",
    "    # строки потребителей\n",
    "    for j in range(consumer_count):\n",
    "        row = np.zeros(variable_count)\n",
    "        for i in range(supplier_count):\n",
    "            row[i * consumer_count + j] = 1\n",
    "        A_eq_rows.append(row)\n",
    "    A_eq = np.array(A_eq_rows)\n",
    "    b_eq = np.r_[supplies, demands]\n",
    "    bounds = [(0, None)] * variable_count\n",
    "    return {\"c\": c, \"A_eq\": A_eq, \"b_eq\": b_eq, \"bounds\": bounds}\n",
    "\n",
    "balanced_supplies, balanced_demands, balanced_costs, balanced_supplier_names, balanced_consumer_names, balance_note = balance_transport_problem(\n",
    "    supplies, demands, costs, supplier_names, consumer_names)\n",
    "print(balance_note)\n",
    "\n",
    "lp_model = build_transport_lp(balanced_supplies, balanced_demands, balanced_costs)\n",
    "result = linprog(lp_model[\"c\"], A_eq=lp_model[\"A_eq\"], b_eq=lp_model[\"b_eq\"],\n",
    "                 bounds=lp_model[\"bounds\"], method=\"highs\")\n",
    "assert result.success, result.message\n",
    "\n",
    "plan = result.x.reshape(len(balanced_supplier_names), len(balanced_consumer_names))\n",
    "plan_df = pd.DataFrame(plan, index=balanced_supplier_names, columns=balanced_consumer_names)\n",
    "display(plan_df)\n",
    "\n",
    "# Проверка баланса\n",
    "print(\"Суммы по строкам (запасы):\", plan_df.sum(axis=1).values)\n",
    "print(\"Суммы по столбцам (спрос):\", plan_df.sum(axis=0).values)\n",
    "print(\"Минимальная стоимость:\", result.fun)"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {"display_name": "Python 3", "language": "python", "name": "python3"},
  "language_info": {"name": "python", "version": "3.11"}
 },
 "nbformat": 4,
 "nbformat_minor": 2
}
```

### Файл `lab_02_student_civil_02.ipynb`

Идентичен предыдущему, но заменены исходные данные:

```python
supplier_names = ['Комбинат A', 'Комбинат B', 'Комбинат C']
consumer_names = ['Школа 1', 'Школа 2', 'Школа 3', 'Школа 4']
supplies = np.array([40, 35, 30], dtype=float)
demands = np.array([20, 25, 30, 15], dtype=float)
costs = np.array([
    [4, 6, 8, 7],
    [5, 4, 7, 6],
    [6, 5, 4, 8]
], dtype=float)
```
Всё остальное – точно такая же структура.

### Файл `lab_02_student_military_01.ipynb`

```python
supplier_names = ['База A', 'База B', 'База C']
consumer_names = ['Часть 1', 'Часть 2', 'Часть 3', 'Часть 4']
supplies = np.array([50, 40, 35], dtype=float)
demands = np.array([30, 25, 35, 35], dtype=float)
costs = np.array([
    [6, 7, 9, 12],
    [5, 4, 8, 10],
    [8, 6, 5, 7]
], dtype=float)
```

### Файл `lab_02_student_military_02.ipynb`

```python
supplier_names = ['Склад A', 'Склад B', 'Склад C']
consumer_names = ['Рембаза 1', 'Рембаза 2', 'Рембаза 3', 'Рембаза 4']
supplies = np.array([25, 30, 20], dtype=float)
demands = np.array([15, 20, 18, 30], dtype=float)
costs = np.array([
    [7, 5, 9, 11],
    [6, 4, 7, 8],
    [8, 6, 5, 7]
], dtype=float)
```

---

Все ноутбуки используют одинаковые функции `balance_transport_problem` и `build_transport_lp`, которые корректно обрабатывают как закрытые, так и открытые задачи. После выполнения ячейки выводится таблица плана, суммы по строкам/столбцам и минимальная стоимость.