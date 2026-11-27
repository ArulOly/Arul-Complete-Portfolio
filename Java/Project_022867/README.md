# Java: Automation — Automation for Public Dataset (Project 022867)

## Project overview

A synthetic public-data exercise provides the setting for this automation exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a repeatable automation workflow with automation.

## Project objectives

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Process a batch in order and keep valid results when individual records fail.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Java 21, Gson for JSON, and the Java standard library.

## System architecture

`JSON file → Gson parsing → validated Java workflow → CLI response`

## Features

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Process a batch in order and keep valid results when individual records fail.

## Prerequisites

Install JDK 21 or later and obtain Gson 2.13.2. Use a text editor and a local practice directory.

## Complete Java source code

```java
// Local Java reference implementation for project 022867.
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

public class Project022867 {
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
        record Task(List<String> needs, boolean ready) {}
        Map<String, Task> pending = new LinkedHashMap<>();
        for (JsonElement item : array(r, "tasks", false)) {
            JsonObject task = object(item, "task");
            String name = text(task, "name");
            JsonElement ready = task.get("ready");
            if (!(ready instanceof JsonPrimitive primitive) || !primitive.isBoolean() || pending.containsKey(name)) {
                throw new IllegalArgumentException("Each task needs a unique name and Boolean ready flag");
            }
            pending.put(name, new Task(strings(task, "needs", true), ready.getAsBoolean()));
        }
        for (Task task : pending.values()) {
            if (!pending.keySet().containsAll(task.needs())) throw new IllegalArgumentException("Missing prerequisite");
        }
        Map<String, String> states = new LinkedHashMap<>();
        while (!pending.isEmpty()) {
            List<String> eligible = pending.keySet().stream()
                .filter(name -> states.keySet().containsAll(pending.get(name).needs())).toList();
            if (eligible.isEmpty()) throw new IllegalArgumentException("Task dependency cycle");
            for (String name : eligible) {
                Task task = pending.remove(name);
                boolean blocked = task.needs().stream().anyMatch(need -> !states.get(need).equals("done"));
                states.put(name, blocked ? "blocked" : task.ready() ? "done" : "failed");
            }
        }
        return Map.of("task_states", states);
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

`evaluate()` applies the automation rules. The CLI dispatch handles automation, while Gson parses the source and serializes results.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "records": [
    {
      "asset": "DATA-12",
      "tasks": [
        {
          "name": "collect",
          "ready": true,
          "needs": []
        },
        {
          "name": "verify",
          "ready": false,
          "needs": [
            "collect"
          ]
        },
        {
          "name": "publish",
          "ready": true,
          "needs": [
            "verify"
          ]
        }
      ]
    },
    {
      "asset": "DATA-12-B",
      "tasks": [
        {
          "name": "collect",
          "ready": true,
          "needs": []
        },
        {
          "name": "verify",
          "ready": false,
          "needs": [
            "collect"
          ]
        },
        {
          "name": "publish",
          "ready": true,
          "needs": [
            "verify"
          ]
        }
      ]
    }
  ]
}
```

## Execution instructions

Save the code as `Project022867.java` and the input as `sample.json` in the same directory. Download the [Gson 2.13.2 JAR](https://repo.maven.apache.org/maven2/com/google/code/gson/gson/2.13.2/gson-2.13.2.jar) into that directory.

```sh
javac -cp gson-2.13.2.jar Project022867.java
java -cp ".:gson-2.13.2.jar" Project022867 sample.json
```

On Windows, use `".;gson-2.13.2.jar"` as the runtime classpath. Pass `--help` for usage.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Testing and validation

Actual validation compiled the source with JDK 21 and used temporary JSON files. The CLI returned status 0 and the sample outcome matched the repeated application run. A mixed batch retained its valid result, reported the invalid row, and returned status 1.

```json
{
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
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

Practice automation using Java and check a repeatable automation workflow in the public dataset scenario.

## Conclusion

The project records an observable automation result for training case `DATA-12`.

[Back to the Java project index](../README.md)
