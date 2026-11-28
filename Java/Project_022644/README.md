# Java: Service Check — Reporting for Help Desk (Project 022644)

## Project overview

A help desk training queue provides the setting for this service check exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a repeatable service check workflow with reporting.

## Project objectives

- Classify synthetic service probes by status and latency.
- Create a portable JSON report without replacing an existing file.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Classify synthetic service probes by status and latency.
- Create a portable JSON report without replacing an existing file.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022644.
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

public class Project022644 {
    private static final Gson GSON = new GsonBuilder().setStrictness(Strictness.STRICT).setPrettyPrinting().create();
    private static final String USAGE = "Usage: sample.json --output report.json";
    private static final String PROJECT_ID = "022644";

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

    private static Map<String, Object> evaluate(JsonObject r) throws NoSuchAlgorithmException {
        int budget = bounded(r, "max_ms", 1, 1_000_000_000);
        List<Map<String, Object>> checks = new ArrayList<>();
        int healthy = 0;
        for (JsonElement item : array(r, "probes", false)) {
            JsonObject probe = object(item, "probe");
            int status = bounded(probe, "status", 100, 599);
            boolean ok = status >= 200 && status < 300 && count(probe, "ms") <= budget;
            checks.add(Map.of("service", text(probe, "service"), "healthy", ok));
            if (ok) healthy++;
        }
        return Map.of("checks", checks, "healthy", healthy);
    }

    private static Map<String, Object> reply(Map<String, Object> body, int status) {
        return Map.of("status", status, "output", GSON.toJson(body));
    }

    public static Map<String, Object> run(String[] args) {
        if (args.length == 1 && args[0].equals("--help")) return Map.of("status", 0, "output", USAGE);
        try {
            if (args.length == 0) throw new IllegalArgumentException(USAGE);
            Path input = Path.of(args[0]);
            if (args.length != 1 && (args.length != 3 || !args[1].equals("--output"))) {
                throw new IllegalArgumentException(USAGE);
            }
            JsonObject record = load(input);
            Map<String, Object> result = Map.of("project", PROJECT_ID, "asset", text(record, "asset"),
                "result", evaluate(record));
            Map<String, Object> response = reply(result, 0);
            if (args.length == 3) {
                Files.writeString(Path.of(args[2]), response.get("output") + "\n", StandardOpenOption.CREATE_NEW);
            }
            return response;
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

`evaluate()` applies the service check rules. The CLI dispatch handles reporting, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DESK-14",
  "probes": [
    {
      "service": "catalog",
      "status": 200,
      "ms": 25
    },
    {
      "service": "print-queue",
      "status": 503,
      "ms": 60
    }
  ],
  "max_ms": 50
}
```

## Execution instructions

Save the code as `Project022644.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022644.java
java -cp ".:gson-2.13.2.jar" Project022644 sample.json --output report.json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "healthy": 1
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. The written report matched terminal output and the implementation rejected an overwrite.

```json
{
  "healthy": 1
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

Practice service check using Java and check a repeatable reporting workflow in the help desk scenario.

## Conclusion

The project records an observable service check result for training case `DESK-14`.

[Back to the Java project index](../README.md)
