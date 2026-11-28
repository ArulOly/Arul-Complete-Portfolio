# MongoDB: Inventory — Baseline Setup for School Lab (Project 027001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Validate counts and calculate reorder status inside the MongoDB pipeline.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MongoDB aggregation and mongosh.

## System architecture

`Local documents → aggregation match → computed projection → sorted report`

## Features

- Record the school lab inventory baseline.
- Validate counts and calculate reorder status inside the MongoDB pipeline.

## Prerequisites

Use a local MongoDB instance and mongosh with practice database collection permissions. Use a text editor and a local practice directory.

## Complete MongoDB source code

```javascript
const lab = db.getSiblingDB("portfolio_lab");
const table = lab.getCollection("inventory_" + new ObjectId().toHexString());

try {
  table.insertMany([
    { asset: "LAB-17", stock: 2, reorder_at: 4 },
    { asset: "LAB-24", stock: 6, reorder_at: 4 }
  ]);
  // Restrict the exercise to numeric, nonnegative counts before projection.
  const output = table.aggregate([
    { $match: {
        stock: { $type: "number", $gte: 0, $lte: 1000000000 },
        reorder_at: { $type: "number", $gte: 1, $lte: 1000000000 }
    } },
    { $match: { $expr: { $and: [
        { $eq: ["$stock", { $floor: "$stock" }] },
        { $eq: ["$reorder_at", { $floor: "$reorder_at" }] }
    ] } } },
    { $project: { _id: 0, asset: 1, stock: 1, reorder_at: 1,
        needs_reorder: { $lt: ["$stock", "$reorder_at"] } } },
    { $sort: { asset: 1 } }
  ]).toArray();
  print(JSON.stringify(output));
} finally {
  table.drop();
}
```

## Implementation explanation

Validate counts and calculate reorder status inside the MongoDB pipeline.

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

Save the block as `inventory.js` and run it in the mongo shell.

```sh
mongosh "mongodb://127.0.0.1:27017/portfolio_lab" --quiet --file inventory.js
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "rows": [
    {
      "asset": "LAB-17",
      "stock": 2,
      "reorder_at": 4,
      "needs_reorder": true
    },
    {
      "asset": "LAB-24",
      "stock": 6,
      "reorder_at": 4,
      "needs_reorder": false
    }
  ]
}
```

## Testing and validation

Ran the embedded script with mongosh and MongoDB 8.0. Two runs produced the same rows. Changing the second row to negative stock excluded it.

```json
{
  "rows": [
    {
      "asset": "LAB-17",
      "stock": 2,
      "reorder_at": 4,
      "needs_reorder": true
    },
    {
      "asset": "LAB-24",
      "stock": 6,
      "reorder_at": 4,
      "needs_reorder": false
    }
  ]
}
```

## Error handling

The pipeline excludes invalid counts. The shell reports connection or permission errors.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The script uses two synthetic records and drops its uniquely named collection on normal completion.

## Learning outcomes

Practice inventory using MongoDB and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the MongoDB project index](../README.md)
