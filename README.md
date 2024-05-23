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
- use Serilog  & use logger in the action methods to log valuable info
- log the output to Console or .txt file
- diff. levels: information, warning & error
- global exception handling: 
1. create custom ExceptionHandlerMiddleware at 1 place & use its method anywhere you want to log info, e.g. Controller, Repository classes
2. use try catch blocks & return back a generic http response with status code 500

## Section 11 
- API versioning: manual doing / use nuget package
- using nuget package : https://im5tu.io/article/2022/09/asp.net-core-versioning-mvc-apis/

## Section 12
- use try catch block & httpClient to consume APIs in the MVC application
- use .EnsureSuccessStatusCode(), once succeed -> extract the reponse body from the http response content like:
await httpResponseMessage.Content.ReadFromJsonAsync<IEnumerable<RegionDto>>();
- in the < a > tag, can specify the routing attributes below:
1. asp-controller = "Regions"
2. asp-action = "Index"
3. asp-route-id = "@region.id"

## Section 13
- deploy ASP.NET web app to Azure
- naming convention: matter-projectName-regionName-dev-codeNumber, e.g. rg-nzwalks-eastus-dev-001, sqlserver-nzwalks-eastus-dev-001 ...
1. Go to Azure portal > Add Subscription > Create Resource group
2. Create App Service 
3. Create SQL Databases > Add Network info of SQL (enable Public access: add client IPv4 address & check the exception)
4. Back to VS:
- config account (File > Account setting)
- create Profile (Right click the API proj & click Publish)
- connect Service Dependencies (2 DBs) to Azure SQL Database
- click the Publish again
- add the Migrations upon the publish of DBs (click the More Actions under Publish & select Edit)
5. Back to App Service, go to the Advanced Tools to troubleshoot with the DLL
- direct to the Kudu  & go to CMD to debug (cd site/wwwroot)
- type dotnet NZWalks.API.dll to spot the exception
- publish the Images folder in local dir to Azure as well & restart the app
6. use Postman to test the API working as expected  with the domain assigned by Azure

  

 
