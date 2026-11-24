# DevTrace
## Terminal-Native Debugging Workbench

> **Understand what happened. Find out why.**

| Project detail | Value |
|---|---|
| Project type | Developer Tools / Terminal UI |
| Language | Go |
| Framework | Bubble Tea |
| Estimated build time | 8–10 hours |
| Difficulty | Ambitious but achievable |

## 1. Project Overview

DevTrace is an interactive terminal-based debugging workbench that transforms raw application logs into navigable timelines, error clusters, and correlated event investigations.

Instead of manually scrolling through thousands of lines of logs, developers can use DevTrace to:

- Monitor application logs in real time.
- Identify errors and warnings.
- Search and filter log events.
- Visualize event timelines.
- Detect repeated errors.
- Investigate sequences of related events.
- Identify unusual spikes in errors.

The main selling point is that DevTrace doesn't simply display logs. It helps developers understand what happened before, during, and after an error.

## 2. Example User Experience

Run DevTrace:

```bash
devtrace ./logs/application.log
```

The application opens an interactive dashboard:

```text
╭─ DEVTRACE ── application.log ───────── LIVE ─╮
│                                              │
│  OVERVIEW                                    │
│  Events: 12,482   Errors: 34   Warnings: 128 │
│                                              │
│  ERROR RATE                                  │
│  10:00 ▂▃▂▂▅▇▃▂▂▁▃▅▇▂▁  10:15                │
│                                              │
│  EVENT TIMELINE                              │
│  10:12:03  INFO   Server started             │
│  10:12:04  INFO   Database connected         │
│  10:12:08  WARN   Slow database query        │
│  10:12:09  ERROR  Connection timeout         │
│  10:12:10  ERROR  Request failed             │
│                                              │
│  ── SELECTED EVENT ────────────────────────  │
│  ERROR: Connection timeout                   │
│  File: database/connection.go:84             │
│  Time: 10:12:09                              │
│                                              │
│  [j/k] Navigate  [/] Search  [f] Filter      │
│  [e] Errors  [t] Timeline  [q] Quit          │
╰──────────────────────────────────────────────╯
```

### Keyboard shortcuts

| Key | Action |
|---|---|
| `j` / `k` | Navigate events |
| `/` | Search logs |
| `f` | Open filters |
| `e` | Show errors |
| `t` | Show timeline |
| `Enter` | Inspect selected event |
| `r` | Refresh |
| `q` | Quit |

## 3. Features

### Core Features

These are essential for the first working version.

- **Log ingestion:** Read application log files.
- **Log parsing:** Parse JSON and conventional text logs.
- **Severity categorization:** Identify INFO, WARN, ERROR, and DEBUG.
- **Interactive timeline:** Navigate events chronologically.
- **Search:** Search messages and source locations.
- **Filtering:** Filter by severity and time.
- **Event inspector:** Display full details of selected events.
- **Statistics:** Display event counts and error rates.

### Advanced Features

These make the project more impressive.

- **Error clustering:** Group repeated errors together.
- **Error correlation:** Identify events occurring before and after an error.
- **Live tailing:** Monitor a log file as it grows.
- **Anomaly detection:** Highlight unusual spikes in errors.
- **Export:** Export filtered results to JSON or CSV.

### Stretch Features

Only implement these if time permits.

- Git integration to associate errors with recent commits.
- Stack trace visualization.
- AI-powered error explanations.
- Compare two log files to identify changes.

> Prioritize error correlation and live tailing over AI integration.

## 4. Technology Stack

### Go

The primary programming language.

Reasons:

- Fast log processing.
- Excellent concurrency support.
- Easy binary distribution.
- Strong terminal UI ecosystem.

### Bubble Tea

Repository: <https://github.com/charmbracelet/bubbletea>

Used for:

- Application state.
- Keyboard input.
- Navigation.
- Rendering.
- Event handling.

### Lipgloss

Repository: <https://github.com/charmbracelet/lipgloss>

Used for:

- Terminal styling.
- Colors.
- Borders.
- Layouts.
- Visual themes.

### Bubbles

Repository: <https://github.com/charmbracelet/bubbles>

