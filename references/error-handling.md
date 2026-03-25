# Error Handling Reference

Extended error handling examples and patterns for Go.

## Error Wrapping Decision Tree

```
Should the caller inspect this error?
├── No → Use %v (opaque wrapping)
│   └── At API boundaries, HTTP handlers, CLI output
└── Yes → Use %w (transparent wrapping)
    └── For errors.Is / errors.As inspection
```

## Complete Validation Pattern

```go
func validateOrder(o Order) error {
    var errs []error

    if o.CustomerID == "" {
        errs = append(errs, fmt.Errorf("customer ID: %w", ErrRequired))
    }
    if o.Total < 0 {
        errs = append(errs, fmt.Errorf("total: %w", ErrNegative))
    }
    if len(o.Items) == 0 {
        errs = append(errs, fmt.Errorf("items: %w", ErrEmpty))
    }

    for i, item := range o.Items {
        if err := validateItem(item); err != nil {
            errs = append(errs, fmt.Errorf("item[%d]: %w", i, err))
        }
    }

    return errors.Join(errs...)
}
```

## Custom Error Type with Context

```go
type ValidationError struct {
    Field   string
    Message string
    Value   any
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s (got %v)", e.Field, e.Message, e.Value)
}

// Use errors.As to extract the typed error from callers
var ve *ValidationError
if errors.As(err, &ve) {
    fmt.Printf("invalid field: %s\n", ve.Field)
}
```

## Error Handling at Layer Boundaries

```go
// Repository layer: wrap with %w for internal use
func (r *UserRepo) FindByID(ctx context.Context, id string) (*User, error) {
    user, err := r.db.QueryRow(ctx, query, id)
    if err != nil {
        return nil, fmt.Errorf("find user %s: %w", id, err)
    }
    return user, nil
}

// Service layer: wrap with %w for internal callers
func (s *UserService) GetUser(ctx context.Context, id string) (*User, error) {
    user, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("get user: %w", err)
    }
    return user, nil
}

// HTTP handler: log internal details, return generic messages to clients
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
    user, err := h.service.GetUser(r.Context(), chi.URLParam(r, "id"))
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            http.Error(w, "user not found", http.StatusNotFound)
            return
        }
        slog.Error("get user", "err", err)
        http.Error(w, "internal error", http.StatusInternalServerError)
        return
    }
    json.NewEncoder(w).Encode(user)
}
```

## Sources

- [Google Go Style Guide - Error Handling](https://google.github.io/styleguide/go/decisions#errors)
- [Uber Go Style Guide - Errors](https://github.com/uber-go/guide/blob/master/style.md#errors)
- [Go Blog - Working with Errors in Go 1.13](https://go.dev/blog/go1.13-errors)
