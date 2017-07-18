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


# <a name="1.1"/>1.1 Module
Modules are fundamental components of TurboScript. each TurboScript module can be split in to different wasm modules or combine everything to single wasm module.

compiler option for this feature is `--bundle=true` or `-b=true` default value is `true`

```typescript
module main {} -> main.wasm
module sevenzip {} -> sevenzip.wasm
```

```typescript
import {SevenZipReader} from "sevenzip"
module main {
    
    var sevenZipReader:SevenZipReader;

    start function entry():void{
        sevenZipReader = new SevenZipReader();
    }

}
```

```typescript
module sevenzip {
    
    export class SevenZipReader {
        constructor(){

        }
    }

}
```

# <a name="1.2"/>1.2 Declaration

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
