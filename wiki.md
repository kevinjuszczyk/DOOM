# DOOM

This repository contains the open source release of the DOOM game engine, allowing developers to study, modify, and build upon the iconic first-person shooter game. The codebase provides a complete implementation of the DOOM game, including the main game logic, networking, rendering, sound, and user interface components.

The repository is organized into two main directories:

1. `/DOOM/linuxdoom-1.10`: Contains the core game engine implementation for Linux.
2. `/DOOM/sndserv`: Houses the UNIX sound server implementation.

The main game engine in `/DOOM/linuxdoom-1.10` is built around a central game loop, implemented in the [`D_DoomLoop()`](/DOOM/linuxdoom-1.10/d_main.c#L354) function. This loop handles:

- Event processing
- Game state updates
- Sound updates
- Rendering

The game engine uses a client-server architecture for multiplayer gameplay, with networking code implemented in `/DOOM/linuxdoom-1.10/d_net.c`. Key networking features include:

- Packet data structures ([`doomdata_t`](/DOOM/ipx/IPXNET.H#L7) and [`doomcom_t`](/DOOM/ipx/DOOMNET.H#L47))
- Game state synchronization across players
- Command broadcasting

For more details on networking, see the [Networking](#networking) section.

The rendering pipeline, crucial for DOOM's 3D-like graphics, is based on a raycasting algorithm. This technique allows for efficient rendering of the game world on limited hardware. The engine uses various data structures to represent the game world, including:

- Vertices
- Sectors
- Line definitions
- Things (objects)

These structures are defined in `/DOOM/linuxdoom-1.10/doomdata.h`. For more information on the game world representation, refer to the [Game World and Level Management](#game%20world%20and%20level%20management) section.

Sound processing is handled by a separate sound server implemented in `/DOOM/sndserv`. This server is responsible for:

- Loading sound effects from WAD files
- Mixing audio samples
- Playing back sound effects and music

The sound server communicates with the main game engine to provide audio output. More details can be found in the [Sound System](#sound%20system) section.

The user interface, including the Heads-Up Display (HUD) and menu system, is implemented in `/DOOM/linuxdoom-1.10/hu_lib.c` and `/DOOM/linuxdoom-1.10/hu_stuff.c`. These components handle:

- Text display
- Status information rendering
- Player chat functionality

For more information on the user interface, see the [User Interface](#user%20interface) section.

Key design choices in the DOOM engine include:

- Use of fixed-point arithmetic for performance on limited hardware
- Binary Space Partitioning (BSP) for efficient rendering and collision detection
- Separate sound server for audio processing
- WAD file format for game data storage and modding support

The DOOM engine's modular design and use of data-driven approaches (via WAD files) have contributed to its longevity and adaptability across various platforms and projects.

## Game Engine
References: `/DOOM/linuxdoom-1.10`

The DOOM game engine is built around a central game loop that handles initialization, event processing, and state updates. This loop is implemented in the `/DOOM/linuxdoom-1.10/d_main.c` file, which serves as the entry point for the game.

The engine's core functionality includes:

• Event handling: User input and game events are managed through a queue system defined in `/DOOM/linuxdoom-1.10/d_event.h`. The [`event_t`](/DOOM/linuxdoom-1.10/d_event.h#L50) structure represents different types of events, such as key presses, mouse movements, and joystick actions.

• Game state management: The overall game state, including player state, level state, and game mode, is handled through various structures and functions. The [`player_t`](/DOOM/linuxdoom-1.10/d_player.h#L166) struct in `/DOOM/linuxdoom-1.10/d_player.h` represents the player character, storing information about health, weapons, and inventory.

• Network communication: For multiplayer games, the engine handles packet data structures and communication with network drivers. This functionality is implemented in `/DOOM/linuxdoom-1.10/d_net.c` and `/DOOM/linuxdoom-1.10/d_net.h`.

• Rendering: The engine includes a rendering pipeline for drawing the game world, sprites, and special effects. Texture and pixel management are crucial components of this system, handled by various functions in the `/DOOM/linuxdoom-1.10` directory.

• Sound system: The engine interfaces with a sound server for handling sound effects and music. The system-level interface is defined in `/DOOM/linuxdoom-1.10/i_sound.h`, while the actual implementation of the UNIX sound server is in the `/DOOM/sndserv` directory.

• User interface: The engine implements both a Heads-Up Display (HUD) and a menu system. These components handle text display, status information, and game settings.

• Level management: The engine loads levels from WAD (Where's All the Data) files and manages the game world during gameplay. Various data structures are used to represent map elements, such as vertices, sectors, and objects.

The DOOM engine's design emphasizes efficiency and modularity, allowing for smooth gameplay across different hardware configurations and easy extension of game functionality.

### Main Game Loop
References: `/DOOM/linuxdoom-1.10/d_main.c`

The main game loop is implemented in the [`D_DoomLoop()`](/DOOM/linuxdoom-1.10/d_main.c#L354) function within `/DOOM/linuxdoom-1.10/d_main.c`. This function orchestrates the core gameplay mechanics:

- Processes events using [`D_ProcessEvents()`](/DOOM/linuxdoom-1.10/d_main.c#L161), handling user input and game events
- Updates game state and performs game logic calculations
- Renders the current frame using [`D_Display()`](/DOOM/linuxdoom-1.10/d_main.c#L193)
- Manages sound updates

[`D_DoomMain()`](/DOOM/linuxdoom-1.10/d_main.c#L796) serves as the entry point, initializing the game:

- Parses command-line arguments
- Identifies game mode (shareware, registered, commercial) via [`IdentifyVersion()`](/DOOM/linuxdoom-1.10/d_main.c#L563)
- Sets up initial game state
- Processes additional WAD files using [`D_AddFile()`](/DOOM/linuxdoom-1.10/d_main.c#L543)

Key game settings are configured through command-line options:

- [`-nomonsters`](/DOOM/linuxdoom-1.10/d_main.c#L96): Disables monster spawning
- [`-respawn`](/DOOM/linuxdoom-1.10/d_main.c#L97): Enables monster respawning
- [`-fast`](/DOOM/linuxdoom-1.10/d_main.c#L98): Increases game speed
- [`-deathmatch`](/DOOM/linuxdoom-1.10/d_main.c#L814): Activates deathmatch mode
- [`-turbo`](/DOOM/linuxdoom-1.10/d_main.c#L885): Modifies player movement speed

The [`D_Display()`](/DOOM/linuxdoom-1.10/d_main.c#L193) function manages rendering for different game states:

- Level gameplay
- Intermission screens
- Finale sequences
- Demo playback

Event handling is crucial for game responsiveness:

- [`D_PostEvent()`](/DOOM/linuxdoom-1.10/d_main.c#L150) adds events to the queue
- [`D_ProcessEvents()`](/DOOM/linuxdoom-1.10/d_main.c#L161) processes queued events, routing them to appropriate handlers

Demo playback and cycling is handled by [`D_AdvanceDemo()`](/DOOM/linuxdoom-1.10/d_main.c#L444) and [`D_DoAdvanceDemo()`](/DOOM/linuxdoom-1.10/d_main.c#L454), managing transitions between demo sequences and intro screens.

### Event Handling
References: `/DOOM/linuxdoom-1.10/d_event.h`, `/DOOM/linuxdoom-1.10/d_main.c`

Event handling in DOOM is managed through a queue-based system defined in `/DOOM/linuxdoom-1.10/d_event.h` and implemented in `/DOOM/linuxdoom-1.10/d_main.c`. The system processes various input events such as key presses, mouse movements, and joystick actions.

Key components:

- [`event_t`](/DOOM/linuxdoom-1.10/d_event.h#L50) struct: Represents an input event, containing the event type and associated data.
- [`events`](/DOOM/linuxdoom-1.10/d_main.c#L141) array: Stores input events for processing.
- [`eventhead`](/DOOM/linuxdoom-1.10/d_main.c#L142) and [`eventtail`](/DOOM/linuxdoom-1.10/d_main.c#L143): Indices for managing the event queue.

Event processing flow:

- [`D_PostEvent()`](/DOOM/linuxdoom-1.10/d_main.c#L150): Adds new events to the queue.
- [`D_ProcessEvents()`](/DOOM/linuxdoom-1.10/d_main.c#L161): Processes queued events, passing them to appropriate responders (e.g., menu, game).

The [`gameaction_t`](/DOOM/linuxdoom-1.10/d_event.h#L65) enum defines various game actions (e.g., loading a level, starting a new game) that can be triggered by events.

[`buttoncode_t`](/DOOM/linuxdoom-1.10/d_event.h#L100) enum represents player input states, such as attacking or using objects.

This event system allows DOOM to handle user input and game events in a structured manner, separating input processing from game logic and ensuring responsive gameplay.

### Game State Management
References: `/DOOM/linuxdoom-1.10/d_main.c`, `/DOOM/linuxdoom-1.10/d_player.h`, `/DOOM/linuxdoom-1.10/doomdata.h`

The game state is primarily managed through the [`player_t`](/DOOM/linuxdoom-1.10/d_player.h#L166) struct defined in `/DOOM/linuxdoom-1.10/d_player.h`. This struct encapsulates all aspects of a player's state, including:

- Current player state ([`playerstate_t`](/DOOM/linuxdoom-1.10/d_player.h#L62))
- Health, armor, and power-ups
- Weapon information (current and pending weapons)
- Kill count, items collected, and secrets found
- Damage taken and attacker information

The [`playerstate_t`](/DOOM/linuxdoom-1.10/d_player.h#L62) enumeration defines possible player states:
- [`PST_LIVE`](/DOOM/linuxdoom-1.10/d_player.h#L56): Player is alive and active
- [`PST_DEAD`](/DOOM/linuxdoom-1.10/d_player.h#L58): Player has died
- [`PST_REBORN`](/DOOM/linuxdoom-1.10/d_player.h#L60): Player is ready to respawn

Game mode and level state are handled in `/DOOM/linuxdoom-1.10/d_main.c`:

- [`D_DoomMain()`](/DOOM/linuxdoom-1.10/d_main.c#L796) initializes the game state, parsing command-line arguments to set up the initial game mode (shareware, registered, or commercial).
- [`IdentifyVersion()`](/DOOM/linuxdoom-1.10/d_main.c#L563) determines the game mode based on available IWAD files.
- [`D_DoomLoop()`](/DOOM/linuxdoom-1.10/d_main.c#L354) manages the main game loop, updating the game state each tick.
- [`D_Display()`](/DOOM/linuxdoom-1.10/d_main.c#L193) renders the current game state, handling different states such as level gameplay, intermission, and finale.

Level state is represented using structures defined in `/DOOM/linuxdoom-1.10/doomdata.h`:

- [`mapvertex_t`](/DOOM/linuxdoom-1.10/doomdata.h#L64), [`mapsidedef_t`](/DOOM/linuxdoom-1.10/doomdata.h#L78), [`maplinedef_t`](/DOOM/linuxdoom-1.10/doomdata.h#L93), [`mapsector_t`](/DOOM/linuxdoom-1.10/doomdata.h#L150): Define the map geometry and properties
- [`mapsubsector_t`](/DOOM/linuxdoom-1.10/doomdata.h#L158), [`mapseg_t`](/DOOM/linuxdoom-1.10/doomdata.h#L171), [`mapnode_t`](/DOOM/linuxdoom-1.10/doomdata.h#L196): Used for BSP-based rendering and collision detection
- [`mapthing_t`](/DOOM/linuxdoom-1.10/doomdata.h#L210): Represents objects in the map (monsters, items, player start positions)

These structures are loaded from WAD files at runtime to construct the game world.

### Network Communication
References: `/DOOM/linuxdoom-1.10/d_net.c`, `/DOOM/linuxdoom-1.10/d_net.h`

Network communication in DOOM is handled through packet data structures and functions defined in `/DOOM/linuxdoom-1.10/d_net.c` and `/DOOM/linuxdoom-1.10/d_net.h`. The key components are:

- [`doomcom_t`](/DOOM/ipx/DOOMNET.H#L47): Structure for communication between the DOOM executable and network driver
- [`doomdata_t`](/DOOM/ipx/IPXNET.H#L7): Defines network packet data format, including checksums and player commands
- [`netbuffer`](/DOOM/linuxdoom-1.10/d_net.c#L45): Holds network packet data

Packet handling is managed by several functions:

- [`NetbufferSize()`](/DOOM/linuxdoom-1.10/d_net.c#L90): Determines packet size
- [`NetbufferChecksum()`](/DOOM/linuxdoom-1.10/d_net.c#L98): Calculates packet checksum
- [`HSendPacket()`](/DOOM/linuxdoom-1.10/d_net.c#L143): Sends packets to other nodes
- [`HGetPacket()`](/DOOM/linuxdoom-1.10/d_net.c#L192): Receives incoming packets

The [`NetUpdate()`](/DOOM/linuxdoom-1.10/d_net.c#L368) function is responsible for:

- Building ticcmds (tic commands) for the console player
- Sending packets to other nodes
- Listening for incoming packets

Game synchronization across players is achieved through:

- [`TryRunTics()`](/DOOM/linuxdoom-1.10/d_net.c#L636): Manages execution of game tics, ensuring synchronized game state
- [`ExpandTics()`](/DOOM/linuxdoom-1.10/d_net.c#L120): Handles tic expansion for smooth gameplay

Network game initialization and management:

- [`D_ArbitrateNetStart()`](/DOOM/linuxdoom-1.10/d_net.c#L476): Arbitrates network game start
- [`D_CheckNetGame()`](/DOOM/linuxdoom-1.10/d_net.c#L555): Initializes network game, setting up player and node information
- [`D_QuitNetGame()`](/DOOM/linuxdoom-1.10/d_net.c#L602): Notifies other players when quitting a network game

### Game State Synchronization
References: `/DOOM/linuxdoom-1.10/d_net.c`, `/DOOM/linuxdoom-1.10/d_net.h`

Game state synchronization in DOOM's multiplayer mode is primarily managed through the creation and broadcasting of ticcmds (tic commands). The [`NetUpdate()`](/DOOM/linuxdoom-1.10/d_net.c#L368) function in `/DOOM/linuxdoom-1.10/d_net.c` is responsible for this process:

- Builds ticcmds for the console player
- Sends packets containing these commands to other nodes in the network
- Listens for incoming packets from other players

The [`doomdata_t`](/DOOM/ipx/IPXNET.H#L7) structure in `/DOOM/linuxdoom-1.10/d_net.h` defines the format of network packets, including:

- Checksum for data integrity
- Retransmission information
- Ticcmds for each player

To ensure consistent gameplay across networked players, the [`TryRunTics()`](/DOOM/linuxdoom-1.10/d_net.c#L636) function in `/DOOM/linuxdoom-1.10/d_net.c`:

- Manages the execution of game tics
- Synchronizes game state across all players

The [`doomcom_t`](/DOOM/ipx/DOOMNET.H#L47) structure in `/DOOM/linuxdoom-1.10/d_net.h` facilitates communication between the DOOM executable and the network driver, storing:

- Number of nodes (players)
- Game settings
- Packet data

For network game initialization and arbitration, [`D_ArbitrateNetStart()`](/DOOM/linuxdoom-1.10/d_net.c#L476) and [`D_CheckNetGame()`](/DOOM/linuxdoom-1.10/d_net.c#L555) in `/DOOM/linuxdoom-1.10/d_net.c` set up player and node information.

### Multiplayer Game Management
References: `/DOOM/linuxdoom-1.10/d_net.c`, `/DOOM/linuxdoom-1.10/d_net.h`

Multiplayer game management in DOOM is primarily handled through functions in `/DOOM/linuxdoom-1.10/d_net.c` and `/DOOM/linuxdoom-1.10/d_net.h`.

Initialization:
- [`D_CheckNetGame()`](/DOOM/linuxdoom-1.10/d_net.c#L555) sets up the network game, configuring player and node information.
- [`D_ArbitrateNetStart()`](/DOOM/linuxdoom-1.10/d_net.c#L476) manages the start of a network game, ensuring all players are ready.

Player joining and leaving:
- The [`doomcom_t`](/DOOM/ipx/DOOMNET.H#L47) structure in `/DOOM/linuxdoom-1.10/d_net.h` tracks the number of nodes (players) in the game.
- When a player joins or leaves, the game updates this structure and broadcasts the changes to other players.

Game termination:
- [`D_QuitNetGame()`](/DOOM/linuxdoom-1.10/d_net.c#L602) notifies other players when a player exits the game.
- This function sends special packets to ensure proper cleanup of the network game state.

Ongoing management:
- [`NetUpdate()`](/DOOM/linuxdoom-1.10/d_net.c#L368) builds and broadcasts ticcmds (game commands) to synchronize game state across players.
- [`TryRunTics()`](/DOOM/linuxdoom-1.10/d_net.c#L636) executes game ticks, maintaining consistency in the multiplayer environment.

The networking code uses the [`doomdata_t`](/DOOM/ipx/IPXNET.H#L7) structure to define the format of network packets, facilitating efficient communication between players.

### Texture and Pixel Management
References: `/DOOM/linuxdoom-1.10`

Textures and pixel blocks in DOOM are managed through several key structures and functions:

• The [`pic_t`](/DOOM/linuxdoom-1.10/d_textur.h#L41) struct in `/DOOM/linuxdoom-1.10/d_textur.h` represents an unmasked block of pixels, used for textures and flat surfaces. It contains fields for width, height, and a pointer to the pixel data.

• Texture management is handled in `/DOOM/linuxdoom-1.10/r_data.c`:
  - [`maptexture_t`](/DOOM/linuxdoom-1.10/r_data.c#L93) and [`texture_t`](/DOOM/linuxdoom-1.10/r_data.c#L125) structures represent wall textures composed of one or more patches.
  - [`R_GenerateLookup()`](/DOOM/linuxdoom-1.10/r_data.c#L296) analyzes texture definitions and creates lookup tables for efficient rendering.
  - [`R_GenerateComposite()`](/DOOM/linuxdoom-1.10/r_data.c#L228) generates composite textures by combining individual patches.
  - [`R_GetColumn()`](/DOOM/linuxdoom-1.10/r_data.c#L383) retrieves a column of texture data, either from the cached composite or by generating it on-the-fly.

• Flat (floor and ceiling) graphics are managed through:
  - [`R_InitFlats()`](/DOOM/linuxdoom-1.10/r_data.c#L581) initializes the list of available flats.
  - [`R_FlatNumForName()`](/DOOM/linuxdoom-1.10/r_data.c#L672) retrieves the index of a flat given its name.

• Sprite management is handled by:
  - [`R_InitSpriteLumps()`](/DOOM/linuxdoom-1.10/r_data.c#L603) loads information about sprite graphics, including width, offset, and top offset.
  - Data is stored in [`spritewidth`](/DOOM/linuxdoom-1.10/r_data.c#L158), [`spriteoffset`](/DOOM/linuxdoom-1.10/r_data.c#L159), and [`spritetopoffset`](/DOOM/linuxdoom-1.10/r_data.c#L160) arrays.

• Colormap management:
  - [`R_InitColormaps()`](/DOOM/linuxdoom-1.10/r_data.c#L633) loads color lookup tables for lighting and shading effects.
  - The [`colormaps`](/DOOM/linuxdoom-1.10/r_data.c#L162) variable stores a pointer to the aligned color lookup table.

• The [`R_InitData()`](/DOOM/linuxdoom-1.10/r_data.c#L654) function initializes all graphics-related data structures, while [`R_PrecacheLevel()`](/DOOM/linuxdoom-1.10/r_data.c#L743) preloads relevant graphics for the current level to optimize rendering performance.

### Rendering Pipeline
References: `/DOOM/linuxdoom-1.10`

The rendering pipeline in DOOM is implemented primarily in the `/DOOM/linuxdoom-1.10/r_segs.c` file. The core rendering process involves:

• Drawing wall segments using [`R_RenderSegLoop()`](/DOOM/linuxdoom-1.10/r_segs.c#L206), which calculates texture coordinates, lighting, and clipping information for each column of the wall.

• Handling masked textures (e.g., doors, windows) with [`R_RenderMaskedSegRange()`](/DOOM/linuxdoom-1.10/r_segs.c#L103).

• Managing visible planes (floors and ceilings) using the [`visplane_t`](/DOOM/linuxdoom-1.10/r_defs.h#L480) structure defined in `/DOOM/linuxdoom-1.10/r_plane.h`.

• Rendering sprites and special effects.

The pipeline uses several optimization techniques:

• Fixed-point arithmetic for efficient calculations.

• Precalculated tables (e.g., [`tantoangle[]`](/DOOM/linuxdoom-1.10/r_main.c#L284) in `/DOOM/linuxdoom-1.10/tables.h`) for fast angle calculations.

• Binary Space Partitioning (BSP) tree traversal for efficient rendering order and occlusion culling.

The [`R_RenderBSPNode()`](/DOOM/linuxdoom-1.10/r_bsp.c#L552) function in `/DOOM/linuxdoom-1.10/r_bsp.h` is responsible for traversing the BSP tree and rendering the game world. It recursively calls itself to traverse the tree and render the visible segments.

Sprite rendering is handled separately, with visible sprites stored in the [`vissprite_t`](/DOOM/linuxdoom-1.10/r_defs.h#L409) structure. The pipeline sorts and draws sprites after rendering the walls and planes to ensure correct depth ordering.

Special effects, such as palette shifting for damage indicators or item pickups, are managed by the [`ST_doPaletteStuff()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1000) function in `/DOOM/linuxdoom-1.10/st_stuff.c`.

The rendering pipeline interacts closely with the game's main loop, updating the display based on the current game state and player actions.

### Heads-Up Display (HUD) Rendering
References: `/DOOM/linuxdoom-1.10`

The HUD rendering is primarily handled by the [`ST_drawWidgets()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1054) function in `/DOOM/linuxdoom-1.10/st_stuff.c`. This function updates and draws various widgets on the status bar, including health, armor, ammunition, and weapons.

Key aspects of the HUD rendering process:

• Widget initialization: The status bar widgets are initialized using functions like [`STlib_initNum()`](/DOOM/linuxdoom-1.10/st_lib.c#L66), [`STlib_initPercent()`](/DOOM/linuxdoom-1.10/st_lib.c#L163), [`STlib_initBinIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L245), and [`STlib_initMultIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L193).

• Face animation: The [`ST_updateFaceWidget()`](/DOOM/linuxdoom-1.10/st_stuff.c#L752) function determines the appropriate face to display based on the player's current state, while [`ST_calcPainOffset()`](/DOOM/linuxdoom-1.10/st_stuff.c#L729) calculates the offset for the player's face based on health.

• Palette and color shifting: [`ST_doPaletteStuff()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1000) handles palette and color shifting effects, such as the red/gold shift when the player takes damage or picks up items.

• Text rendering: The [`HU_Drawer()`](/DOOM/linuxdoom-1.10/hu_stuff.c#L486) function in `/DOOM/linuxdoom-1.10/hu_stuff.c` is responsible for rendering text on the HUD, including messages and chat input.

The HUD rendering system uses the [`screens`](/DOOM/linuxdoom-1.10/v_video.c#L44) array defined in `/DOOM/linuxdoom-1.10/v_video.h` to draw graphics on the screen. The [`V_DrawPatch()`](/DOOM/linuxdoom-1.10/v_video.c#L204) and [`V_DrawPatchDirect()`](/DOOM/linuxdoom-1.10/v_video.c#L337) functions are used to draw pre-rendered graphics (patches) to the screen.

To optimize performance, the system uses dirty rectangle tracking with the [`dirtybox`](/DOOM/linuxdoom-1.10/v_video.c#L46) array to update only the modified portions of the screen.

### Sound Server
References: `/DOOM/sndserv`

The UNIX sound server for DOOM is implemented in `/DOOM/sndserv/soundsrv.c`. Key components include:

- Sound effect loading: [`grabdata()`](/DOOM/sndserv/soundsrv.c#L304) loads sound data from WAD files into the [`S_sfx`](/DOOM/sndserv/sounds.c#L127) array.
- Mixing: [`mix()`](/DOOM/sndserv/soundsrv.c#L142) combines active sound channels into the [`mixbuffer`](/DOOM/sndserv/soundsrv.c#L95).
- Playback: [`updatesounds()`](/DOOM/sndserv/soundsrv.c#L410) submits the mixed buffer to the sound device via [`I_SubmitOutputBuffer()`](/DOOM/sndserv/linux.c#L102).
- Channel management: [`addsfx()`](/DOOM/sndserv/soundsrv.c#L419) assigns new sound effects to available channels.

The server's main loop in [`main()`](/DOOM/linuxdoom-1.10/i_main.c#L35):
- Listens for commands from DOOM
- Processes play ('p'), save ('s'), and quit ('q') commands

Sound initialization and shutdown are handled by:
- [`initdata()`](/DOOM/sndserv/soundsrv.c#L537): Sets up channel arrays, step tables, and volume lookup tables
- [`quit()`](/DOOM/sndserv/soundsrv.c#L573): Shuts down sound and music subsystems

WAD file handling is implemented in `/DOOM/sndserv/wadread.c`:
- [`openwad()`](/DOOM/sndserv/wadread.c#L152): Opens WAD file, reads header, parses directory
- [`loadlump()`](/DOOM/sndserv/wadread.c#L198): Loads specific sound effect lump
- [`getsfx()`](/DOOM/sndserv/wadread.c#L231): High-level function for loading and padding sound data

The sound server interfaces with the Linux sound system through `/DOOM/sndserv/linux.c`:
- [`I_InitSound()`](/DOOM/sndserv/linux.c#L72): Opens `/dev/dsp`, sets audio parameters
- [`I_SubmitOutputBuffer()`](/DOOM/sndserv/linux.c#L102): Writes audio samples to `/dev/dsp`
- [`I_ShutdownSound()`](/DOOM/sndserv/linux.c#L109): Closes `/dev/dsp`

### Sound System Interface
References: `/DOOM/linuxdoom-1.10`

The sound system interface is implemented in `/DOOM/linuxdoom-1.10/i_sound.h` and `/DOOM/linuxdoom-1.10/i_sound.c`. Key functions include:

- [`I_InitSound()`](/DOOM/sndserv/linux.c#L72) and [`I_ShutdownSound()`](/DOOM/sndserv/linux.c#L109): Initialize and shut down the sound system.
- [`I_GetSfxLumpNum()`](/DOOM/linuxdoom-1.10/i_sound.c#L451): Retrieves the raw data lump index for a given sound descriptor.
- [`I_StartSound()`](/DOOM/linuxdoom-1.10/i_sound.c#L471): Starts playing a sound effect on a specific channel, handling volume, separation, pitch, and priority.
- [`I_StopSound()`](/DOOM/linuxdoom-1.10/i_sound.c#L505): Stops playback of a sound on a given channel.
- [`I_SoundIsPlaying()`](/DOOM/linuxdoom-1.10/i_sound.c#L517): Checks if a sound is still playing on a channel.
- [`I_UpdateSoundParams()`](/DOOM/linuxdoom-1.10/i_sound.c#L675): Updates volume, separation, and pitch of a playing sound.

Music playback is managed through:

- [`I_InitMusic()`](/DOOM/sndserv/linux.c#L67) and [`I_ShutdownMusic()`](/DOOM/sndserv/linux.c#L116): Initialize and shut down the music system.
- [`I_SetMusicVolume()`](/DOOM/linuxdoom-1.10/i_sound.c#L438): Sets the music volume.
- [`I_PauseSong()`](/DOOM/linuxdoom-1.10/i_sound.c#L848) and [`I_ResumeSong()`](/DOOM/linuxdoom-1.10/i_sound.c#L854): Pause and resume music playback.
- [`I_RegisterSong()`](/DOOM/linuxdoom-1.10/i_sound.c#L875): Registers a song handle to song data.
- [`I_PlaySong()`](/DOOM/linuxdoom-1.10/i_sound.c#L841): Starts playing a song in a loop.
- [`I_StopSong()`](/DOOM/linuxdoom-1.10/i_sound.c#L860): Stops the current song.
- [`I_UnRegisterSong()`](/DOOM/linuxdoom-1.10/i_sound.c#L869): Unregisters a song handle.

The interface uses the [`sndserver`](/DOOM/linuxdoom-1.10/i_sound.c#L64) executable for sound server integration on UNIX platforms. Sound effects are loaded from WAD files using [`getsfx()`](/DOOM/sndserv/wadread.c#L231), which pads the data to the required sample count. The [`addsfx()`](/DOOM/sndserv/soundsrv.c#L419) function adds sound effects to the list of active sounds, managing internal channels and setting volume and panning parameters.

Sound mixing is handled by [`I_UpdateSound()`](/DOOM/linuxdoom-1.10/i_sound.c#L539), which loops through active sound channels, retrieves samples, applies volume and panning adjustments, and mixes the samples into the global [`mixbuffer`](/DOOM/sndserv/soundsrv.c#L95). The [`I_SubmitSound()`](/DOOM/linuxdoom-1.10/i_sound.c#L666) function writes the [`mixbuffer`](/DOOM/sndserv/soundsrv.c#L95) contents to the audio device for playback.

### WAD File Handling
References: `/DOOM/sndserv`

WAD (Where's All the Data) file handling in DOOM's sound server is primarily managed by the [`wadread.c`](/DOOM/sndserv/wadread.c#L4) and [`wadread.h`](/DOOM/sndserv/wadread.h#L4) files in the `/DOOM/sndserv` directory. The key functions for loading sound effects from WAD files are:

- [`openwad()`](/DOOM/sndserv/wadread.c#L152): Opens a WAD file, reads its header, and parses the directory information to populate the [`lumpinfo`](/DOOM/sndserv/wadread.c#L84) data structure.

- [`loadlump()`](/DOOM/sndserv/wadread.c#L198): Loads a specific sound effect lump from the WAD file by searching the [`lumpinfo`](/DOOM/sndserv/wadread.c#L84) table and reading the lump data into memory.

- [`getsfx()`](/DOOM/sndserv/wadread.c#L231): A higher-level function that loads a sound effect lump, pads the data to a specific sample count, and returns the padded sound effect data.

The WAD file handling process:

• Searches for various WAD files (e.g., [`doom1.wad`](/DOOM/sndserv/soundsrv.c#L326), [`doom2.wad`](/DOOM/sndserv/soundsrv.c#L329)) to load sound effect data.
• Parses the WAD file header and directory to locate individual sound effect lumps.
• Loads sound effects into the [`S_sfx`](/DOOM/sndserv/sounds.c#L127) array for use by the sound server.

Endianness handling is implemented using the [`LONG()`](/DOOM/ipx/IPXNET.H#L29) and [`SHORT()`](/DOOM/sndserv/wadread.c#L100) macros, along with the [`SwapLONG()`](/DOOM/sndserv/wadread.c#L107) and [`SwapSHORT()`](/DOOM/sndserv/wadread.c#L116) functions, ensuring compatibility across different hardware architectures.

The [`grabdata()`](/DOOM/sndserv/soundsrv.c#L304) function in `/DOOM/sndserv/soundsrv.c` utilizes these WAD file handling functions to load sound effect data into the sound server's memory.

### Heads-Up Display (HUD)
References: `/DOOM/linuxdoom-1.10`

The HUD functionality is implemented primarily in `/DOOM/linuxdoom-1.10/st_stuff.c`. Key components include:

• Text display: The [`STlib_initNum()`](/DOOM/linuxdoom-1.10/st_lib.c#L66), [`STlib_initPercent()`](/DOOM/linuxdoom-1.10/st_lib.c#L163), [`STlib_initBinIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L245), and [`STlib_initMultIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L193) functions initialize various widgets for displaying numeric values, percentages, binary icons, and multi-state icons respectively.

• Widget rendering: [`ST_drawWidgets()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1054) updates the display of these widgets based on the player's current state, including health, armor, ammunition, and weapons.

• Scrolling messages: The file handles scrolling messages, though the specific implementation details are not provided in the summary.

• Chat input: [`ST_Responder()`](/DOOM/linuxdoom-1.10/st_stuff.c#L518) handles chat-related input events, allowing players to communicate during gameplay.

• Face animation: [`ST_updateFaceWidget()`](/DOOM/linuxdoom-1.10/st_stuff.c#L752) and [`ST_calcPainOffset()`](/DOOM/linuxdoom-1.10/st_stuff.c#L729) manage the player's face animation on the status bar, reflecting health status and damage taken.

• Palette effects: [`ST_doPaletteStuff()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1000) handles color shifting effects when the player takes damage or picks up items.

• Cheat code handling: [`ST_Responder()`](/DOOM/linuxdoom-1.10/st_stuff.c#L518) also intercepts keyboard input to check for cheat code sequences using [`cht_CheckCheat()`](/DOOM/linuxdoom-1.10/m_cheat.c#L43).

The HUD system is initialized with [`ST_Init()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1466), which loads necessary graphics and data. [`ST_Start()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1444) and [`ST_Stop()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1456) handle setup and cleanup when entering or leaving a level.

### Menu System
References: `/DOOM/linuxdoom-1.10`

The menu system in DOOM is implemented primarily in `/DOOM/linuxdoom-1.10/st_stuff.c`. The file handles the display of various widgets and information on the status bar, including options and settings. Key components include:

- Initialization of menu widgets using functions like [`STlib_initNum()`](/DOOM/linuxdoom-1.10/st_lib.c#L66), [`STlib_initPercent()`](/DOOM/linuxdoom-1.10/st_lib.c#L163), [`STlib_initBinIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L245), and [`STlib_initMultIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L193).

- The [`ST_drawWidgets()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1054) function updates the display of these widgets based on the player's current state.

- User input for the menu is handled by the [`ST_Responder()`](/DOOM/linuxdoom-1.10/st_stuff.c#L518) function, which intercepts keyboard events and processes them accordingly.

- The [`ST_Ticker()`](/DOOM/linuxdoom-1.10/st_stuff.c#L988) function is called by the main game loop to update the menu state, such as animating elements or updating palette indicators.

- [`ST_Drawer()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1108) is responsible for rendering the menu on the screen, in both fullscreen and non-fullscreen modes.

- The [`ST_Start()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1444) function initializes the menu state when the console player spawns on each level.

- [`ST_Init()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1466) is called during game startup to set up the entire menu system.

The menu system uses two key enumerations:

1. [`st_stateenum_t`](/DOOM/linuxdoom-1.10/st_stuff.h#L64): Defines possible states for the status bar, including [`AutomapState`](/DOOM/linuxdoom-1.10/st_stuff.h#L61) and [`FirstPersonState`](/DOOM/linuxdoom-1.10/st_stuff.h#L62).
2. [`st_chatstateenum_t`](/DOOM/linuxdoom-1.10/st_stuff.h#L74): Defines states for the chat functionality, including [`StartChatState`](/DOOM/linuxdoom-1.10/st_stuff.h#L70), [`WaitDestState`](/DOOM/linuxdoom-1.10/st_stuff.h#L71), and [`GetChatState`](/DOOM/linuxdoom-1.10/st_stuff.h#L72).

The system also includes support for cheat codes, which are processed within the [`ST_Responder()`](/DOOM/linuxdoom-1.10/st_stuff.c#L518) function. Cheat codes are defined as arrays of characters, and the [`cht_CheckCheat()`](/DOOM/linuxdoom-1.10/m_cheat.c#L43) function verifies if a valid cheat code sequence has been entered.

### Input Handling
References: `/DOOM/linuxdoom-1.10`

Input handling for the HUD and menu systems is primarily managed through the [`ST_Responder()`](/DOOM/linuxdoom-1.10/st_stuff.c#L518) function in `/DOOM/linuxdoom-1.10/st_stuff.c`. This function intercepts keyboard input events and processes them for both the HUD and menu interactions.

Key aspects of the input handling:

• The function checks for cheat code sequences entered by the player, using [`cht_CheckCheat()`](/DOOM/linuxdoom-1.10/m_cheat.c#L43) to validate potential cheat inputs.

• HUD-specific inputs, such as toggling automap or changing weapon, are processed and the appropriate game state is updated.

• Menu navigation inputs are handled, allowing players to move through menu options and select items.

• Chat functionality is also managed here, with the function handling the transition between different chat states ([`StartChatState`](/DOOM/linuxdoom-1.10/st_stuff.h#L70), [`WaitDestState`](/DOOM/linuxdoom-1.10/st_stuff.h#L71), and [`GetChatState`](/DOOM/linuxdoom-1.10/st_stuff.h#L72)) as defined in the [`st_chatstateenum_t`](/DOOM/linuxdoom-1.10/st_stuff.h#L74) enumeration in `/DOOM/linuxdoom-1.10/st_stuff.h`.

The input handling system integrates closely with the game's event system, utilizing the [`event_t`](/DOOM/linuxdoom-1.10/d_event.h#L50) structure defined in `/DOOM/linuxdoom-1.10/d_event.h` to process various types of input events, including key presses, mouse movements, and joystick actions.

For more general input processing, the [`G_Responder()`](/DOOM/linuxdoom-1.10/g_game.c#L504) function in `/DOOM/linuxdoom-1.10/g_game.c` handles user input events and responds to them in the context of the overall game state, working in conjunction with the HUD and menu-specific input handling.

### Map Data Structures
References: `/DOOM/linuxdoom-1.10/doomdata.h`

The DOOM map structure is defined using several key data types in `/DOOM/linuxdoom-1.10/doomdata.h`:

- [`mapvertex_t`](/DOOM/linuxdoom-1.10/doomdata.h#L64): Represents map vertices with x and y coordinates.
- [`mapsidedef_t`](/DOOM/linuxdoom-1.10/doomdata.h#L78): Defines wall visual properties, including texture offsets and textures for top, bottom, and middle sections.
- [`maplinedef_t`](/DOOM/linuxdoom-1.10/doomdata.h#L93): Describes map lines using vertex references and property flags.
- [`mapsector_t`](/DOOM/linuxdoom-1.10/doomdata.h#L150): Defines map sectors with floor/ceiling heights, textures, light levels, and special properties.
- [`mapsubsector_t`](/DOOM/linuxdoom-1.10/doomdata.h#L158): Represents subsectors, which are lists of line segments generated by BSP.
- [`mapseg_t`](/DOOM/linuxdoom-1.10/doomdata.h#L171): Describes line segments created by BSP partitioning.
- [`mapnode_t`](/DOOM/linuxdoom-1.10/doomdata.h#L196): Represents nodes in the BSP tree for efficient visibility and collision detection.
- [`mapthing_t`](/DOOM/linuxdoom-1.10/doomdata.h#L210): Defines map objects such as monsters, items, and player start positions.

These structures form the foundation of DOOM's level representation, enabling efficient rendering, collision detection, and gameplay mechanics. The BSP-based approach allows for quick traversal and culling of the game world during rendering and collision checks.

### Level Loading and Management
References: `/DOOM/linuxdoom-1.10`

Level loading and management in DOOM is primarily handled by functions in `/DOOM/linuxdoom-1.10/p_setup.c` and `/DOOM/linuxdoom-1.10/g_game.c`. The process involves:

• Loading level data from WAD files using [`P_SetupLevel()`](/DOOM/linuxdoom-1.10/p_setup.c#L584) in [`p_setup.c`](/DOOM/linuxdoom-1.10/p_setup.c#L26). This function:
  - Initializes the level structure
  - Loads map geometry (vertices, linedefs, sectors)
  - Sets up BSP nodes for efficient rendering
  - Spawns map objects and items

• [`G_DoLoadLevel()`](/DOOM/linuxdoom-1.10/g_game.c#L445) in [`g_game.c`](/DOOM/linuxdoom-1.10/g_game.c#L25) coordinates the level loading process:
  - Calls [`P_SetupLevel()`](/DOOM/linuxdoom-1.10/p_setup.c#L584)
  - Initializes player states
  - Sets the sky texture
  - Prepares the game world for play

• During gameplay, the level state is managed through:
  - Thinker system: Updates game objects over time
  - Collision detection: Handled by functions like [`P_CheckPosition()`](/DOOM/linuxdoom-1.10/p_map.c#L375) and [`P_TryMove()`](/DOOM/linuxdoom-1.10/p_map.c#L451)
  - Sector special effects: Managed by functions in `/DOOM/linuxdoom-1.10/p_lights.c` for dynamic lighting

• Level transitions are handled by [`G_ExitLevel()`](/DOOM/linuxdoom-1.10/g_game.c#L1002) and [`G_SecretExitLevel()`](/DOOM/linuxdoom-1.10/g_game.c#L1009) in [`g_game.c`](/DOOM/linuxdoom-1.10/g_game.c#L25), which trigger the loading of the next level.

• The [`visplane_t`](/DOOM/linuxdoom-1.10/r_defs.h#L480) structure in `/DOOM/linuxdoom-1.10/r_plane.h` is used to represent visible floor and ceiling planes for efficient rendering.

• Sound management during gameplay is handled by functions in `/DOOM/linuxdoom-1.10/s_sound.c`, which adjust sound parameters based on the player's position in the level.

### Player Representation
References: `/DOOM/linuxdoom-1.10`

The [`player_t`](/DOOM/linuxdoom-1.10/d_player.h#L166) struct, defined in `/DOOM/linuxdoom-1.10/d_player.h`, represents the player character in DOOM. It contains various fields that store information about the player's state, including:

- [`playerstate_t`](/DOOM/linuxdoom-1.10/d_player.h#L62): An enumeration defining the player's current state ([`PST_LIVE`](/DOOM/linuxdoom-1.10/d_player.h#L56), [`PST_DEAD`](/DOOM/linuxdoom-1.10/d_player.h#L58), or [`PST_REBORN`](/DOOM/linuxdoom-1.10/d_player.h#L60)).
- Health and armor values.
- Weapon information:
  - [`readyweapon`](/DOOM/linuxdoom-1.10/d_player.h#L114): Currently equipped weapon.
  - [`pendingweapon`](/DOOM/linuxdoom-1.10/d_player.h#L117): Weapon to be switched to.
  - [`weaponowned`](/DOOM/linuxdoom-1.10/d_player.h#L119): Array indicating which weapons the player possesses.
- Ammo counts for different weapon types.
- Inventory items and power-ups.
- Movement-related fields:
  - [`viewz`](/DOOM/linuxdoom-1.10/d_player.h#L92): Player's view height.
  - [`viewheight`](/DOOM/linuxdoom-1.10/d_player.h#L94): Standard view height.
  - [`deltaviewheight`](/DOOM/linuxdoom-1.10/d_player.h#L96): Change in view height when moving.
  - [`bob`](/DOOM/linuxdoom-1.10/d_player.h#L98): Weapon bobbing offset.

The [`player_t`](/DOOM/linuxdoom-1.10/d_player.h#L166) struct also includes fields for tracking the player's score, kills, and other statistics. The [`psprites`](/DOOM/linuxdoom-1.10/d_player.h#L161) field is an array of [`pspdef_t`](/DOOM/linuxdoom-1.10/p_pspr.h#L75) structs that store information about the player's weapon sprites.

Player movement and input are handled through the [`ticcmd_t`](/DOOM/linuxdoom-1.10/d_ticcmd.h#L44) struct, which stores the player's input commands for each game tick. The [`G_BuildTiccmd()`](/DOOM/linuxdoom-1.10/g_game.c#L237) function in `/DOOM/linuxdoom-1.10/g_game.c` is responsible for building these input commands based on keyboard, mouse, and joystick input.

The [`P_PlayerThink()`](/DOOM/linuxdoom-1.10/p_user.c#L236) function, declared in `/DOOM/linuxdoom-1.10/p_local.h`, handles the player's thinking and decision-making processes, updating the player's state based on input and game events.

### Game State Management
References: `/DOOM/linuxdoom-1.10`

Game state in DOOM is managed through several key structures and functions:

• The [`gamestate`](/DOOM/linuxdoom-1.10/g_game.c#L98) variable tracks the overall state of the game, including menu, gameplay, and intermission states.

• Level transitions are handled by the [`G_DoCompleted()`](/DOOM/linuxdoom-1.10/g_game.c#L1020) function in `/DOOM/linuxdoom-1.10/g_game.c`. This function is called when a level is completed and manages the transition to the next level or victory sequence.

• Intermission data is stored in the [`wbstartstruct_t`](/DOOM/linuxdoom-1.10/d_player.h#L211) structure, defined in `/DOOM/linuxdoom-1.10/d_player.h`. This structure contains information about the current episode, including:
  - Previous and next level numbers
  - Maximum number of kills, items, and secrets
  - Par time for the level
  - Player index

• The [`wbplayerstruct_t`](/DOOM/linuxdoom-1.10/d_player.h#L185) structure, also in `/DOOM/linuxdoom-1.10/d_player.h`, stores individual player statistics for the intermission screen, such as:
  - Frags (kills in deathmatch)
  - Item count
  - Secret count
  - Time taken
  - Score

• The [`G_WorldDone()`](/DOOM/linuxdoom-1.10/g_game.c#L1147) function in `/DOOM/linuxdoom-1.10/g_game.h` is called to indicate that the current game world is finished, triggering the transition to the intermission or next level.

• The [`gameaction`](/DOOM/linuxdoom-1.10/g_game.c#L97) variable, defined in `/DOOM/linuxdoom-1.10/doomstat.h`, is used to queue up game state changes, such as loading a new level or starting a new game.

• Level loading is handled by [`G_DoLoadLevel()`](/DOOM/linuxdoom-1.10/g_game.c#L445) in `/DOOM/linuxdoom-1.10/g_game.c`, which sets up the new level, initializes player states, and prepares the game world.

### Player Input and Commands
References: `/DOOM/linuxdoom-1.10`

Player input processing and command handling in DOOM are primarily managed through the [`G_BuildTiccmd()`](/DOOM/linuxdoom-1.10/g_game.c#L237) function in `/DOOM/linuxdoom-1.10/g_game.c`. This function constructs a [`ticcmd_t`](/DOOM/linuxdoom-1.10/d_ticcmd.h#L44) structure, which represents the player's input command for a single game tick. The process involves:

- Reading input from various sources (keyboard, mouse, joystick) and translating it into game actions.
- Applying speed modifiers based on the game's difficulty settings.
- Handling movement, strafing, and turning actions.
- Processing button presses for actions like shooting or using items.

The [`G_Responder()`](/DOOM/linuxdoom-1.10/g_game.c#L504) function in the same file handles user input events and determines appropriate responses. It processes events such as key presses, weapon changes, and cheat code inputs.

Key aspects of the input handling system:

- The [`event_t`](/DOOM/linuxdoom-1.10/d_event.h#L50) structure in `/DOOM/linuxdoom-1.10/d_event.h` represents different types of input events.
- The [`buttoncode_t`](/DOOM/linuxdoom-1.10/d_event.h#L100) enum defines various button codes for player actions.
- The [`P_PlayerThink()`](/DOOM/linuxdoom-1.10/p_user.c#L236) function in `/DOOM/linuxdoom-1.10/p_local.h` processes player thinking and decision-making based on input.

The game loop in [`D_DoomLoop()`](/DOOM/linuxdoom-1.10/d_main.c#L354) calls [`I_StartTic()`](/DOOM/linuxdoom-1.10/i_video.c#L309) and [`I_StartFrame()`](/DOOM/linuxdoom-1.10/i_video.c#L183) from `/DOOM/linuxdoom-1.10/i_system.h` to handle synchronous operations and post events using [`D_PostEvent()`](/DOOM/linuxdoom-1.10/d_main.c#L150).

## Networking
References: `/DOOM/linuxdoom-1.10`

Networking in DOOM is handled primarily through the [`d_net.c`](/DOOM/linuxdoom-1.10/d_net.c#L26) and [`d_net.h`](/DOOM/linuxdoom-1.10/d_net.h#L0) files. The core data structures for network communication are:

- [`doomdata_t`](/DOOM/ipx/IPXNET.H#L7): Defines the format of network packet data, including checksums, retransmission information, and ticcmds (game commands) for each player.
- [`doomcom_t`](/DOOM/ipx/DOOMNET.H#L47): Holds overall network communication information, such as the number of nodes (players) and game settings.

Key networking functions include:

- [`NetUpdate()`](/DOOM/linuxdoom-1.10/d_net.c#L368): Creates new ticcmds and broadcasts them to other players.
- [`D_QuitNetGame()`](/DOOM/linuxdoom-1.10/d_net.c#L602): Broadcasts special packets to notify other players of game exit.
- [`TryRunTics()`](/DOOM/linuxdoom-1.10/d_net.c#L636): Ensures synchronization across players by running the appropriate number of game ticks.

Game state synchronization is achieved through:

- Ticcmds: Small data structures containing player input and game state changes.
- Broadcast and processing of ticcmds to maintain consistent gameplay across networked players.

The networking system uses a peer-to-peer model, where each player's game instance communicates directly with others. This approach helps minimize latency and ensures responsive gameplay.

Packet handling involves:

- Checksums for data integrity verification.
- Retransmission information to handle packet loss.
- Sequencing to maintain proper order of game events.

The [`NetUpdate()`](/DOOM/linuxdoom-1.10/d_net.c#L368) function is called regularly to:

- Generate new ticcmds based on local player input.
- Broadcast these ticcmds to other players in the network.
- Process incoming ticcmds from other players.

This constant exchange of ticcmds allows all players' game states to remain synchronized, even with varying network conditions.

## Rendering and Graphics
References: `/DOOM/linuxdoom-1.10`

The rendering pipeline in DOOM is implemented across several files, with key components in `/DOOM/linuxdoom-1.10/r_data.c`, `/DOOM/linuxdoom-1.10/r_bsp.h`, and `/DOOM/linuxdoom-1.10/r_segs.c`.

Texture management is handled in `/DOOM/linuxdoom-1.10/r_data.c`:

- [`R_GenerateLookup()`](/DOOM/linuxdoom-1.10/r_data.c#L296) analyzes texture definitions and creates lookup tables for efficient rendering.
- [`R_GenerateComposite()`](/DOOM/linuxdoom-1.10/r_data.c#L228) combines individual patches into a single texture.
- [`R_GetColumn()`](/DOOM/linuxdoom-1.10/r_data.c#L383) retrieves texture columns, either from cache or by generating them on-the-fly.

The Binary Space Partitioning (BSP) tree traversal, crucial for efficient rendering, is managed in `/DOOM/linuxdoom-1.10/r_bsp.h`:

- [`R_RenderBSPNode()`](/DOOM/linuxdoom-1.10/r_bsp.c#L552) recursively traverses the BSP tree to determine visible segments.

Wall segment rendering is handled in `/DOOM/linuxdoom-1.10/r_segs.c`:

- [`R_RenderSegLoop()`](/DOOM/linuxdoom-1.10/r_segs.c#L206) is the core loop for drawing wall segments, calculating texture coordinates, lighting, and clipping information.
- [`R_RenderMaskedSegRange()`](/DOOM/linuxdoom-1.10/r_segs.c#L103) handles rendering of masked textures for elements like doors and windows.

Plane (floor and ceiling) rendering is managed in `/DOOM/linuxdoom-1.10/r_plane.h`:

- [`R_FindPlane()`](/DOOM/linuxdoom-1.10/r_plane.c#L218) and [`R_CheckPlane()`](/DOOM/linuxdoom-1.10/r_plane.c#L266) handle the creation and management of visible planes.
- [`R_MapPlane()`](/DOOM/linuxdoom-1.10/r_plane.c#L121) maps planes to the screen.
- [`R_DrawPlanes()`](/DOOM/linuxdoom-1.10/r_plane.c#L367) is the main entry point for rendering all visible planes.

The status bar, which displays player information, is implemented in `/DOOM/linuxdoom-1.10/st_stuff.c`:

- Various widgets (health, armor, ammo) are initialized and updated.
- [`ST_drawWidgets()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1054) updates the display of these widgets based on the player's current state.
- [`ST_updateFaceWidget()`](/DOOM/linuxdoom-1.10/st_stuff.c#L752) handles the animation of the player's face on the status bar.

Screen wipe effects for transitions are implemented in `/DOOM/linuxdoom-1.10/f_wipe.c`:

- Different wipe effects (color transform, melt) are implemented using functions like [`wipe_doColorXForm()`](/DOOM/linuxdoom-1.10/f_wipe.c#L83) and [`wipe_doMelt()`](/DOOM/linuxdoom-1.10/f_wipe.c#L172).
- [`wipe_ScreenWipe()`](/DOOM/linuxdoom-1.10/f_wipe.c#L262) coordinates the execution of these effects.

## Sound System
References: `/DOOM/linuxdoom-1.10`, `/DOOM/sndserv`

The sound system in DOOM is implemented primarily in `/DOOM/linuxdoom-1.10/s_sound.c` and `/DOOM/linuxdoom-1.10/s_sound.h`. It handles sound effects and music playback, providing functions for initialization, volume control, and spatial audio.

Key components:

- Sound Initialization: [`S_Init()`](/DOOM/linuxdoom-1.10/s_sound.c#L161) sets up the sound system, configuring default volumes and allocating internal channels.

- Sound Effect Management:
  • [`S_StartSoundAtVolume()`](/DOOM/linuxdoom-1.10/s_sound.c#L255) plays a sound effect, considering origin, pitch, and priority.
  • [`S_StopSound()`](/DOOM/linuxdoom-1.10/s_sound.c#L471) halts a playing sound effect.
  • [`S_AdjustSoundParams()`](/DOOM/linuxdoom-1.10/s_sound.c#L753) modifies volume, stereo separation, and pitch based on listener position.

- Channel Management: [`S_getChannel()`](/DOOM/linuxdoom-1.10/s_sound.c#L828) finds or frees a channel for new sounds, while [`S_StopChannel()`](/DOOM/linuxdoom-1.10/s_sound.c#L708) stops sounds on specific channels.

- Music Control:
  • [`S_StartMusic()`](/DOOM/linuxdoom-1.10/s_sound.c#L644) and [`S_ChangeMusic()`](/DOOM/linuxdoom-1.10/s_sound.c#L650) start new music tracks.
  • [`S_StopMusic()`](/DOOM/linuxdoom-1.10/s_sound.c#L689) halts the current music.
  • [`S_PauseSound()`](/DOOM/linuxdoom-1.10/s_sound.c#L497) and [`S_ResumeSound()`](/DOOM/linuxdoom-1.10/s_sound.c#L506) pause/resume all audio.

- Sound Update: [`S_UpdateSounds()`](/DOOM/linuxdoom-1.10/s_sound.c#L519) refreshes all active sound effects, adjusting parameters and stopping inaudible sounds.

The system uses a thinker-based approach for efficient updates and provides spatial audio by adjusting sound parameters based on game world positions. Volume control is handled separately for music and sound effects through [`S_SetMusicVolume()`](/DOOM/linuxdoom-1.10/s_sound.c#L616) and [`S_SetSfxVolume()`](/DOOM/linuxdoom-1.10/s_sound.c#L631).

## User Interface
References: `/DOOM/linuxdoom-1.10`

The Heads-Up Display (HUD) and menu system are implemented primarily in `/DOOM/linuxdoom-1.10/st_stuff.c` and `/DOOM/linuxdoom-1.10/st_stuff.h`. The HUD displays crucial information such as player health, armor, ammunition, and weapons.

Key components of the user interface include:

• Status Bar Widgets: Defined and initialized using functions like [`STlib_initNum()`](/DOOM/linuxdoom-1.10/st_lib.c#L66), [`STlib_initPercent()`](/DOOM/linuxdoom-1.10/st_lib.c#L163), [`STlib_initBinIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L245), and [`STlib_initMultIcon()`](/DOOM/linuxdoom-1.10/st_lib.c#L193). These widgets display various player stats.

• Face Animation: The [`ST_updateFaceWidget()`](/DOOM/linuxdoom-1.10/st_stuff.c#L752) function updates the player's face on the status bar based on health, damage, and other factors. [`ST_calcPainOffset()`](/DOOM/linuxdoom-1.10/st_stuff.c#L729) calculates the appropriate offset for the face animation.

• Palette and Color Shifting: [`ST_doPaletteStuff()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1000) handles palette and color shifting effects, such as red/gold shifts when the player takes damage or picks up items.

• Cheat Code Handling: [`ST_Responder()`](/DOOM/linuxdoom-1.10/st_stuff.c#L518) intercepts keyboard input events to check for cheat code sequences using [`cht_CheckCheat()`](/DOOM/linuxdoom-1.10/m_cheat.c#L43).

The menu system is likely implemented in separate files, but specific details are not provided in the given summaries.

The [`ST_Ticker()`](/DOOM/linuxdoom-1.10/st_stuff.c#L988) function updates the status bar state, while [`ST_Drawer()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1108) renders the status bar on screen. [`ST_Start()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1444) initializes the status bar when a player spawns, and [`ST_Init()`](/DOOM/linuxdoom-1.10/st_stuff.c#L1466) sets up the status bar system during game startup.

The HUD can be in different states, defined by [`st_stateenum_t`](/DOOM/linuxdoom-1.10/st_stuff.h#L64), including [`AutomapState`](/DOOM/linuxdoom-1.10/st_stuff.h#L61) and [`FirstPersonState`](/DOOM/linuxdoom-1.10/st_stuff.h#L62). Chat functionality is also integrated, with states defined in [`st_chatstateenum_t`](/DOOM/linuxdoom-1.10/st_stuff.h#L74).

French language support for the user interface is provided through string constants defined in `/DOOM/linuxdoom-1.10/d_french.h`, covering various game messages, menu items, and HUD elements.

## Game World and Level Management
References: `/DOOM/linuxdoom-1.10`

The game world in DOOM is represented using various data structures defined in `/DOOM/linuxdoom-1.10/r_defs.h`. Key structures include:

- [`vertex_t`](/DOOM/linuxdoom-1.10/r_defs.h#L76): Represents 2D vertices with x and y coordinates.
- [`sector_t`](/DOOM/linuxdoom-1.10/r_defs.h#L135): Defines sectors with floor/ceiling heights, textures, lighting, and lists of objects.
- [`side_t`](/DOOM/linuxdoom-1.10/r_defs.h#L161): Represents sidedefs with texture offsets and indices.
- [`line_t`](/DOOM/linuxdoom-1.10/r_defs.h#L215): Defines linedefs with vertices, flags, and sector references.
- [`subsector_t`](/DOOM/linuxdoom-1.10/r_defs.h#L233): Contains lists of segs that define convex BSP leaves.
- [`seg_t`](/DOOM/linuxdoom-1.10/r_defs.h#L258): Represents line segments within subsectors.
- [`node_t`](/DOOM/linuxdoom-1.10/r_defs.h#L279): Defines nodes in the BSP tree for visibility determination.

Level loading is handled primarily in `/DOOM/linuxdoom-1.10/p_setup.c`. The process involves:

- Loading map data from WAD files.
- Constructing the BSP tree for efficient rendering.
- Initializing sectors, linedefs, and other map elements.
- Setting up the blockmap for collision detection.

The [`P_SetupLevel()`](/DOOM/linuxdoom-1.10/p_setup.c#L584) function in `/DOOM/linuxdoom-1.10/p_setup.c` is responsible for loading and setting up a new level. It performs the following tasks:

- Loads the level data from the appropriate WAD file.
- Initializes the map structures (vertices, sectors, lines, etc.).
- Builds the BSP tree for rendering optimization.
- Creates the blockmap for efficient collision detection.
- Spawns map objects and initializes their states.

The [`R_PrecacheLevel()`](/DOOM/linuxdoom-1.10/r_data.c#L743) function in `/DOOM/linuxdoom-1.10/r_data.c` preloads graphics for the current level to optimize rendering performance. It caches textures, flats, and sprites used in the level.

The game uses a Binary Space Partitioning (BSP) tree for efficient rendering and collision detection. The BSP tree is traversed during rendering to determine which walls and objects are visible from the player's current position.

## Player and Game State
References: `/DOOM/linuxdoom-1.10`

The [`player_t`](/DOOM/linuxdoom-1.10/d_player.h#L166) struct in `/DOOM/linuxdoom-1.10/d_player.h` represents the player character, storing information such as health, armor, weapons, and inventory. It includes fields for the player's current state ([`playerstate_t`](/DOOM/linuxdoom-1.10/d_player.h#L62)), input commands ([`ticcmd_t`](/DOOM/linuxdoom-1.10/d_ticcmd.h#L44)), and visual representation.

Player state management:

- [`P_PlayerThink()`](/DOOM/linuxdoom-1.10/p_user.c#L236) handles the player's thinking and decision-making processes.
- [`P_SetupPsprites()`](/DOOM/linuxdoom-1.10/p_pspr.c#L831) and [`P_MovePsprites()`](/DOOM/linuxdoom-1.10/p_pspr.c#L851) manage the player's weapon sprites.
- [`P_DeathThink()`](/DOOM/linuxdoom-1.10/p_user.c#L182) handles player behavior when dead.

Game state is primarily managed through:

- Global variables in `/DOOM/linuxdoom-1.10/doomstat.h` store game mode, skill level, episode, and map information.
- [`gameaction`](/DOOM/linuxdoom-1.10/g_game.c#L97) determines pending game actions like loading levels or starting new games.
- [`gamestate`](/DOOM/linuxdoom-1.10/g_game.c#L98) tracks the current state (e.g., level, intermission, finale).

Key game state functions:

- [`G_DoLoadLevel()`](/DOOM/linuxdoom-1.10/g_game.c#L445) sets up a new level, initializing player states and the game world.
- [`G_PlayerReborn()`](/DOOM/linuxdoom-1.10/g_game.c#L800) resets player state on respawn.
- [`G_DoCompleted()`](/DOOM/linuxdoom-1.10/g_game.c#L1020) handles level completion and transitions.

The [`wbplayerstruct_t`](/DOOM/linuxdoom-1.10/d_player.h#L185) and [`wbstartstruct_t`](/DOOM/linuxdoom-1.10/d_player.h#L211) structs in `/DOOM/linuxdoom-1.10/d_player.h` store intermission statistics and level transition information.

Demo recording and playback:

- [`G_RecordDemo()`](/DOOM/linuxdoom-1.10/g_game.c#L1530) and [`G_PlayDemo()`](/DOOM/linuxdoom-1.10/g_game.c#L1571) manage demo recording and playback.
- [`G_ReadDemoTiccmd()`](/DOOM/linuxdoom-1.10/g_game.c#L1491) reads player input commands from demo data.

The game loop in [`D_DoomLoop()`](/DOOM/linuxdoom-1.10/d_main.c#L354) (`/DOOM/linuxdoom-1.10/d_main.c`) coordinates game state updates, including:

- Processing events with [`D_ProcessEvents()`](/DOOM/linuxdoom-1.10/d_main.c#L161)
- Updating game tick with [`G_Ticker()`](/DOOM/linuxdoom-1.10/g_game.c#L605)
- Rendering the current game state
