# Xamarin To Do List
Aplicacion de ejemplo usando una base de datos SQLite, para almacenar todas las tareas que se vayan introduciendo en la aplicacion por parte del usuario.

## Colaboradores 鉁掞笍 
- Diaz Carlos
- Vasconez John
- Alexis Yepez

## Infografia 馃摉
Explicacion sobre la herramienta de [Xamarin](https://github.com/JoelDiaz93/Xamarin-ToDoList/blob/master/Infograf%C3%ADa.pdf)

## Comenzando 馃殌
Integre SQLite.NET aplicaciones m贸viles siguiendo estos pasos:

Instale el NuGet .
Configure constantes.
Cree una clase de acceso a la base de datos.
Access data in Xamarin.Forms.
Configuraci贸n avanzada.
Instalaci贸n del paquete de NuGet SQLite
Use el NuGet paquetes para buscar sqlite-net-pcl y agregar la versi贸n m谩s reciente al proyecto de c贸digo compartido.

##Explicacion del codigo 鈱笍

### Configuracion
Los valores SQLiteOpenFlag de enumeraci贸n predeterminados que se usan para inicializar la conexi贸n de base de datos. La SQLiteOpenFlag enumeraci贸n admite estos valores:

- Create: la conexi贸n crear谩 autom谩ticamente el archivo de base de datos si no existe.
- FullMutex: la conexi贸n se abre en modo de subproceso serializado.
- NoMutex: la conexi贸n se abre en modo multiproceso.
- PrivateCache: la conexi贸n no participar谩 en la cach茅 compartida, incluso si est谩 habilitada.
- ReadWrite: la conexi贸n puede leer y escribir datos.
- SharedCache: la conexi贸n participar谩 en la cach茅 compartida, si est谩 habilitada.
- ProtectionComplete: el archivo est谩 cifrado e inaccesible mientras el dispositivo est谩 bloqueado.
- ProtectionCompleteUnlessOpen: el archivo se cifra hasta que se abre, pero luego es accesible incluso si el usuario bloquea el dispositivo.
- ProtectionCompleteUntilFirstUserAuthentication: el archivo se cifra hasta despu茅s de que el usuario haya arrancado y desbloqueado el dispositivo.
- ProtectionNone: el archivo de base de datos no est谩 cifrado.

```
public static class Constants
{
    public const string DatabaseFilename = "TodoSQLite.db3";

    public const SQLite.SQLiteOpenFlags Flags =
        // open the database in read/write mode
        SQLite.SQLiteOpenFlags.ReadWrite |
        // create the database if it doesn't exist
        SQLite.SQLiteOpenFlags.Create |
        // enable multi-threaded database access
        SQLite.SQLiteOpenFlags.SharedCache;

    public static string DatabasePath
    {
        get
        {
            var basePath = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
            return Path.Combine(basePath, DatabaseFilename);
        }
    }
}
```
### Creaci贸n de una clase de acceso a la base de datos
### Inicializaci贸n diferida
usa TodoItemDatabase la inicializaci贸n diferida asincr贸nica, representada por la clase personalizada, para retrasar la inicializaci贸n de la base de datos hasta que se accede AsyncLazy<T> por primera vez:
  
```
public class TodoItemDatabase
{
    static SQLiteAsyncConnection Database;

    public static readonly AsyncLazy<TodoItemDatabase> Instance = new AsyncLazy<TodoItemDatabase>(async () =>
    {
        var instance = new TodoItemDatabase();
        CreateTableResult result = await Database.CreateTableAsync<TodoItem>();
        return instance;
    });

    public TodoItemDatabase()
    {
        Database = new SQLiteAsyncConnection(Constants.DatabasePath, Constants.Flags);
    }

    //...
}
```

### Inicializaci贸n diferida asincr贸nica
Para iniciar la inicializaci贸n de la base de datos, evitar el bloqueo de la ejecuci贸n y tener la oportunidad de detectar excepciones, la aplicaci贸n de ejemplo usa la inicializaci贸n diferida asincr贸nica, representada por la AsyncLazy<T> clase :
  
```
public class AsyncLazy<T>
{
    readonly Lazy<Task<T>> instance;

    public AsyncLazy(Func<T> factory)
    {
        instance = new Lazy<Task<T>>(() => Task.Run(factory));
    }

    public AsyncLazy(Func<Task<T>> factory)
    {
        instance = new Lazy<Task<T>>(() => Task.Run(factory));
    }

    public TaskAwaiter<T> GetAwaiter()
    {
        return instance.Value.GetAwaiter();
    }
}
```
### M茅todos de manipulaci贸n de datos
La TodoItemDatabase clase incluye m茅todos para los cuatro tipos de manipulaci贸n de datos: crear, leer, editar y eliminar. La SQLite.NET proporciona un mapa relacional de objetos (ORM) simple que permite almacenar y recuperar objetos sin escribir SQL instrucciones.
  
```
public class TodoItemDatabase
{
    // ...
    public Task<List<TodoItem>> GetItemsAsync()
    {
        return Database.Table<TodoItem>().ToListAsync();
    }

    public Task<List<TodoItem>> GetItemsNotDoneAsync()
    {
        // SQL queries are also possible
        return Database.QueryAsync<TodoItem>("SELECT * FROM [TodoItem] WHERE [Done] = 0");
    }

    public Task<TodoItem> GetItemAsync(int id)
    {
        return Database.Table<TodoItem>().Where(i => i.ID == id).FirstOrDefaultAsync();
    }

    public Task<int> SaveItemAsync(TodoItem item)
    {
        if (item.ID != 0)
        {
            return Database.UpdateAsync(item);
        }
        else
        {
            return Database.InsertAsync(item);
        }
    }

    public Task<int> DeleteItemAsync(TodoItem item)
    {
        return Database.DeleteAsync(item);
    }
}
```
### Acceso a datos en Xamarin.Forms
La clase expone el campo , a trav茅s del cual se pueden invocar las operaciones de acceso a datos TodoItemDatabaseInstance de la clase TodoItemDatabase :
  
```
async void OnSaveClicked(object sender, EventArgs e)
{
    var todoItem = (TodoItem)BindingContext;
    TodoItemDatabase database = await TodoItemDatabase.Instance;
    await database.SaveItemAsync(todoItem);

    // Navigate backwards
    await Navigation.PopAsync();
}
```
Para habilitar WAL en SQLite.NET, llame al EnableWriteAheadLoggingAsync m茅todo en la instancia de SQLiteAsyncConnection :
```
await Database.EnableWriteAheadLoggingAsync();
```


## Capturas de Funcionamiento de la aplicacion 馃搫
- Lista de las actividades

![Captura de pantalla (93)](https://user-images.githubusercontent.com/58042087/150618999-3045dae2-6da7-42d5-8ecc-c1de691e223a.png)

- Creacion o actualizacion de actividades

![Captura de pantalla (92)](https://user-images.githubusercontent.com/58042087/150619039-91406d23-fa4b-4c7a-af0c-db40a51e9fd9.png)

- Ejecucion del apk en BlueStacks

![Captura de pantalla (95)](https://user-images.githubusercontent.com/58042087/150619049-a44e63f0-22d0-4a37-96b1-156696134756.png)
