# Agentic Code Generation — Complete Guide
> Two paths to generate production-ready code from a requirement spec with minimal human effort.
> Both paths use the same two input files: `constitution.md` + `requirement-spec.md`

---

## FILE STRUCTURE IN YOUR REPO

```
your-project/
  devin/
    constitution.md          ← Global standards (never changes)
    requirement-spec.md      ← Per-task spec (you fill this per component)
  scripts/
    generate_code.py         ← SpecKit orchestration script (Copilot/API path)
  src/                       ← Generated code lands here
  tests/                     ← Generated tests land here
```

---

## THE TWO INPUT FILES

### File 1: constitution.md (permanent, never changes)
Stored in your repo. Contains:
- Hexagonal Architecture rules
- Code quality standards (SOLID, clean code, naming)
- Error handling patterns
- Logging & observability requirements
- Security rules (no hardcoded secrets, input validation)
- Google Cloud ADK best practices
- Testing requirements (unit + integration)
- Project structure conventions
- Environment variable conventions

### File 2: requirement-spec.md (per task, you fill this)
A new file per component. Contains:
- Component name
- What to build (functional requirements)
- Input schema
- Output / side effects
- Environment variables specific to this component
- Non-functional requirements (throughput, latency, availability)
- Branch name
- Explicitly out of scope

---

## PATH 1 — DEVIN (FULLY AUTONOMOUS)

### How it works
Devin reads both files, writes tests first, generates code, runs tests, fixes failures,
and loops until 80%+ tests pass — then opens a PR. Zero human intervention.

### Steps
1. Commit both `constitution.md` and your filled `requirement-spec.md` to the repo
2. Open app.devin.ai
3. Give Devin this instruction:

```
Read devin/constitution.md for all project standards and coding rules.
Read devin/requirement-spec.md for the component requirement.

Then do the following autonomously:
1. Ask any clarifying questions BEFORE starting to code (this is the only time)
2. Write test cases first (TDD approach)
3. Generate implementation code
4. Run the tests
5. If tests fail — analyse failures, fix the code, re-run tests
6. Keep iterating until at least 80% of tests pass
7. Open a PR against the base branch specified in the spec

Do not stop until step 7 is complete.
```

4. Answer any clarifying questions Devin asks upfront
5. Wait for the PR — review and merge

### What Devin does autonomously
- Reads your repo and understands existing code patterns
- Plans implementation steps
- Writes tests first (TDD)
- Generates production code
- Runs tests in its sandbox
- Self-heals: reads error logs, fixes code, re-runs — loops until 80%+ pass
- Opens a PR with all changes

### Human touchpoints
- Answer clarifying questions at the start (one time only)
- Review and approve the final PR

---

## PATH 2 — COPILOT + SPECKIT PYTHON SCRIPT (HUMAN-IN-THE-LOOP)

### How it works
A Python script (`generate_code.py`) reads both files, calls the Sonnet API to
generate code and tests, runs the tests, feeds failures back to the API,
and iterates automatically. You run one command and review the output.

### Steps
1. Install dependencies:
```bash
pip install anthropic pytest
```

2. Set your API key:
```bash
export ANTHROPIC_API_KEY=your_key_here
```

3. Fill in your `requirement-spec.md`

4. Run the script:
```bash
python scripts/generate_code.py \
  --constitution devin/constitution.md \
  --spec devin/requirement-spec.md \
  --output src/ \
  --tests tests/
```

5. Script runs automatically — watch the terminal for progress

6. **Human checkpoint**: When the script stops (80%+ tests pass or max iterations reached),
   review the generated code and tests in `src/` and `tests/`

7. Fix any remaining failures manually if needed, then open a PR

### Human touchpoints
- Run the script (one command)
- Review generated code after the loop completes
- Fix any remaining edge cases manually
- Open and merge the PR

---

## SPECKIT PYTHON SCRIPT

Save this as `scripts/generate_code.py` in your repo:

```python
#!/usr/bin/env python3
"""
SpecKit Code Generator
Reads constitution.md + requirement-spec.md, calls Claude Sonnet API,
generates code and tests, runs tests, fixes failures — loops until 80%+ pass.

Usage:
  python generate_code.py \
    --constitution devin/constitution.md \
    --spec devin/requirement-spec.md \
    --output src/ \
    --tests tests/
"""

import argparse
import os
import subprocess
import sys
import json
import re
from pathlib import Path
import anthropic

# ── Config ────────────────────────────────────────────────────────────────────
MODEL = "claude-sonnet-4-20250514"
MAX_TOKENS = 8000
MAX_ITERATIONS = 5
PASS_THRESHOLD = 0.80   # 80% tests must pass to stop

client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))


# ── File helpers ──────────────────────────────────────────────────────────────
def read_file(path: str) -> str:
    with open(path, "r") as f:
        return f.read()


def write_files(file_map: dict):
    """Write generated files to disk. file_map = {relative_path: content}"""
    for rel_path, content in file_map.items():
        full_path = Path(rel_path)
        full_path.parent.mkdir(parents=True, exist_ok=True)
        with open(full_path, "w") as f:
            f.write(content)
        print(f"  ✓ Written: {rel_path}")


def extract_files_from_response(response_text: str) -> dict:
    """
    Parse Claude's response and extract files.
    Expects blocks like:
      ### FILE: src/adapters/kafka_consumer.py
      ```python
      ... code ...
      ```
    """
    file_map = {}
    pattern = r"### FILE:\s*(\S+)\s*\n```[a-z]*\n(.*?)```"
    matches = re.findall(pattern, response_text, re.DOTALL)
    for filepath, content in matches:
        file_map[filepath.strip()] = content.strip()
    return file_map


# ── Test runner ───────────────────────────────────────────────────────────────
def run_tests(test_dir: str) -> tuple[bool, str, float]:
    """
    Run pytest. Returns (all_passed, output, pass_rate).
    """
    result = subprocess.run(
        ["python", "-m", "pytest", test_dir, "-v", "--tb=short", "--json-report",
         "--json-report-file=test_report.json"],
        capture_output=True, text=True
    )
    output = result.stdout + result.stderr

    # Parse pass rate from JSON report
    pass_rate = 0.0
    try:
        with open("test_report.json") as f:
            report = json.load(f)
        total = report["summary"].get("total", 0)
        passed = report["summary"].get("passed", 0)
        pass_rate = passed / total if total > 0 else 0.0
        print(f"\n  Test results: {passed}/{total} passed ({pass_rate*100:.0f}%)")
    except Exception:
        # Fallback: parse from stdout
        match = re.search(r"(\d+) passed", output)
        total_match = re.search(r"(\d+) (passed|failed|error)", output)
        if match:
            passed = int(match.group(1))
            total = passed + sum(
                int(m.group(1)) for m in re.finditer(r"(\d+) failed", output)
            )
            pass_rate = passed / total if total > 0 else 0.0

    all_passed = result.returncode == 0
    return all_passed, output, pass_rate


# ── Claude API calls ──────────────────────────────────────────────────────────
def generate_code(constitution: str, spec: str, output_dir: str, test_dir: str) -> str:
    """First call: generate implementation + tests from scratch."""
    prompt = f"""
You are an expert software engineer. Use the constitution and requirement spec below
to generate production-ready code and tests.

IMPORTANT — Response format:
- Output each file using this exact format:
  ### FILE: <relative/path/to/file.py>
  ```python
  <file content>
  ```
- Generate ALL implementation files AND test files in one response
- Write tests first (TDD), then implementation
- Follow all rules in the constitution exactly

---
# CONSTITUTION (project-wide standards)
{constitution}

---
# REQUIREMENT SPEC (what to build)
{spec}

---
Generate all files now. Place implementation files under `{output_dir}` and
test files under `{test_dir}`. Include __init__.py files where needed.
"""
    print("\n⏳ Calling Claude API — generating code and tests...")
    response = client.messages.create(
        model=MODEL,
        max_tokens=MAX_TOKENS,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.content[0].text


def fix_code(constitution: str, spec: str, previous_code: str,
             test_output: str, iteration: int) -> str:
    """Subsequent calls: fix failures from test run."""
    prompt = f"""
You are an expert software engineer. Tests are failing. Analyse the failures and fix the code.

IMPORTANT — Response format:
- Output ONLY the files that need to change
- Use the same format: ### FILE: <path> then ```python ... ```
- Do not output files that are already correct

---
# CONSTITUTION (rules to follow)
{constitution}

---
# REQUIREMENT SPEC
{spec}

---
# CURRENT CODE (iteration {iteration})
{previous_code}

---
# TEST FAILURES
{test_output}

---
Fix the failures. Output only changed files.
"""
    print(f"\n⏳ Iteration {iteration} — fixing failures...")
    response = client.messages.create(
        model=MODEL,
        max_tokens=MAX_TOKENS,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.content[0].text


# ── Main loop ─────────────────────────────────────────────────────────────────
def main():
    parser = argparse.ArgumentParser(description="SpecKit Code Generator")
    parser.add_argument("--constitution", required=True, help="Path to constitution.md")
    parser.add_argument("--spec",         required=True, help="Path to requirement-spec.md")
    parser.add_argument("--output",       default="src/",   help="Output dir for src files")
    parser.add_argument("--tests",        default="tests/", help="Output dir for test files")
    args = parser.parse_args()

    # Read inputs
    constitution = read_file(args.constitution)
    spec = read_file(args.spec)

    print("=" * 60)
    print("  SpecKit Code Generator")
    print(f"  Constitution : {args.constitution}")
    print(f"  Spec         : {args.spec}")
    print(f"  Output       : {args.output}")
    print(f"  Tests        : {args.tests}")
    print(f"  Max iters    : {MAX_ITERATIONS}")
    print(f"  Pass target  : {PASS_THRESHOLD*100:.0f}%")
    print("=" * 60)

    # Step 1: Generate initial code + tests
    response = generate_code(constitution, spec, args.output, args.tests)
    all_code = response

    file_map = extract_files_from_response(response)
    if not file_map:
        print("\n❌ No files extracted from API response. Check response format.")
        print(response[:500])
        sys.exit(1)

    print(f"\n📁 Writing {len(file_map)} files...")
    write_files(file_map)

    # Step 2: Test → fix loop
    for iteration in range(1, MAX_ITERATIONS + 1):
        print(f"\n🧪 Running tests (iteration {iteration}/{MAX_ITERATIONS})...")
        all_passed, test_output, pass_rate = run_tests(args.tests)

        if all_passed or pass_rate >= PASS_THRESHOLD:
            print(f"\n✅ Done! Pass rate: {pass_rate*100:.0f}% — target met.")
            print("   Review generated files, then open a PR.")
            break

        if iteration == MAX_ITERATIONS:
            print(f"\n⚠️  Max iterations reached. Pass rate: {pass_rate*100:.0f}%")
            print("   Manual review required for remaining failures.")
            break

        # Fix and re-write
        fix_response = fix_code(constitution, spec, all_code, test_output, iteration)
        all_code += "\n\n" + fix_response

        fixed_files = extract_files_from_response(fix_response)
        if fixed_files:
            print(f"  📁 Updating {len(fixed_files)} file(s)...")
            write_files(fixed_files)
        else:
            print("  ⚠️  No file changes extracted from fix response.")

    print("\n" + "=" * 60)
    print("  SpecKit complete. Review src/ and tests/ before committing.")
    print("=" * 60)


if __name__ == "__main__":
    main()
```