Useful components:

- Tables.
- Viewports.
- Text inputs.
- Lists.
- Spinners.

### Additional Libraries

| Library | Purpose |
|---|---|
| `fsnotify` | File change monitoring |
| SQLite | Optional persistent storage |
| `encoding/json` | JSON log parsing |
| `regexp` | Text log parsing |

## 5. Architecture

The application should have several independent components.

```text
             LOG FILES
                 |
                 v
          LOG INGESTION
                 |
                 v
               PARSER
                 |
                 v
             EVENT STORE
                 |
                 v
          ANALYSIS ENGINE
            /    |    \
           /     |     \
          v      v      v
      CLUSTER  ANOMALY  TIMELINE
          \      |      /
           \     |     /
            v    v    v
             BUBBLE TEA
                 |
                 v
             TERMINAL UI
              /       \
             v         v
        INSPECTOR    EXPORTER
```

### Component responsibilities

#### Parser

- Read incoming log lines.
- Detect JSON or text formats.
- Extract timestamps, severity, messages, and source locations.
- Preserve raw log content.

#### Event Store

- Store normalized events.
- Support chronological ordering.
- Provide filtering and searching.

#### Analysis Engine

- Cluster repeated errors.
- Correlate related events.
- Calculate event statistics.
- Detect unusual error spikes.

#### Watcher

- Monitor file changes.
- Read newly appended lines.
- Send parsed events to the event store.

#### UI

- Display the dashboard.
- Handle keyboard navigation.
- Render timelines and statistics.
- Display event details.

## 6. Project Structure

```text
devtrace/
├── cmd/
│   └── devtrace/
│       └── main.go
├── internal/
│   ├── parser/
│   │   ├── parser.go
│   │   └── formats.go
│   ├── engine/
│   │   ├── analyzer.go
│   │   ├── clustering.go
│   │   └── anomalies.go
│   ├── watcher/
│   │   └── watcher.go
│   ├── store/
│   │   └── events.go
│   └── ui/
│       ├── app.go
│       ├── dashboard.go
│       ├── timeline.go
│       └── inspector.go
├── testdata/
│   └── application.log
├── go.mod
└── README.md
```

Keep the parser and analysis engine independent of the UI. This makes them easier to test.

## 7. Event Data Model

Every parsed log line should become a normalized event.

```go
type Event struct {
    ID        int
    Timestamp time.Time
    Level     string
    Message   string
    Source    string
    Line      int
    Raw       string
}
```

This structure makes searching, sorting, filtering, clustering, and displaying events easier.

Support at least two input formats:

- JSON structured logs.
- Conventional text logs with timestamps and severity levels.

For unstructured logs, use a fallback parser that preserves the original line instead of discarding it.

## 8. Standout Feature: Error Correlation

This is the centerpiece of DevTrace.

Consider these application logs:

```text
10:15:01 INFO  User authenticated
10:15:02 INFO  Request received
10:15:03 WARN  Database latency: 900ms
10:15:04 ERROR Database timeout
10:15:04 ERROR Request failed
10:15:05 ERROR Retry exhausted
```

Instead of treating these as six unrelated events, DevTrace attempts to identify a sequence of related events.

The UI could display:

```text
╭─ ERROR INVESTIGATION ─────────────────╮
│                                       │
│  ROOT EVENT                           │
│  Database timeout                     │
│                                       │
│  RELATED EVENTS                       │
│                                       │
│  10:15:02  Request received           │
│      ↓                                │
│  10:15:03  Database latency warning   │
│      ↓                                │
│  10:15:04  Database timeout           │
│      ↓                                │
│  10:15:05  Retry exhausted            │
│                                       │
│  POSSIBLE CAUSE                       │
│  Database response latency increased │
│                                       │
╰───────────────────────────────────────╯
```

### Implementation approach

Start with simple correlation:

1. Find an error event.
2. Look at events within a configurable time window.
3. Match shared request IDs where available.
4. Identify repeated error signatures.
5. Display surrounding events chronologically.
6. Highlight possible relationships.

> **Important:** Temporal proximity alone does not prove causation. Label inferred relationships as possible correlations.

