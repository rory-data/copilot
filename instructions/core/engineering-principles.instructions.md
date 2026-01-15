---
description: "Shared engineering fundamentals, design patterns, and quality principles"
applyTo: "**"
---

# Core Engineering Principles

## Quick Reference

- **SOLID Principles**: Single responsibility, Open/closed, Liskov substitution, Interface segregation, Dependency inversion
- **Quality Mantras**: DRY, YAGNI, KISS - applied pragmatically based on context
- **Test Strategy**: Test pyramid (70% unit, 20% integration, 10% e2e), AAA pattern, Given/When/Then
- **Clean Code**: Readable, maintainable code that tells a story and minimises cognitive load

## Engineering Fundamentals

### Design Patterns

Apply pragmatically based on context:

- **Strategy Pattern**: Encapsulate interchangeable algorithms for different implementations
- **Dependency Injection**: Provide dependencies rather than instantiating them internally
- **Adapter Pattern**: Make incompatible interfaces work together

For comprehensive coverage, refer to established design pattern resources. Focus on solving the specific problem rather than applying patterns prematurely.

### SOLID Principles

- **Single Responsibility**: Each class/function has one reason to change

  ```python
  # Good: Single responsibility
  class UserValidator:
      def validate_email(self, email: str) -> bool:
          return "@" in email

  class UserRepository:
      def save(self, user: User) -> None:
          # Save to database
          pass
  ```

- **Open/Closed**: Open for extension, closed for modification

  ```python
  # Good: Use protocols for extension
  from typing import Protocol

  class PaymentProcessor(Protocol):
      def process(self, amount: float) -> bool: ...

  class StripeProcessor:
      def process(self, amount: float) -> bool:
          # Stripe implementation
          return True
  ```

- **Liskov Substitution**: Subtypes must be substitutable for their base types
- **Interface Segregation**: Many client-specific interfaces better than one general-purpose
- **Dependency Inversion**: Depend on abstractions, not concretions
  ```python
  # Good: Depend on abstraction
  def process_payment(processor: PaymentProcessor, amount: float):
      return processor.process(amount)
  ```

### Quality Attributes Balance

- **Testability**: Design for easy unit and integration testing
- **Maintainability**: Code should be easy to understand and modify
- **Scalability**: Consider future growth and load requirements
- **Performance**: Optimise where it matters, measure first
- **Security**: Build security in from the ground up
- **Understandability**: Code should tell a story

## Clean Code Practices

### Code Organisation

- **Separation of Concerns**: Each module has a single, well-defined responsibility
- **Minimal Coupling**: Reduce dependencies between modules
- **High Cohesion**: Elements within a module work together toward a common goal
- **Clear Module Boundaries**: Well-defined interfaces between components

### Naming and Structure

- **Descriptive Names**: Functions and variables should explain their purpose
- **Consistent Conventions**: Follow language-specific naming patterns
- **Meaningful Comments**: Explain WHY, not WHAT (see commenting-instructions.md)
- **Function Size**: Small, focused functions that do one thing well

## Test Automation Strategy

### Test Pyramid Implementation

- **Unit Tests (70%)**: Fast, isolated, test individual components
- **Integration Tests (20%)**: Test component interactions
- **End-to-End Tests (10%)**: Test complete user journeys

### Test Structure Standards

- **AAA Pattern**: Arrange, Act, Assert for clear test structure
- **Given/When/Then**: Structure test cases for clarity and consistency
- **Comprehensive Coverage**: Target >80% with meaningful tests, not just metrics
- **Edge Case Coverage**: Include boundary conditions, invalid inputs, error scenarios

### Quality Patterns

- **Error Handling**: Graceful degradation and meaningful error messages
- **Testing Strategies**: Unit, integration, property-based testing where appropriate
- **Refactoring Patterns**: Safe transformation of code structure
- **Architectural Best Practices**: Layered architecture, dependency injection

## Implementation Excellence

### Requirements Analysis

- **Clear Requirements**: Document assumptions explicitly
- **Edge Case Identification**: Consider boundary conditions and failure modes
- **Risk Assessment**: Identify and mitigate technical and business risks
- **Dependency Mapping**: Understand system interactions and constraints

### Pragmatic Craft

- **Good Over Perfect**: Balance engineering excellence with delivery needs
- **Never Compromise Fundamentals**: Maintain code quality and architectural integrity
- **Forward Thinking**: Anticipate future needs and technical evolution
- **Technical Debt Management**: Document debt, plan remediation, assess impact

## Technical Leadership Principles

### Code Review Excellence

- **Clear Feedback**: Specific, actionable improvement recommendations
- **Knowledge Sharing**: Explain the reasoning behind suggestions
- **Consistent Standards**: Apply principles fairly and consistently
- **Growth Mindset**: Focus on learning and improvement opportunities

### Quality Assurance

- **Continuous Improvement**: Regular retrospectives on processes and practices
- **Metric-Driven Decisions**: Use data to guide technical choices
- **Automation First**: Automate repetitive quality checks
- **Documentation Culture**: Keep documentation current and useful

## Language and Style

- Use New Zealand English spelling and grammar
- Follow language-specific best practices (see language/ directory)
- Prioritise readability and maintainability over cleverness
- Write code that tells a story and minimises cognitive load
