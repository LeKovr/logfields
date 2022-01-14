# logfileds

## Задача

При выводе в лог или возврате ошибок иметь дополнительную информацию о

1. имени файла и строке возникновения собыия
2. имени функции
3. ее аргументах
4. возможных значимых переменных

## Текущее состояние

Текущие библиотеки логгирования позволяют получить информацию по пунктам 1 и 2 автоматически, а для 3 и 4 позволяют использовать конструкцию вида

```
log = log.WithFields("var1", value1, "var2", value2, ...)
```

Использовать эти поля при возникновении ошибки можно только в том случае, когда ошибка протоколируется в точке возникновения командой `log.Error(...)`.

## Цели

1. при возникновении возвращать ошибку без записи в журнал, но без потери дополнительной информации
2. при анализе ошибки иметь возможность использовать `err.Is()`
3. задавать доп информацию для ошибок и журнала без дублирования

## Пример желаемого

```go

var ErrNoZero = errors.New("arg must not be zero")
var ErrNoLuck = errors.New("no luck")

func DoSomething(arg int) error {
  lf := logfields.New("arg",arg) // func name saved here
  lf.Debug("starting") // filename & line saved here
  if arg == 0 {
    return lf.Error(ErrNoZero) // filename & line saved here
  }
  ...
  if err != nil {
    return lf.Wrap(ErrNoLuck, err) // filename & line saved here if empty
  }
  return nil
}

func DoMain() {
  err := DoSomething(1)
  if err != nil {
    switch {
        case errors.Is(err, ErrNoZero):
            fmt.Printf("not allowed: %s", err) // prints message, arg, func, filename, line
    }
  }
}

```

