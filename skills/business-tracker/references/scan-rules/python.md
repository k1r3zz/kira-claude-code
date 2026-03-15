# Scan Rules — Python

File priority rules for Python projects (tag: `python`).

| Priority | File patterns |
|----------|--------------|
| P0 | `services/**`, `domain/**`, `usecases/**`, `handlers/**`, `views.py` (with business logic), `tasks.py`, `signals.py` |
| P1 | `models.py`, `models/**`, `serializers.py`, `schemas/**`, `routes/**`, `urls.py`, `middleware/**` |
| P2 | `utils/**` / `helpers/**` / `config/**` / `constants.py` with business vocabulary |
| Skip | `__pycache__/`, `*.pyc`, `venv/`, `.venv/`, `env/`, `migrations/` (auto-generated), `tests/`, `test_*.py`, `*_test.py`, `conftest.py`, `setup.py`, `manage.py` |
