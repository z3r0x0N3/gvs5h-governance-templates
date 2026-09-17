# C++20 Governance Templates

**Rule enforcement as a library.** A drop-in C++20 pattern for gating tool calls, API routes, and agent actions through a pre-check rule engine — with strike counting, containment, and *steer-don't-stop* redirects instead of hard failures.

Part of the [**C++20 Governance Templates** pack](https://ipoole.gumroad.com/l/tafts?utm_source=github&utm_medium=repo&utm_campaign=launch) ($39, lifetime updates).

## The problem

Every agent stack eventually needs the same thing: intercept every action before it executes, match it against rules, block or allow with evidence. Ad-hoc implementations rot. This pack gives you the load-bearing pieces:

```cpp
gov::RuleEngine engine("demo");
engine.add_rule({
    .id = "R1",
    .description = "no raw secrets in logs",
    .severity = gov::Severity::Medium,
    .strike_threshold = 3,
    .matcher = [](const std::string& tool, const std::string& payload) {
        return tool == "log_write" && payload.find("token") != std::string::npos;
    },
    .mode = gov::Enforcement::Redirect,   // steer, don't stop
});

gov::Proxy proxy(engine);
auto result = proxy.dispatch("log_write", "api_token=xyz", /*call_through=*/[](auto t, auto p) {
    return "ok";
});
// result.outcome == Advisory — call ran, reminder appended to output
```

## What's in the pack

| Component | Role |
|---|---|
| `RuleEngine` | Rule registry, per-rule strike counters, match/redirect/block verdicts |
| `Proxy` | Pre-check interceptor wrapping any dispatch function |
| Three enforcement modes | `Allow`, `Redirect` (advisory appended), `Block` (refused with reason) |
| Working CMake project | Static lib + example, builds clean with `-Wall -Wextra` |

## Build & run

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release && cmake --build build
./build/govtpl_example
```

## Who this is for

- Agent runtime authors who need tool-call gating
- Platform teams enforcing per-endpoint policy in C++ services
- Anyone who wants the *steer-don't-stop* pattern (advisory injection instead of hard 4xx walls) without designing it from scratch

## License

MIT. [Full pack on Gumroad](https://ipoole.gumroad.com/l/tafts?utm_source=github&utm_medium=repo&utm_campaign=launch) — this repo shows the core; the pack includes the extended rule catalog, proxy chain patterns, and integration recipes.
