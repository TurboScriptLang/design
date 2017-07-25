# TurboScript Language Specification

Version 1.0

July, 2017

This Specification available under the Open Web Foundation Final Specification Agreement Version 1.0 ("OWF 1.0") as of October 1, 2012. The OWF 1.0 is available at http://www.openwebfoundation.org/legal/the-owf-1-0-agreements/owfa-1-0.

## Table of Contents

* [1 Introduction](#1)


# <a name="1"/>1 Introduction

TurboScript designed to fill missing featrures of JavaScript which offered by WebAssembly. TurboScript's primary goals is to leverage features of WebAssembly while progamming in JavaScript like language. This specification provide a complete picture of what TurboScript is and not. There is no intention to replace JavaScript or TypeScript with TurboScript. Initial version of language will concentrate on parallel programming and SIMD features targeting post-MVP WebAssembly. Since language mainly targeting parallel programming features, integration with WebGL Next and WebGPU are in radar. 

TurboScript intented to work closely with JavaScript therefore JavaScript is an integral part of TurboScript. TurboScript project can be composed of JavaScript and WebAssembly. Since JavaScript is a dyanamic language JavaScript modules in TurboScript can be without types but any intraction with TurboScript modules need to be strongly typed. That is any function exporting from JavaScript module need to be strongly typed. 

There are some difference between concept of JavaScript and TurboScript. 1) variables and members cannot be undefined therefore no `undefined` type. default value will be `null`. 2) TurboScript doesnot support closure functions. 3) TurboScript's runtime memory is managed by programmer, ie. there is no garbage collector in TurboScript. 

# <a name="2"/>2 Basic Concepts

TurboScript stands for hassel free parallel programming in web using JavaScript dialect with additional type information on top of JavaScript syntax and parallel programming features like shared memory, context sharing workers, SIMD, channels etc... TurboScript may look like TypeScript but it is not TypeScript.

# <a name="3"/>3 Module
Modules are fundamental components of TurboScript. Each TurboScript file is a module and modules can be combined to single wasm module. Worker modules always compiled to separate .wasm modules.

```typescript
// main.tbs
import {SevenZipReader} from "sevenzip"
var sevenZipReader:SevenZipReader;

start function main():void{
    sevenZipReader = new SevenZipReader();
}
```

```typescript
// sevenzip.tbs
export class SevenZipReader {
    constructor(){

    }
}
```

# <a name="3.1">3.1 Import Declarations
Import declarations are used to import entities from other modules and provide bindings for them in the current module.

An import declaration of the form
```typescript
import * as awesome from "./components/awesome-component"
```
imports the module with the given name and creates a local binding for the module itself. The local binding is classified as a value (representing the module instance) and a namespace (representing a container of types and namespaces).

An import declaration of the form
```typescript
import {_funcName, _className, _varName} from "./module"
```
imports a given module and creates local bindings for a specified list of exported members of the module. The specified names must each reference an entity in the export member set of the given module. The local bindings have the same names and classifications as the entities they represent unless as clauses are used to that specify different local names:
```typescript
import {_funcName as a, _className as b, _varName as c} from "./module"
```

An import declaration of the form
```typescript
import awesome from "./components/awesome-component"
```
is exactly equivalent to the import declaration
```typescript
import {default as awesome} from "./components/awesome-component"
```

# <a name="1.2"/>1.2 Declarations

```typescript
declare module console {
    function log(value:int32):void
}
```
Above declaration will import `{ console: { log: value => {} } }` to WebAssembly
implementation of the import is up to the user.

```typescript
declare var PI:float64 = 3.141592653589793;
var twoPI:float64 = 2.0 * PI;
```

# <a name="1.x">1.x Numeric types

Type      | Alias | Description
----------|-------------|-------------
`int8`   | sbyte         | signed  8-bit integers (-128 to 127)
`uint8`    | byte         | unsigned  8-bit integers (0 to 255)
`int16`   | short         | signed 16-bit integers (-32768 to 32767)
`uint16`  | ushort         | unsigned 16-bit integers (0 to 65535)
`int32`     | int         | signed 32-bit integers (-2147483648 to 2147483647)
`uint32`    | uint         | unsigned 32-bit integers (0 to 4294967295)
`int64`    | long         | signed 64-bit integers (-9223372036854775808 to 9223372036854775807)
`uint64`   | ulong         | unsigned 64-bit integers (0 to 18446744073709551615)
`float32`   | float         | IEEE-754 32-bit floating-point numbers
`float64`  | double         | IEEE-754 64-bit floating-point numbers

# <a name="1.x">1.x Numeric literals

```typescript
// Integers literals
let _uint = 1; //default number type is uint32
let _int = -1; //default negative number type is int32
let _int64 = 1l //number+l represent 64bit integers
let _int64 = -1l //-number+l represent 64bit unsigned integers

//Floating point literals
let _float64 = 1.0 //default floating point number type is float64
let _float32 = 1.0f //number.fraction+f represent 32bit floating point numbers
```

# <a name="1.x"/>1.x Interface

An interface specifies properties and method set of an unknown implementation. implementations can be completely isolated, invoking unimplemented methods or properties raise compiler time error.

```typescript
interface Robot {
    name:string
    id:uint32
    serialNumber:uint32

    wake()
    eat()
    sleep()
    walk()
    run()
    reproduce()
    die()
}
```
# <a name="1.x"/>1.x Class

