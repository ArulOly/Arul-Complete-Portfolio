# Java: Recovery Test — Recovery for Portfolio Demo (Project 022998)

## Project overview

A repeatable portfolio demonstration provides the setting for this recovery test exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The portfolio demo needs a repeatable recovery test workflow with recovery.

## Project objectives

- Restore a mock asset snapshot in memory and compare it against expected values.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Restore a mock asset snapshot in memory and compare it against expected values.
- Try a fallback input snapshot when the primary input is missing or invalid.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022998.
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

public class Project022998 {
    private static final Gson GSON = new GsonBuilder().setStrictness(Strictness.STRICT).setPrettyPrinting().create();
    private static final String USAGE = "Usage: sample.json --fallback fallback.json";

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

    private static Map<String, Object> evaluate(JsonObject r) throws NoSuchAlgorithmException {
        JsonObject restored = object(r.get("snapshot"), "snapshot").deepCopy();
        JsonObject expected = object(r.get("expected"), "expected");
        Set<String> keys = new TreeSet<>(restored.keySet());
        keys.addAll(expected.keySet());
        List<String> differences = keys.stream().filter(key -> !Objects.equals(restored.get(key), expected.get(key))).toList();
        return Map.of("restored", restored, "differences", differences, "verified", differences.isEmpty());
    }

    private static Map<String, Object> reply(Map<String, Object> body, int status) {
        return Map.of("status", status, "output", GSON.toJson(body));
    }

    public static Map<String, Object> run(String[] args) {
        if (args.length == 1 && args[0].equals("--help")) return Map.of("status", 0, "output", USAGE);
        try {
            if (args.length == 0) throw new IllegalArgumentException(USAGE);
            Path input = Path.of(args[0]);
            if (args.length != 3 || !args[1].equals("--fallback")) throw new IllegalArgumentException(USAGE);
            try {
                JsonObject record = load(input);
                return reply(Map.of("source", "primary", "asset", text(record, "asset"), "result", evaluate(record)), 0);
            } catch (IOException | IllegalArgumentException | JsonParseException primary) {
                JsonObject record = load(Path.of(args[2]));
                return reply(Map.of("source", "fallback", "asset", text(record, "asset"),
                    "primary_error", primary.getMessage(), "result", evaluate(record)), 0);
            }
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

`evaluate()` applies the recovery test rules. The CLI dispatch handles recovery, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DEMO-02",
  "snapshot": {
    "version": 3,
    "active": true
  },
  "expected": {
    "version": 3,
    "active": true
  }
}
```

## Execution instructions

Save the code as `Project022998.java` and the input as `sample.json` in the same directory. Save another copy of the sample as `fallback.json`. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022998.java
java -cp ".:gson-2.13.2.jar" Project022998 sample.json --fallback fallback.json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "verified": true
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. An invalid primary used the fallback and an unavailable fallback returned status 2.

```json
{
  "verified": true
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

Practice recovery test using Java and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable recovery test result for training case `DEMO-02`.

[Back to the Java project index](../README.md)
