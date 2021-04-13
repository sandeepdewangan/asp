# ASP.NET Core Code Snippets

## Convert any string to title case / pascal case
```csharp
Name = CultureInfo.CurrentCulture.TextInfo.ToTitleCase(Name.ToLower());
```

## Dropdown

1. Declare properties
```csharp
public IEnumerable<SelectListItem> Departments { get; set; }
```
2. In controller
```csharp
empDepartmentVM.Departments = _context.Departments.ToList().Select(
                        d =>
                            new SelectListItem
                            {
                                Text = d.Name,
                                Value = d.Id.ToString()
                            }
                    );
```
3. In Views
```html
 <select asp-for="Employee.DepartmentId" asp-items="Model.Departments" class="form-control"></select>
 ```
 
## Go back or cancel button implementation

```html
<a href="##" onClick="history.go(-1); return false;" class="btn btn-danger form-control">Cancel</a>
```

## Pass parameter to redirect to action
```csharp
return RedirectToAction(nameof(Index), new { id = 10 });
```
