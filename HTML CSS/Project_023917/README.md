# HTML CSS: Visual Reporting — Automation for Public Dataset (Project 023917)

## Project overview

A synthetic public-data exercise provides the setting for this visual reporting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The public dataset needs a readable visual reporting display focused on automation.

## Project objectives

- Present the visual reporting outcome with a labeled result.
- Expand the example batch to check its records in order.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

HTML, CSS Grid, responsive media queries, and native HTML disclosure controls.

## System architecture

`Synthetic fixture → semantic HTML → CSS layout → accessible report in the browser`

## Features

- Present the visual reporting outcome with a labeled result.
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
  <title>Visual Reporting — Automation for Public Dataset | Project 023917</title>
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
      <h1>Visual Reporting — Automation for Public Dataset</h1>
      <p>A synthetic public-data exercise · Case DATA-12</p>
    </header>
    <div class="grid">
      <section class="panel result" aria-labelledby="result-title">
        <h2 id="result-title">Batch review</h2>
        <dl>
          <dt>peak</dt>
          <dd><pre id="result">12</pre></dd>
        </dl>
      </section>
      <section class="panel" aria-labelledby="sample-title">
        <h2 id="sample-title">Synthetic source data</h2>
        <details>
          <summary>Input records</summary>
          <pre id="input-records">{
  &quot;records&quot;: [
    {
      &quot;asset&quot;: &quot;DATA-12&quot;,
      &quot;values&quot;: {
        &quot;available&quot;: 12,
        &quot;reserved&quot;: 7,
        &quot;repair&quot;: 3
      },
      &quot;width&quot;: 12
    },
    {
      &quot;asset&quot;: &quot;DATA-12-B&quot;,
      &quot;values&quot;: {
        &quot;available&quot;: 12,
        &quot;reserved&quot;: 7,
        &quot;repair&quot;: 3
      },
      &quot;width&quot;: 12
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
      "asset": "DATA-12",
      "values": {
        "available": 12,
        "reserved": 7,
        "repair": 3
      },
      "width": 12
    },
    {
      "asset": "DATA-12-B",
      "values": {
        "available": 12,
        "reserved": 7,
        "repair": 3
      },
      "width": 12
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
  "peak": 12
}
```

## Testing and validation

Actual tests used Chromium at 1100 and 360 pixels. Both sizes displayed the expected result without horizontal overflow. Enter toggled the input disclosure and the revealed JSON matched the sample.

```json
{
  "peak": 12
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

Practice visual reporting using HTML CSS and check a repeatable automation workflow in the public dataset scenario.

## Conclusion

The project records an observable visual reporting result for training case `DATA-12`.

[Back to the HTML CSS project index](../README.md)
