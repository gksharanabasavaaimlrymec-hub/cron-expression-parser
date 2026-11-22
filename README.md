# cron-expression-parser

A small command-line application that expands the five standard numeric cron time fields and prints the values in the format requested in the GOAVEGA take-home task.

## Requirements

- Java 25 or later
- Apache Maven 3.9 or later

Check your environment:

```bash
java -version
mvn -version
```

## Build and test

From the project root:

```bash
mvn clean test
mvn package
```

The executable JAR is created at `target/cron-expression-parser-1.0.0.jar`.

## Run

Pass the entire cron expression as **one quoted command-line argument**:

```bash
java -jar target/cron-expression-parser-1.0.0.jar "*/15 0 1,15 * 1-5 /usr/bin/find"
```

Expected output:

```text
minute        0 15 30 45
hour          0
day of month  1 15
month         1 2 3 4 5 6 7 8 9 10 11 12
day of week   1 2 3 4 5
command       /usr/bin/find
```

## Supported syntax

Each of the five time fields supports:

| Syntax | Meaning | Example |
|---|---|---|
| `*` | Every allowed value | `*` |
| `n` | One specific value | `5` |
| `a-b` | Inclusive range | `1-5` |
| `*/s` | Every `s` values, starting at the field minimum | `*/15` |
| `a-b/s` | Inclusive range in steps of `s` | `10-20/2` |
| `n/s` | Starting at `n`, then every `s` values through the field maximum | `5/10` |
| `a,b,c` | Comma-separated items | `1,5,10` |

Comma-separated items can combine the supported forms, for example `1,10-20/2,*/15`.

## Field bounds

| Field | Minimum | Maximum |
|---|---:|---:|
| Minute | 0 | 59 |
| Hour | 0 | 23 |
| Day of month | 1 | 31 |
| Month | 1 | 12 |
| Day of week | 0 | 7 |

Day of week accepts both `0` and `7` as numeric values because cron implementations commonly use either for Sunday. The parser prints the numeric values supplied/expanded; it does not normalize 7 to 0.

## Design

- `CronParser` splits the input into five time fields and the remaining command. A limit of six split parts preserves spaces inside the command.
- `CronField` owns the parsing and expansion logic for one field. Bounds are supplied by the caller, so the logic is reusable for other fields.
- `CronExpression` is an immutable result object.
- `Main` handles command-line input, output formatting, and user-facing errors.

The field expansion uses a sorted set, so output values are ascending and duplicate values from overlapping list items appear only once.

## Assumptions and limitations

- Supports the standard five numeric time fields plus a command.
- Does not support aliases such as `@yearly`, named months (`JAN`) or weekdays (`MON`), or seconds/year fields.
- Does not calculate actual future execution dates or apply cron day-of-month/day-of-week scheduling semantics; it only expands each field independently, as requested.
- Does not validate whether the command exists or can be executed.
- Field lists are printed in ascending numeric order.
- A single value with a step (such as `5/10`) expands from that value through the field maximum.
- A descending range (such as `10-2`), zero step, empty list item, non-numeric value, or out-of-range value is rejected with an error.
- Day of week accepts 0 through 7. It does not map 7 to 0.

## Error handling

Invalid input prints an error message to standard error and exits with a non-zero status. Invoking the program without exactly one argument prints usage instructions.

## Project layout

```text
cron-expression-parser/
├── pom.xml
├── README.md
├── .gitignore
└── src/
    ├── main/java/com/goavega/cron/
    │   ├── Main.java
    │   ├── CronParser.java
    │   ├── CronField.java
    │   └── CronExpression.java
    └── test/java/com/goavega/cron/
        ├── CronFieldTest.java
        └── CronParserTest.java
```

## GitHub submission instructions

1. Create a **public** repository on GitHub named `cron-expression-parser`.
2. In a terminal, change directory to this project folder.
3. Initialize Git and commit the project in logical steps:

```bash
git init
git add pom.xml README.md .gitignore
git commit -m "Add Maven project setup and documentation"

git add src/main/java
git commit -m "Implement cron field expansion and parser"

git add src/test
git commit -m "Add cron parser unit tests"

git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/cron-expression-parser.git
git push -u origin main
```

Replace `YOUR_GITHUB_USERNAME` with your GitHub username. If you create the repository with a README or license on GitHub, avoid initializing it with extra files to keep the first push straightforward.

Before submitting, run:

```bash
mvn clean test
mvn package
java -jar target/cron-expression-parser-1.0.0.jar "*/15 0 1,15 * 1-5 /usr/bin/find"
```

Then reply to the recruiter with the public repository URL.
