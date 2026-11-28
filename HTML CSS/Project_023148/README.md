# HTML CSS: Audit — Recovery for Portfolio Demo (Project 023148)

## Project overview

A repeatable portfolio demonstration provides the setting for this audit exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The portfolio demo needs a readable audit display focused on recovery.

## Project objectives

- Present the audit outcome with a labeled result.
- Compare a restored example with the preserved source data.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

HTML, CSS Grid, responsive media queries, and native HTML disclosure controls.

## System architecture

`Synthetic fixture → semantic HTML → CSS layout → accessible report in the browser`

## Features

- Present the audit outcome with a labeled result.
- Compare a restored example with the preserved source data.

## Prerequisites

Use a modern browser. Use a text editor and a local practice directory.

## Complete HTML CSS source code

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Audit — Recovery for Portfolio Demo | Project 023148</title>
  <style>
    :root {
      color-scheme: light;
      font-family: system-ui, sans-serif;
      color: #172438;
      background: #f8fafc;
      line-height: 1.55;
      --accent: #74401d;
      --tint: #fff4e9;
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
      <h1>Audit — Recovery for Portfolio Demo</h1>
      <p>A repeatable portfolio demonstration · Case DEMO-02</p>
    </header>
    <p class="notice">Review the preserved snapshot before replacing a working copy.</p>
    <div class="grid">
      <section class="panel result" aria-labelledby="result-title">
        <h2 id="result-title">Restored snapshot</h2>
        <dl>
          <dt>follow up</dt>
          <dd><pre id="result">true</pre></dd>
        </dl>
      </section>
      <section class="panel" aria-labelledby="sample-title">
        <h2 id="sample-title">Synthetic source data</h2>
        <details>
          <summary>Input records</summary>
          <pre id="input-records">{
  &quot;asset&quot;: &quot;DEMO-02&quot;,
  &quot;reviewed_on&quot;: &quot;2026-05-03&quot;,
  &quot;cutoff&quot;: &quot;2026-06-01&quot;,
  &quot;owner&quot;: &quot;Lab Operations&quot;
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
  "asset": "DEMO-02",
  "reviewed_on": "2026-05-03",
  "cutoff": "2026-06-01",
  "owner": "Lab Operations"
}
```

## Execution instructions

Copy the HTML block into `index.html` and open that file in a browser. Use **Input records** to toggle the synthetic data. Resize the window to compare the single-column and two-column layouts.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "follow_up": true
}
```

## Testing and validation

Actual tests used Chromium at 1100 and 360 pixels. Both sizes displayed the expected result without horizontal overflow. Enter toggled the input disclosure and the revealed JSON matched the sample.

```json
{
  "follow_up": true
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

Practice audit using HTML CSS and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable audit result for training case `DEMO-02`.

[Back to the HTML CSS project index](../README.md)
