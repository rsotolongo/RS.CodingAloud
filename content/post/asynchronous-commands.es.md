---
title: "Comandos asíncronos"
date: 2021-03-15
draft: false
slug: asynchronous-commands
tags: ["paralelismo", "WPF"]
categories: ["Desarrollo"]
---
[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/AsyncCommands)

Recientemente tuve que implementar una aplicación simple con Windows Presentation Framework (WPF) usando el patrón Model-View-ViewModel (MVVM). Me sorprendí cuando noté que no había una solución simple lista para usar para enlazar comandos asíncronos, así que decidí crear mis implementaciones para ellos.

También quiero ser genérico, así que comencé a definir la interfaz más general que hereda de ICommand (incluida en el espacio de nombres: "System.Windows.Input").

``` C#
public interface IGenericCommand<TInput> : ICommand
{
    bool CanExecute(TInput parameter);

    void Execute(TInput parameter);

    void RaiseCanExecuteChanged();
}
```

Seguido por la interfaz principal para comandos asíncronos:

``` C#
public interface IAsyncCommand<TInput, TOutput> : IGenericCommand<TInput>
{
    TOutput ExecuteAsync();

    TOutput ExecuteAsync(TInput parameter);
}
```

En teoría, los comandos no deberían devolver nada en absoluto (nulo) pero si queremos trabajar en un entorno async-await, necesitamos devolver Task al menos (observe el: "... al menos" porque se mejorará más adelante en este artículo).

Con esta interfaz definida, podemos definir un conjunto concreto de interfaces para todos los escenarios posibles:

1. Comando que acepta un parámetro en la llamada y devuelve algo.
2. Comando que acepta un parámetro en la llamada pero no devuelve nada.
3. Comando que no acepta parámetros en la llamada pero devuelve algo.
4. Comando que no acepta parámetros y no devuelve nada.

No soy el único al que le gusta devolver el resultado de la ejecución (éxito o fracaso, mensaje, etc.) aunque esto no sea exactamente lo que se le pide a un comando como tal. De esta manera tenemos los cuatro escenarios anteriores.

Las cuatro interfaces correspondientes (observe que los nombres son sugerentes):

``` C#
public interface IParameterizedReturnAsyncCommand<TInput, TOutput> : IAsyncCommand<TInput, Task<TOutput>>
{
}

public interface IParameterizedVoidAsyncCommand<TInput> : IAsyncCommand<TInput, Task>
{
}

public interface IParameterlessParameterizedAsyncCommand<TOutput> : IAsyncCommand<Void, Task<TOutput>>
{
}

public interface IParameterlessVoidAsyncCommand : IAsyncCommand<Void, Task>
{
}
```

Luego, presentemos las implementaciones, comenzando con el comando genérico:

``` C#
public abstract class GenericCommand<TInput> : IGenericCommand<TInput>
{
    protected GenericCommand(Func<TInput, bool> canExecute = null, IErrorHandler errorHandler = null)
    {
        CanExecuteMethodParameterized = canExecute;
        ErrorHandler = errorHandler;
        Variant = 0;
    }

    protected GenericCommand(Func<bool> canExecute = null, IErrorHandler errorHandler = null)
    {
        CanExecuteMethodParameterless = canExecute;
        ErrorHandler = errorHandler;
        Variant = 1;
    }

    public event EventHandler CanExecuteChanged;
    
    protected readonly Func<TInput, bool> CanExecuteMethodParameterized;
    
    protected readonly Func<bool> CanExecuteMethodParameterless;
    
    protected readonly IErrorHandler ErrorHandler;
    
    protected readonly int Variant;
    
    protected bool IsExecuting;
    
    public bool CanExecute(TInput parameter)
    {
        return !IsExecuting &&
            (((Variant == 0) && (CanExecuteMethodParameterized?.Invoke(parameter) ?? true)) ||
             ((Variant == 1) && (CanExecuteMethodParameterless?.Invoke() ?? true)));
    }
    
    public bool CanExecute(object parameter)
    {
        return CanExecute((TInput)parameter);
    }
    
    public abstract void Execute(TInput parameter);
    
    public void Execute(object parameter)
    {
        Execute((TInput)parameter);
    }
    
    public void RaiseCanExecuteChanged()
    {
        CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}
```