---

## COMPARISON: DEVIN vs COPILOT+SPECKIT

| Aspect                  | Devin (Path 1)              | Copilot + SpecKit (Path 2)       |
|-------------------------|-----------------------------|----------------------------------|
| Human interaction       | Clarifying questions only   | Run script + review output       |
| Test-fix loop           | Built-in, fully autonomous  | Python script handles it         |
| Where it runs           | Devin's cloud sandbox       | Your local machine or CI         |
| Cost                    | Per-task ACU credits        | Anthropic API tokens             |
| PR creation             | Automatic                   | Manual after review              |
| Best for                | Large, complex components   | Smaller components, CI pipelines |
| Internet access         | Yes (reads docs, SO)        | No (API only)                    |
| Pass threshold          | 80% configurable in prompt  | 80% in script (PASS_THRESHOLD)   |

---

## QUICK REFERENCE — WHICH PATH TO USE?

```
Is the task complex (multi-file, multi-service, needs research)?
  → YES → Use Devin (Path 1)
  → NO  → Use SpecKit script (Path 2)

Do you need fully zero human interaction?
  → YES → Use Devin (Path 1)
  → NO  → Either path works

Do you want to embed this in a CI/CD pipeline?
  → YES → Use SpecKit script (Path 2)
  → NO  → Either path works
```
