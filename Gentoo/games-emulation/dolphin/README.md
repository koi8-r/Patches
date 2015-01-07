Need to check haptic use flag on libsdl2, if dolphin found libsdl2, then it add haptic to sdl init flags.
If libsdl2 build without this flag dolphin silently don't show djoystick to you because SDL_Init raise error.
This is DIRTY patch: https://github.com/koi8-r/Patches/blob/master/Gentoo/games-emulation/dolphin/dolphin-9999.ebuild-SDL2-missing-haptic-use.patch

```
https://github.com/dolphin-emu/dolphin
Master: c6e2449bff926b1e241b4b9bad8569136be5eba4
```

```c
// Source/Core/InputCommon/ControllerInterface/SDL/SDL.h: line 14
#if SDL_VERSION_ATLEAST(1, 3, 0)
        #define USE_SDL_HAPTIC
#endif
#ifdef USE_SDL_HAPTIC
        #include <SDL_haptic.h>
        #define SDL_INIT_FLAGS  SDL_INIT_JOYSTICK | SDL_INIT_HAPTIC
#else
        #define SDL_INIT_FLAGS  SDL_INIT_JOYSTICK
#endif
```

```c
// Source/Core/InputCommon/ControllerInterface/SDL/SDL.cpp: line 30
void Init( std::vector<Core::Device*>& devices )
{
	// this is used to number the joysticks
	// multiple joysticks with the same name shall get unique ids starting at 0
	std::map<std::string, int> name_counts;

	if (SDL_Init( SDL_INIT_FLAGS ) >= 0)
	{
		// joysticks
		for (int i = 0; i < SDL_NumJoysticks(); ++i)
		{
			SDL_Joystick* dev = SDL_JoystickOpen(i);
			if (dev)
			{
				Joystick* js = new Joystick(dev, i, name_counts[GetJoystickName(i)]++);
				// only add if it has some inputs/outputs
				if (js->Inputs().size() || js->Outputs().size())
					devices.push_back( js );
				else
					delete js;
			}
		}
	}
}
```
