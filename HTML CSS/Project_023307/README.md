# HTML CSS: Data Export — Automation for Small Business (Project 023307)

## Project overview

Local shop operations provides the setting for this data export exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The small business needs a readable data export display focused on automation.

## Project objectives

- Present the data export outcome with a labeled result.
- Expand the example batch to check its records in order.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

HTML, CSS Grid, responsive media queries, and native HTML disclosure controls.

## System architecture

`Synthetic fixture → semantic HTML → CSS layout → accessible report in the browser`

## Features

- Present the data export outcome with a labeled result.
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
  <title>Data Export — Automation for Small Business | Project 023307</title>
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
      <h1>Data Export — Automation for Small Business</h1>
      <p>Local shop operations · Case SHOP-22</p>
    </header>
    <div class="grid">
      <section class="panel result" aria-labelledby="result-title">
        <h2 id="result-title">Batch review</h2>
        <dl>
          <dt>csv</dt>
          <dd><pre id="result">&quot;code,quantity\nKIT-01,4\nKIT-02,2\n&quot;</pre></dd>
        </dl>
      </section>
      <section class="panel" aria-labelledby="sample-title">
        <h2 id="sample-title">Synthetic source data</h2>
        <details>
          <summary>Input records</summary>
          <pre id="input-records">{
  &quot;records&quot;: [
    {
      &quot;asset&quot;: &quot;SHOP-22&quot;,
      &quot;entries&quot;: [
        {
          &quot;code&quot;: &quot;KIT-01&quot;,
          &quot;quantity&quot;: 4
        },
        {
          &quot;code&quot;: &quot;KIT-02&quot;,
          &quot;quantity&quot;: 2
        }
      ]
    },
    {
      &quot;asset&quot;: &quot;SHOP-22-B&quot;,
      &quot;entries&quot;: [
        {
          &quot;code&quot;: &quot;KIT-01&quot;,
          &quot;quantity&quot;: 4
        },
        {
          &quot;code&quot;: &quot;KIT-02&quot;,
          &quot;quantity&quot;: 2
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
      "asset": "SHOP-22",
      "entries": [
        {
          "code": "KIT-01",
          "quantity": 4
        },
        {
          "code": "KIT-02",
          "quantity": 2
        }
      ]
    },
    {
      "asset": "SHOP-22-B",
      "entries": [
        {
          "code": "KIT-01",
          "quantity": 4
        },
        {
          "code": "KIT-02",
          "quantity": 2
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
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Testing and validation

Actual tests used Chromium at 1100 and 360 pixels. Both sizes displayed the expected result without horizontal overflow. Enter toggled the input disclosure and the revealed JSON matched the sample.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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

Practice data export using HTML CSS and check a repeatable automation workflow in the small business scenario.

## Conclusion

The project records an observable data export result for training case `SHOP-22`.

[Back to the HTML CSS project index](../README.md)
