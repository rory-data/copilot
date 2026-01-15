---
description: "Optimised testing strategy leveraging core engineering principles"
---

# Testing Strategy Generator

## Goal

Generate comprehensive testing strategies following **Core Engineering Principles** (see `../instructions/core/engineering-principles.instructions.md`) and **Python best practices** (see `../instructions/language/python.instructions.md`).

## Quick Reference

- **Test Strategy**: Test pyramid (70% unit, 20% integration, 10% e2e)
- **Quality Standards**: SOLID principles applied to test design
- **Python Conventions**: Modern Python 3.11+ with type hints
- **Tool Patterns**: Efficient testing workflows

## Testing Framework

### Test Pyramid Implementation

```
E2E Tests (10%) - Full user journeys
Integration Tests (20%) - Component interactions
Unit Tests (70%) - Individual functions/classes
```

### Test Structure Standards

- **AAA Pattern**: Arrange, Act, Assert
- **Given/When/Then**: Clear scenario structure
- **SOLID Principles**: Applied to test design for maintainability

## Test Categories

### Unit Testing

- **Coverage Target**: >80% with meaningful tests
- **Focus Areas**: Business logic, algorithms, data transformations
- **Mocking Strategy**: External dependencies, I/O operations
- **Parametrization**: Multiple test scenarios efficiently

### Integration Testing

- **Database Integration**: Connection mocking and test data
- **API Integration**: Service interaction validation
- **File System**: Path operations and data persistence
- **Configuration**: Environment-specific testing

### End-to-End Testing

- **User Journeys**: Complete workflow validation
- **Error Scenarios**: Graceful failure handling
- **Performance**: Load and stress testing
- **Security**: Input validation and authorization

## Python-Specific Best Practices

### Test Organization

```
tests/
├── unit/           # Mirror source structure
├── integration/    # Service interaction tests
├── e2e/           # End-to-end scenarios
├── fixtures/      # Shared test data
└── conftest.py    # Pytest configuration
```

### Code Quality

- **Type Hints**: Full type annotation for test functions
- **Modern Python**: 3.11+ features and syntax
- **Pytest Features**: Fixtures, parametrize, markers
- **Property-Based Testing**: Hypothesis for edge case discovery

## Execution Strategy

### Phase 1: Analyse

1. **Code Analysis**: Use `codebase` and `usages` tools
2. **Existing Tests**: Analyse current test coverage with `findTestFiles`
3. **Gap Assessment**: Identify untested code paths

### Phase 2: Design

1. **Test Strategy**: Design comprehensive test approach
2. **Mock Strategy**: Plan external dependency mocking
3. **Data Strategy**: Test data generation and management

### Phase 3: Implement

1. **Test Creation**: Write tests following AAA pattern
2. **Fixture Development**: Create reusable test utilities
3. **Continuous Validation**: Run tests with `runTests` tool

## Quality Metrics

Track and optimise:

- **Coverage Percentage**: Maintain >80% meaningful coverage
- **Test Execution Time**: Keep unit tests under 1s each
- **Flaky Test Rate**: Target <1% flaky tests
- **Bug Escape Rate**: Measure production defects vs test coverage
