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

Use the Vaadin MCP tools (`search_vaadin_docs`, `get_component_java_api`) to look up the latest documentation whenever uncertain about a specific API detail. Always set `vaadin_version` to `"25"` and `ui_language` to `"java"`.

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

## Separating the Form into Its Own Class

Encapsulate the form in a dedicated class that extends `Composite<FormLayout>`. This keeps the Binder, fields, and form data object together, and exposes a small `setFormDataObject` / `getFormDataObject` API to the surrounding view. The constructor builds the fields, adds them to `getContent()`, and wires up the Binder.

**Buffered mode** — the form owns the form data object and writes to it on demand:

```java
public class ProposalForm extends Composite<FormLayout> {
    private final Binder<Proposal> binder;
    private Proposal formDataObject;

    public void setFormDataObject(@Nullable Proposal formDataObject) {
        this.formDataObject = formDataObject;
        if (formDataObject != null) {
            binder.readBean(formDataObject);
        } else {
            binder.refreshFields();
        }
    }

    public Optional<Proposal> getFormDataObject() {
        if (formDataObject == null) {
            formDataObject = new Proposal();
        }
        return binder.writeBeanIfValid(formDataObject)
            ? Optional.of(formDataObject)
            : Optional.empty();
    }
}
```

**Write-through mode** — the bean is set on the Binder and edits flow through immediately; the getter only validates before handing it back:

```java
public class ProposalForm extends Composite<FormLayout> {
    private final Binder<Proposal> binder;

    public void setFormDataObject(@Nullable Proposal formDataObject) {
        binder.setBean(formDataObject);
    }

    public Optional<Proposal> getFormDataObject() {
        if (binder.getBean() == null) {
            throw new IllegalStateException("No form data object");
        }
        return binder.validate().isOk()
            ? Optional.of(binder.getBean())
            : Optional.empty();
    }
}
```

## Best Practices

1. **Use buffered mode for most forms** — it gives you control over when data is written and lets you implement Cancel without manual state tracking.
2. **Prefer explicit binding over `bindInstanceFields`** — it's more readable, easier to maintain, and doesn't rely on field naming conventions.
3. **Validate at the right level** — field format/range → binding-level. Cross-field rules → binder-level. Business rules → service layer.
4. **Use converters for type safety** — domain primitives with converters catch invalid data at the type system level.
5. **Set `asRequired()` on mandatory fields** — it provides both the visual indicator and the empty-value check in one call.
6. **Show binder-level errors prominently** — they don't attach to a specific field, so users need a clear error display area.
7. **Encapsulate the form in its own `Composite<FormLayout>` class** — keep the Binder, fields, and form data object together, and expose a small `setFormDataObject` / `getFormDataObject` API to the surrounding view.

## Detailed Reference

For the complete list of built-in validators, converter patterns, and form templates, see `references/form-patterns.md`.
