# TP 2 — Object-Oriented Data Cleaning with Design Patterns

**Course:** Advanced Programming for Data Science (ADV) — ITC, AMS-FIE
**Lecturer:** Mr. VANN Visal
**Group 1:** Duk Pagnarith, Leng Devid, Sophorn Rachana

This TP builds a small, reusable data-cleaning toolkit in Python step by step. Each exercise introduces one object-oriented concept or design pattern, and the final exercise combines all of them into a complete cleaning pipeline.

---

## Repository Structure

```
.
├── Exercise1.ipynb              # CSVReader class (OOP foundation)
├── Exercise2.ipynb              # Missing values — Strategy pattern
├── Exercise3.ipynb              # Logging & timing — Decorator pattern
├── Exercise4.ipynb              # Transformations — Factory pattern
├── Exercise5.ipynb              # Full pipeline — Template Method pattern
├── random_employee_data.csv     # Messy employee dataset (Exercise 3)
└── sample_data (2).csv          # Small dataset with missing values (Exercises 1, 2, 4, 5)
```

---

## Exercises

### Exercise 1 — `CSVReader` (OOP Foundation)

A first class that encapsulates loading a CSV file.

| Member | Description |
|---|---|
| `__init__(file_path)` | Stores the path; `data` starts as `None` |
| `read()` | Loads the file with `pd.read_csv` and stores it in `self.data` |
| `preview(n=5)` | Prints the first `n` rows |

**Concepts:** classes, attributes, methods, encapsulation.

---

### Exercise 2 — Missing Values with the Strategy Pattern

Different ways of handling missing values are implemented as interchangeable strategies behind one common interface.

| Class | Role |
|---|---|
| `MissingValueStrategy` (ABC) | Interface with abstract method `handle(df)` |
| `DropMissing` | Drops every row containing a missing value |
| `FillMean` | Fills numeric columns with their mean |
| `FillMode` | Fills every column with its most frequent value |
| `DataCleaner` | Context class: holds a strategy and calls it in `clean(df)` |

The strategy can be swapped at runtime on the same cleaner:

```python
cleaner = DataCleaner(FillMean())
mean_result = cleaner.clean(df)

cleaner.strategy = FillMode()
mode_result = cleaner.clean(df)
```

**Concepts:** abstract base classes, polymorphism, Strategy pattern, composition over inheritance.

---

### Exercise 3 — Logging and Timing with the Decorator Pattern

Reusable decorators add behaviour around a cleaning method without modifying its logic.

| Decorator | Effect |
|---|---|
| `@log_action` | Prints `[START]` / `[FINISH]` around the call (`finally` guarantees the finish message) |
| `@log_time` | Measures execution time with `time.perf_counter()` |
| `@wraps(func)` | Preserves the original function's name and docstring |

The example uses `random_employee_data.csv` (32 rows: `employee_id, name, age, department, salary_usd, city`), which contains duplicates, inconsistent casing and whitespace, missing values, and impossible ages and salaries. The decorated cleaning method:

1. removes duplicate rows,
2. strips and title-cases `name`, `department`, `city`,
3. treats ages outside 18–100 and salaries below 0 or above 100,000 as missing,
4. fills missing text with `"Unknown"` and missing numbers with the median,
5. casts `age` to `int` and rounds `salary_usd`.

**Concepts:** higher-order functions, decorators, decorator stacking order, Open/Closed Principle.

---

### Exercise 4 — Transformations with the Factory Pattern

Data transformations share a common abstract interface and are created by name through a factory.

| Class | Role |
|---|---|
| `DataTransform` (ABC) | Interface with abstract method `apply(df)` |
| `NormalizeColumns` | Min–max scales numeric columns to `[0, 1]` |
| `RemoveDuplicates` | Drops duplicate rows (optionally on a subset of columns) |
| `StandardizeText` | Strips extra spaces and unifies case (`lower`, `upper`, `title`) |
| `TransformFactory` | `create(name, **kwargs)` returns a transform; `register(name, cls)` adds new ones without editing the factory |

Each `apply` method is decorated with `@log_action` and `@log_time` from Exercise 3.

```python
steps = [
    ("remove_duplicates", {}),
    ("standardize_text", {"columns": ["name", "city"]}),
    ("normalize", {"columns": ["age", "height_cm", "weight_kg", "score"]}),
]

for name, params in steps:
    df = TransformFactory.create(name, **params).apply(df)
```

**Concepts:** Factory pattern, class methods, registry, Open/Closed Principle.

---

### Exercise 5 — Full Cleaning Pipeline with the Template Method Pattern

An abstract `DataPipeline` fixes the order of the steps in `run()`; subclasses implement each step.

```
run():  load()  →  clean()  →  transform()  →  save()
```

| Class | Role |
|---|---|
| `DataPipeline` (ABC) | Defines `run()`; declares abstract `load`, `clean`, `transform`, `save` |
| `CSVDataPipeline` | Implements the steps by reusing `CSVReader` (Ex. 1), `DataCleaner` + strategy (Ex. 2) and `DataTransform` objects from the factory (Ex. 4), then writes the result to CSV |

**Concepts:** Template Method pattern, composition, abstraction.

---

## Design Patterns Summary

| Exercise | Pattern | What it solves |
|---|---|---|
| 1 | Class / Encapsulation | Bundles file path, data and reading logic |
| 2 | Strategy | Swap missing-value handling without changing the cleaner |
| 3 | Decorator | Add logging and timing without touching method logic |
| 4 | Factory | Create transformations by name; add new ones via `register` |
| 5 | Template Method | Fix the pipeline skeleton while letting subclasses define each step |

---

## Requirements

- Python 3.9+
- pandas
- Jupyter Notebook or JupyterLab

```bash
pip install pandas jupyter
```

## How to Run

1. Clone the repository and place the CSV files in the same folder as the notebooks.
2. Launch Jupyter:
   ```bash
   jupyter notebook
   ```
3. Run the notebooks in order (Exercise 1 → 5). Exercise 5 depends on the classes defined in Exercises 1, 2 and 4, so those must be defined (or imported) in the same session.
