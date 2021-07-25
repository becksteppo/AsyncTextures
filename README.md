# Asynchronous Textures

Enables loading (and uploading to GPU) textures asynchronously using compute shaders.

## Quickstart

Here's how you could create a texture:

```c#
var httpClient = new HttpClient();
var image = await httpClient.GetStreamAsync("https://picsum.photos/2048");
var texture = await AsyncTextureLoader.Instance.LoadTextureAsync(image);
```

## Usage

See [Quickstart](#quickstart) for basic usage. The platform has to support compute shaders for this to work.

### Advanced Usage

Take a look at the implementation of `LoadTextureAsync()` to get a good overview of the inner workings:

```c#
var image = await DecodeImageAsync(input);
var texture = await AcquireTextureAsync(image.Width, image.Height);
await UploadDataAsync(texture, 0, 0, image.Width, image.Height, 0, image.Data);
return texture;
```

As you can see, if you have a `RenderTexture`, you can also update only part of it if you want. This could, for example,
be used to create and manage a texture atlas.

### Changing the Decoder Implementation

The package comes with a C# port of the well-known [stb_image.h](https://github.com/nothings/stb) header
file. ([StbImageSharp](https://github.com/StbSharp/StbImageSharp))

While it's good enough in most cases, you may want to replace this with a more optimized variant for your specific use
case.

To do so, simply implement `IImageDecoder` and hook it up:

```c#
AsyncTextureLoader.Instance.ImageDecoder = new MyImageDecoder();
```

Naturally, this step is superfluous if you use `AsyncTextureLoader.UploadDataAsync` directly.

**Note that the class currently expects the data layout to be RGBA32.**

## Background

As of the time of this writing, Unity only allows asynchronously uploading textures to the GPU when they have been
compiled with game. If, however, you need to dynamically load textures at runtime, you're kind of left in a bind.

There are basically two approaches to solve this (as far as I know):

1. Manage the texture using native plugins.
2. Manage the texture using compute shaders.

The preferred approach would probably be #1, but it is also harder to manage because you have to cater for all supported
platforms.

This package tries to implement approach #2.