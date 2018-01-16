## 问题起因
fmt.println(err)
但是error 没有实现String()方法，却依然打印出信息

## 源码
根据源码一路探寻
```
func (p *pp) printArg(arg interface{}, verb rune) {
  p.arg = arg
 p.value = reflect.Value{}    if arg == nil {
  switch verb {
  case 'T', 'v':
         p.fmt.padString(nilAngleString)
  default:
         p.badVerb(verb)
 }  return
  }    // Special processing considerations.
 // %T (the value's type) and %p (its address) are special; we always do them first.  switch verb {
  case 'T':
      p.fmt.fmt_s(reflect.TypeOf(arg).String())
  return
 case 'p':
      p.fmtPointer(reflect.ValueOf(arg), 'p')
  return
  }    // Some types can be done without reflection.
  switch f := arg.(type) {
  case bool:
      p.fmtBool(f, verb)
  case float32:
      p.fmtFloat(float64(f), 32, verb)
  case float64:
      p.fmtFloat(f, 64, verb)
  case complex64:
      p.fmtComplex(complex128(f), 64, verb)
  case complex128:
      p.fmtComplex(f, 128, verb)
  case int:
      p.fmtInteger(uint64(f), signed, verb)
  case int8:
      p.fmtInteger(uint64(f), signed, verb)
  case int16:
      p.fmtInteger(uint64(f), signed, verb)
  case int32:
      p.fmtInteger(uint64(f), signed, verb)
  case int64:
      p.fmtInteger(uint64(f), signed, verb)
  case uint:
      p.fmtInteger(uint64(f), unsigned, verb)
  case uint8:
      p.fmtInteger(uint64(f), unsigned, verb)
  case uint16:
      p.fmtInteger(uint64(f), unsigned, verb)
  case uint32:
      p.fmtInteger(uint64(f), unsigned, verb)
  case uint64:
      p.fmtInteger(f, unsigned, verb)
  case uintptr:
      p.fmtInteger(uint64(f), unsigned, verb)
  case string:
      p.fmtString(f, verb)
  case []byte:
      p.fmtBytes(f, verb, "[]byte")
  case reflect.Value:
      // Handle extractable values with special methods
 // since printValue does not handle them at depth 0.  if f.IsValid() && f.CanInterface() {
  p.arg = f.Interface()
  if p.handleMethods(verb) {
  return
  }
 }  p.printValue(f, verb, 0)
  default:
      // If the type is not simple, it might have methods.
  if !p.handleMethods(verb) {
  // Need to use reflection, since the type had no
 // interface methods that could be used for formatting.  p.printValue(reflect.ValueOf(f), verb, 0)
 } } }
```
当是error类型或结构体时走default
```
func (p *pp) handleMethods(verb rune) (handled bool) {
  if p.erroring {
  return
  }
  // Is it a Formatter?
  if formatter, ok := p.arg.(Formatter); ok {
  handled = true
  defer p.catchPanic(p.arg, verb)
  formatter.Format(p, verb)
  return
  }    // If we're doing Go syntax and the argument knows how to supply it, take care of it now.
  if p.fmt.sharpV {
  if stringer, ok := p.arg.(GoStringer); ok {
  handled = true
  defer p.catchPanic(p.arg, verb)
  // Print the result of GoString unadorned.
  p.fmt.fmt_s(stringer.GoString())
  return
  }
 } else {
  // If a string is acceptable according to the format, see if
 // the value satisfies one of the string-valued interfaces. // Println etc. set verb to %v, which is "stringable".  switch verb {
  case 'v', 's', 'x', 'X', 'q':
         // Is it an error or Stringer?
 // The duplication in the bodies is necessary: // setting handled and deferring catchPanic // must happen before calling the method.  switch v := p.arg.(type) {
  case error:
            handled = true
  defer p.catchPanic(p.arg, verb)
  p.fmtString(v.Error(), verb)
  return   case Stringer:
            handled = true
  defer p.catchPanic(p.arg, verb)
  p.fmtString(v.String(), verb)
  return
  }
 } }  return false }
```
可以看出当是error接口时时，调用error接口的Error()方法
当是结构体时，调用Stringer接口的String()方法