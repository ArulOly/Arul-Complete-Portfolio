# SQLite: Inventory — Baseline Setup for School Lab (Project 028001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Calculate the stock status in SQL and guard inputs with table constraints.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

SQLite SQL.

## System architecture

`Sample rows → temporary table constraints → ordered CASE query → inventory report`

## Features

- Record the school lab inventory baseline.
- Calculate the stock status in SQL and guard inputs with table constraints.

## Prerequisites

Use SQLite and its command-line client in a local practice environment. Use a text editor and a local practice directory.

## Complete SQLite source code

```sql
BEGIN;

CREATE TEMPORARY TABLE lab_inventory (
    asset VARCHAR(32) PRIMARY KEY,
    stock INTEGER NOT NULL CHECK (stock >= 0),
    reorder_at INTEGER NOT NULL CHECK (reorder_at > 0)
);

INSERT INTO lab_inventory (asset, stock, reorder_at)
VALUES ('LAB-17', 2, 4), ('LAB-24', 6, 4);

SELECT asset, stock, reorder_at,
       CASE WHEN stock < reorder_at THEN 'reorder' ELSE 'available' END AS stock_status
FROM lab_inventory
ORDER BY asset;

ROLLBACK;
```

## Implementation explanation

Calculate the stock status in SQL and guard inputs with table constraints.

## Sample input data

Use this example input.

```json
[
  {
    "asset": "LAB-17",
    "stock": 2,
    "reorder_at": 4
  },
  {
    "asset": "LAB-24",
    "stock": 6,
    "reorder_at": 4
  }
]
```

## Execution instructions

Save the block as `inventory.sql`. Run these statements as a script with access to a practice database and temporary tables.

```sh
sqlite3 :memory: < inventory.sql
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "columns": [
    "asset",
    "stock",
    "reorder_at",
    "stock_status"
  ],
  "rows": [
    [
      "LAB-17",
      2,
      4,
      "reorder"
    ],
    [
      "LAB-24",
      6,
      4,
      "available"
    ]
  ]
}
```

## Testing and validation

Executed the statements on SQLite 3.53.1. Repeated reads returned the expected rows, and a negative stock insert triggered the table constraint.

```json
{
  "columns": [
    "asset",
    "stock",
    "reorder_at",
    "stock_status"
  ],
  "rows": [
    [
      "LAB-17",
      2,
      4,
      "reorder"
    ],
    [
      "LAB-24",
      6,
      4,
      "available"
    ]
  ]
}
```

## Error handling

The primary key rejects duplicate assets and CHECK constraints reject invalid quantities. If a statement fails inside the transaction, roll back before retrying.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

Temporary rows are scoped to the practice session. The embedded fixture demonstrates two inventory items.

## Learning outcomes

Practice inventory using SQLite and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the SQLite project index](../README.md)
