# HTML CSS: Backup Verification — Baseline Setup for Home Development (Project 023211)

## Project overview

A home development workspace provides the setting for this backup verification exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The home development needs a readable backup verification display focused on baseline setup.

## Project objectives

- Present the backup verification outcome with a labeled result.
- Show the baseline result alongside its source records.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

HTML, CSS Grid, responsive media queries, and native HTML disclosure controls.

## System architecture

`Synthetic fixture → semantic HTML → CSS layout → accessible report in the browser`

## Features

- Present the backup verification outcome with a labeled result.
- Show the baseline result alongside its source records.

## Prerequisites

Use a modern browser. Use a text editor and a local practice directory.

## Complete HTML CSS source code

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Backup Verification — Baseline Setup for Home Development | Project 023211</title>
  <style>
    :root {
      color-scheme: light;
      font-family: system-ui, sans-serif;
      color: #172438;
      background: #f8fafc;
      line-height: 1.55;
      --accent: #1b4781;
      --tint: #eef4ff;
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
      <h1>Backup Verification — Baseline Setup for Home Development</h1>
      <p>A home development workspace · Case DEV-05</p>
    </header>
    <div class="grid">
      <section class="panel result" aria-labelledby="result-title">
        <h2 id="result-title">Reference snapshot</h2>
        <dl>
          <dt>verified</dt>
          <dd><pre id="result">true</pre></dd>
        </dl>
      </section>
      <section class="panel" aria-labelledby="sample-title">
        <h2 id="sample-title">Synthetic source data</h2>
        <details open>
          <summary>Input records</summary>
          <pre id="input-records">{
  &quot;asset&quot;: &quot;DEV-05&quot;,
  &quot;payload&quot;: &quot;Backup manifest for DEV-05, revision 1&quot;,
  &quot;sha256&quot;: &quot;41483b649918b02313ecc8d7759ec466c5f64526a1fc8380d89b76c597672551&quot;
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
  "asset": "DEV-05",
  "payload": "Backup manifest for DEV-05, revision 1",
  "sha256": "41483b649918b02313ecc8d7759ec466c5f64526a1fc8380d89b76c597672551"
}
```

## Execution instructions

Copy the HTML block into `index.html` and open that file in a browser. Use **Input records** to toggle the synthetic data. Resize the window to compare the single-column and two-column layouts.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "verified": true
}
```

## Testing and validation

Actual tests used Chromium at 1100 and 360 pixels. Both sizes displayed the expected result without horizontal overflow. Enter toggled the input disclosure and the revealed JSON matched the sample.

```json
{
  "verified": true
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

Practice backup verification using HTML CSS and check a repeatable baseline setup workflow in the home development scenario.

## Conclusion

The project records an observable backup verification result for training case `DEV-05`.

[Back to the HTML CSS project index](../README.md)
