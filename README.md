# typescript-ignore
Every time I'm frustrated with TypeScript i'll to explain the limitation here

## Callable class
I was able to create a function that makes a class callable:

```typescript
// Because typeof class === "function"
interface AnyClass extends Function {
   new (...args: any[]): any
}

function CallableClass (func: Function, cls: AnyClass) {
   return new Proxy(cls, {
      apply (_target: typeof cls, thisArg: any, argArray: any[]) {
         func.apply(thisArg, argArray)
      }
   })
}
```

However, when I tried using it, it says that ```Value of type AnyClass is not callable```

```typescript
const test = (CallableClass(() => 7, class _ {}))()
```

It seems that TypeScript thinks the ```new Proxy``` is still just a regular AnyClass.

I typed the return value, but although that got a different error -  
```Type 'AnyClass' is not assignable to type 'AnyCallableClass'.``` - TypeScript still thinks ```new Proxy``` is AnyClass.

```typescript
// Return value type
interface AnyCallableClass extends AnyClass {
   (...args: any[]): any
}
```

I found https://github.com/microsoft/TypeScript/issues/183, but the pull request that closed it only allows for defining callable classes whose name is already known.

### Fix

Oh wait, it turns out that you can just force TypeScript to accept a type: https://basarat.gitbook.io/typescript/type-system/type-assertion

```typescript
// Because typeof class === "function"
interface AnyClass extends Function {
   new (...args: any[]): any
}

interface AnyCallableClass extends AnyClass {
   (...args: any[]): any
}

function CallableClass (func: Function, cls: AnyClass) {
   return new Proxy(cls, {
      apply (_target: typeof cls, thisArg: any, argArray: any[]) {
         func.apply(thisArg, argArray)
      }
   }) as AnyCallableClass // finally it works
}

const test = (CallableClass(() => 7, class _ {}))()
```

## Return type as the return type of another function

```typescript
// Because typeof class === "function"
interface AnyClass extends Function {
   new (...args: any[]): any
}

interface AnyCallableClass extends AnyClass {
   (...args: any[]): any
}

// But where the return type === ```func``` return type
function CallableClass (func: Function, cls: AnyClass): AnyCallableClass {
   return new Proxy(cls, {
      apply (_target: typeof cls, thisArg: any, argArray: any[]) {
         func.apply(thisArg, argArray)
      }
   }) as AnyCallableClass & Args<func> => Return<func>
}

const test = (CallableClass(() => 7, class _ {}))()
```

### Fix

This is possible, actually.

```typescript
// The return type is now (typeof func & typeof cls) because it gives more information
function CallableClass <FuncReturn> (func: (...FuncArgs: any[]) => FuncReturn, cls: AnyClass): typeof func & typeof cls {
   return new Proxy(cls, {
      apply (_target: typeof cls, thisArg: any, argArray: any[]) {
         return func.apply(thisArg, argArray)
      }
   }) as typeof func & typeof cls
}
```

### Edit

Oh, it actually exists: https://www.typescriptlang.org/docs/handbook/utility-types.html#returntypetype
