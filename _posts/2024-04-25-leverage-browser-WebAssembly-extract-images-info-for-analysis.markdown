---
layout: post
title:  "Leverage Browser WebAssembly to Extract Images Info for Analysis"
date:   2024-04-25 14:30:10 -0500
categories: Engineering
---
In recent years, the web development landscape has been revolutionized by the introduction of WebAssembly (Wasm), a binary instruction format designed to run code efficiently on any modern browser. This technology brings near-native performance to web applications,
enabling developers to leverage languages like C, C++, and Rust for their web-based projects. One of the key tools facilitating the use of C++ in web development is Emscripten, a powerful LLVM-based compiler that transforms C/C++ code into WebAssembly.

# 1. What is WebAssembly?

WebAssembly is a low-level, binary format designed for safe and efficient execution across different platforms. It is designed to be a portable compilation target for high-level languages, allowing code written in these languages to run on the web at near-native speeds. WebAssembly is:

- Fast: It offers performance close to native execution.
- Safe: It enforces strong security measures, ensuring safe execution.
- Portable: It can run on any platform with a WebAssembly runtime, including all modern web browsers.

# 2. Emscripten
Emscripten is a complete compiler toolchain that takes C/C++ source code and compiles it to WebAssembly. It simplifies the process of porting existing C/C++ codebases to the web, making it possible to reuse existing libraries and applications in a web environment.

Emscripten provides several benefits:

- Broad Compatibility: Supports most C/C++ codebases.
- Comprehensive Libraries: Includes many commonly used libraries, such as SDL, OpenGL, and POSIX.
- High Performance: Produces highly optimized WebAssembly code.
- Integration with JavaScript: Allows seamless integration with JavaScript, enabling interaction between WebAssembly modules and JavaScript code.

# 3. Example: Extracting Image Info with Emscripten & C++, and Send to Backend API
Let's dive into an example of using Emscripten to extract metadata from an image file using C++ and compile it to WebAssembly. We'll use the stb_image library to handle image loading in C++.

### Step 1: Setup Your Environment
First, ensure you have Emscripten installed. Follow the instructions on the [Emscripten website](https://emscripten.org/docs/getting_started/index.html) to set up.

### Step 2: Write C++ Code to Extract Image Metadata and Post to Backend Data API
```cpp
// image_metadata.cpp
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
#include <string>
#include <iostream>

#include <emscripten/fetch.h>

extern "C" {
    struct ImageMetadata {
        int width;
        int height;
        int channels;
    };

    ImageMetadata getImageMetadata(const char* imagePath) {
        ImageMetadata metadata;
        int width, height, channels;
        unsigned char* img = stbi_load(imagePath, &width, &height, &channels, 0);
        if (img) {
            metadata.width = width;
            metadata.height = height;
            metadata.channels = channels;
            stbi_image_free(img);
        } else {
            metadata.width = metadata.height = metadata.channels = -1;
        }
        return metadata;
    }
}

bool PostAPI(const std::string url,
             const std::string &image_name,
             ImageMetadata &image_data) {
  emscripten_fetch_attr_t attr;
  emscripten_fetch_attr_init(&attr);
  strcpy(attr.requestMethod, "POST");
  attr.attributes = EMSCRIPTEN_FETCH_SYNCHRONOUS | EMSCRIPTEN_FETCH_LOAD_TO_MEMORY | EMSCRIPTEN_FETCH_REPLACE;

  const char * headers[] = {"Content-Type", "application/x-www-form-urlencoded",
                            0};
  attr.requestHeaders = headers;

  std::string data = "image_name=" + image_name +
                     "&image_w=" + std::to_string(image_data.width) +
                     "&image_h=" + std::to_string(image_data.height) +
                     "&image_c=" + std::to_string(image_data.channels);
  attr.requestData = data.c_str();
  attr.requestDataSize = strlen(attr.requestData);

  emscripten_fetch_t *fetch = emscripten_fetch(&attr, url.c_str());
  return fetch->status == 200; 
}

int main() {
    # uploaded by web UI
    const char* image_name = "example.jpg";
    ImageMetadata metadata = getImageMetadata(image_name);
    if (metadata.width != -1) {
        std::cout << "Width: " << metadata.width << "\n";
        std::cout << "Height: " << metadata.height << "\n";
        std::cout << "Channels: " << metadata.channels << "\n";
    } else {
        std::cout << "Failed to load image.\n";
        return -1;
    }
    
    bool res = PostAPI("http://127.0.0.1/api/analysis", image_name, metadata);
    if (res) return 0;
    return -1;
}
```

### Step 3: Compile C++ Code to WebAssembly
```bash
em++ image_metadata.cpp -sFETCH --proxy-to-worker -o image_metadata.html
```

# 4. Conclusion
WebAssembly and Emscripten provide a powerful combination for bringing high-performance C++ code to the web. By leveraging these technologies, developers can reuse existing C++ libraries and codebases, unlocking new possibilities for web applications. 
