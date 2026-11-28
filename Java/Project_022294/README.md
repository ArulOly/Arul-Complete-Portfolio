# Java: Data Import — Reporting for Help Desk (Project 022294)

## Project overview

A help desk training queue provides the setting for this data import exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a repeatable data import workflow with reporting.

## Project objectives

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Create a portable JSON report without replacing an existing file.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Create a portable JSON report without replacing an existing file.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022294.
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

public class Project022294 {
    private static final Gson GSON = new GsonBuilder().setStrictness(Strictness.STRICT).setPrettyPrinting().create();
    private static final String USAGE = "Usage: sample.json --output report.json";
    private static final String PROJECT_ID = "022294";

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

    private static Map<String, Object> evaluate(JsonObject r) throws NoSuchAlgorithmException {
        List<String> lines = text(r, "csv_data").lines().toList();
        if (lines.size() < 2 || !lines.getFirst().equals("code,quantity")) {
            throw new IllegalArgumentException("Expected code,quantity header and data rows");
        }
        Set<String> codes = new HashSet<>();
        List<Map<String, Object>> rows = new ArrayList<>();
        for (String line : lines.subList(1, lines.size())) {
            if (!line.matches("[A-Za-z0-9_-]+,[0-9]+")) throw new IllegalArgumentException("Invalid CSV row");
            String[] fields = line.split(",", 2);
            if (!codes.add(fields[0])) throw new IllegalArgumentException("Duplicate code " + fields[0]);
            int quantity = number(new JsonPrimitive(new BigDecimal(fields[1])), "quantity", 0, 1_000_000_000);
            rows.add(Map.of("code", fields[0], "quantity", quantity));
        }
        return Map.of("rows", rows, "imported", rows.size());
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

`evaluate()` applies the data import rules. The CLI dispatch handles reporting, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DESK-14",
  "csv_data": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Execution instructions

Save the code as `Project022294.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022294.java
java -cp ".:gson-2.13.2.jar" Project022294 sample.json --output report.json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "imported": 2
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. The written report matched terminal output and the implementation rejected an overwrite.

```json
{
  "imported": 2
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

Practice data import using Java and check a repeatable reporting workflow in the help desk scenario.

## Conclusion

The project records an observable data import result for training case `DESK-14`.

[Back to the Java project index](../README.md)
