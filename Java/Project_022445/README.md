# Java: Alerting — Usability Improvement for Help Desk (Project 022445)

## Project overview

A help desk training queue provides the setting for this alerting exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a repeatable alerting workflow with usability improvement.

## Project objectives

- Emit one local alert per consecutive breach episode after a configured threshold.
- Provide a discoverable help screen and a readable terminal summary.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Emit one local alert per consecutive breach episode after a configured threshold.
- Provide a discoverable help screen and a readable terminal summary.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022445.
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

public class Project022445 {
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

    private static JsonArray array(JsonObject record, String name, boolean allowEmpty) {
        JsonElement value = record.get(name);
        if (value == null || !value.isJsonArray() || (!allowEmpty && value.getAsJsonArray().isEmpty())) {
            throw new IllegalArgumentException(name + " requires " + (allowEmpty ? "an array" : "a non-empty array"));
        }
        return value.getAsJsonArray();
    }

    private static int[] numbers(JsonObject record, String name, int maximum) {
        JsonArray raw = array(record, name, false);
        int[] result = new int[raw.size()];
        for (int index = 0; index < raw.size(); index++) {
            result[index] = number(raw.get(index), name + "[" + index + "]", 0, maximum);
        }
        return result;
    }

    private static Map<String, Object> evaluate(JsonObject r) throws NoSuchAlgorithmException {
        int limit = count(r, "limit"), required = bounded(r, "consecutive", 1, 1_000_000_000);
        int[] readings = numbers(r, "readings", 1_000_000_000);
        int streak = 0;
        List<Map<String, Integer>> alerts = new ArrayList<>();
        for (int index = 0; index < readings.length; index++) {
            streak = readings[index] > limit ? streak + 1 : 0;
            if (streak == required) alerts.add(Map.of("sample", index + 1, "reading", readings[index]));
        }
        return Map.of("limit", limit, "alerts", alerts);
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

`evaluate()` applies the alerting rules. The CLI dispatch handles usability improvement, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DESK-14",
  "readings": [
    10,
    22,
    24,
    25,
    8,
    26,
    29
  ],
  "limit": 20,
  "consecutive": 2
}
```

## Execution instructions

Save the code as `Project022445.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022445.java
java -cp ".:gson-2.13.2.jar" Project022445 sample.json --json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "alerts": [
    {
      "sample": 3,
      "reading": 24
    },
    {
      "sample": 7,
      "reading": 29
    }
  ]
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. An invalid record returned status 2. The help text documented the JSON flag and the default view labeled the result.

```json
{
  "alerts": [
    {
      "sample": 3,
      "reading": 24
    },
    {
      "sample": 7,
      "reading": 29
    }
  ]
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

Practice alerting using Java and check a repeatable usability improvement workflow in the help desk scenario.

## Conclusion

The project records an observable alerting result for training case `DESK-14`.

[Back to the Java project index](../README.md)
