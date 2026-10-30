# Library Borrowing and Returning Management (`library_mini`)

Database assignment 05. A relational database design and implementation in MySQL for managing library readers, book copies, loan registrations, return records and the fees collected for overdue or damaged books.

## Technologies

- MySQL 8.0.16 or later (CHECK constraints are enforced only from this version).
- No application or user interface is included.

## Repository structure

| File | Content |
|---|---|
| `schema.sql` | Creates the `library_mini` database, 6 tables, keys and constraints |
| `seed.sql` | Sample data and data verification queries (PASS/FAIL) |
| `queries.sql` | 10 independent queries, numbered as in the assignment |
| `views.sql` | 3 views: `v_detail`, `v_group_summary`, `v_debt`, plus verification queries |

## How to run

Run the files in this order on a fresh server (`schema.sql` drops and recreates `library_mini`):

```bash
mysql -u <user> -p --default-character-set=utf8mb4 < schema.sql
mysql -u <user> -p --default-character-set=utf8mb4 < seed.sql
mysql -u <user> -p --default-character-set=utf8mb4 < queries.sql
mysql -u <user> -p --default-character-set=utf8mb4 < views.sql
```

In MySQL Workbench, open and execute the four files one after another in the same order.

> Warning: `schema.sql` contains `DROP DATABASE IF EXISTS library_mini;`, which deletes any existing database with that name.

## Tables

| Table | Primary key | Foreign keys and main constraints |
|---|---|---|
| `categories` | `category_id` | `name` UNIQUE |
| `copies` | `copy_id` | `category_id` references `categories`; `barcode` UNIQUE |
| `readers` | `reader_id` | |
| `loans` | `loan_id` | `reader_id` references `readers`; `copy_id` references `copies`; `status` ENUM (`Pending`, `Active`, `Completed`, `Cancelled`); CHECK `due_date >= loan_date` |
| `return_records` | `return_id` | `loan_id` references `loans` and is UNIQUE (at most one return record per loan); CHECK `fine_amount >= 0` |
| `payments` | `payment_id` | `loan_id` references `loans`; CHECK `amount > 0`; `method` ENUM (`Cash`, `BankTransfer`, `Card`) |

Money uses `DECIMAL(12,2)` and dates use `DATE`.

Relationships:

- One category has many copies; each copy belongs to exactly one category.
- Each loan belongs to exactly one reader and one copy.
- Each loan has at most one return record and may have many payments.

## Views (cover the whole time range)

- `v_detail`: one row per loan. Loans without a return record are kept and their fee is `NULL`.
- `v_group_summary`: one row per category, with total loans, Completed loans, Cancelled loans and total fee. Categories without loans are kept.
- `v_debt`: one row per reader, with total confirmed cost, total paid and balance due (cost minus money collected). Costs and payments are aggregated separately before joining, so a cost is never counted twice when a loan has several payments.

## Rules the schema cannot enforce

A CHECK constraint must not query other tables, and triggers or stored procedures are not required by the assignment. The following rules are therefore verified by queries in `seed.sql` (group B), not blocked at insert time:

- Only `Completed` loans have a return record, and each `Completed` loan has exactly one (B06, B07).
- `return_date >= loan_date` (B08).
- Money is collected only when a confirmed cost exists (B12).
- The total collected for a loan does not exceed its confirmed cost (B13).

These checks validate the sample data. They are not a runtime protection mechanism.

## Testing

Running `seed.sql` prints three verification tables (A: record counts, B: integrity and business rules, C: required data scenarios) and a fourth table D with extra scenarios that make mistakes in the queries visible (exactly 3 loans, boundary dates, ties). Every row of the column `result` must be `PASS`.

Expected results of the 10 queries are written in the comments above each query in `queries.sql`. The verification queries at the bottom of `views.sql` confirm that the views have the right number of rows and that totals match the base tables (total cost 110000, total paid 96000).

To test the constraints, remove the comment marks of one illustration at the bottom of `seed.sql` at a time and run it in a separate session; each statement must raise an error.

## Members

| Name | Student ID 
|---|---|---|
| Nguyen Quang Cuong | 11230740 
| Nguyen Nho Thang | 11230778 
| Duong Duc Thang | 11230777 
