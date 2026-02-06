# AGENTS.md

## Repository Overview

**nanobot** is an ultra-lightweight personal AI assistant framework (~4,000 lines of code) written in Python. The project provides a modular agent architecture with LLM integration, tool execution, and multi-channel support (Telegram, Discord, WhatsApp, Feishu).

## Build, Test, and Lint Commands

### Testing
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_tool_validation.py

# Run with verbose output
pytest -v

# Run with coverage (requires pytest-cov)
pytest --cov=nanobot
```

### Linting and Formatting
```bash
# Run linting with ruff
ruff check nanobot/

# Fix linting issues automatically
ruff check --fix nanobot/

# Format code (if ruff format is configured)
ruff format nanobot/
```

### Installation and Build
```bash
# Install in development mode
pip install -e ".[dev]"

# Install with vLLM provider support
pip install -e ".[vllm]"

# Build Docker image
docker build -t nanobot .

# Run full Docker test suite
./test_docker.sh
```

## Code Style Guidelines

### Python Version and Syntax
- **Minimum Python version**: 3.11 (uses modern syntax like `|` for Union types)
- **Target Python version**: 3.11+ (configured in pyproject.toml)
- **Line length**: 100 characters (configured in ruff)

### Typing and Type Hints
- **Use type hints for all functions and methods**: All public functions must have complete type annotations
- **Use modern Union syntax**: Prefer `str | None` over `Optional[str]`
- **Use `Any` sparingly**: Only for truly dynamic data structures
- **Generic types**: Use proper generic types from `typing` module (e.g., `dict[str, Any]`)

Example:
```python
def load_config(config_path: Path | None = None) -> Config:
    """Load configuration from file or create default."""
    path = config_path or get_config_path()
    return Config()
```

### Imports
1. **Order**: Standard library → Third-party → Local imports
2. **Style**: Use absolute imports, not relative imports
3. **Grouping**: Separate groups with a blank line

Example:
```python
import json
from pathlib import Path
from typing import Any

from pydantic import BaseModel

from nanobot.config.schema import Config
```

4. **Avoid star imports**: Explicitly import required names
5. **__all__ exports**: Define `__all__` in `__init__.py` files for public API

### Naming Conventions
- **Classes**: PascalCase (e.g., `AgentLoop`, `ToolRegistry`)
- **Functions and methods**: snake_case (e.g., `load_config`, `validate_params`)
- **Variables**: snake_case (e.g., `config_path`, `max_iterations`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DEFAULT_MODEL`, `API_BASE`)
- **Private members**: Prefixed with underscore (e.g., `_validate`, `_running`)

### Error Handling
- **Specific exceptions**: Catch specific exception types, not broad `Exception`
- **User-friendly messages**: Include helpful error messages for user-facing errors
- **Warning for recoverable errors**: Use warnings for non-critical issues

Example:
```python
try:
    with open(path) as f:
        data = json.load(f)
    return Config.model_validate(convert_keys(data))
except (json.JSONDecodeError, ValueError) as e:
    print(f"Warning: Failed to load config from {path}: {e}")
    print("Using default configuration.")
```

### Documentation
- **Docstrings**: All public classes and functions must have docstrings
- **Format**: Use triple double quotes with Google or reStructuredText style
- **Content**: Explain what the function does, parameters, return values, and raised exceptions

Example:
```python
def load_config(config_path: Path | None = None) -> Config:
    """
    Load configuration from file or create default.
    
    Args:
        config_path: Optional path to config file. Uses default if not provided.
    
    Returns:
        Loaded configuration object.
    """
```

### Code Structure
- **Small modules**: Keep modules focused and under ~200 lines where possible
- **Clear separation**: Separate concerns across modules (config, providers, agent, tools, etc.)
- **Abstract base classes**: Define interfaces using ABCs with abstract methods
- **Pydantic models**: Use Pydantic for all configuration and data models

### Async/Await Patterns
- **Async throughout**: Core functionality uses async/await (asyncio)
- **Tool execution**: All tool `execute()` methods must be async
- **LLM calls**: Use async LLM provider methods (e.g., `acompletion` from litellm)

Example:
```python
class Tool(ABC):
    @abstractmethod
    async def execute(self, **kwargs: Any) -> str:
        """Execute the tool with given parameters."""
        pass
```

### Logging
- **Use loguru**: Structured logging with `from loguru import logger`
- **Appropriate levels**: Use appropriate log levels (debug, info, warning, error)
- **Context**: Include relevant context in log messages

Example:
```python
from loguru import logger

logger.info(f"Loaded config from {path}")
logger.warning(f"Missing tool: {tool_name}")
```

### Testing
- **pytest style**: Write test functions with `def test_*():` naming
- **Descriptive names**: Test names should describe what is being tested
- **Assert messages**: Use clear assertion messages
- **Async tests**: Tests can be async (pytest-asyncio configured)

Example from test_tool_validation.py:
```python
def test_validate_params_missing_required() -> None:
    """Test validation catches missing required parameters."""
    tool = SampleTool()
    errors = tool.validate_params({"query": "hi"})
    assert "missing required count" in "; ".join(errors)
```

### Configuration and Data
- **Pydantic validation**: Use Pydantic models for all configuration
- **Schema definition**: Define JSON schemas for tool parameters
- **Environment variables**: Support env vars for sensitive data (API keys)

### Tool Development
- **Inherit from Tool**: All tools must inherit from `Tool` ABC
- **Define properties**: Implement `name`, `description`, and `parameters` properties
- **JSON Schema**: Use proper JSON Schema format for parameters
- **Validation**: Use built-in parameter validation in `Tool` base class

Example:
```python
class SampleTool(Tool):
    @property
    def name(self) -> str:
        return "sample"
    
    @property
    def description(self) -> str:
        return "Sample tool for demonstration"
    
    @property
    def parameters(self) -> dict[str, Any]:
        return {
            "type": "object",
            "properties": {
                "query": {"type": "string", "minLength": 2},
            },
            "required": ["query"],
        }
    
    async def execute(self, **kwargs: Any) -> str:
        return f"Result: {kwargs['query']}"
```

### CLI Development
- **Use typer**: All CLI commands use Typer framework
- **Clear help**: Provide clear help text and descriptions
- **Type annotations**: Use type annotations for automatic validation
- **Rich output**: Use rich library for formatted output

### Development Workflow
1. Run tests before committing: `pytest`
2. Run linter before committing: `ruff check nanobot/`
3. Fix any linting issues: `ruff check --fix nanobot/`
4. Test Docker build if making infrastructure changes: `./test_docker.sh`

### Ruff Configuration
- **Line length**: 100 characters
- **Selected rules**: E (pycodestyle errors), F (Pyflakes), I (import sorting), N (naming), W (pycodestyle warnings)
- **Ignored**: E501 (line length) - trust the formatter line length setting
- **Target Python**: 3.11+

### Key Dependencies
- **typer**: CLI framework
- **pydantic**: Data validation and settings management
- **litellm**: Unified LLM API
- **loguru**: Structured logging
- **pytest**: Testing framework
- **ruff**: Linting and formatting
- **websockets**: WebSocket support