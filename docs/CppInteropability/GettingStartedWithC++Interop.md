
## General Feedback
Most of it makes sense to me in a general sense, but there are some parts I am not clear on, and it may just be due to my lack of C++ experience, so feel free to discard any feedback or suggested changes as you see appropriate
- I think that more screenshots would be helpful (I've noted the spots I think would be most helpful)
- I would suggest trying to maintain consistency in using `code` notation in your text and the bullet point/non-bullet points
- I would suggest using additional levels on your bullet points to help differentiate steps vs details


#  Getting started with C++ Interoperability

This document is designed to get you started with bidirectional API-level interoperability between Swift and C++.

## Table of Contents

- [Creating a Module to contain your C++ source code](#creating-a-module-to-contain-your-c-source-code)
- [Adding C++ to an Xcode project](#adding-c-to-an-xcode-project)
- [Creating a Swift Package](#Creating-a-Swift-Package)
- [Building with CMake](#building-with-cmake)

## Creating a Module to contain your C++ source code

- Create your C++ implementation and header files
  - In this example we are naming our `module` Cxx, but this can be whatever you choose
  - Our implementation and header files will be named `Cxx.cpp` and `Cxx.hpp` respectively
- Next create an empty file named `module.modulemap`, 
  - Inside this file create the `module` for your source code, and 
  - Define your C++ `header` 
  - Note: `requires cplusplus` isn't required but it is convention for a C++ `module`, especially if the `module` uses C++ features

```
//In module.modulemap
module Cxx {
    //note that your header should be the file that containts your method implementations
    header "Cxx.hpp"
    requires cplusplus
}
```

- Move the newly created files (`Cxx.cpp`, `Cxx.hpp`, `module.modulemap`) into a separate directory
  - This directory should remain in your project directory 
  - // Where should it go? At the root level? Top level inside project? Possibly include a screenshot showing the C++ directory in relation to the project directory? //

<img width="252" alt="Screen Shot 2022-02-26 at 9 14 06 PM" src="https://user-images.githubusercontent.com/62521716/155867937-9d9d6c62-4418-414d-bc4e-5d12c2055022.png">

## Adding C++ to an Xcode project
After [Creating a Module to contain your C++ source code](#creating-a-module-to-contain-your-c-source-code) in your project directory:

- Add the C++ module to the `Build Settings` to enable C++ interop
  - Navigate to your project directory 
  - In `Project` navigate to `Build Settings` -> `Swift Compiler`
  - Under `Custom Flags` -> `Other Swift Flags` add `-Xfrontend -enable-cxx-interop`
  - Under `Search Paths` -> `Import Paths` add your search path to the C++ `module` (i.e, `./ProjectName/Cxx`).
  - // A screenshot showing these nested settings/navigation would be huge, especially with yellow boxes around the specific new setting or arrows or something //

```
//Add to Other Swift Flags and Import Paths respectively
-Xfrontend -enable-cxx-interop 
-I./ProjectName/Cxx // Should the "I" be here? If so, what is it for? Possibley provide a brief explanation if it is out of scope //
```

- You should now be able to import your C++ `module` into any `.swift` file in your project // Should "Module" be capitalized? Some places it is, some places not//

**Importing our `module` and using the `cxxFunction` from our .swift file**

// Possibly add captions as to what you are demo-ing in these example snippets //
```
//In ViewController.swift
import UIKit
import Cxx

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let result = cxxFunction(7)
        print(result)
    }
}
```

**Defining our `cxxFunction` method in our implementation file**
```
//In Cxx.cpp

#include "Cxx.hpp"
int cxxFunction(int n) {
    return n;
}

```

**Exposing our `cxxFunction` method in our header file**
```
//In Cxx.hpp

#ifndef Cxx_hpp
#define Cxx_hpp

int cxxFunction(int n);

#endif

```


## Creating a Swift Package
- Create a Swift Package project in Xcode // Should "project" be capitalized? Is there a link to an Apple doc on how to create a Swift Package that you could direct people to? //
  - In this example, we named the package `Cxx_Interop`
- Add your C++ `module` directory to the `Source` directory in your project // Screenshot would be great showing file structure relationship //
  - C++ source code follows the example shown in [Creating a Module to contain your C++ source code](#creating-a-module-to-contain-your-c-source-code)
- Configure the Swift target dependencies and compiler flags in the project `Package Manifest`
- Create a Swift file in your C++ `module` directory to contain your Swift code
  - In the example we name this file `main.swift` and it is located in the `Sources/Cxx_Interop` directory // Another screenshot //
- Add yopur C++ `module` to the appropriate targets
  - Navigate to `Targets` // where is this? //
  - Select the `Add` button
  - Add your C++ module
  - Add the directory containing the Swift code // I have no idea the actual steps, just giving an example to replace the next bullet //
- Under targets, add the name of your C++ module and the directory containing the Swift code as a target. // More explicit step by step instructions would be clearer, especially with screenshots with highlighting, for this bullet and the next //
- In the target defining your Swift target, add a`dependencies` to the C++ Module, the `path`, `source`, and `swiftSettings` with `unsafeFlags` with the source to the C++ Module, and enable `-enable-cxx-interop` // The wording on this is a little confusing //
- Add C++ `module` and underlying targets and dependencies to the `Package Manifest`, including:
  - C++ `module` `dependencies`
  - `path`
  - `sources` 
  - `swiftSettings` with `unsafeFlags` to enable `cxx-interop`

**Your `Package Manifest` should look like the following, if you have been following the example**
```
//In Package Manifest

import PackageDescription

let package = Package(
    name: "Cxx_Interop",
    platforms: [.macOS(.v12)],
    products: [
        .library(
            name: "Cxx",
            targets: ["Cxx"]),
        .library(
            name: "Cxx_Interop",
            targets: ["Cxx_Interop"]),
    ],
    targets: [
        .target(
            name: "Cxx",
            dependencies: []
        ),
        .executableTarget(
            name: "Cxx_Interop",
            dependencies: ["Cxx"],
            path: "./Sources/Cxx_Interop",
            sources: [ "main.swift" ],
            swiftSettings: [.unsafeFlags([
                "-I", "Sources/Cxx",
                "-Xfrontend", "-enable-cxx-interop",
            ])]
        ),
    ]
)

```

- You should now be able to import your C++ Module into your Swift code, and import the package into existing projects

**Using the Cxx `module` in a Swift Struct**
```
//In main.swift

import Cxx

public struct Cxx_Interop {
    
    public func callCxxFunction(n: Int32) -> Int32 {
        return cxxFunction(n: n)
    }
}

print(Cxx_Interop().callCxxFunction(n: 7))
//outputs: 7

```

## Building with CMake
After creating your project follow the steps [Creating a Module to contain your C++ source code](#creating-a-module-to-contain-your-c-source-code) // Is this the same as the first bullet point - create a CMakeList.txt file?

- Create a `CMakeLists.txt` file and configure for your project
- In `add_library` invoke `cxx-support` with the path to the C++ implementation file
- Add the `target_include_directories` with `cxx-support` and `module` path to the C++ Module `${CMAKE_SOURCE_DIR}/Sources/Cxx`
- Add the `add_executable` to the specific files/directory you would like to generate source, with`SHELL:-Xfrontend -enable-cxx-interop`

**Example CMakeLists.txt file, using the same file structure used in [Creating a Swift Package](#Creating-a-Swift-Package)**
```
//In CMakeLists.txt

cmake_minimum_required(VERSION 3.18)

project(Cxx_Interop LANGUAGES CXX Swift)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS OFF)

add_library(cxx-support ./Sources/Cxx/Cxx.cpp)
target_compile_options(cxx-support PRIVATE
  -I${SWIFT_CXX_TOOLCHAIN}/usr/include/c++/v1
  -fno-exceptions
  -fignore-exceptions
  -nostdinc++)
target_include_directories(cxx-support PUBLIC
  ${CMAKE_SOURCE_DIR}/Sources/Cxx)

add_executable(Cxx_Interop ./Sources/Cxx_Interop/main.swift)
target_compile_options(Cxx_Interop PRIVATE
  "SHELL:-Xfrontend -enable-cxx-interop"
target_link_libraries(Cxx_Interop PRIVATE cxx-support)

```

```
//In main.swift

import Cxx

public struct Cxx_Interop {
    public static func main() {
        let result = cxxFunction(7)
        print(result)
    }
}

Cxx_Interop.main()

```

- In your projects directory, run `cmake` to generate the systems build files // Is this being done in terminal? If so, I would explicitly state that //

- To generate an Xcode project run `cmake -GXcode` 
- To generate with Ninja run `cmake -GNinja`

- For more information on `cmake` see the  'GettingStarted' documentation: (https://github.com/apple/swift/blob/main/docs/HowToGuides/GettingStarted.md)


