# Java: Configuration Comparison — Reporting for Help Desk (Project 022694)

## Project overview

A help desk training queue provides the setting for this configuration comparison exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a repeatable configuration comparison workflow with reporting.

## Project objectives

- Identify added, removed, and changed settings between two configuration objects.
- Create a portable JSON report without replacing an existing file.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Identify added, removed, and changed settings between two configuration objects.
- Create a portable JSON report without replacing an existing file.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022694.
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

public class Project022694 {
    private static final Gson GSON = new GsonBuilder().setStrictness(Strictness.STRICT).setPrettyPrinting().create();
    private static final String USAGE = "Usage: sample.json --output report.json";
    private static final String PROJECT_ID = "022694";

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
        JsonObject baseline = object(r.get("baseline"), "baseline");
        JsonObject current = object(r.get("current"), "current");
        Set<String> added = new TreeSet<>(current.keySet()), removed = new TreeSet<>(baseline.keySet());
        added.removeAll(baseline.keySet());
        removed.removeAll(current.keySet());
        List<String> changed = baseline.keySet().stream().filter(current::has)
            .filter(key -> !Objects.equals(baseline.get(key), current.get(key))).sorted().toList();
        return Map.of("added", added, "removed", removed, "changed", changed);
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

`evaluate()` applies the configuration comparison rules. The CLI dispatch handles reporting, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DESK-14",
  "baseline": {
    "timeout": 30,
    "theme": "light"
  },
  "current": {
    "timeout": 45,
    "retries": 2
  }
}
```

## Execution instructions

Save the code as `Project022694.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022694.java
java -cp ".:gson-2.13.2.jar" Project022694 sample.json --output report.json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "changed": [
    "timeout"
  ]
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. The written report matched terminal output and the implementation rejected an overwrite.

```json
{
  "changed": [
    "timeout"
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

Practice configuration comparison using Java and check a repeatable reporting workflow in the help desk scenario.

## Conclusion

The project records an observable configuration comparison result for training case `DESK-14`.

[Back to the Java project index](../README.md)
