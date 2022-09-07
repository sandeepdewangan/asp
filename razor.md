# Asp.Net-6.0 Razor

[TOC]

## Notes

1. **Temp Data** - Stays for only one request.
	
	Usage: `TempData['success'] = "Successfully  inserted record!";`

2. To add a page without page model use `razor view`.

3. **Partial View** can be used to clean up the code and shift to another place.
	
	`<partial name="_PartialPageName"/>`


## Initial Setup
### Step 01: Install Packages
1. Microsoft.EntityFrameworkCore
2. Microsoft.EntityFrameworkCore.SqlServer
3. Microsoft.EntityFrameworkCore.Tools

### Step 02: Database Setup
`appsettings.json`
```json
"ConnectionStrings": {
    "DefaultConnection": "Server=DESKTOP-790MJGB\\SQLEXPRESS;Database=eShoppe;Trusted_Connection=True"
  }
```
`program.cs`
```csharp
// SQL Server
builder.Services.AddDbContext<ApplicationDbContext>(options => 
        options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```
`Data/ApplicationDbContext.cs`
```csharp
public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {
            
        }
    }
```

## Data Annotations
 - `[Key]` Primary Key
 - `[Required]` Required Field
 - `[Display(Name="Employee Name")]` For displaying the name in form as mentioned
 - `[Range(1,100,ErrorMessage ="The Display order must be between 1 to 100")]` Custom error message



## Creating Table
Create table category

`Model/Category.cs`
```csharp
public class Category
    {
        [Key]
        public int Id { get; set; }
        [Required]
        public string Name { get; set; }
        public int DisplayOrder { get; set; }
    }
```

`ApplicationDbContext.cs`
```csharp
public class ApplicationDbContext : DbContext
    {
    	.....

    	public DbSet<Category> Category { get; set; }
    }
```
Migrate the changes

`add-migration AddCategoryToDb`

Update the database - reflect the changes into DB

`update-database`

## Category - READ
Create razor pages under `Pages/Categories/Index.cshtml`

```csharp
public class IndexModel : PageModel
    {
        private readonly ApplicationDbContext _db;
        public IEnumerable<Category> Categories { get; set; }

        // Dependency injection 
        public IndexModel(ApplicationDbContext db)
        {
            _db = db;
        }

        public void OnGet()
        {
            // Get all the category from the database
            Categories = _db.Category;
        }
    }
```
```csharp
	// Usage in cshtml file
	<a asp-page="Create">Create</a>
	@foreach(var cat in Model.Categories){
        @cat.Name
    }
```

Navigating to the page
```csharp
<a asp-page="/Categories/Index">Category</a>
```

## Category - CREATE

```csharp
public class CreateModel : PageModel
    {
        private readonly ApplicationDbContext _db;
        // When we bind property the data given by the html pages are automatically binded.
        // if we have multiple property to bind use [BindProperties] at class level.
        [BindProperty]
        public Category Category { get; set; }

        public CreateModel(ApplicationDbContext db)
        {
            _db = db;
        }

        public void OnGet()
        {
        }

        // IActionResult - to redirect to some page or a view.
        public async Task<IActionResult> OnPost()
        {
        	// Validation
            if (ModelState.IsValid)
            {
                await _db.Category.AddAsync(Category);
                await _db.SaveChangesAsync();
                return RedirectToPage(nameof(Index));
            }
            return Page();
        }
    }
```
```html
<div>
    <form method="post">
        <label>Name</label>
        <input asp-for="Category.Name" />
        <span asp-validation-for="Category.Name"></span>

        <label>Display Order</label>
        <input asp-for="Category.DisplayOrder" />
        <span asp-validation-for="Category.DisplayOrder"></span>

        <button type="submit">Create</button>
    </form>
</div>
```

## Custom Validation
Custom validation if we want to check the Name and Display Order cannot be same. We need to throw the error.

```csharp

    if(Category.Name == Category.DisplayOrder.ToString())
    {
        ModelState.AddModelError("Category.Name", "Name and DisplayOrder cannot be same.");
    }

```
Displaying error
```
<div asp-validation-summary="All"></div>
```

## Client Side Validation
For handling the client side validation we have `_ValidationScriptsPartial.cshtml` file which is not included by default.
We can add Javascripts files in here.
```csharp
@section Scripts{
    @{
    <partial name="_ValidationScriptsPartial"/>
    }
}
```

## Category - Edit
Navigating to Edit Page

` <a asp-page="Edit" asp-route-id="@cat.Id">Edit</a>`

