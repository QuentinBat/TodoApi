> Web API basique en .NET Core 3.1 avec connexion SQL Server + Entity Framework

# Création  d'une API web

## Création du projet

* Dans le menu **Fichier**, sélectionnez **Nouveau** > **Projet**.
* Sélectionnez le modèle **Application web ASP.NET Core** et cliquez sur **Suivant**.
* Nommez le projet *TodoApi* et cliquez sur **Créer**.
* Dans la boîte de dialogue **créer une application Web ASP.net Core** , vérifiez que **.net Core** et **ASP.net Core 3,1** sont sélectionnés. Sélectionnez le modèle **API** et cliquez sur **Créer**.

## Ajouter une classe de modèle

- Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet. Sélectionnez **Ajouter** > **Nouveau dossier**. Nommez le dossier *Models*.

- Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Classe**. Nommez la classe *TodoItem* et sélectionnez sur **Ajouter**.

- Remplacez le code du modèle par le code suivant :

  ```c#
  namespace TodoApi.Models
  {
      public class TodoItem
      {
          public long Id { get; set; }
          public string Name { get; set; }
          public bool IsComplete { get; set; }
      }
  }
  ```

## Ajouter un contexte de base de données

### Ajouter le package EntityFramework SQL Server

- Dans le menu **Outils**, sélectionnez **Gestionnaire de package NuGet > Gérer les packages NuGet pour la solution**.
- Sélectionnez l’onglet **Parcourir**, puis entrez **Microsoft.EntityFrameworkCore.SqlServer** dans la zone de recherche.
- Dans le volet gauche, sélectionnez **Microsoft. EntityFrameworkCore. SqlServer** .
- Cochez la case **Projet** dans le volet droit, puis sélectionnez **Installer**.

### Création du contexte de la base de données

* Cliquez avec le bouton droit sur le dossier *Models* et sélectionnez **Ajouter** > **Classe**. Nommez la classe *TodoContext* et cliquez sur **Ajouter**.

```c#
using Microsoft.EntityFrameworkCore;

namespace TodoApi.Models
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

        public DbSet<TodoItem> TodoItems { get; set; }
    }
}
```

### Inscription du contexte de la base de données

* Dans le Startup.cs, ajouter les éléments suivants :

```c#
...
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;

namespace TodoApi
{
    public class Startup
    {
        ...
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<TodoContext>(options => options.UseSqlServer(Configuration.GetConnectionString("TodoContext")));
            services.AddControllers();
        }
        ...
    }
}
```

* Dans le fichier appsettings.json, ajouter la chaine de connexion à la base de données

```json
"ConnectionStrings": {
    "TodoContext": "Server=(localdb)\\mssqllocaldb;Database=Todo;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
```

## Création de la base de données

* Créer la base de donnée
* Créer la migration :

```powershell
Add-Migration "Initial migration"
```

* Appliquer la mise à jour :

```powershell
Update-Database
```

## Création des actions du contrôleur

```c#
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using TodoApi.Models;

namespace TodoApi.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class TodoItemsController : ControllerBase
    {
        private readonly TodoContext _context;

        public TodoItemsController(TodoContext context)
        {
            _context = context;
        }

        // GET: api/TodoItems
        [HttpGet]
        public async Task<ActionResult<IEnumerable<TodoItem>>> GetTodoItems()
        {
            return await _context.TodoItems.ToListAsync();
        }

        // GET: api/TodoItems/5
        [HttpGet("{id}")]
        public async Task<ActionResult<TodoItem>> GetTodoItem(long id)
        {
            var todoItem = await _context.TodoItems.FindAsync(id);

            if (todoItem == null)
            {
                return NotFound();
            }

            return todoItem;
        }

        // PUT: api/TodoItems/5
        [HttpPut("{id}")]
        public async Task<IActionResult> PutTodoItem(long id, TodoItem todoItem)
        {
            if (id != todoItem.Id)
            {
                return BadRequest();
            }

            _context.Entry(todoItem).State = EntityState.Modified;

            try
            {
                await _context.SaveChangesAsync();
            }
            catch (DbUpdateConcurrencyException)
            {
                if (!TodoItemExists(id))
                {
                    return NotFound();
                }
                else
                {
                    throw;
                }
            }

            return NoContent();
        }

        // POST: api/TodoItems
        [HttpPost]
        public async Task<ActionResult<TodoItem>> PostTodoItem(TodoItem todoItem)
        {
            _context.TodoItems.Add(todoItem);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetTodoItem), new { id = todoItem.Id }, todoItem);
        }

        // DELETE: api/TodoItems/5
        [HttpDelete("{id}")]
        public async Task<ActionResult<TodoItem>> DeleteTodoItem(long id)
        {
            var todoItem = await _context.TodoItems.FindAsync(id);
            if (todoItem == null)
            {
                return NotFound();
            }

            _context.TodoItems.Remove(todoItem);
            await _context.SaveChangesAsync();

            return todoItem;
        }

        private bool TodoItemExists(long id)
        {
            return _context.TodoItems.Any(e => e.Id == id);
        }
    }
}

```

