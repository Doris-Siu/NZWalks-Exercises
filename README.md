# ASP.NET8 overview
This will serve as quick notes for refreshing the relevant concepts and syntax covered in each section. Feel free to use this repo if you find it is useful.

## Section 1
- install and configure VS, SQL server, SMSS
- connect APP to DB :  
"ConnectionStrings": {
   "NZWalksConnectionString": "Server=< serverName >;Database=< dBName >;Trusted_Connection=True;TrustServerCertificate=True"
}

## Section 2
- REST API & HTTP verbs
- Domain Models
- EF migration :
1. Add-Migration < nameOfMigration >
2. Update-Migration

- DI:

builder.Services.AddDbContext<NZWalksDbContext>(options =>
options.UseSqlServer(builder.Configuration.GetConnectionString("NZWalksConnectionString")));

builder.Services.AddScoped<IWalkRepository, SQLWalkRepository>();

builder.Services.AddAutoMapper(typeof(AutoMapperProfiles));

## Section 3
- controller actions - CRUD operations
- DTOs 

## Section 4
- utilize Repository Pattern for best practice
- make all actions in Controller, Repository ASYNC
- utilize 1-line AutoMapper

## Section 5
- seed data to DB using EF:

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
...

modelBuilder.Entity<Difficulty>().HasData(difficulties);
}

## Section 6
- Data Annotation in DTOs: e.g. [Required], [MinLength(3, ErrorMessage="...")], [Range(0,50)]
- utilize [ValidateModel] : 
create new DIR CustomActionFilters\ValidateModelAttribute.cs
 
public class ValidateModelAttribute : ActionFilterAttribute
 {
     public override void OnActionExecuting(ActionExecutingContext context)
     {
         if (context.ModelState.IsValid == false) 
         {
             context.Result = new BadRequestResult();
         }
     }
 }

## Section 7
- Query Parameters for Filtering, Sorting & Pagination
- SQLRepository Implementation:
1. convert IEnumerable to IQueryable
2. LINQ Query:
e.g. 
- .Where(x=> x.Name.Contains(filterQuery))
- .OrderBy(x => x.Length)
- 1-line formula for pagination :
var skipResults = (pageNumber - 1) * pageSize;
3. return await walks.Skip(skipResults).Take(pageSize).ToListAsync();

## Section 8

 