```csharp
public class EditModel : PageModel
    {
        private readonly ApplicationDbContext _db;
        [BindProperty]
        public Category Category { get; set; }

        public EditModel(ApplicationDbContext db)
        {
            _db = db;
        }

        public void OnGet(int id)
        {
            Category = _db.Category.Find(id);
            //Category = _db.Category.First(u=>u.Id == id); // throw exception if not found
            //Category = _db.Category.FirstOrDefault(u=>u.Id == id); // throw null if not found
            //Category = _db.Category.Where(u=>u.Id == id); // return multiple records
        }

        public async Task<IActionResult> OnPost()
        {
            if (ModelState.IsValid)
            {
                _db.Category.Update(Category);
                await _db.SaveChangesAsync();
                return RedirectToPage(nameof(Index));
            }
            return Page();
        }
    }
```
```csharp
<div>
    <form method="post">
        <input type="hidden" asp-for="Category.Id" />
        <label>Name</label>
        <input asp-for="Category.Name" />
        <span asp-validation-for="Category.Name"></span>

        <label>Display Order</label>
        <input asp-for="Category.DisplayOrder" />
        <span asp-validation-for="Category.DisplayOrder"></span>

        <div asp-validation-summary="All"></div>

        <button type="submit">Edit</button>
    </form>
</div>
```

## Repository

`Repos/ICategoryRepository.cs`
```csharp
	public interface ICategoryRepository
    {
        IEnumerable<Category> GetAll();
    }
```
`Repos/CategoryRepository.cs`
```csharp
public class CategoryRepository : ICategoryRepository
    {
        private readonly ApplicationDbContext _db;

        public CategoryRepository(ApplicationDbContext db)
        {
            _db = db;
        }

        public IEnumerable<Category> GetAll()
        {
            var categories = _db.Categories.ToList();
            return categories;
        }
    }
```
Register `program.cs`
```csharp
// Dependency Injection
//Transient objects are always different; a new instance is provided to every controller and every service.
//Scoped objects are the same within a request, but different across different requests.
//Singleton objects are the same for every object and every request.

builder.Services.AddScoped<ICategoryRepo, CategoryRepo>();
```
Usage
```csharp
	private readonly ICategoryRepository repoCategory
    public HomeController(ICategoryRepository repoCategory)
    {
        this.repoCategory = repoCategory;
    }
    public IActionResult Index()
    {
        var categories = repoCategory.GetAll();
    }
```

## Foreign Key
Here `CategoryId` is the foreign key.
```csharp
	  public int CategoryId { get; set; }	
      // navigation property and declaring the foreign key
      [ForeignKey("CategoryId")]
      public Category Category { get; set; }
```

## Dropdown
```csharp
public IEnumerable<SelectListItem> DepartmentList { get; set; }
...
...
	DepartmentList = _deptRepo.GetAll().Select(i => new SelectListItem
         {
             Text = i.DepartmentName,
             Value = i.Id.ToString()
         }),

```
Usage
```csharp
<select asp-for="@Model.DepartmentId" asp-items="@Model.DepartmentList">
     <option disabled selected>-Select Department-</option>
</select>
```

## Image Upload

```c#
	public class IndexModel : PageModel
    {
        public IEnumerable<Category> Categories { get; set; }

        // for image upload
        private readonly IWebHostEnvironment _hostingEnv;

        // Dependency injection 
        public IndexModel(IWebHostEnvironment hostingEnv)
        {
            _hostingEnv = hostingEnv;       
        }

        public async Task<IActionResult> OnPost()
        {
            string webRootPath = _hostingEnv.WebRootPath;
            var files = HttpContext.Request.Form.Files;
            string filename = Guid.NewGuid().ToString();
            var uploads = Path.Combine(webRootPath, @"images\products");
            var extension = Path.GetExtension(files[0].FileName);

            using (var fileStream = new FileStream(Path.Combine(uploads, filename + extension), FileMode.Create))
            {
                files[0].CopyTo(fileStream);
            }
            string publicImageUrl = @"\images\products\"+ filename + extension;

            return Page();
        }

    }
```

## Identity

**Step 01: Change the db context class.**

```c#
using Microsoft.AspNetCore.Identity.EntityFrameworkCore; 
public class ApplicationDbContext : IdentityDbContext {}
```

**Step 02: Add new scaffolded Identity to the project** 

The below code is automatically generated in `program.cs`

```c#
builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

We can remove the `RequireConfirmedAccount`.

**Step 03: Add and update migration**

`add-migration AddIdentityToDb`

`update-database`

**Step 04: We can use login partial view to display login or register links** 

`<partial name="_LoginPartial" />`



### Extending User Model

**Step 01:** Create model

```c#
  public class ApplicationUser : IdentityUser
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }
```

**Step 02:** Add to dB context class

`public DbSet<ApplicationUser>  ApplicationUser { get; set; }`

**Step 03:** Apply migration.

**Step 04:** Register page of identity pages

```c#
	public class InputModel
        {
            [Required]
            public string FirstName { get; set; }
            [Required]
            public string LastName { get; set; }
        }
```

```c#
<div class="form-floating">
   <input asp-for="Input.FirstName" class="form-control" aria-required="true" />
   <label asp-for="Input.FirstName"></label>
   <span asp-validation-for="Input.FirstName" class="text-danger"></span>
