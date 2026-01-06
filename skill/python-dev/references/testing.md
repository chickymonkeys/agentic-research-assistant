# Testing in Python

This guide covers best practices for testing Python applications using `pytest`.

## Core Principles

1.  **Tests are Code**: Treat test code with the same care as production code.
2.  **Fast & Deterministic**: Tests should run quickly and reliably.
3.  **Isolation**: Tests should not depend on each other or shared global state.

## Configuration (`pyproject.toml`)

Standard configuration for `pytest`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
addopts = "-ra -q --cov=src"
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks tests as integration tests",
]
```

## Structure

```
myproject/
├── src/
│   └── mypackage/
│       └── core.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py       # Shared fixtures
│   ├── unit/
│   │   └── test_core.py
│   └── integration/
│       └── test_api.py
```

## Writing Tests

### Basic Test Function

```python
def test_addition():
    assert 1 + 1 == 2
```

### Using Fixtures

Use fixtures for setup/teardown logic.

```python
import pytest

@pytest.fixture
def user():
    return {"username": "testuser", "email": "test@example.com"}

def test_user_email(user):
    assert user["email"] == "test@example.com"
```

### Parameterization

Avoid duplicating test logic.

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("", ""),
])
def test_uppercase(input, expected):
    assert input.upper() == expected
```

### Exception Testing

Assert that errors are raised correctly.

```python
def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        1 / 0
```

## Mocking

Use `unittest.mock` or `pytest-mock` to isolate unit tests from external dependencies.

```python
def test_api_call(mocker):
    mock_get = mocker.patch("requests.get")
    mock_get.return_value.status_code = 200
    
    response = fetch_data()
    assert response.status_code == 200
```

## Best Practices

1.  **Arrange-Act-Assert**: Structure tests clearly.
    *   **Arrange**: Set up inputs and mocks.
    *   **Act**: Call the function under test.
    *   **Assert**: Verify the results.
2.  **One Concept per Test**: Don't test everything in one function.
3.  **Descriptive Names**: `test_calculate_total_with_discount` vs `test_func`.
4.  **Use `conftest.py`**: Share fixtures across multiple test files.
