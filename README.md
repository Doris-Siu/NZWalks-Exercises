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
- Authentication with JWT
- Identity Core: setting up Identity/Auth DB

run EF: specify a particular dBContext e.g. -Context "NZWalksAuthDbContext"
- create AuthController with Register & Login action methods
1. Register: Role-based authorization, add [Authorize Roles = "..."] to individual action method to protect API resources
2. Login: generate JWT after credential validation using tokenRepository
- 401 Unauthorized, 403 Forbidden
- userManager to interact with the Identity DB, e.g. .CreateAsync() , .AddToRolesAsync(), .FindByEmailAsync(), .CheckPasswordAsync()
- config Swagger option, Identity solution option & JWT option in Program.cs

## Section 9
- a new domain model Images to handle image upload by API
- imageRepository.Upload(imageDomainModel): 
1. find local path where the image will reside -> use Path.Combine()
2. upload image to the local path -> open FileStream & use image.CopyToAsync(stream) 
3. compose a urlFilePath -> use httpContextAccessor.HttpContext.Request.sth & set it to thr image's FilePath prop
4. add Image to the Images table & return Image back 
- enable serving of static files with FilePath -> 
config in Program.cs

app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(Path.Combine(Directory.GetCurrentDirectory(), "Images")),
    RequestPath = "/Images"
}
);

## Section 10


  

 
