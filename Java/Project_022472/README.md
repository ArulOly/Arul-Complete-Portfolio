# Java: Scheduled Reporting — Automation for Branch Office Simulator (Project 022472)

## Project overview

A simulated branch office provides the setting for this scheduled reporting exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs a repeatable scheduled reporting workflow with automation.

## Project objectives

- Calculate the next due date from an explicit reporting interval and reference date.
- Process a batch in order and keep valid results when individual records fail.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Calculate the next due date from an explicit reporting interval and reference date.
- Process a batch in order and keep valid results when individual records fail.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022472.
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

public class Project022472 {
    private static final Gson GSON = new GsonBuilder().setStrictness(Strictness.STRICT).setPrettyPrinting().create();
    private static final String USAGE = "Usage: sample.json";

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

    private static JsonArray array(JsonObject record, String name, boolean allowEmpty) {
        JsonElement value = record.get(name);
        if (value == null || !value.isJsonArray() || (!allowEmpty && value.getAsJsonArray().isEmpty())) {
            throw new IllegalArgumentException(name + " requires " + (allowEmpty ? "an array" : "a non-empty array"));
        }
        return value.getAsJsonArray();
    }

    private static LocalDate day(JsonObject record, String name) {
        String value = text(record, name);
        if (!value.matches("[0-9]{4}-[0-9]{2}-[0-9]{2}")) throw new IllegalArgumentException(name + " must use YYYY-MM-DD");
        try {
            return LocalDate.parse(value);
        } catch (DateTimeParseException error) {
            throw new IllegalArgumentException(name + " must be a real calendar date", error);
        }
    }

    private static Map<String, Object> evaluate(JsonObject r) throws NoSuchAlgorithmException {
        LocalDate previous = day(r, "previous_report"), asOf = day(r, "as_of");
        if (asOf.isBefore(previous)) throw new IllegalArgumentException("Reference date precedes previous report");
        LocalDate due = previous.plusDays(bounded(r, "interval_days", 1, 365));
        return Map.of("due_on", due.toString(), "due", !asOf.isBefore(due),
            "days_overdue", Math.max(0, ChronoUnit.DAYS.between(due, asOf)));
    }

    private static Map<String, Object> reply(Map<String, Object> body, int status) {
        return Map.of("status", status, "output", GSON.toJson(body));
    }

    public static Map<String, Object> run(String[] args) {
        if (args.length == 1 && args[0].equals("--help")) return Map.of("status", 0, "output", USAGE);
        try {
            if (args.length == 0) throw new IllegalArgumentException(USAGE);
            Path input = Path.of(args[0]);
            if (args.length != 1) throw new IllegalArgumentException(USAGE);
            JsonArray records = array(load(input), "records", false);
            List<Map<String, Object>> rows = new ArrayList<>();
            int status = 0, index = 0;
            for (JsonElement candidate : records) {
                index++;
                try {
                    JsonObject record = object(candidate, "record");
                    rows.add(Map.of("row", index, "asset", text(record, "asset"), "result", evaluate(record)));
                } catch (IllegalArgumentException error) {
                    status = 1;
                    rows.add(Map.of("row", index, "error", error.getMessage()));
                }
            }
            return reply(Map.of("rows", rows), status);
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

`evaluate()` applies the scheduled reporting rules. The CLI dispatch handles automation, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "records": [
    {
      "asset": "BRANCH-08",
      "previous_report": "2026-06-01",
      "as_of": "2026-06-16",
      "interval_days": 7
    },
    {
      "asset": "BRANCH-08-B",
      "previous_report": "2026-06-01",
      "as_of": "2026-06-16",
      "interval_days": 7
    }
  ]
}
```

## Execution instructions

Save the code as `Project022472.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022472.java
java -cp ".:gson-2.13.2.jar" Project022472 sample.json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "days_overdue": 8
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. A mixed batch retained its valid result, reported the invalid row, and returned status 1.

```json
{
  "days_overdue": 8
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

Practice scheduled reporting using Java and check a repeatable automation workflow in the branch office simulator scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `BRANCH-08`.

[Back to the Java project index](../README.md)
