---
name: forms-and-validation
description: >
  Guide Claude on building forms with Binder and robust validation in Vaadin 25 Flow.
  This skill should be used when the user asks to "create a form", "bind fields",
  "validate input", "use Binder", "use BeanValidationBinder", "add validation",
  "convert field values", "handle form submission", "cross-field validation",
  or needs help with field binding, converters, required fields, custom validators,
  or form error handling in Vaadin Flow.
version: 0.1.0
---

# Forms with Binder and Validation in Vaadin 25

## Workflow for Creating Forms

When building or reviewing forms, follow this workflow to ensure accuracy. **Prioritize correctness over speed.**

1. **Review Vaadin Documentation First**
   - Use `search_vaadin_docs` to find relevant form, binder, and validation documentation
   - Use `get_full_document` to retrieve complete guides on Binder and validation
   - Use `get_component_java_api` for specific component API details
   - Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`

2. **Review Reference Materials**
   - Check `references/form-patterns.md` for established patterns and examples
   - Review best practices for the specific form type being created

3. **Verify Form Creation Approach**
   - Confirm Binder mode (buffered vs write-through) matches the use case
   - Review field binding patterns and validation strategies
   - Check that converters are properly ordered with validators

4. **Validate Component Selection**
   - Ensure each field uses the most appropriate Vaadin input component
   - Review the "Data Entry components" section below for component guidance
   - Check component API documentation for specific features

5. **Review Validation Strategy**
   - Verify required field markers are properly set
   - Check that validation runs at the correct level (field, binding, binder)
   - Ensure cross-field validation uses binder-level validators

6. **Verify Layout Structure**
   - Confirm FormLayout responsive steps match requirements
   - Check field spans and grouping
   - Ensure error display components are properly configured

**Remember:** Take time to consult documentation thoroughly. Accurate implementation is more important than rapid completion.

## Review Vaadin documentation

ALWAYS use the Vaadin MCP tools (`search_vaadin_docs`, `get_full_document`) to look up the latest documentation. Use (`get_component_java_api`) to look up the documentation for specific API details. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

## Binder Fundamentals

`Binder` connects UI fields to a Form Data Object (FDO) — a Java bean, record, or DTO. It handles reading values from the FDO into fields, writing field values back, converting between field types and model types, and validating at every level.

Binder can only bind components that implement `HasValue` (TextField, ComboBox, DatePicker, Checkbox, etc.).

### Two Modes of Operation

**Buffered mode** — changes are held in the Binder until explicitly written:

```java
Binder<Person> binder = new Binder<>(Person.class);
binder.readBean(person);           // populate fields from bean
// ... user edits fields ...
if (binder.writeBeanIfValid(person)) {
    service.save(person);          // only writes if all validation passes
}
```

**Write-through mode** — changes are written to the bean immediately on field change:

```java
binder.setBean(person);            // binds directly; changes write through
```

Use buffered mode for forms with Save/Cancel buttons. Use write-through mode for settings panels or simple filters where every change should apply immediately.

## Binding Fields

### Explicit binding (recommended)

```java
binder.forField(nameField)
    .asRequired("Name is required")
    .withValidator(new StringLengthValidator("1-100 characters", 1, 100))
    .bind(Person::getName, Person::setName);

binder.forField(emailField)
    .asRequired()
    .withValidator(new EmailValidator("Invalid email"))
    .bind(Person::getEmail, Person::setEmail);
```

The chain order matters: `forField()` → `asRequired()` → `withValidator()` → `withConverter()` → `withValidator()` → `bind()`. Validators and converters execute in the order they appear.

### Shorthand binding

```java
binder.bind(nameField, Person::getName, Person::setName);
```

No validation configuration — only useful for simple cases.

### BeanValidationBinder with Jakarta annotations

If your bean uses Jakarta Bean Validation annotations (`@NotEmpty`, `@Max`, `@Email`, etc.), use `BeanValidationBinder` to pick them up automatically:

```java
BeanValidationBinder<Person> binder = new BeanValidationBinder<>(Person.class);
binder.bindInstanceFields(this);  // binds fields matching property names
```

`bindInstanceFields()` scans the view for fields whose names match bean properties. This is convenient but less explicit — prefer `forField().bind()` for complex forms.

## Converters

When the field's value type doesn't match the bean property type, add a converter:

```java
binder.forField(yearOfBirthField)
    .withConverter(new StringToIntegerConverter("Enter a number"))
    .bind(Person::getYearOfBirth, Person::setYearOfBirth);
```

Converters implicitly validate — if conversion fails, the error message is shown as a validation error.

### Domain Primitives pattern

Create type-safe value objects with converters:

```java
binder.forField(emailField)
    .withConverter(new EmailAddressConverter())  // String → EmailAddress
    .withValidator(emailService::notAlreadyInUse, "Email already in use")
    .bind(Person::getEmail, Person::setEmail);
```

Validators can be added after converters — they then validate the converted type.

## Validation Layers

### 1. Required fields

```java
binder.forField(titleField)
    .asRequired()                              // visual indicator, empty check
    .bind(Proposal::getTitle, Proposal::setTitle);