Hay dos constructores que aceptan el método para evaluar si el comando se puede ejecutar o no basándose en pasar un parámetro opcional. El segundo parámetro opcional es útil para manejar posibles excepciones de ejecución. Variable: "Variant" discriminará a qué constructor se llamó para saber qué función se debe llamar (la que acepta parámetros o la que no acepta parámetros). El resto son implementaciones de métodos muy simples.

Ahora, la implementación de la interfaz asíncrona:

``` C#
public abstract class AsyncCommand<TInput, TOutput> : GenericCommand<TInput>, IAsyncCommand<TInput, TOutput> where TOutput : Task
{
    protected AsyncCommand(Func<TInput, bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
    }
    
    protected AsyncCommand(Func<bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
    }
    
    public override async void Execute(TInput parameter)
    {
        try
        {
            await ExecuteAsync(parameter);
        }
        catch (Exception exception)
        {
            ErrorHandler?.HandleError(exception);
        }
    }

    public abstract TOutput ExecuteAsync();
    
    public abstract TOutput ExecuteAsync(TInput parameter);
}
```

Éste es más fácil que su antepasado y prepara el camino a sus clases secundarias. Los dos primeros corresponden a comandos que aceptan un parámetro para su ejecución y control de ejecución:

``` C#
public class ParameterizedReturnAsyncCommand<TInput, TOutput> : AsyncCommand<TInput, Task<TOutput>>, IParameterizedReturnAsyncCommand<TInput, TOutput>
{
    public ParameterizedReturnAsyncCommand(Func<TInput, Task<TOutput>> execute, Func<TInput, bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
        ExecuteMethodParameterized = execute;
    }

    public ParameterizedReturnAsyncCommand(Func<Task<TOutput>> execute, Func<TInput, bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
        ExecuteMethodParameterless = execute;
    }

    public ParameterizedReturnAsyncCommand(Func<Task<TOutput>> execute, Func<bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
        ExecuteMethodParameterless = execute;
    }

    private readonly Func<TInput, Task<TOutput>> ExecuteMethodParameterized;
    
    private readonly Func<Task<TOutput>> ExecuteMethodParameterless;
    
    public override async Task<TOutput> ExecuteAsync()
    {
        return await ExecuteAsync(default);
    }
    
    public override async Task<TOutput> ExecuteAsync(TInput parameter)
    {
        TOutput result = default;
        if (CanExecute(parameter))
        {
            try
            {
                IsExecuting = true;
                result = (Variant == 0)
                    ? await ExecuteMethodParameterized(parameter)
                    : await ExecuteMethodParameterless();
            }
            finally
            {
                IsExecuting = false;
            }
        }
        RaiseCanExecuteChanged();
        return result;
    }
}

public class ParameterizedVoidAsyncCommand<TInput> : AsyncCommand<TInput, Task>, IParameterizedVoidAsyncCommand<TInput>
{
    public ParameterizedVoidAsyncCommand(Func<TInput, Task> execute, Func<TInput, bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
        ExecuteMethodParameterized = execute;
    }

    public ParameterizedVoidAsyncCommand(Func<Task> execute, Func<TInput, bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
        ExecuteMethodParameterless = execute;
    }

    public ParameterizedVoidAsyncCommand(Func<Task> execute, Func<bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(canExecute, errorHandler)
    {
        ExecuteMethodParameterless = execute;
    }

    private readonly Func<TInput, Task> ExecuteMethodParameterized;
    
    private readonly Func<Task> ExecuteMethodParameterless;
    
    public override async Task ExecuteAsync()
    {
        await ExecuteAsync(default);
    }

    public override async Task ExecuteAsync(TInput parameter)
    {
        if (CanExecute(parameter))
        {
            try
            {
                IsExecuting = true;
                if (Variant == 0)
                {
                    await ExecuteMethodParameterized(parameter);
                }
                else
                {
                    await ExecuteMethodParameterless();
                }
            }
            finally
            {
                IsExecuting = false;
            }
        }
        RaiseCanExecuteChanged();
    }
}
```

