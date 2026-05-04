---
title: LLD - Logging Framework
description: Design log4j from scratch. Levels with per-logger thresholds, a chain of appenders that each format and write somewhere, and an async path so a log call doesn't block on a disk write. The patterns are Chain of Responsibility, Strategy, and a bounded queue.
date: 2026-05-04
draft: false
slug: /system-design/logging-framework
tags:
  - System Design
  - Low-Level Design
  - OOD
---

`log.info("order placed", orderId)` should be cheap when the level is below the threshold (a comparison and a return), cheap when it's enabled (enqueue and move on), and it should be able to fan the same message to a console, a rolling file, and a network collector, each with its own format. That's what log4j, Logback, and `java.util.logging` do, and it's a compact LLD problem: levels, a chain of outputs, and an async write path.

## Scope

- Levels: `TRACE < DEBUG < INFO < WARN < ERROR < FATAL`.
- Named loggers with per-logger thresholds, inherited by hierarchy (`com.app.db` inherits from `com.app`).
- Multiple destinations (appenders), each with its own level and format.
- Low overhead; thread-safe; async option; flush on shutdown.

## Model

```java
enum Level { TRACE, DEBUG, INFO, WARN, ERROR, FATAL }

record LogEvent(Instant time, Level level, String loggerName,
                String message, Object[] args, Thread thread, Throwable error) {}

interface Formatter { String format(LogEvent e); }          // Strategy

interface Appender {
    void append(LogEvent e);                                // write it somewhere
    Level threshold();
    void close();
}

class Logger {
    final String name;
    private Level level;                    // null ⇒ inherit from parent
    private final List<Appender> appenders = new CopyOnWriteArrayList<>();
    private final Logger parent;

    void info(String msg, Object... args)  { log(Level.INFO, msg, args, null); }
    void error(String msg, Throwable t)    { log(Level.ERROR, msg, new Object[0], t); }

    void log(Level lvl, String msg, Object[] args, Throwable t) {
        if (lvl.compareTo(effectiveLevel()) < 0) return;    // cheap early-out
        LogEvent e = new LogEvent(Instant.now(), lvl, name, msg, args, currentThread(), t);
        for (Logger l = this; l != null; l = l.parent)      // walk up, hit each logger's appenders
            for (Appender a : l.appenders)
                if (lvl.compareTo(a.threshold()) >= 0) a.append(e);
    }

    private Level effectiveLevel() {
        for (Logger l = this; l != null; l = l.parent)
            if (l.level != null) return l.level;
        return Level.INFO;                                   // root default
    }
}
```

`effectiveLevel` walks up the logger hierarchy until it finds a set level — so configuring `com.app` to `DEBUG` turns on debug for every logger under it. The level check is the first line and is just an enum comparison, so a disabled log call costs almost nothing (defer string formatting until an appender actually wants it).

## Appenders as a chain

Each appender is one destination and format:

```java
class ConsoleAppender implements Appender {
    private final Formatter fmt; private final Level threshold;
    public void append(LogEvent e) { System.out.print(fmt.format(e)); }
}

class RollingFileAppender implements Appender {
    private final Formatter fmt; private final Level threshold;
    private final long maxBytes; private final int maxFiles;
    private Writer out; private long written;
    public synchronized void append(LogEvent e) {
        if (written > maxBytes) roll();        // close, rename .1 -> .2, open fresh
        String line = fmt.format(e);
        out.write(line); written += line.length();
    }
}
```

Others: `SocketAppender` (to a log collector), `KafkaAppender`, `AsyncAppender` (below). A logger holds a list, and the message goes to all of them whose threshold it passes — a fan-out, effectively Chain of Responsibility where every link handles the event rather than one consuming it.

`Formatter` is a Strategy: `PatternFormatter("%time %level [%thread] %logger - %msg%n")`, `JsonFormatter` for structured logs.

## The async path

A synchronous `RollingFileAppender.append` blocks the calling thread on a disk write. `AsyncAppender` wraps another appender with a bounded queue and a background thread:

```java
class AsyncAppender implements Appender {
    private final Appender delegate;
    private final BlockingQueue<LogEvent> queue = new ArrayBlockingQueue<>(8192);
    private final Thread worker;
    private final OverflowPolicy overflow;   // BLOCK, DROP_OLDEST, DROP_NEW

    public void append(LogEvent e) {
        if (!queue.offer(e)) applyOverflow(e);   // queue full: what now?
    }
    // worker: while running, take() and delegate.append(); drain on shutdown
}
```

The overflow policy is a real decision: **block** (backpressure — a slow disk slows the app, but no logs lost), **drop oldest** (keep the app fast, lose the least-recent buffered lines), or **drop new** (keep the app fast, lose the newest). Most services pick drop-oldest for non-critical logs and block for audit logs. Flush the queue on JVM shutdown (a shutdown hook) so the last few lines aren't lost.

## Configuration

A `LoggerFactory` (singleton) parses a config file (`logging.yaml`) into logger levels and appender definitions, and `LoggerFactory.getLogger("com.app.db")` returns (and caches) the logger, wiring up the hierarchy. Support hot-reload — re-read the config and adjust levels without a restart — since "turn on DEBUG for this package in prod" is a common need.

## Concurrency

- The level check and event creation are lock-free.
- `CopyOnWriteArrayList` for the appender list — reads (every log call) are unsynchronized; the rare reconfiguration copies.
- Each appender is responsible for its own thread safety: `ConsoleAppender` relies on `System.out` being synchronized; `RollingFileAppender.append` is `synchronized`; `AsyncAppender` moves the contention off the caller onto its single worker.
- Add a **thread/request context** (MDC — mapped diagnostic context) as a `ThreadLocal<Map>` so every log line in a request carries its correlation ID.

## Extensions

- **Sampling** — an appender that logs 1 in N of repetitive events.
- **Rate limiting** — collapse a burst of identical messages into "…repeated 4,213 times".
- **Structured logging** — `log.info("order", kv("id", orderId), kv("amount", amt))` and a `JsonFormatter`.
- **Correlation** — MDC integration with tracing (see [observability](/citadel/interview/observability)).

## The one idea to keep

A logging framework is a level check (cheap enum comparison, first line, so disabled logs are nearly free), a hierarchy of loggers where a child inherits its parent's threshold, and a list of appenders that each format via a `Formatter` strategy and write to one destination. Wrap the slow appenders (file, socket) in an `AsyncAppender` — a bounded queue plus a worker thread — and decide its overflow policy deliberately: block for audit logs, drop-oldest for the rest.
