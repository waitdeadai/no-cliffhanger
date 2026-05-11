# Receipts

Five reproducible local fixture tests. `bash` + `jq` + 30 seconds.

## Setup

```bash
git clone https://github.com/waitdeadai/no-cliffhanger
cd no-cliffhanger
mkdir -p /tmp/cliff-tests
```

## Test 1 — "let me know if you would like me to continue" → BLOCK

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Here is the function. Let me know if you would like me to continue with tests."}' \
  > /tmp/cliff-tests/t1.json
bash no-cliffhanger.sh < /tmp/cliff-tests/t1.json; echo "exit=$?"
```

Expected: `BLOCKED: dangling permission-loop ending`, `exit=2`.

## Test 2 — substantive next-action statement → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Here is the function. Tests are next; running pytest now."}' \
  > /tmp/cliff-tests/t2.json
bash no-cliffhanger.sh < /tmp/cliff-tests/t2.json; echo "exit=$?"
```

Expected: `exit=0`. The model is *doing* the next thing, not asking permission.

## Test 3 — verification-framework partial closeout (allow-clause) → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Status: partial. Verification: not run because deps missing. Next step: pip install -r requirements.txt."}' \
  > /tmp/cliff-tests/t3.json
bash no-cliffhanger.sh < /tmp/cliff-tests/t3.json; echo "exit=$?"
```

Expected: `exit=0`. The `Status:` / `Next step:` allow-clause matches.

## Test 4 — "Want me to continue?" → BLOCK

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Want me to continue with the second migration?"}' \
  > /tmp/cliff-tests/t4.json
bash no-cliffhanger.sh < /tmp/cliff-tests/t4.json; echo "exit=$?"
```

Expected: `BLOCKED`, `exit=2`. The work was already authorized — keep going.

## Test 5 — explicit multiple-choice question (allow-clause) → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Here are three options. Pick one of: A) bash, B) python, C) go."}' \
  > /tmp/cliff-tests/t5.json
bash no-cliffhanger.sh < /tmp/cliff-tests/t5.json; echo "exit=$?"
```

Expected: `exit=0`. When a real decision is needed, ask explicitly with named options — the allow-clause matches.

## Summary

| # | Scenario | Expected | Exit |
|---|----------|----------|------|
| 1 | "let me know if you would like me to continue" | BLOCK | 2 |
| 2 | Substantive next-action statement | ALLOW | 0 |
| 3 | `Status: partial / Next step:` (allow-clause) | ALLOW | 0 |
| 4 | "Want me to continue?" | BLOCK | 2 |
| 5 | Explicit `pick one of: A) ... B) ...` (allow-clause) | ALLOW | 0 |
