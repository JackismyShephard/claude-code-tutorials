# Python Coding Guidelines for Claude

## Technical Stack

- **Project config**: `pyproject.toml` for configuration and dependency management
- **Environment**: Use virtual environment in `.venv` for dependency isolation
- **Package Management**: Always use uv with pyproject.toml, never pip
  - Run scripts with: `uv run script.py`
  - Use environment variables: `env VAR_NAME="value" uv run command`
  - When adding/updating dependencies, update `pyproject.toml` first
- **Dependencies**: Separate production and dev dependencies in `pyproject.toml`
- **Type checking**: Use VS Code with Pylance (pyright) for static type checking
- **Project layout**: Organize code with `src/` layout

## Code Style Guidelines

- **Formatting**: Black-compatible formatting via `ruff format` as defined in `pyproject.toml`
- **Imports**: Sort imports with `ruff` (stdlib, third-party, local) as defined in `pyproject.toml`
- **Linting**: Use `ruff` for style and error checking as defined in `pyproject.toml`.
- **Type Hints**: Use the typing library for clear type annotations to improve code understanding and tooling
  - Type hints should be used for all function parameters and return values
  - ideally, no type errors should be reported when using pyright in strict mode
  - However, do not write code that is overly verbose or complex just to appease pyright.
    - The goal of type hints is to improve code understanding and tooling, not to make the code harder to read or maintain.
  - If type hints make the code significantly harder to read, they can be omitted.
- **Documentation**: numpy-style docstrings for all public/exported modules, classes, functions
- **Naming**: snake_case for variables/functions, PascalCase for classes, private items prefixed with an underscore, constants in UPPER_SNAKE_CASE

## Python Best Practices

- **Debugging**: Use `logging` module instead of `print`
  - Configure logging with appropriate levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  - Use structured logging where possible to include context in log messages
- **Error handling**: Use specific exceptions with context messages and proper logging
- **Data structures**: Use list/dict comprehensions for concise, readable code
- **NO Conditional Imports**
  - Never use try/except blocks for imports of required packages
  - Handle specific errors during usage, not during import
- **Configuration**: Use environment variables (via `python-dotenv`) for configuration
- **MagicMock Ban**: MagicMock is strictly forbidden for testing core functionality
- **Test coverage**: Write comprehensive unit and integration tests with `pytest`; include fixtures
- **File size**: Keep files under 500 lines; refactor when exceeding this limit

## Build & Test Commands

### Testing and linting

- When running tests, always use the venv: e.g. `./venv/bin/pytest'
- Run tests: `pytest`
- Run single test: `pytest tests/path/to/test_file.py::test_function_name -v`
- Run tests with coverage: `python -m pytest --cov=src/aws_mcp_server tests/`
- Run linter: `ruff check src/ tests/`
- Format code: `ruff format src/ tests/`
