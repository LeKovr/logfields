# logfileds

## Задача

При выводе в лог или возврате ошибок иметь дополнительную информацию про

1. имя файла и номер строки возникновения события
2. имя функции
3. ее аргументы
4. возможные значимые переменные

## Текущее состояние

Текущие библиотеки логгирования позволяют получить информацию по пунктам 1 и 2 автоматически, а для 3 и 4 позволяют использовать конструкцию вида

```
log = log.WithFields("var1", value1, "var2", value2, ...)
```

Использовать эти поля при возникновении ошибки можно только в том случае, когда ошибка протоколируется в точке возникновения командой `log.Error(...)`.

## Цели

1. при возникновении ошибки возвращать ее без журналирования, но и без потери дополнительной информации
2. при анализе ошибки иметь возможность использовать `errors.Is()`
3. не дублировтаь доп информацию для ошибок и журнала

## Пример желаемого

```go

var ErrNoZero = errors.New("arg must not be zero")
var ErrNoLuck = errors.New("no luck")

func DoSomething(ctx context.Context, arg int) error {
  log := logr.FromContextOrDiscard(ctx)
  lf := logfields.New("arg", arg) // func name saved here
  lf.Debug(log, "starting") // filename & line printed also
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
            fmt.Printf("not allowed: %s", err) // err.Error() contains message, arg, func, filename, line
        case errors.Is(err, ErrNoLuck):
            if logfields.ValueString(err, "arg") == "1" {
              fmt.Printf("you call 1")
            }
    }
  }
}

```

