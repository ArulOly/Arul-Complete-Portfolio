# HTML CSS: Data Validation — Automation for Branch Office Simulator (Project 023772)

## Project overview

A simulated branch office provides the setting for this data validation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The branch office simulator needs a readable data validation display focused on automation.

## Project objectives

- Present the data validation outcome with a labeled result.
- Expand the example batch to check its records in order.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

HTML, CSS Grid, responsive media queries, and native HTML disclosure controls.

## System architecture

`Synthetic fixture → semantic HTML → CSS layout → accessible report in the browser`

## Features

- Present the data validation outcome with a labeled result.
- Expand the example batch to check its records in order.

## Prerequisites

Use a modern browser. Use a text editor and a local practice directory.

## Complete HTML CSS source code

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Data Validation — Automation for Branch Office Simulator | Project 023772</title>
  <style>
    :root {
      color-scheme: light;
      font-family: system-ui, sans-serif;
      color: #172438;
      background: #f8fafc;
      line-height: 1.55;
      --accent: #225b50;
      --tint: #edf8f4;
    }
    body { margin: 0; }
    main { max-width: 66rem; margin: auto; padding: clamp(1rem, 4vw, 3rem); }
    h1 { color: var(--accent); line-height: 1.15; overflow-wrap: anywhere; }
    h2 { margin-block: 0 .75rem; }
    .grid { display: grid; gap: 1rem; align-items: start; }
    .panel { min-width: 0; border: 1px solid #bcc8d6; border-radius: .75rem; padding: 1rem; background: white; }
    .result { background: var(--tint); border-inline-start: .35rem solid var(--accent); }
    .notice { border-inline-start: .25rem solid var(--accent); padding-inline-start: .75rem; }
    dt { font-weight: 700; }
    dd { margin: .5rem 0 0; }
    summary { cursor: pointer; font-weight: 650; }
    summary:focus-visible { outline: 3px solid var(--accent); outline-offset: 4px; }
    pre { white-space: pre-wrap; overflow-wrap: anywhere; }
    @media (min-width: 48rem) {
      .grid { grid-template-columns: minmax(0, 3fr) minmax(0, 2fr); }
    }
    @media print {
      :root, .panel, .result { background: white; color: black; }
      main { padding: 0; }
      .panel { break-inside: avoid; }
    }
  </style>
</head>
<body>
  <main>
    <header>
      <h1>Data Validation — Automation for Branch Office Simulator</h1>
      <p>A simulated branch office · Case BRANCH-08</p>
    </header>
    <div class="grid">
      <section class="panel result" aria-labelledby="result-title">
        <h2 id="result-title">Batch review</h2>
        <dl>
          <dt>accepted</dt>
          <dd><pre id="result">{&quot;KIT-01&quot;: 4}</pre></dd>
        </dl>
      </section>
      <section class="panel" aria-labelledby="sample-title">
        <h2 id="sample-title">Synthetic source data</h2>
        <details>
          <summary>Input records</summary>
          <pre id="input-records">{
  &quot;records&quot;: [
    {
      &quot;asset&quot;: &quot;BRANCH-08&quot;,
      &quot;rows&quot;: [
        {
          &quot;code&quot;: &quot;KIT-01&quot;,
          &quot;count&quot;: 4
        },
        {
          &quot;code&quot;: &quot;KIT-01&quot;,
          &quot;count&quot;: 2
        },
        {
          &quot;code&quot;: &quot;KIT-02&quot;,
          &quot;count&quot;: -1
        }
      ]
    },
    {
      &quot;asset&quot;: &quot;BRANCH-08-B&quot;,
      &quot;rows&quot;: [
        {
          &quot;code&quot;: &quot;KIT-01&quot;,
          &quot;count&quot;: 4
        },
        {
          &quot;code&quot;: &quot;KIT-01&quot;,
          &quot;count&quot;: 2
        },
        {
          &quot;code&quot;: &quot;KIT-02&quot;,
          &quot;count&quot;: -1
        }
      ]
    }
  ]
}</pre>
        </details>
      </section>
    </div>
  </main>
</body>
</html>
```

## Implementation explanation

A labeled result and preserved source records make the example inspectable. CSS Grid adapts to the viewport, and the native details element can be opened with the keyboard.

## Sample input data

The standalone page already contains this synthetic data.

```json
{
  "records": [
    {
      "asset": "BRANCH-08",
      "rows": [
        {
          "code": "KIT-01",
          "count": 4
        },
        {
          "code": "KIT-01",
          "count": 2
        },
        {
          "code": "KIT-02",
          "count": -1
        }
      ]
    },
    {
      "asset": "BRANCH-08-B",
      "rows": [
        {
          "code": "KIT-01",
          "count": 4
        },
        {
          "code": "KIT-01",
          "count": 2
        },
        {
          "code": "KIT-02",
          "count": -1
        }
      ]
    }
  ]
}
```

## Execution instructions

Copy the HTML block into `index.html` and open that file in a browser. Use **Input records** to toggle the synthetic data. Resize the window to compare the single-column and two-column layouts.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "accepted": {
    "KIT-01": 4
  }
}
```

## Testing and validation

Actual tests used Chromium at 1100 and 360 pixels. Both sizes displayed the expected result without horizontal overflow. Enter toggled the input disclosure and the revealed JSON matched the sample.

```json
{
  "accepted": {
    "KIT-01": 4
  }
}
```

## Error handling

Long values wrap at narrow widths. The native disclosure control works without scripts, and its focus is visible during keyboard navigation.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This is a static report. To change the inputs or result, edit the HTML fixture. The page displays a precomputed example and does not run calculations in the browser.

## Learning outcomes

Practice data validation using HTML CSS and check a repeatable automation workflow in the branch office simulator scenario.

## Conclusion

The project records an observable data validation result for training case `BRANCH-08`.

[Back to the HTML CSS project index](../README.md)