Ambas tienen tres constructores diferentes para permitir diferentes posibilidades para el control de ejecución. Los dos últimos son muy simples:

``` C#
public class ParameterlessParameterizedAsyncCommand<TOutput> : ParameterizedReturnAsyncCommand<Void, TOutput>, IParameterlessParameterizedAsyncCommand<TOutput>
{
    public ParameterlessParameterizedAsyncCommand(Func<Task<TOutput>> execute, Func<bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(execute, canExecute, errorHandler)
    {
    }
}

public class ParameterlessVoidAsyncCommand : ParameterizedVoidAsyncCommand<Void>, IParameterlessVoidAsyncCommand
{
    public ParameterlessVoidAsyncCommand(Func<Task> execute, Func<bool> canExecute = null, IErrorHandler errorHandler = null)
        : base(execute, canExecute, errorHandler)
    {
    }
}
```

Así es como todas estas interfaces e implementaciones podrían usarse en un modelo de vista:

``` C#
public class MainWindowViewModel : INotifyPropertyChanged
{
    public MainWindowViewModel()
    {
        ParameterizedReturnCommand = new ParameterizedReturnAsyncCommand<string, string>(ParameterizedReturnMethod, parameter => OddLength(parameter));
        //// ParameterizedReturnCommand = new ParameterizedReturnAsyncCommand<string, string>(ParameterlessReturnMethod, parameter => OddLength(parameter));
        //// ParameterizedReturnCommand = new ParameterizedReturnAsyncCommand<string, string>(ParameterlessReturnMethod, () => OddLength("OddString"));
        ParameterizedVoidCommand = new ParameterizedVoidAsyncCommand<string>(ParameterizedVoidMethod, parameter => OddLength(parameter));
        //// ParameterizedVoidCommand = new ParameterizedVoidAsyncCommand<string>(ParameterlessVoidMethod, parameter => OddLength(parameter));
        //// ParameterizedVoidCommand = new ParameterizedVoidAsyncCommand<string>(ParameterlessVoidMethod, () => OddLength("OddString"));
        ParameterlessReturnCommand = new ParameterlessParameterizedAsyncCommand<string>(ParameterlessReturnMethod, () => OddLength("OddString"));
        ParameterlessVoidCommand = new ParameterlessVoidAsyncCommand(ParameterlessVoidMethod, () => OddLength("OddString"));
    }
    
    public IParameterizedReturnAsyncCommand<string, string> ParameterizedReturnCommand { get; set; }
    
    public IParameterizedVoidAsyncCommand<string> ParameterizedVoidCommand { get; set; }
    
    public IParameterlessParameterizedAsyncCommand<string> ParameterlessReturnCommand { get; set; }
    
    public IParameterlessVoidAsyncCommand ParameterlessVoidCommand { get; set; }
    
    private async Task<string> ParameterizedReturnMethod(string text)
    {
        //// Command's code goes here.
    }
    
    private async Task ParameterizedVoidMethod(string text)
    {
        //// Command's code goes here.
    }
    
    private async Task<string> ParameterlessReturnMethod()
    {
        //// Command's code goes here.
    }
    
    private async Task ParameterlessVoidMethod()
    {
        //// Command's code goes here.
    }
    
    private bool OddLength(string value)
    {
        return !string.IsNullOrWhiteSpace(value) && (value.Length % 2 != 0);
    }
}
```

Esa es la forma en que diseñé un conjunto de interfaces y clases para proporcionar soporte asincrónico para los comandos vinculados en la aplicación WPF. Tengo el plan de convertir esto en un paquete NuGet. El código fuente incluye código menos teórico y funcionalidades más comprobables, por favor, revísenlo.

[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/AsyncCommands)
