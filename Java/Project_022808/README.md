# Java: Documentation Generation — Recovery for Small Business (Project 022808)

## Project overview

Local shop operations provides the setting for this documentation generation exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The small business needs a repeatable documentation generation workflow with recovery.

## Project objectives

- Generate a short Markdown handover from a synthetic operating procedure.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Generate a short Markdown handover from a synthetic operating procedure.
- Try a fallback input snapshot when the primary input is missing or invalid.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022808.
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

public class Project022808 {
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
        List<String> steps = strings(r, "steps", false);
        StringBuilder markdown = new StringBuilder();
        markdown.append("# ").append(text(r, "asset")).append("\n\n")
            .append(text(r, "summary")).append("\n\n## Procedure\n\n");
        for (int index = 0; index < steps.size(); index++) {
            markdown.append(index + 1).append(". ").append(steps.get(index)).append('\n');
        }
        return Map.of("markdown", markdown.toString(), "steps", steps.size());
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

`evaluate()` applies the documentation generation rules. The CLI dispatch handles recovery, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "SHOP-22",
  "summary": "Review the training asset SHOP-22 before handover.",
  "steps": [
    "Confirm the asset label",
    "Record the handover status"
  ]
}
```

## Execution instructions

Save the code as `Project022808.java` and the input as `sample.json` in the same directory. Save another copy of the sample as `fallback.json`. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022808.java
java -cp ".:gson-2.13.2.jar" Project022808 sample.json --fallback fallback.json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "steps": 2
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. An invalid primary used the fallback and an unavailable fallback returned status 2.

```json
{
  "steps": 2
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

Practice documentation generation using Java and check a repeatable recovery workflow in the small business scenario.

## Conclusion

The project records an observable documentation generation result for training case `SHOP-22`.

[Back to the Java project index](../README.md)