## 9. CLI Interface

DevTrace should feel like a real developer tool.

```bash
# Open a log file
devtrace ./app.log

# Follow logs live
devtrace ./app.log --follow

# Load JSON logs
devtrace ./app.jsonl --format json

# Filter to errors
devtrace ./app.log --level error

# Export investigation results
devtrace ./app.log --export report.json
```

## 10. Development Roadmap

**Target:** approximately 9 hours.

### Roadmap overview

| Time | Task | Deliverable |
|---|---|---|
| 0–1h | Project setup and TUI skeleton | Working dashboard |
| 1–2.5h | Log ingestion and parsing | Normalized events |
| 2.5–4h | Interactive dashboard | Timeline and inspector |
| 4–5h | Search and filtering | Interactive log search |
| 5–6.5h | Error correlation | Investigation view |
| 6.5–7.5h | Live tailing and statistics | Real-time updates |
| 7.5–8.5h | Polish and testing | Stable application |
| 8.5–9h | Demo preparation | Reproducible demo |

### Hour 0–1: Setup

- Initialize Go module.
- Install Bubble Tea, Bubbles, and Lipgloss.
- Create application model.
- Implement basic terminal layout.
- Add keyboard navigation.

### Hour 1–2.5: Parsing

- Implement file reader.
- Parse JSON logs.
- Parse conventional text logs.
- Normalize entries into Event structs.
- Add sample test data.

### Hour 2.5–4: Dashboard

- Build event list.
- Build event inspector.
- Add severity colors.
- Implement navigation.
- Add event statistics.

### Hour 4–5: Search and Filtering

- Implement text search.
- Add severity filters.
- Add time filtering.
- Display filtered results.

### Hour 5–6.5: Error Correlation

- Identify repeated error messages.
- Find related events.
- Build chronological sequences.
- Create investigation view.

### Hour 6.5–7.5: Live Tailing

- Integrate `fsnotify`.
- Detect file modifications.
- Read appended content.
- Update dashboard statistics.

### Hour 7.5–8.5: Polish

- Improve styling.
- Handle malformed logs.
- Test keyboard navigation.
- Test large files.
- Fix bugs.

### Hour 8.5–9: Demo

- Prepare realistic sample logs.
- Create a reproducible error scenario.
- Test the complete demo flow.
- Write README and usage instructions.

## 11. Hackathon Demo Strategy

Don't just demonstrate that the application can read logs.

Prepare a fictional broken application with:

- A normal startup sequence.
- Gradually increasing database latency.
- A burst of failed requests.
- Repeated timeout errors.
- A recovery sequence.

Then demonstrate DevTrace:

1. Launch DevTrace and show the dashboard.
2. Identify an error spike on the timeline.
3. Select an error and inspect its details.
4. Open error correlation and reveal preceding events.
5. Enable live tailing and append new events.
6. Filter to errors and export the investigation.

This gives the demo a clear story and showcases the actual value of the application.

## 12. Feature Prioritization

| Feature | Priority |
|---|---|
| Beautiful dashboard | Essential |
| Log parsing | Essential |
| Timeline | Essential |
| Search and filters | Essential |
| Event inspector | Essential |
| Error correlation | Essential |
| Live tailing | High |
| Anomaly detection | If time |
| Git integration | Skip |
| AI explanations | Skip |

> Do not sacrifice the core debugging experience for extra features.

## 13. Project Identity

**Name:** DevTrace

**Tagline:** Understand what happened. Find out why.

### Target audience

- Backend developers.
- DevOps engineers.
- Software engineers working with application logs.

### Core selling point

Turn thousands of log lines into an interactive debugging investigation.

### GitHub description

> DevTrace is a terminal-native debugging workbench that transforms raw application logs into interactive timelines, error clusters, and correlated event investigations. Built with Go and Bubble Tea.

## Final Recommendation

Make error correlation the feature judges remember.

There are plenty of terminal dashboards, but a TUI that takes a messy log file and reconstructs a useful debugging story has a much stronger identity.

Build a working engine first, get the dashboard running early, and spend the final hours making the experience polished and memorable.
