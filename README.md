# Xamarin To Do List
Aplicacion de ejemplo usando una base de datos SQLite, para almacenar todas las tareas que se vayan introduciendo en la aplicacion por parte del usuario.

## Colaboradores ✒️ 
- Diaz Carlos
- Vasconez John
- Alexis Yepez

## Infografia 📖
Explicacion sobre la herramienta de [Xamarin](https://github.com/JoelDiaz93/Xamarin-ToDoList/blob/master/Infograf%C3%ADa.pdf)

## Comenzando 🚀
Integre SQLite.NET aplicaciones móviles siguiendo estos pasos:

Instale el NuGet .
Configure constantes.
Cree una clase de acceso a la base de datos.
Access data in Xamarin.Forms.
Configuración avanzada.
Instalación del paquete de NuGet SQLite
Use el NuGet paquetes para buscar sqlite-net-pcl y agregar la versión más reciente al proyecto de código compartido.

##Explicacion del codigo ⌨️

### Configuracion
Los valores SQLiteOpenFlag de enumeración predeterminados que se usan para inicializar la conexión de base de datos. La SQLiteOpenFlag enumeración admite estos valores:

- Create: la conexión creará automáticamente el archivo de base de datos si no existe.
- FullMutex: la conexión se abre en modo de subproceso serializado.
- NoMutex: la conexión se abre en modo multiproceso.
- PrivateCache: la conexión no participará en la caché compartida, incluso si está habilitada.
- ReadWrite: la conexión puede leer y escribir datos.
- SharedCache: la conexión participará en la caché compartida, si está habilitada.
- ProtectionComplete: el archivo está cifrado e inaccesible mientras el dispositivo está bloqueado.
- ProtectionCompleteUnlessOpen: el archivo se cifra hasta que se abre, pero luego es accesible incluso si el usuario bloquea el dispositivo.
- ProtectionCompleteUntilFirstUserAuthentication: el archivo se cifra hasta después de que el usuario haya arrancado y desbloqueado el dispositivo.
- ProtectionNone: el archivo de base de datos no está cifrado.

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
### Creación de una clase de acceso a la base de datos
### Inicialización diferida
usa TodoItemDatabase la inicialización diferida asincrónica, representada por la clase personalizada, para retrasar la inicialización de la base de datos hasta que se accede AsyncLazy<T> por primera vez:
  
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

### Inicialización diferida asincrónica
Para iniciar la inicialización de la base de datos, evitar el bloqueo de la ejecución y tener la oportunidad de detectar excepciones, la aplicación de ejemplo usa la inicialización diferida asincrónica, representada por la AsyncLazy<T> clase :
  
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
### Métodos de manipulación de datos
La TodoItemDatabase clase incluye métodos para los cuatro tipos de manipulación de datos: crear, leer, editar y eliminar. La SQLite.NET proporciona un mapa relacional de objetos (ORM) simple que permite almacenar y recuperar objetos sin escribir SQL instrucciones.
  
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
La clase expone el campo , a través del cual se pueden invocar las operaciones de acceso a datos TodoItemDatabaseInstance de la clase TodoItemDatabase :
  
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
Para habilitar WAL en SQLite.NET, llame al EnableWriteAheadLoggingAsync método en la instancia de SQLiteAsyncConnection :
```
await Database.EnableWriteAheadLoggingAsync();
```


## Capturas de Funcionamiento de la aplicacion 📄
- Lista de las actividades

![Captura de pantalla (93)](https://user-images.githubusercontent.com/58042087/150618999-3045dae2-6da7-42d5-8ecc-c1de691e223a.png)

- Creacion o actualizacion de actividades

![Captura de pantalla (92)](https://user-images.githubusercontent.com/58042087/150619039-91406d23-fa4b-4c7a-af0c-db40a51e9fd9.png)

- Ejecucion del apk en BlueStacks

![Captura de pantalla (95)](https://user-images.githubusercontent.com/58042087/150619049-a44e63f0-22d0-4a37-96b1-156696134756.png)
