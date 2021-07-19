# ASP.NET Core API
### Create Project
Create API Project
### Models
Define Models
```csharp
namespace ParkyAPI.Models
{
    public class NationalPark
    {
        [Key]
        public int Id { get; set; }
        [Required]
        public string Name { get; set; }
        [Required]
        public string State { get; set; }
        public DateTime Created { get; set; }
        public DateTime Established { get; set; }
    }
}
```
### Add Connection String
```csharp
"ConnectionStrings": {
    "DefaultConnection": "Server=SANDEEP-DEWANGA;Database=Parky;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
```
### Application Db Context
```csharp
public class ApplicationDBContext : DbContext
    {
        public ApplicationDBContext(DbContextOptions<ApplicationDBContext> options) : base(options)
        {

        }
        public DbSet<NationalPark> NationalParks { get; set; }
    }
```
### Startup.cs - Inject the dependencies
```csharp
public void ConfigureServices(IServiceCollection services)
        {

            // Inject the database
            services.AddDbContext<ApplicationDBContext>(
                        options => options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"))
                        );
...
		}
```
### Migrate and Update Database
`add-migration AddNationalParktToDb`
`update-database`

### Repository Pattern
```csharp
public interface INationalParkRepo
    {
        ICollection<NationalPark> GetNationalParks();
        NationalPark GetNationalParkById(int id);
        bool NationalParkExists(string name);
        bool NationalParkExists(int id);
        bool CreateNationalPark(NationalPark nationalPark);
        bool UpdateNationalPark(NationalPark nationalPark);
        bool DeleteNationalPark(NationalPark nationalPark);
        bool Save();
    }
```
```csharp
public class NationalParkRepo : INationalParkRepo
    {
        private readonly ApplicationDBContext _db;

        public NationalParkRepo(ApplicationDBContext db)
        {
            _db = db;
        }

        public bool CreateNationalPark(NationalPark nationalPark)
        {
            _db.NationalParks.Add(nationalPark);
            return Save();
        }

        public bool DeleteNationalPark(NationalPark nationalPark)
        {
            _db.NationalParks.Remove(nationalPark);
            return Save();
        }

        public NationalPark GetNationalParkById(int id)
        {
            return _db.NationalParks.FirstOrDefault(a => a.Id == id);
        }

        public ICollection<NationalPark> GetNationalParks()
        {
            return _db.NationalParks.OrderBy(a => a.Name).ToList();
        }

        public bool NationalParkExists(string name)
        {
            return _db.NationalParks.Any(a => a.Name.ToLower().Trim() == name.ToLower().Trim());
        }

        public bool NationalParkExists(int id)
        {
            return _db.NationalParks.Any(a => a.Id == id);
        }

        public bool Save()
        {
            return _db.SaveChanges() >= 0 ? true : false;
        }

        public bool UpdateNationalPark(NationalPark nationalPark)
        {
            _db.NationalParks.Update(nationalPark);
            return Save();
        }
    }
```
Add to startup file
`services.AddScoped<INationalParkRepo, NationalParkRepo>();`

### API Controller
Create a blank API controller.

```csharp
[Route("api/[controller]")]
    [ApiController]
    public class NationalParkController : ControllerBase
    {
        private readonly INationalParkRepo _npRepo;

        public NationalParkController(INationalParkRepo npRepo)
        {
            _npRepo = npRepo;
        }

        // URL: https://localhost:44375/api/nationalpark

        [HttpGet]
        public IActionResult GetNationalParks() {
            var objList = _npRepo.GetNationalParks();
            return Ok(objList);
        }

        // URL: https://localhost:44375/api/nationalpark/3

        [HttpGet("{id:int}", Name = "GetNationalPark")]
        public IActionResult GetNationalPark(int id)
        {
            var obj = _npRepo.GetNationalParkById(id);
            if(obj == null)
            {
                return NotFound();
            }
            return Ok(obj);
        }

        // URL: https://localhost:44375/api/nationalpark

        [HttpPost]
        public IActionResult CreateNationalPark([FromBody] NationalPark np)
        {
            if(np == null)
            {
                return BadRequest(ModelState);
            }
            if (_npRepo.NationalParkExists(np.Name))
            {
                ModelState.AddModelError("", "National Park Exists!");
                return StatusCode(404, ModelState);
            }
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }
            if (!_npRepo.CreateNationalPark(np))
            {
                ModelState.AddModelError("", "Somethign went wrong.");
                return StatusCode(500, ModelState);
            }
            // return Ok();

            // Use CreatedAtRoute for creating new record and return that new record.
            // Calls to GetNationalPark with created ID
            return CreatedAtRoute("GetNationalPark", new { id = np.Id }, np);
        }

        // URL: https://localhost:44375/api/nationalpark/1

        [HttpPatch("{id:int}", Name = "UpdateNationalPark")]
        public IActionResult UpdateNationalPark(int id, [FromBody] NationalPark np)
        {
            if (np == null || id != np.Id)
            {
                return BadRequest(ModelState);
            }
            if (!_npRepo.UpdateNationalPark(np))
            {
                ModelState.AddModelError("", "Somethign went wrong in updating the record.");
                return StatusCode(500, ModelState);
            }

            return NoContent();
        }

        // URL: https://localhost:44375/api/nationalpark/3

        [HttpDelete("{id:int}", Name = "DeleteNationalPark")]
        public IActionResult DeleteNationalPark(int id)
        {
            if (!_npRepo.NationalParkExists(id))
            {
                return NotFound();
            }

            var npObj = _npRepo.GetNationalParkById(id);
            if (!_npRepo.DeleteNationalPark(npObj))
            {
                ModelState.AddModelError("", "Somethign went wrong in deleting the record.");
                return StatusCode(500, ModelState);
            }

            return NoContent();
        }

    }
```

