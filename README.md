# SDL on Zig

This is a fork of [SDL](https://www.libsdl.org/), packaged for Zig (Initially by Andrew Kelley) and in-sync with the latest SDL releases in the main repository.
Unnecessary files have been deleted, and the build system has been replaced with `build.zig

## Usage

We can easily fetch this library using `zig fetch`, for example:

```bash
zig fetch --save https://github.com/pwbh/SDL/archive/refs/tags/release-2.30.3.tar.gz
```

If another SDL version needed, we can use a different tag instead of the `release-2.30.3` in the url we fetch.

In your `build.zig`:

```zig
const exe = b.addExecutable(.{
      .name = "my-project",
      .root_source_file = b.path("src/main.zig"),
      .target = target,
      .optimize = optimize,
  });

if (target.result.os.tag == .linux) {
    // The SDL package doesn't work for Linux yet, so we rely on system
    // packages for now.
    exe.linkSystemLibrary("SDL2");
    exe.linkLibC();
} else {
    const sdl_dep = b.dependency("SDL", .{
        .optimize = .ReleaseFast,
        .target = target,
    });
    exe.linkLibrary(sdl_dep.artifact("SDL2"));
}

b.installArtifact(exe);
```

## Test it out

Now lets test it out and see if it creates a window. It should create a window and exit after 5 seconds.

```zig
const c = @cImport({
    @cInclude("SDL2/SDL.h");
});
const assert = @import("std").debug.assert;

pub fn main() !void {
    if (c.SDL_Init(c.SDL_INIT_VIDEO) != 0) {
        c.SDL_Log("Unable to initialize SDL: %s", c.SDL_GetError());
        return error.SDLInitializationFailed;
    }
    defer c.SDL_Quit();

    const screen = c.SDL_CreateWindow("My Game Window", c.SDL_WINDOWPOS_UNDEFINED, c.SDL_WINDOWPOS_UNDEFINED, 400, 140, c.SDL_WINDOW_OPENGL) orelse
        {
        c.SDL_Log("Unable to create window: %s", c.SDL_GetError());
        return error.SDLInitializationFailed;
    };
    defer c.SDL_DestroyWindow(screen);

    const renderer = c.SDL_CreateRenderer(screen, -1, 0) orelse {
        c.SDL_Log("Unable to create renderer: %s", c.SDL_GetError());
        return error.SDLInitializationFailed;
    };
    defer c.SDL_DestroyRenderer(renderer);

    c.SDL_Delay(5000);
}
```