</div>
```

Change instance of Identity User to Application User

`private IdentityUser CreateUser()`to `private ApplicationUser.CreateUser()`

**Step 05:** 

```c#
user.FirstName = Input.FirstName; // <-- add this
user.LastName = Input.LastName;   // <-- add this
...
var result = await _userManager.CreateAsync(user, Input.Password);
```



### Role Manager

Make changes in `program.cs` to make role manager work.

```c#
builder.Services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

Create a static class

```c#
public static class SD
    {
        public const string Admin = "Admin";
        public const string Customer = "Customer";
    }
```

In Register

```c#
@using eShoppe.Utility
    
<div>
    <input type="radio" name="role" value="@SD.Customer" />@SD.Customer
    <input type="radio" name="role" value="@SD.Admin" />@SD.Admin
</div>
```

Define role manager and inject it

```c#
`private readonly RoleManager<IdentityRole> roleManager;`
    
public RegisterModel(RoleManager<IdentityRole> _roleManager)
  {
            _roleManager = roleManager;
  }
```

Create role if not exists

```c#
if(!await _roleManager.RoleExistsAsync(SD.Admin))
  {
      _roleManager.CreateAsync(new IdentityRole(SD.Admin)).GetAwaiter().GetResult();
      _roleManager.CreateAsync(new IdentityRole(SD.Customer)).GetAwaiter().GetResult();
  }
```

Assign Roles and add to database

```c#
string role = Request.Form["role"].ToString();
if(role == SD.Admin)
{
    await _userManager.AddToRoleAsync(user, SD.Admin);
}
if (role == SD.Customer)
{
    await _userManager.AddToRoleAsync(user, SD.Customer);
}
```



## Authorization

Just use as below

```c#
[Authorize] //<-- This
public class IndexModel : PageModel
```

If the user is not authorized, then user will be redirected to the login URL.

Configure login redirect URL in `program.cs`

```c#
// configure identity paths
builder.Services.ConfigureApplicationCookie(options =>
{
    options.LoginPath = "/Identity/Account/Login";
    options.LogoutPath = "/Identity/Account/Logout";
    options.AccessDeniedPath = "/Identity/Account/AccessDenied";
});
```

**Getting user id when user has already logged in**

**Getting Claims**

```c#
// fetch all the claims
var claimsIdentity = (ClaimsIdentity) User.Identity;
// get user id
var claims = claimsIdentity.FindFirst(ClaimTypes.NameIdentifier);
// claims.Value <-- This will contain the USER ID
```

**User role check**

```csharp
@using Microsoft.AspNetCore.Identity

@if(User.IsInRole('Admin')){}
```

## Database Seeding

**Step 01:** Create an initializer.

```c#
public interface IDbInitializer
    {
        void Initialize();
    }
```

```c#
public class DbInitializer : IDbInitializer
    {
        private readonly UserManager<IdentityUser> _userManager;
        private readonly RoleManager<IdentityRole> _roleManager;
        private readonly ApplicationDbContext _db;

        public DbInitializer(UserManager<IdentityUser> userManager, 
            RoleManager<IdentityRole> roleManager, 
                ApplicationDbContext applicationDbContext)
        {
            _userManager = userManager;
            _roleManager = roleManager;
            _db = applicationDbContext;
        }

        public void Initialize()
        {
            // push the pending migrations into DB
            try
            {
                if(_db.Database.GetPendingMigrations().Count() > 0)
                {
                    _db.Database.Migrate();
                }
            }
            catch (Exception){}

            // create roles
            if (!_roleManager.RoleExistsAsync(SD.Admin).GetAwaiter().GetResult())
            {
                _roleManager.CreateAsync(new IdentityRole(SD.Admin)).GetAwaiter().GetResult();
                _roleManager.CreateAsync(new IdentityRole(SD.Customer)).GetAwaiter().GetResult();

                // if roles donot exists, it means it is a fresh start.
                // create a new admin user
                _userManager.CreateAsync(new ApplicationUser
                {
                    UserName = "sandeep@gmail.com",
                    Email = "sandeep@gmail.com",
                    EmailConfirmed = true,
                    FirstName = "Sandeep",
                    LastName = "Dewangan"
                }, "Sandeep123@").GetAwaiter().GetResult();
                // Assign role ADMIN to the newly created user
                ApplicationUser user = _db.ApplicationUser.FirstOrDefault(u=> u.Email == "sandeep@gmail.com");
                _userManager.AddToRoleAsync(user, SD.Admin).GetAwaiter().GetResult();
            }
            return;
        }
    }
```

Under `program.cs` 

```csharp
// db seeder
builder.Services.AddScoped<IDbInitializer, DbInitializer>();  // <-- dependency injection

-------
app.UseRouting();
SeedDatabase(); // <-- call database seeder
app.UseAuthentication();;
------
void SeedDatabase() // <-- seeder method
{
    using (var scope = app.Services.CreateScope())
    {
        var dbInitalizer = scope.ServiceProvider.GetRequiredService<DbInitializer>();
        dbInitalizer.Initialize();
    }
}
```









































