# Data Provider Patterns Reference

## Decision Matrix: Which Binding Approach?

| Scenario | Approach | Method |
|----------|----------|--------|
| Small dataset (<1000 items) | In-memory | `grid.setItems(list)` |
| Large dataset, Spring backend | Lazy with Spring | `grid.setItemsPageable(service::list)` |
| Large dataset, non-Spring | Lazy with callbacks | `grid.setItems(query -> ...)` |
| ComboBox with many options | Lazy ComboBox | `comboBox.setItemsPageable(service::list)` |
| Reusable across views | Custom DataProvider | Extend `AbstractBackEndDataProvider` |

## Spring Integration Pattern

### Service layer

```java
@Service
public class PersonService {

    private final PersonRepository repository;

    public PersonService(PersonRepository repository) {
        this.repository = repository;
    }

    public List<Person> list(Pageable pageable) {
        return repository.findAllBy(pageable).getContent();
    }

    public List<Person> list(Pageable pageable, String nameFilter) {
        if (nameFilter == null || nameFilter.isBlank()) {
            return list(pageable);
        }
        return repository.findByNameContainingIgnoreCase(nameFilter, pageable);
    }

    public long count() {
        return repository.count();
    }

    public long count(String nameFilter) {
        if (nameFilter == null || nameFilter.isBlank()) {
            return count();
        }
        return repository.countByNameContainingIgnoreCase(nameFilter);
    }
}
```

### View binding

```java
TextField filter = new TextField("Search");
filter.setValueChangeMode(ValueChangeMode.LAZY);

Grid<Person> grid = new Grid<>(Person.class);
grid.setColumns("name", "email", "department");

GridLazyDataView<Person> dataView = grid.setItemsPageable(
    pageable -> personService.list(pageable, filter.getValue()),
    pageable -> personService.count(filter.getValue())
);

filter.addValueChangeListener(e -> dataView.refreshAll());
```

## ComboBox Lazy Loading Pattern

```java
ComboBox<Product> productCombo = new ComboBox<>("Product");
productCombo.setItemLabelGenerator(Product::getName);

// Spring: service automatically receives filter string
productCombo.setItemsPageable(productService::search);

// Service signature:
// List<Product> search(Pageable pageable, String filterText)
```

## ConfigurableFilterDataProvider Pattern

For reusable, filterable data providers:

```java
// Create the provider
PersonDataProvider provider = new PersonDataProvider();

// Wrap with configurable filter
ConfigurableFilterDataProvider<Person, Void, String> filtered =
    provider.withConfigurableFilter();

// Bind to grid
grid.setDataProvider(filtered);

// Update filter from UI
filterField.addValueChangeListener(e ->
    filtered.setFilter(e.getValue()));
```

## Grid Sorting with Lazy Data

```java
// Declare sortable columns
grid.addColumn(Person::getName).setHeader("Name").setKey("name").setSortable(true);
grid.addColumn(Person::getEmail).setHeader("Email").setKey("email").setSortable(true);
grid.addColumn(Person::getDepartment).setHeader("Dept").setKey("department"); // not sortable

// Handle in callback
grid.setItems(query -> {
    String sortProperty = null;
    boolean ascending = true;
    if (!query.getSortOrders().isEmpty()) {
        QuerySortOrder sort = query.getSortOrders().get(0);
        sortProperty = sort.getSorted();
        ascending = sort.getDirection() == SortDirection.ASCENDING;
    }
    return service.list(query.getOffset(), query.getLimit(),
                        sortProperty, ascending).stream();
});
```

With Spring's `setItemsPageable`, sorting is handled automatically — `Pageable` includes sort info.

## Refreshing Data

| Need | Method | Effect |
|------|--------|--------|
| Single item changed | `dataView.refreshItem(item)` | Re-renders one row |
| Filter/sort changed | `dataView.refreshAll()` | Re-fetches all visible data |
| Complete data replacement | `grid.setItems(newList)` | Replaces the data source |
| Add item (in-memory) | `listDataView.addItem(item)` | Adds and refreshes |
| Remove item (in-memory) | `listDataView.removeItem(item)` | Removes and refreshes |

## hashCode/equals Template

```java
// With manual implementation
public class Person {
    private final long id;
    // ... other fields ...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        return Objects.equals(id, ((Person) o).id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}

// With Lombok
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Person {
    @EqualsAndHashCode.Include
    private final long id;
    // ... other fields ...
}
```

## Anti-patterns

**Loading everything for lazy data:**
```java
// BAD: defeats the purpose of lazy loading
grid.getGenericDataView().getItems().forEach(...);
```

**Missing refreshAll after filter change:**
```java
// BAD: grid won't update
filterField.addValueChangeListener(e -> {
    // forgot dataView.refreshAll()
});
```

**Mutable hashCode/equals:**
```java
// BAD: changing email changes identity, breaking Grid
@Override
public boolean equals(Object o) {
    return Objects.equals(email, ((Person) o).email); // email can change!
}
```

**Spring-managing Grids or DataProviders:**
```java
// BAD: Grid holds UI state — not a Spring bean
@SpringComponent
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class PersonGrid extends Grid<Person> { ... }

// BAD: DataProvider holds query state — not a singleton
@SpringComponent
public class PersonDataProvider extends AbstractBackEndDataProvider<...> {
    @Autowired private PersonRepository repo; // field injection too
}

// GOOD: plain classes with constructor-injected dependencies
public class PersonGrid extends Grid<Person> {
    public PersonGrid(PersonRepository repo) { ... }
}
```
