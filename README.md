# 🥽 XRBridge32

**Run 32-bit VR games and mods on your headset's native 64-bit OpenXR runtime. No SteamVR needed.**

```
32-bit game ─► OpenComposite (SteamVR games) ─► xrbridge32.dll ─► xrbridge_host.exe (64-bit) ─► your runtime
```

---

## 🚀 Quick start

1. Set your real runtime active (Pimax OpenXR, PimaxXR, or any other).
2. Unzip the whole package, then run **`XRBridge32-Setup.exe`** from it → **Install** → **Enable** → **Test with headset**.
3. Start your game.

📦 Everything is in the package: Setup installs OpenComposite and, if yours is too old, the
32-bit Visual C++ runtime from the folders beside it. Nothing is downloaded.

| In the package | |
|---|---|
| `XRBridge32-Setup.exe` | Installer, bridge, host and D3D9 screen |
| `OpenComposite\` | OpenComposite 32- and 64-bit DLLs (official builds, unmodified) |
| `VC_redist.x86.exe` | Microsoft's signed 32-bit Visual C++ runtime installer |
| `licenses\` | Licences for everything bundled |
| `source\` | Source for XRBridge32 and OpenComposite |

Each graphics API shows for about 4 seconds in the test: 🟩 D3D11 · 🟦 D3D12 · 🟧 Vulkan · 🟪 OpenGL.
The white half should be at the top. If it is at the bottom, please report it.

---

## ✅ What works

| | |
|---|---|
| 🎮 **Graphics APIs** | D3D11, D3D12, Vulkan, OpenGL |
| 🕹️ **SteamVR games** | Through OpenComposite, set up by Setup |
| 🧩 **64-bit OpenXR layers** | OpenXR Toolkit, Quad-Views-Foveated and others load in the host |
| 👁️ **Foveated rendering** | Fixed and eye-tracked: quad views (`XR_VARJO_quad_views`, `XR_VARJO_foveated_rendering`), FB / Meta / HTC foveation and eye gaze pass through. Quad-Views-Foveated in the host gives quad views to every graphics API, because the host is always D3D11 |
| 📦 **Extensions** | 223 passed through, generated from the Khronos registry |
| 🖼️ **Direct3D 9** | Stereo mods, or a virtual screen for flat games (see below) |

---

## 🖼️ Direct3D 9 games

OpenXR has no D3D9 support, so there are two ways in:

- **🎯 Stereo mods** that bring their own OpenXR code work through the bridge:
  [Alan Wake VR](https://alanwake.vrmods.eu), [Dishonored VR](https://github.com/VR-Stereo-Hub/Dishonored-VR),
  [Mirror's Edge VR](https://github.com/letsgosportsteam/mirrors-edge-vr-mod), [openRBRVR](https://github.com/Detegr/openRBRVR).
- **📺 Virtual screen** for any flat D3D9 game: in Setup, click **D3D9 game screen...** and pick the game.
  You get a large curved screen, not stereo. **F8** recenters. Avoid it in online games with anti-cheat.

---

## 🛠️ "Missing required XR extension"?

The OpenXR loader hides the real error, so check Setup's **Offered to 32-bit games** row:

| The row shows | Fix |
|---|---|
| ⚠️ A list, and the 32-bit runtime row says **Native** | Tick **Replace an existing native 32-bit runtime**, then **Enable** |
| ❌ `could not start the 64-bit host` | Antivirus may have removed `xrbridge_host.exe`. Allow it, then reinstall |
| ❌ `could not load the 64-bit OpenXR runtime` | Set your headset's runtime active |
| ⚠️ Everything except Vulkan | Update your GPU driver (the 32-bit `vulkan-1.dll` is missing) |
| ✅ D3D11, D3D12, Vulkan, OpenGL | The bridge is fine. Check the game's own log |

📄 Logs: `%LOCALAPPDATA%\XRBridge32\shim.log` and `host.log`.

---

## 🏎️ Performance

- Calls go through shared memory, about 1 µs each, with no kernel round trip in a burst.
- The host opts out of Windows power throttling and runs at above-normal priority, so it
  isn't parked on efficiency cores.
- Extension lists and path names are cached, and lookups never start the host twice.
- D3D9 screen: the back buffer is copied on the GPU and read a frame later, so the game never
  waits for it.
- Image releases are posted: the game's render thread doesn't wait while the host copies the
  image and runs 64-bit layers such as Neural Rendering VR's DLSS 5 submit. The host finishes
  them before that thread's `xrEndFrame`, so order is kept. `XRB_NO_POST=1` turns this off.
- `host.log` gets a timing line every 10 s: frame rate, frame interval, how long the runtime's
  `xrWaitFrame` blocks, and what release and `xrEndFrame` cost. It also says when a game had
  to fall back to CPU sync. Send it along when a game feels choppy.

---

## 🤝 Plays well with other mods

- 32-bit layers the game enables stay on the 32-bit side; 64-bit layers load in the host.
- A layer that a game or mod switches off for itself (Dishonored VR, AC Unity VR) still loads
  in the host, where the reason doesn't apply. Switch-offs set in Windows' environment
  settings are kept; `XRB_KEEP_LAYER_SWITCH_OFF=1` keeps them all.
- **Neural Rendering VR:** DLSS 5 runs once, in the host, with fixed or eye-tracked foveation.
- Extension functions are offered only when the real runtime has them, so feature checks are
  honest.
- D3D12 and Vulkan games get NT shared handles, which their drivers support; D3D11 and OpenGL
  keep the legacy handles every driver accepts.
- The D3D9 screen keeps each hooked function per object type, so it chains cleanly with
  ReShade, overlays and other `d3d9.dll` wrappers.

---

## ⚡ Never blocks a game

- The bridge uses a Vulkan game's queue only where OpenXR allows it, so DXVK-based mods are safe.
- A slow first start (antivirus scanning the host) gets 20 seconds before it counts as a failure.
- A stuck runtime returns an error after 20 seconds instead of freezing the game, and the bridge
  doesn't wait on it again.
- Frame calls are never cut short, and exiting a game never waits on the bridge.

---

## ⚠️ Limits

- The game and headset must use the same GPU.
- Your real runtime must support D3D11.
- OpenXR Toolkit's variable rate shading can't reach a 32-bit game.
- OpenComposite's own limits apply to SteamVR games.

---

## 💻 Command line

```
XRBridge32-Setup --install | --enable [--force] | --disable | --uninstall | --status
XRBridge32-Setup --selftest [--api d3d11|d3d12|vulkan|opengl]
XRBridge32-Setup --d3d9-add <game.exe> | --d3d9-remove <game.exe>
XRBridge32-Setup --zero-copy on|off
```

🔧 Built on Linux with mingw-w64. See `build.sh`.

---

## 🙏 Credits

| Project | Used for |
|---|---|
| [Khronos OpenXR SDK](https://github.com/KhronosGroup/OpenXR-SDK) | Loader (statically linked) and `xr.xml` registry |
| [Vulkan Headers](https://github.com/KhronosGroup/Vulkan-Headers) | Vulkan interop |
| [OpenComposite](https://gitlab.com/znixian/OpenOVR) by znixian, and the [QuestCraft fork](https://github.com/QuestCraftPlusPlus/OpenComposite) | SteamVR to OpenXR (downloaded by Setup, not bundled) |
| [PimaxXR](https://github.com/mbucchia/Pimax-OpenXR) by mbucchia | The shared-texture recipe for each graphics API |
| [OpenVR](https://github.com/ValveSoftware/openvr) by Valve | Testing SteamVR games |
| [DXVK](https://github.com/doitsujin/dxvk), [vkd3d-proton](https://github.com/HansKristian-Work/vkd3d-proton), [Wine](https://www.winehq.org) | Testing on Linux |
| VR mod authors: fholger (Alan Wake VR), Detegr (openRBRVR), the Dishonored VR and Mirror's Edge VR teams | Showing what D3D9 VR can do |

📜 Licences: the OpenXR loader and Vulkan headers are Apache-2.0. OpenComposite is GPLv3; its
source is in `source\OpenComposite-source.zip`. The Visual C++ runtime is redistributed under
Microsoft's terms.
