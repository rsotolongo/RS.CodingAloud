---
title: "Asynchronous commands"
date: 2021-03-15
draft: false
slug: asynchronous-commands
tags: ["parallelism", "WPF"]
categories: ["Development"]
---
[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/AsyncCommands)

Recently I had to implement a simple application with Windows Presentation Framework (WPF) using Model-View-ViewModel (MVVM) pattern. I got surprised when I noticed that there was not a simple solution out-of-the-box for bind asynchronous commands, so I decided to create my implementations for them.

I also want genericity, so I started defining the most general interface that inheritance from ICommand (included in namespace: "System.Windows.Input").

``` C#
public interface IGenericCommand<TInput> : ICommand
{
    bool CanExecute(TInput parameter);

    void Execute(TInput parameter);

    void RaiseCanExecuteChanged();
}
```

Followed by the main interface for asynchronous commands:

``` C#
public interface IAsyncCommand<TInput, TOutput> : IGenericCommand<TInput>
{
    TOutput ExecuteAsync();

    TOutput ExecuteAsync(TInput parameter);
}
```

In theory, commands should return nothing at all (void) but if we want to work in async-await environment, we need to return Task at least (notice the: "...at least" because it will be enhanced later in this post).

With this interface defined, we can define a concrete set of interfaces for all possible scenarios:

1. Command that accepts a parameter in the call and returns something.
2. Command that accepts a parameter in the call but returns nothing.
3. Command that does not accept parameters in the call but returns something.
4. Command that does not accept parameters and returns nothing.

I am not the only one who likes to return the result of the execution of a command (success or failure, message, etc.) although this is not exactly what a command states. Thereby we have the previous four scenarios.

The four corresponding interfaces (notice that names are suggestive):

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

Then, let's me present the implementations, starting with the generic command:

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

There are two constructors accepting the method to evaluate if the command can be executed or not based on passing an optional parameter. The second optional parameter is useful to handle possible execution exceptions. Variable: "Variant" will discriminate which constructor was called in order to know which function should be called (the one that accepts parameters or the one that doesn't accept parameters). The rest is very simple method implementations.

Now, the implementation for the asynchronous interface:

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

This one is easy than its ancestor and prepares the way to its child classes. The first two corresponding to commands accepting a parameter for its execution and execution control:

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

Both have three different constructors to allows different possibilities for the execution control. The last two ones are very simple:

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

This is how all of these interfaces and implementations could be used in a ViewModel:

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

That is the way I designed a set of interfaces and classes to provides asynchronous support for commands binding in WPF application. I have the plan to convert this into a NuGet package. The source code includes less theoretical code and more testable functionalities, please take a look at it.

[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/AsyncCommands)
