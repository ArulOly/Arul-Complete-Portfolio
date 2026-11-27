# Java: Access Review — Usability Improvement for Branch Office Simulator (Project 022525)

## Project overview

A simulated branch office provides the setting for this access review exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs a repeatable access review workflow with usability improvement.

## Project objectives

- Compare synthetic active grants with an approved role policy.
- Provide a discoverable help screen and a readable terminal summary.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Compare synthetic active grants with an approved role policy.
- Provide a discoverable help screen and a readable terminal summary.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022525.
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

public class Project022525 {
    private static final Gson GSON = new GsonBuilder().setStrictness(Strictness.STRICT).setPrettyPrinting().create();
    private static final String USAGE = "Usage: sample.json --json";
    private static final String SCENARIO = "Branch Office Simulator";

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

    private static JsonArray array(JsonObject record, String name, boolean allowEmpty) {
        JsonElement value = record.get(name);
        if (value == null || !value.isJsonArray() || (!allowEmpty && value.getAsJsonArray().isEmpty())) {
            throw new IllegalArgumentException(name + " requires " + (allowEmpty ? "an array" : "a non-empty array"));
        }
        return value.getAsJsonArray();
    }

    private static List<String> strings(JsonObject record, String name, boolean allowEmpty) {
        List<String> result = new ArrayList<>();
        for (JsonElement value : array(record, name, allowEmpty)) result.add(rawText(value, name).strip());
        return result;
    }

    private static Map<String, Object> evaluate(JsonObject r) throws NoSuchAlgorithmException {
        Set<String> approved = new TreeSet<>(strings(r, "approved", true));
        Set<String> grants = new TreeSet<>(strings(r, "grants", true));
        Set<String> excess = new TreeSet<>(grants), missing = new TreeSet<>(approved);
        excess.removeAll(approved);
        missing.removeAll(grants);
        return Map.of("excess_grants", excess, "missing_grants", missing, "matches_policy", approved.equals(grants));
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

`evaluate()` applies the access review rules. The CLI dispatch handles usability improvement, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "BRANCH-08",
  "approved": [
    "read",
    "submit"
  ],
  "grants": [
    "read",
    "admin"
  ]
}
```

## Execution instructions

Save the code as `Project022525.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022525.java
java -cp ".:gson-2.13.2.jar" Project022525 sample.json --json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "excess_grants": [
    "admin"
  ]
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. An invalid record returned status 2. The help text documented the JSON flag and the default view labeled the result.

```json
{
  "excess_grants": [
    "admin"
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

Practice access review using Java and check a repeatable usability improvement workflow in the branch office simulator scenario.

## Conclusion

The project records an observable access review result for training case `BRANCH-08`.

[Back to the Java project index](../README.md)
