# Pointers

## Вопрос
нужно ли использовать для структуры во всех методах либо только указатели, либо только копии объектов, не перемешивая?
https://www.reddit.com/r/golang/comments/10yrpic/why_not_mix_value_and_pointer_receivers/?rdt=35963
- например:
	```go
	type A struct {
		param string
		b     *stirng
	}
	
	func (a *A) setParam() { a.param = "test" }
	func (a A) getParam() { return a.param } 
	// or func (a *A) getParam() { return a.param } ???
	```
	для метода `getParam` можно было бы использовать `(a *A)` вместо копии `(a A)`,  так как `setParam` уже использует указатель, но так как метод ничего не изменяет, используем копию (указатель не нужен),
	или  надо все-таки использовать указатель для согласованности?  

## Рекомендации
наиболее **полные рекомендации по использованию указателей** для методов структур:
 https://stackoverflow.com/a/27775558/4791642
- If the receiver is a map, func or chan, don't use a pointer to it.
- If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it.
- If the method needs to mutate the receiver, the receiver must be a pointer.
- If the receiver is a struct that contains a `sync.Mutex` or similar synchronizing field, the receiver must be a pointer to avoid copying.
- If the receiver is a large struct or array, a pointer receiver is more efficient. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver.
- Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer.
- If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention more clear to the reader.
- **If the receiver is a small array or struct that is naturally a value type** (for instance, something like the `time.Time` type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, **a value receiver makes sense**.  
	**A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap.** (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.
- Finally, when in doubt, use a pointer receiver.

## Теория

- Набор методов у объекта типа `A` состоит из методов с получателем `A`: `(a A)`. 
	- Из примера выше для `var a A` набор методов будет включать только `getParam` 
- Набор методов у объекта типа `*A` состоит из методов типа `A` и `*A`.
	- Из примера выше для `var a *A` набор методов будет включать и `setParam`, и `getParam` 
- Особенность языка: для `a := &A{}` можно писать `a.getParam()`, несмотря на то, что метод принимаем не указатель, а копию объекта. [Go Tour. Methods and pointer indirection (2)](https://go.dev/tour/methods/7)
	- Реально строка `a.getParam()` интерпретируется как `(*a).getParam()`
- При вызове, для `A` в метод реально будет передаваться не объект (как в случае с `*A`), а его копия, что не позволит изменять исходный объект.
	- поэтому для методов, в которых нужно **запретить изменения исходного объекта используют именно копии**, а не указатели (=ссылки на исходный объект)
	- при этом если в объекте есть параметры со ссылками на объект (в примере это параметр `b`), то в копии объекта типа `A` будет тоже сслыка (=копия адреса параметра `b` из примера). 
		- то есть `func (a A) getParam() { a.b = "564"; return a.param }` реально изменил бы значение в исходном объекте `a`, а не в копии
		- то же самое касается некоторых других типов, например `map`, для которых реально хранится адрес объекта, а не сам объект
- копирование объекта тоже операция, занимающее некоторое время и ресурсы, поэтому **для больших объектов рекомендуется использовать указатель**, даже если метод не меняет объект.
	- это сократит время и используемые ресурсы, так как передать указатель быстрее, чем выделять память и копировать объект
- **для маленьких объектов лучше отказаться от указателей** там, где указатели не нужны
	- https://stackoverflow.com/questions/27775376/value-receiver-vs-pointer-receiver
	> **docs** also say "For types such as basic types, slices, and small structs, a value receiver is very cheap so unless the semantics of the method requires a pointer, a value receiver is efficient and clear."
	
	- копирование небольших объектов оптимизировано - выполнится достаточно быстро, 
	- при этом получим  лучшую производительность, так как при передаче указателя методу при работе с указателем постоянно придется обращаться к адресу объекта, который в памяти находится значительно дальше, чем при копировании в память, выделенную специально для этого метода.
- см. подробнее в доках  [Why do T and *T have different method sets?](https://go.dev/doc/faq#different_method_sets)
	- некоторые делают вывод из этих доков, что если хоть один метод принимает указатель, то и остальные должны тоже, хотя в доках этого не сказано - там лишь объясняется, почему наборы методов `T` и `*T` отличаются. 
	- информация в доках не противоречит тому, что для небольших структур **не надо использовать указатели** там, где они не нужны
 - доп. источники 
	https://go.dev/tour/methods/7
	[https://go.dev/doc/faq#different_method_sets](https://go.dev/doc/faq#different_method_sets "https://go.dev/doc/faq#different_method_sets")
	[https://stackoverflow.com/questions/27775376/value-receiver-vs-pointer-receiver](https://stackoverflow.com/questions/27775376/value-receiver-vs-pointer-receiver "https://stackoverflow.com/questions/27775376/value-receiver-vs-pointer-receiver")	[https://www.reddit.com/r/golang/comments/10yrpic/why_not_mix_value_and_pointer_receivers/](https://www.reddit.com/r/golang/comments/10yrpic/why_not_mix_value_and_pointer_receivers/ "https://www.reddit.com/r/golang/comments/10yrpic/why_not_mix_value_and_pointer_receivers/")

## Квест
сложный пример с использованием интерфейсов и методов, возвращающих функции, где тем не менее можно отказаться от указателей в методах (запускается как unit-test)

```go
package main

import (
	"fmt"
	"log"
	"testing"
)

type T struct {
	i  int
	s  string
	m  map[*string]any
	c  chan int
	sp *string

	savedF  func(initialTp *T) bool
	savedFp func(initialTp *T) bool
}

func NewT(i int, s string, m map[*string]any, c chan int, sp *string) *T {
	t := &T{}

	log.Println("NewT")
	t.savedF = t.f(nil) // -> t.m is not yet initialized, so it will be empty on any call of t.savedF
	t.savedFp = t.fp(nil)

	t.m = m // -> initial map pointer is changed after t.savedF and t.savedFp got
	t.i = i
	t.s = s
	t.c = c
	t.sp = sp
	newString := "new"
	t.m[&newString] = nil
	return t
}

func (t T) f(initialTp *T) func(initialTp *T) bool {
	compareWithInitialValue("value method", initialTp, &t)
	// log.Printf("value method: %d, %s, %v, %v, %v", t.i, t.s, t.m, cap(t.c), t.sp)
	// log.Printf("%v, %v, %v, %v, %v", &t.i, &t.s, &t.m, &t.c, &t.sp)
	// log.Printf("%v", &t)
	return func(initialTp *T) bool {
		compareWithInitialValue("value method", initialTp, &t)
		// log.Printf("inner value method: %d, %s, %v, %v, %v", t.i, t.s, t.m, cap(t.c), t.sp)
		// log.Printf("%v, %v, %v, %v, %v", &t.i, &t.s, &t.m, &t.c, &t.sp)
		// log.Printf("%v", &t)
		return true
	}
}

func (t *T) fp(initialTp *T) func(initialTp *T) bool {
	compareWithInitialValue("pointer method", initialTp, t)
	// log.Printf("pointer method: %d, %s, %v, %v, %v", t.i, t.s, t.m, cap(t.c), t.sp)
	// log.Printf("%v, %v, %v, %v, %v", &t.i, &t.s, &t.m, &t.c, &t.sp)
	// log.Printf("%v", &t)

	return func(initialTp *T) bool {
		compareWithInitialValue("pointer method", initialTp, t)
		// log.Printf("inner pointer method: %d, %s, %v, %v, %v", t.i, t.s, t.m, cap(t.c), t.sp)
		// log.Printf("%v, %v, %v, %v, %v", &t.i, &t.s, &t.m, &t.c, &t.sp)
		// log.Printf("%v", &t)
		return true
	}
}

type I interface {
	f(initialTp *T) func(initialTp *T) bool
	fp(initialTp *T) func(initialTp *T) bool
}

type TWithI struct {
	Interf I
}

func Test_value_vs_pointer_structs(t *testing.T) {
	s := "test"
	initialTp := NewT(5, "test", make(map[*string]any), make(chan int, 5), &s)
	log.Printf("=== source value: %v, %v, %v, %v, %v", &initialTp.i, &initialTp.s, &initialTp.m, &initialTp.c, &initialTp.sp)
	var interf I = initialTp

	testInterface(TWithI{Interf: interf}, initialTp)

	log.Println("=== INITIAL inner funcs ===")
	initialTp.savedF(initialTp) // -> without pointer map is empty, because map was initialized after savedF inited
	initialTp.savedFp(initialTp)
	// values of inner funcs are not equal to inital values because inner funcs were inited before other struct params
}

func testInterface(ti TWithI, initialTp *T) {

	log.Println("=== Interface funcs ===")
	savedF := ti.Interf.f(initialTp)
	savedFp := ti.Interf.fp(initialTp)

	log.Println("=== NEW inner funcs ===")
	savedF(initialTp) // -> map is not empty, because map already initialized when saveF inited
	savedFp(initialTp)
}

func compareWithInitialValue(caseName string, initialTp *T, gotT *T) {
	// log.Printf("initial: %d, %s, %v, %v, %v", t.i, t.s, t.m, cap(t.c), t.sp)
	// log.Printf("got: %d, %s, %v, %v, %v", t.i, t.s, t.m, cap(t.c), t.sp)
	if initialTp == nil {
		return
	}
	compare := func(paramName string, equal bool) string {
		return fmt.Sprintf("%s: %v,\t ", paramName, equal)
	}

	result := caseName + ". Addresses are equal? "
	result += compare("int", &(initialTp.i) == &(gotT.i))
	result += compare("string", &(initialTp.s) == &(gotT.s))
	result += compare("map", &(initialTp.m) == &(gotT.m))
	result += compare("chan", &(initialTp.c) == &(gotT.c))
	result += compare("string ptr", &(initialTp.sp) == &(gotT.sp))
	log.Println(result)

	result = caseName + ". Values are equal? "
	result += compare("int", initialTp.i == gotT.i)
	result += compare("string", initialTp.s == gotT.s)
	result += compare("map", len(initialTp.m) == len(gotT.m))
	result += compare("chan", cap(initialTp.c) == cap(gotT.c))
	result += compare("string ptr", initialTp.sp == gotT.sp)
	log.Println(result + "\n")
}

```