##### Example 1
```typescript
class GenericRobot {
    
    name = "GENERIC_ROBOT"
    id = 1

    constructor(public serialNumber:uint32){
    }

    wake(){/*implementation*/}
    eat(){/*implementation*/}
    sleep(){/*implementation*/}
    walk(){/*implementation*/}
    run(){/*implementation*/}
    reproduce(){/*implementation*/}
    die(){/*implementation*/}
}

class Dog extends GenericRobot {
    name = "DOG"
    bark(){/*implementation*/}
}

class Bird extends GenericRobot {
    name = "BIRD"
    fly(){/*implementation*/}
}

function main() {
    let robots = [
        new GenericRobot(0),
        new Dog(1),
        new Bird(2),
    ]

    robots.forEach(robot:Robot => {
        robot.wake();
        console.log(robot.name);
        console.log(robot.id);
        console.log(robot.serialNumber);
        robot.sleep();
    });
}
```

##### Example 2
```typescript
//common.wasm
class Ray {
    origin:Vector3D
    direction:Vector3D
}
interface Shape {
    bbox:Box
    intersect(r:Ray):HitInfo
}
```
```typescript
//intersectable.wasm
class Triangle {
    bbox:Box
    points:Vector3D[3]
    normals:Vector3D[3]
    textureCoord:Vector2D[3]
    intersect(r:Ray):HitInfo{/*implementation*/}
}
class Cube {
    bbox:Box
    min:Vector3D
    max:Vector3D
    intersect(r:Ray):HitInfo{/*implementation*/}
}
class Sphere {
    bbox:Box
    radius:float64
    center:Vector3D
    intersect(r:Ray):HitInfo{/*implementation*/}
}
```

```typescript
//intersector.wasm
function intersect(thing:Shape, ray:Ray):HitInfo {
    return thing.intersect(ray)
}
```

```typescript
//main.wasm
declare class Triangle {}
declare function intersect(thing:Shape, ray:Ray):HitInfo

function main() {
    let shapes:Shape[] = [
        new Triangle,
        new Sphere,
        new Cube
    ]
    let ray = new Ray()
    shapes.forEach(shape => intersect(shape, r))
}
```

# <a name="1.x"/>1.x Generic Types and Functions

```typescript
class Foo<T> {
    value:T;
    constructor(value:T){
        this.value = value;
    }
    getValue():T {
        return this.value;
    }
}

export function testI32(value:int32):int32 {
    let instance = new Foo<int32>(value);
    return instance.getValue();
}

export function testI64(value:int32):int32 {
    let value2 = value as int64;
    let instance = new Foo<int64>(value2);
    return instance.getValue() as int32;
}

export function testF32(value:float32):float32 {
    let instance = new Foo<float32>(value);
    return instance.getValue();
}

export function testF64(value:float64):float64 {
    let instance = new Foo<float64>(value);
    return instance.getValue();
}
```

# <a name="1.x"/>1.x SIMD (128bit)
```typescript
let int8_x16:simd128<int8>;
let int16_x8:simd128<int16>;
let int32_x4:simd128<int32>;
let int64_x2:simd128<int64>;
let float32_x4:simd128<float32>;
let float64_x2:simd128<float64>;
```

```typescript
class RGBA_x2 {
    constructor(
        public r:simd128<float64> = new simd128<float64>()
        public g:simd128<float64> = new simd128<float64>()
        public b:simd128<float64> = new simd128<float64>()
        public a:simd128<float64> = new simd128<float64>()){       
    }
}
let rgba = new RGBA_x2();
console.log(rgba)   // {r:[0.0, 0.0], g:[0.0, 0.0], b:[0.0, 0.0], a:[0.0, 0.0]}

class RGBA32_x4 {
    constructor(
        public r:simd128<float32> = new simd128<float32>()
        public g:simd128<float32> = new simd128<float32>()
        public b:simd128<float32> = new simd128<float32>()
        public a:simd128<float32> = new simd128<float32>()){       
    }
}
let rgba = new RGBA32_x4();
console.log(rgba)   // {r:[0.0, 0.0, 0.0, 0.0], g:[0.0, 0.0, 0.0, 0.0], b:[0.0, 0.0, 0.0, 0.0], a:[0.0, 0.0, 0.0, 0.0]}
```

# <a name="1.x"/>1.x Channel
```typescript
class RGBA {
    constructor(
        public r:float64,
        public g:float64,
        public b:float64,
        public a:float64){       
    }
}
channel color:RGBA4 = new channel<RGBA>
let rgba = new RGBA(1.0, 1.0, 1.0, 1.0);
console.log(rgba)
color <- rgba
console.log(rgba)
rgba <- color
console.log(rgba)
```

# <a name="1.x"/>1.x Worker

```typescript
worker {
    export function filterKernel(in:float64[], out:float64[], start:int32, end:int32) {
        for(let i:int32 = start; i < end; i++){
            out[i] = in[i];
        }
    }
}

async function applyFilter(in:RGBA[]): RGBA[] {
    let length = in.length;
    let numCpu = process.NUM_CPU;
    let blockSize = length / process.NUM_CPU;
    let out = new RGBA[](length);
    for(let i = 0; i < numCpu; i++) {
        let start = i * blockSize;
        worker filterKernel(in, out, start, start + blockSize);
    }
    return out;
}

class RGBA {
    constructor(
        public r:float64 = 0.0,
        public g:float64 = 0.0,
        public b:float64 = 0.0,
        public a:float64 = 0.0){
    }
}

async function main() {
    let i = 0;
    let colors = new RGBA[](1024 * 1024);
    let result = await applyFilter(colors);
}


```
