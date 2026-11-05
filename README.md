# SDL3 for Larva

Bindings of [SDL 3.5](https://libsdl.org) for [Larva](https://www.antril.org): windows, 2D rendering, input, audio,
gamepads, timers and the rest of SDL, called directly from Antril (or any language Larva compiles).

- **1243 functions, 131 structs and unions, 104 enums** from SDL's own headers.
- **Antril names**: `SDL_CreateWindow` is `Video.CreateWindow`, `SDL_INIT_VIDEO` is `InitFlags.Video`,
  `SDLK_ESCAPE` is `Keycode.Escape`. Each SDL header is a namespace and a class (`SDL.Video`, `SDL.Render`,
  `SDL.Events`…).
- **Typed flags and keys**: `InitFlags`, `WindowFlags` and `Keymod` are bitflag enums; `Keycode`, `Scancode`,
  `EventType` and SDL's other enums are enums. Parameters, results and fields that SDL declares with them take them.
- **`SDL3.dll` included**: a project that uses the package gets it copied next to its program.

The package is `Antril-Organization/SDL3`, published with [Hive](https://www.antril.org), Larva's package manager.

## Use it

Add the package to your project (`.queen`) and install it:

```
larva hive add Antril-Organization/SDL3@^3.5
```

A window with a square that bounces (vertical sync on; Escape or closing the window ends it):

```antril
import System;
import SDL.Init;
import SDL.Video;
import SDL.Render;
import SDL.Events;
import SDL.Keycode;
import SDL.Rect;

/// SDL's handles are pointers, so the program is `unsafe` (the project allows it: `unsafe: allow` in BUILD).
unsafe Main() : i32
{
	if (!Init.Initialize(InitFlags.Video | InitFlags.Events))
		return 1;
	void* window = Video.CreateWindow("Larva + SDL3", 800, 600, WindowFlags.Resizable);
	void* renderer = Render.CreateRenderer(window, null);   // null: SDL picks the renderer
	if (window == null || renderer == null)
		return 1;
	Render.SetRenderVSync(renderer, 1);                     // one frame per refresh of the screen

	f32 x = 100.0f32, y = 100.0f32, dx = 4.0f32, dy = 3.0f32;
	bool running = true;
	Event e;
	FRect square;
	square.W = 80.0f32;
	square.H = 80.0f32;
	loop (running)
	{
		loop (Events.PollEvent(&e))
		{
			if (e.Type == (u32)EventType.Quit || e.Type == (u32)EventType.KeyDown && e.Key.Key == Keycode.Escape)
				running = false;
		}
		x += dx;
		y += dy;
		if (x < 0.0f32 || x > 720.0f32)
			dx = -dx;
		if (y < 0.0f32 || y > 520.0f32)
			dy = -dy;
		square.X = x;
		square.Y = y;
		Render.SetRenderDrawColor(renderer, 20, 24, 40, 255);
		Render.RenderClear(renderer);
		Render.SetRenderDrawColor(renderer, 120, 200, 90, 255);
		Render.RenderFillRect(renderer, &square);
		Render.RenderPresent(renderer);
	}
	Render.DestroyRenderer(renderer);
	Video.DestroyWindow(window);
	Init.Quit();
	return 0;
}
```

```
larva run
```

Notes:

- SDL's C strings that may be `NULL` are `str?`: pass a string or `null`.
- What SDL gives through pointers (`Window*`, `Renderer*`…) is `void*`, compared with `null` in `unsafe` code.
- Functions SDL declares with a variable number of arguments (`SDL_Log`, `SDL_SetError`…) or that take a struct by
  value (`SDL_GUIDToString`…) are not in the binding: Larva calls C with what C can take one by one.

## Build it

What is in this repository:

| Path | What it is |
| --- | --- |
| `SDLBindingProfile.ant` | The binding profile: what is read, how each name is written, where it goes. |
| `include/SDL3/` | SDL's headers, read by the profile. |
| `build/SDL3.dll` | SDL itself (Windows x64). |
| `bindings/` | The binding as Antril source, written by the profile: to read what it has. |
| `package/SDL3.queen` | The Hive package: the binding (`SDL.Bindings.dll`) and `SDL3.dll`. |

With [Larva](https://www.antril.org) 0.8 or newer (and clang, which reads the headers):

```
larva bind --profile SDLBindingProfile.ant
```

writes `SDL.Bindings.dll` (the binding, compiled) and the source in `bindings/`. Then

```
larva hive pack package/SDL3.queen
```

packs `package/bin/SDL3-3.5.0.hive`, signed with the key `package/hive.key` (private: not in this repository).

To change the binding, change the profile: its `Configure` sets the prefixes, cases, namespaces and files; `Start`
makes enums from groups of constants (`m.MakeEnum("Keycode", "SDLK_*", false).Typedef("SDL_Keycode")`); `Function`,
`Struct`, `Enum` and `Constant` decide what each declaration is, or leave it out; `Name` renames.

## License

SDL is © Sam Lantinga and its contributors, under the [zlib license](https://github.com/libsdl-org/SDL/blob/main/LICENSE.txt):
its headers (`include/`) and `SDL3.dll` are distributed here as SDL publishes them. The binding is under the same
zlib license.
