# Exempel på modifierad användare i .NET 9
Ett litet exempel som visar hur en användare kan modifieras med extra fält, i ett MVC-projekt med Identity.

## Instruktioner
``dotnet run`` för att starta applikationen.
Registrera ett konto, där nu möjlighet finns för förnamn och efternamn.
Beskök undersidan "Privacy" för utskrift (se även inloggnings-partialen överst).

## Steg för steg
1: Skapa ett nytt projekt med MVC och Identity:
```bash
dotnet new mvc --auth Individual -o ModifiedUser
```
2: Ska en ny model under **Models** med namn **ApplicationUser**:
```bash
using Microsoft.AspNetCore.Identity;

namespace ModifiedUser.Models
{
    public class ApplicationUser : IdentityUser
    {
        public string FirstName { get; set; } = string.Empty;
        public string LastName { get; set; } = string.Empty;
    }
}
```
3: Konfigurera att denna skall användas i **Program.cs**:
```bash
builder.Services.AddDefaultIdentity<ApplicationUser>(options => options.SignIn.RequireConfirmedAccount = false)
    .AddEntityFrameworkStores<ApplicationDbContext>();
```
(Notera **ApplicationUser** har ersatt **IdentityUser**)
4: Konfigurera **ApplicationDbContext** att använda denna modell:
```bash
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
```

5: Ändra överst i Views/Shared/_LoginPartial.cshtml till att använda ApplicationUser:
```bash
@using Microsoft.AspNetCore.Identity
@inject SignInManager<ApplicationUser> SignInManager
@inject UserManager<ApplicationUser> UserManager

6: Skapa en ny migrations-fil och kör denna:
```bash
dotnet ef migrations add ModifiedUserFields
dotnet ef database update
```
7: Generera filer för "scaffolding":
```bash
dotnet aspnet-codegenerator identity -dc ApplicationDbContext --force
```
("force" skriver över eventuellt befintliga filer, så kör denna växel med försiktighet)

8: Modifiera Areas/Identity/Pages/Account/Register.cshtml med dessa nya två fält:
```bash
<div class="form-floating mb-3">
    <input asp-for="Input.FirstName" class="form-control" placeholder="John" />
    <label asp-for="Input.FirstName">First Name:</label>
    <span asp-validation-for="Input.FirstName" class="text-danger"></span>
</div>

<div class="form-floating mb-3">
    <input asp-for="Input.LastName" class="form-control" placeholder="Doe" />
    <label asp-for="Input.LastName">First Name:</label>
    <span asp-validation-for="Input.LastName" class="text-danger"></span>
</div>

<button id="registerSubmit" type="submit" class="w-100 btn btn-lg btn-primary">Register</button>
```

9: Modifiera **Register.cshtml.cs** med följande ändring längst ner i InputModel:
```bash
// Extra fields
[Display(Name = "First Name")]
public string FirstName { get; set; }

[Display(Name = "Last Name")]
public string LastName { get; set; }
```

10: Modifiera metoden för **CreateUser** med:
```bash
try
    {
        var user = Activator.CreateInstance<ApplicationUser>();

        // Custom fields
        user.FirstName = Input.FirstName;
        user.LastName = Input.LastName;

        return user;
    }
```
11: Klart för testkörning - registrera ett konto och fyll i för- och efternamn.

12: För att läsa ut dessa fält går det exempelvis att göra såhär:
```bash
@{
    var user = await UserManager.GetUserAsync(User);

    if (user != null && user.FirstName != null && user.LastName != null)
    {
        var firstName = user.FirstName;
        var lastName = user.LastName;

        <p>Hello @firstName @lastName!</p>
    }
}
```

## Av
Mattias Dahlgren, mattias.dahlgren@miun.se

