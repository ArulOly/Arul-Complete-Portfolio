# Java: Inventory — Usability Improvement for Help Desk (Project 022045)

## Project overview

A help desk training queue provides the setting for this inventory exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a repeatable inventory workflow with usability improvement.

## Project objectives

- Count items and identify stock below the reorder level.
- Provide a discoverable help screen and a readable terminal summary.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Count items and identify stock below the reorder level.
- Provide a discoverable help screen and a readable terminal summary.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022045.
import com.google.gson.*;
import java.io.IOException;
import java.math.BigDecimal;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.time.LocalDate;
import java.time.format.DateTimeParseException;
import java.time.temporal.ChronoUnit;
import java.util.*;

public class Project022045 {
    private static final Gson GSON = new GsonBuilder().setStrictness(Strictness.STRICT).setPrettyPrinting().create();
    private static final String USAGE = "Usage: sample.json --json";
    private static final String SCENARIO = "Help Desk";

    private static JsonObject load(Path path) throws IOException {
        return object(GSON.fromJson(Files.readString(path), JsonElement.class), "input");
    }

    private static JsonObject object(JsonElement element, String name) {
        if (element == null || !element.isJsonObject()) throw new IllegalArgumentException(name + " must be an object");
        return element.getAsJsonObject();
    }

    private static String rawText(JsonElement value, String name) {
        if (!(value instanceof JsonPrimitive primitive) || !primitive.isString() || primitive.getAsString().isBlank()) {
            throw new IllegalArgumentException(name + " must be non-empty text");
        }
        return primitive.getAsString();
    }

    private static String text(JsonObject record, String name) {
        return rawText(record.get(name), name).strip();
    }

    private static int number(JsonElement value, String name, int minimum, int maximum) {
        String error = name + " must be an integer from " + minimum + " to " + maximum;
        if (!(value instanceof JsonPrimitive primitive) || !primitive.isNumber()) {
            throw new IllegalArgumentException(error);
        }
        try {
            int result = primitive.getAsBigDecimal().intValueExact();
            if (result < minimum || result > maximum) throw new IllegalArgumentException(error);
            return result;
        } catch (ArithmeticException | NumberFormatException cause) {
            throw new IllegalArgumentException(error, cause);
        }
    }

    private static int bounded(JsonObject record, String name, int minimum, int maximum) {
        return number(record.get(name), name, minimum, maximum);
    }

    private static int count(JsonObject record, String name) {
        return bounded(record, name, 0, 1_000_000_000);
    }

    private static Map<String, Object> evaluate(JsonObject r) throws NoSuchAlgorithmException {
        int stock = count(r, "stock"), threshold = bounded(r, "reorder_at", 1, 1_000_000_000);
        return Map.of("stock", stock, "reorder_at", threshold, "needs_reorder", stock < threshold);
    }

    private static Map<String, Object> reply(Map<String, Object> body, int status) {
        return Map.of("status", status, "output", GSON.toJson(body));
    }

    public static Map<String, Object> run(String[] args) {
        if (args.length == 1 && args[0].equals("--help")) return Map.of("status", 0, "output", USAGE);
        try {
            if (args.length == 0) throw new IllegalArgumentException(USAGE);
            Path input = Path.of(args[0]);
            if (args.length != 1 && (args.length != 2 || !args[1].equals("--json"))) {
                throw new IllegalArgumentException(USAGE);
            }
            JsonObject record = load(input);
            String asset = text(record, "asset");
            Map<String, Object> result = evaluate(record);
            if (args.length == 2) return reply(Map.of("asset", asset, "result", result), 0);
            StringBuilder output = new StringBuilder(SCENARIO).append(" | ").append(asset);
            new TreeMap<>(result).forEach((key, value) ->
                output.append('\n').append(key.replace('_', ' ')).append(": ").append(GSON.toJson(value)));
            return Map.of("status", 0, "output", output.toString());
        } catch (IOException | JsonParseException | IllegalArgumentException | NoSuchAlgorithmException error) {
            return Map.of("status", 2, "error", "Input or output error: " + error.getMessage());
        }
    }

    public static void main(String[] args) {
        Map<String, Object> result = run(args);
        if (result.containsKey("output")) System.out.println(result.get("output"));
        if (result.containsKey("error")) System.err.println(result.get("error"));
        System.exit((Integer) result.get("status"));
    }
}
```

## Implementation explanation

`evaluate()` applies the inventory rules. The CLI dispatch handles usability improvement, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DESK-14",
  "stock": 2,
  "reorder_at": 4
}
```

## Execution instructions

Save the code as `Project022045.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022045.java
java -cp ".:gson-2.13.2.jar" Project022045 sample.json --json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "needs_reorder": true
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. An invalid record returned status 2. The help text documented the JSON flag and the default view labeled the result.

```json
{
  "needs_reorder": true
}
```

## Error handling

Invalid invocation, malformed input, and file failures produce a diagnostic and a status of 2. The batch focus also reports per-row errors with status 1.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This command-line exercise works on local synthetic files. Structured input fields follow the schema shown in the sample.

## Learning outcomes

Practice inventory using Java and check a repeatable usability improvement workflow in the help desk scenario.

## Conclusion

The project records an observable inventory result for training case `DESK-14`.

[Back to the Java project index](../README.md)
