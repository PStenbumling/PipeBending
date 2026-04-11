# Pipe Bending App — AI-Assisted Development

## Purpose

Deterministic pipe calculation tool for field use.
Geometry engine calculates marks and cut length consistently.
Safety layer prevents common human measurement and tool errors.
System is execution-driven and must preserve physical correctness.
Only `index.html` and `README.md` are public deployment files.

## Files

- `todo.md`: implementation source of truth; do not modify via AI.
- `v0.5.5.rules.md`: compressed hard behavior rules.
- `v0.5.5.md`: full design document for reference.
- `CLAUDE.md`: strict AI execution constraints.
- `index.html`: single-file app implementation.

## How to prompt Claude

- `Implement ONLY S-böj from todo.md. Output only code.`
- `Implement ONLY Takoffset warning persistence from todo.md. No redesign.`
- `Implement ONLY requested section from todo.md. If unclear, ask first.`