binder.forField(typeComboBox)
    .asRequired("Please select a type")        // custom error message
    .bind(Proposal::getType, Proposal::setType);
```

### 2. Binding-level validators (per-field)

Run whenever the field value changes. Use built-in validators when possible:

- `StringLengthValidator`, `EmailValidator`, `RegexpValidator`
- `IntegerRangeValidator`, `DoubleRangeValidator`, `LongRangeValidator`
- `DateRangeValidator`, `DateTimeRangeValidator`
- `RangeValidator` (generic, with Comparator)

Custom validator with lambda:

```java
binder.forField(ageField)
    .withValidator(age -> age >= 0, "Age must be positive")
    .bind(Person::getAge, Person::setAge);
```

Custom validator class:

```java
public class PositiveIntegerValidator implements Validator<Integer> {
    @Override
    public ValidationResult apply(Integer value, ValueContext context) {
        return value >= 0
            ? ValidationResult.ok()
            : ValidationResult.error("Must be positive");
    }
}
```

### 3. Default validators (component built-in)

Some components have built-in validation (e.g., DatePicker min/max). These work alongside Binder validators. Disable them if needed:

```java
binder.forField(datePicker)
    .withDefaultValidator(false)
    .bind(Bean::getDate, Bean::setDate);
```

### 4. Binder-level validators (cross-field)

Validate the entire FDO after all fields are processed. Essential for rules that span multiple fields:

```java
binder.withValidator((bean, context) -> {
    if (bean.getStartDate() != null && bean.getEndDate() != null
            && bean.getStartDate().isAfter(bean.getEndDate())) {
        return ValidationResult.error("Start date must be before end date");
    }
    return ValidationResult.ok();
});
```

In buffered mode, binder-level validators only run when `writeBean()` or `writeBeanIfValid()` is called. In write-through mode, they run on every field change.

## Triggering Validation

- **Automatic:** binding-level validators run on every field value change
- **Programmatic:** `binder.validate()` — runs all validators and updates UI
- **Check only:** `binder.isValid()` — checks without updating UI
- **Write with validation:** `binder.writeBeanIfValid(bean)` — returns false if invalid

## Handling Validation Errors

Binding-level errors display next to the field automatically.

Binder-level errors need a status label:

```java
Div errorDisplay = new Div();
errorDisplay.addClassName(LumoUtility.TextColor.ERROR); // Lumo theme only; for Aura, use a custom CSS class
binder.setStatusLabel(errorDisplay);
```

## Form Layout

Use `FormLayout` for automatic responsive column adjustment:

```java
FormLayout form = new FormLayout();
form.add(nameField, emailField, phoneField);
form.setColspan(descriptionField, 2);  // span multiple columns
form.setResponsiveSteps(
    new ResponsiveStep("0", 1),        // 1 column on mobile
    new ResponsiveStep("500px", 2)     // 2 columns at 500px+
);
```

## Data Entry components in Vaadin

Use the appropriate input component for the type of data being entered.

- **Checkbox** is an input field representing a binary choice.
- **Combo Box** allows the user to choose a value from a filterable list of options presented in an overlay.
- **Custom Field** is a component for wrapping multiple components as a single field.
- **Date Picker** is an input field that allows the user to enter a date by typing or by selecting from a calendar overlay.
- **Date Time Picker** is an input field for selecting both a date and a time.
- **Email Field** is an extension of Text Field that accepts only email addresses as input.
- **List Box** allows the user to select one or more values from a scrollable list of items.
- **Number Field**, **IntegerField** and **BigDecimalField** are input fields that accept only numeric input.
- **Message Input** allows users to author and send messages.
- **Multi-Select Combo Box** allows the user to choose one or more values from a filterable list of options presented in an overlay.
- **Password Field** is an input field for entering passwords.
- **Radio Button Group** allows users to select one value among multiple choices.
- **Rich Text Editor** allows the user to author text that has rich formatting.
- **Select** allows users to choose a single value from a list of options presented in an overlay.
- **Text Area** is an input field component that allows entry of multiple lines of text.
- **Text Field** allows users to enter text.
- **Time Picker** is an input field for used entering or selecting a specific time.
- **Upload** allows the user to upload files, giving feedback to the user during the upload process.


## Best Practices

1. **Use buffered mode for most forms** — it gives you control over when data is written and lets you implement Cancel without manual state tracking.
2. **Prefer explicit binding over `bindInstanceFields`** — it's more readable, easier to maintain, and doesn't rely on field naming conventions.
3. **Validate at the right level** — field format/range → binding-level. Cross-field rules → binder-level. Business rules → service layer.
4. **Use converters for type safety** — domain primitives with converters catch invalid data at the type system level.
5. **Set `asRequired()` on mandatory fields** — it provides both the visual indicator and the empty-value check in one call.
6. **Show binder-level errors prominently** — they don't attach to a specific field, so users need a clear error display area.
7. **Use components that are optimized for the type of input** — this reduces the need for complex validation and improves UX.

## Detailed Reference

For the complete list of built-in validators, converter patterns, and form templates, see `references/form-patterns.md`.
