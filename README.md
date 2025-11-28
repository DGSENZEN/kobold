# kobold
A VTT made for newbies and experienced players alike to bring their stories and games to life


# Virtual Tabletop Feature Roadmap
## D&D VTT

**Tech Stack**: Vue.js Frontend | Rust Backend | WebSocket for Real-time | PostgreSQL for Persistence

---

## PHASE 1: CORE FOUNDATION (MVP - 3-4 months)

### 1.1 Backend Architecture (Rust)

#### WebSocket Server Infrastructure
- **Native WebSocket Implementation** (not Socket.io - following Foundry V12 direction)
  - Binary protocol support for efficient token/map updates
  - Room-based connection management (isolated game sessions)
  - Automatic reconnection handling with state recovery
  - Connection state: connecting, connected, disconnecting, error
  - Per-room WebSocket instances

#### Data Layer
- **Document-based Data Model**
  - Base Document abstract class with CRUD operations
  - Scene documents (maps, grids, lighting config)
  - Actor documents (characters, NPCs, monsters)
  - Item documents (equipment, spells, features)
  - Token documents (scene instances of actors)
  - User documents and permissions
  - Combat documents (encounter state)
  - Journal documents (notes, handouts)

- **Hierarchical Permission System**
  - Ownership levels: None, Limited, Observer, Owner, Game Master
  - Per-document permission inheritance
  - Token-level visibility control

- **Database Layer** (PostgreSQL with LevelDB-style optimization)
  - Document storage with JSON fields for flexible schemas
  - Binary blob storage for assets (images, audio)
  - Efficient indexing for searches
  - Transaction support for atomic updates
  - Migration system for schema evolution

#### Real-time Synchronization Engine
- **State Broadcast System**
  - Differential updates (only changed fields)
  - Batch update buffering (100ms window)
  - Priority queuing (token movement > lighting > fog)
  - Conflict resolution: last-write-wins with timestamps
  - Operation log for rollback/debugging

- **Pub/Sub Architecture** (Redis for multi-server scaling)
  - Room-specific channels
  - User presence tracking
  - Typing indicators / active cursors
  - Live collaborative editing

### 1.2 Frontend Architecture (Vue.js)

#### Core UI Framework
- **Modular Component System**
  - Sidebar: Actors, Items, Scenes, Combat, Journal, Playlists
  - Canvas renderer (WebGL via PixiJS)
  - Drag-and-drop system
  - Hotbar for macros (10 visible, 50 total slots)
  - Context menus (right-click actions)
  - Modal dialogs and forms

- **State Management** (Pinia)
  - Canvas state (viewport, zoom, pan)
  - Active scene and tokens
  - UI panel visibility
  - User settings and preferences
  - Real-time document cache

#### Canvas Rendering System (PixiJS/WebGL)
- **Multi-layer Architecture** (following Foundry pattern)
  - Background layer (map image)
  - Grid layer (square, hex, gridless)
  - Drawings layer (persistent shapes)
  - Tiles layer (map decorations, furniture)
  - Tokens layer (characters, NPCs)
  - Lighting layer (light sources)
  - Walls layer (vision blocking, doors)
  - Fog layer (fog of war)
  - Interface layer (UI overlays, cursors)
  - Effects layer (weather, animations)

- **Viewport Management**
  - Pan: click-drag or arrow keys
  - Zoom: mouse wheel, pinch, or buttons (50%-300%)
  - Smooth easing transitions
  - Grid snapping toggle (Alt key override)
  - Ruler measurement (distance calculation)
  - Waypoint system for movement paths

- **Performance Optimization**
  - Object culling (off-screen rendering skip)
  - Texture atlasing for tokens
  - LOD for large scenes
  - Sprite pooling/recycling
  - Lazy loading of assets
  - Progressive rendering for large maps

---

## PHASE 2: MAP & SCENE MANAGEMENT (Month 2-3)

### 2.1 Scene System

#### Scene Configuration
- **Basic Settings**
  - Name, description, thumbnail
  - Background image with dimensions
  - Initial viewport position
  - Navigation bar visibility
  - Padding/border percentage

- **Grid Configuration** (critical feature)
  - Grid types: Square, Hexagonal (rows/columns), Gridless
  - Grid size in pixels (50-300px recommended)
  - Grid color and opacity (0-100%)
  - Grid offset (X/Y for alignment)
  - Scale: distance per grid unit (5ft default)
  - Diagonal movement rules (5-10-5 or 5-5-5)

#### Grid Alignment Tool
- **Smart Alignment System** (Owlbear inspiration)
  - Visual alignment rulers with 4-point calibration
  - Precision rails for sub-pixel adjustment
  - Automatic grid detection from filename (e.g., "map_40x28.jpg")
  - Manual controls: columns, rows, offset X/Y
  - Live preview overlay with orange guide lines
  - Save alignment as template for reuse

- **Import Optimization**
  - Auto-detect if image has pre-rendered grid
  - Suggest crop to align with corner
  - Validate pixel dimensions divisible by grid size
  - Support for common map creator formats (Dungeondraft, Inkarnate)

### 2.2 Asset Management

#### File Storage System
- **Cloud Storage Integration**
  - S3-compatible object storage
  - User quotas and usage tracking
  - CDN delivery for assets
  - Automatic image optimization (WebP conversion)
  - Lazy loading with placeholder thumbnails

- **Asset Library Interface** (Owlbear Asset Manager pattern)
  - Scene tab, Map tab, Token tab, Tile tab
  - Search and filter by tags
  - Folder organization with drag-drop
  - Batch import wizard
  - Preview modal with metadata
  - Usage tracking (which scenes use this asset)

#### Media Handling
- **Image Support**
  - Formats: PNG, JPG, WebP, AVIF, SVG
  - Automatic format conversion for optimization
  - Transparent backgrounds for tokens
  - Multi-frame animated images

- **Video Support** (animated maps/tokens)
  - Formats: WebM (preferred), MP4, M4V
  - Looping control
  - Playback speed adjustment
  - Frame-by-frame scrubbing

- **Audio Support**
  - Formats: MP3, OGG, WAV, FLAC
  - Streaming vs. preload options
  - Volume normalization
  - Playlist integration

---

## PHASE 3: TOKEN & ACTOR SYSTEM (Month 3-4)

### 3.1 Token Management

#### Token Properties
- **Visual Attributes**
  - Token image with rotation (0-360°)
  - Scale multiplier (0.5x - 3x)
  - Size: Tiny, Small, Medium, Large, Huge, Gargantuan (1x1 to 4x4 grid)
  - Tint color overlay
  - Mirror horizontal/vertical
  - Elevation (Z-order for overlapping)
  - Opacity for transparency

- **Gameplay Attributes**
  - Name display (always, hover, owner, GM)
  - Health bar display (bar1, bar2, bar3)
  - Status effects/conditions (icons)
  - Vision radius and angle
  - Light emission radius and color
  - Movement speed
  - Disposition: Friendly, Neutral, Hostile, Secret

#### Token Interaction
- **Movement System**
  - Click-drag with pathfinding
  - Waypoint mode (Q key) for distance measurement
  - Snap to grid toggle (Alt key)
  - Wall collision detection
  - Movement history (last path highlighted)
  - Group movement for multiple selected tokens

- **Token Context Menu** (right-click)
  - Toggle visibility (hidden from players)
  - Change elevation
  - Assign ownership
  - Add/remove status effects
  - Toggle light source
  - Lock position
  - Duplicate
  - Delete

### 3.2 Actor System

#### Actor Types
- **Character** (player-controlled)
  - Full character sheet integration
  - Resource tracking (HP, spell slots, abilities)
  - Inventory management
  - Skill proficiencies
  - Biography and appearance

- **NPC** (non-player character)
  - Simplified stat blocks
  - CR calculation
  - Action economy
  - Loot tables

- **Vehicle**
  - Crew positions
  - Vehicle-specific stats
  - Component damage

#### Character Sheet Framework
- **Modular Sheet System** (game system agnostic)
  - HTML/CSS templates
  - Reactive data binding
  - Computed fields (modifiers, totals)
  - Roll integration from sheet fields
  - Drag-drop item management
  - Image customization

- **D&D 5e Implementation** (Phase 1 target)
  - Standard ability scores
  - Skills with proficiency
  - Saving throws
  - Class features and spells
  - Equipment and encumbrance
  - Conditions and exhaustion

---

## PHASE 4: DYNAMIC LIGHTING & FOG OF WAR (Month 4-5)

### 4.1 Vision System

#### Token Vision Configuration
- **Per-Token Settings**
  - Vision enabled toggle
  - Vision range (bright/dim light distances)
  - Vision angle (360° or limited cone)
  - Vision color tint
  - Vision modes: Normal, Darkvision, Blindsight, Tremorsense, Truesight
  - Detection modes for invisible creatures

- **Scene-Level Vision**
  - Global illumination toggle and intensity
  - Darkness level slider (0-1, affects ambient lighting)
  - Token vision required (hide unrevealed areas)
  - Fog exploration enabled
  - Fog reset button

#### Fog of War System
- **Exploration Tracking**
  - Per-user fog exploration progress
  - Saved to database (persistent across sessions)
  - Explored vs. currently visible distinction
  - Dimmed/darkened explored areas
  - Manual fog reveal tools (rectangle, polygon, circle)

- **Hybrid Fog Modes** (GM flexibility)
  - **Dynamic Fog** (Foundry style): Auto-reveals based on token vision
  - **Manual Fog** (Owlbear/Roll20 style): GM manually reveals areas
  - **Hybrid**: Dynamic with manual override tools

### 4.2 Lighting Engine

#### Light Sources
- **Ambient Lights** (scene lights)
  - Position (X, Y)
  - Radius (bright/dim)
  - Color and intensity
  - Animated effects: Torch, Pulse, Wave, Fog, etc.
  - Animation speed and intensity
  - Darkness threshold activation
  - Walls block light toggle

- **Token Lights** (carried light sources)
  - Inherit token position
  - Same configuration as ambient lights
  - Auto-update on token movement
  - Toggle on/off via token config

#### Advanced Lighting Features
- **Light Animations** (PixiJS shaders)
  - Torch flicker
  - Pulsing aura
  - Rotating beams
  - Swirling fog
  - Color shift
  - Black hole (negative light)

- **Performance Optimization**
  - Light culling (off-screen)
  - Shadow caching
  - WebGL shader optimization
  - Adaptive quality settings

### 4.3 Walls & Doors

#### Wall System
- **Wall Types**
  - Normal: blocks movement and vision
  - Terrain: blocks movement, not vision
  - Invisible: blocks vision, not movement
  - Ethereal: blocks neither (for organization)

- **Door Types**
  - Normal door (open/close toggle)
  - Secret door (hidden until discovered)
  - Locked door (requires key/check)
  - Double door
  - Sliding door
  - Door states: Closed, Open, Locked

- **Wall Drawing Tool**
  - Click-to-place points
  - Snap to grid
  - Chain mode for continuous walls
  - Rectangle and circle presets
  - Wall type selection
  - One-way vision toggle

---

## PHASE 5: DICE ROLLING & MACROS (Month 5-6)

### 5.1 Dice System

#### Dice Notation Engine
- **Core Notation** (standard dice expressions)
  - Basic: `2d6`, `1d20+5`, `3d8-2`
  - Multiple dice types: `1d20+2d6+3`
  - Math operations: `+`, `-`, `*`, `/`, `()`
  - Negative dice: `-1d4`

- **Dice Modifiers** (Foundry comprehensive system)
  - **Keep/Drop**: `4d6k3` (keep highest 3), `4d6dl` (drop lowest)
  - **Reroll**: `1d20r<3` (reroll once if <3), `4d6rr1` (reroll 1s repeatedly)
  - **Exploding**: `1d6x` (explode on max), `1d6x>5` (explode on 5-6)
  - **Minimum/Maximum**: `1d20min10`, `1d6max4`
  - **Success Counting**: `10d10>7` (count successes), `5d10<3` (count failures)
  - **Conditional**: `1d20=20` (check for crit)

#### Roll Execution
- **Roll Modes**
  - Public Roll (visible to all)
  - Private GM Roll (GM + roller see)
  - Blind GM Roll (only GM sees)
  - Self Roll (only roller sees)
  - Whisper to specific users

- **Inline Rolls** (in chat/journals)
  - Immediate: `[[1d20+5]]` auto-evaluates
  - Deferred: `[[/roll 1d20+5]]` creates roll button

- **Roll Display**
  - Individual die results with highlighting
  - Critical success/failure indication
  - Modifier breakdown
  - Formula and total prominently displayed
  - Flavor text support

### 5.2 Macro System

#### Macro Types
- **Chat Macros** (simple text/rolls)
  - Send text to chat
  - Execute dice rolls
  - Display images
  - Play sounds

- **Script Macros** (JavaScript API access)
  - Full API access (with permissions)
  - Manipulate actors, items, scenes
  - Create automation workflows
  - Custom UI dialogs

#### Macro Interface
- **Macro Directory**
  - Folder organization
  - Search and filter
  - Import/export macros
  - Share to compendium
  - Permission settings

- **Macro Hotbar**
  - 10 visible slots per page
  - 5 pages (50 total slots)
  - Drag-drop from directory
  - Keyboard shortcuts (1-0 keys)
  - Page switching
  - Execute via `/macro` command

#### Example Macro Use Cases
- Quick weapon attack (to-hit + damage)
- Spell slot consumption
- Condition toggles
- Heal/damage application
- Ability check with modifiers
- Initiative roller

---

## PHASE 6: COMBAT TRACKER (Month 6-7)

### 6.1 Combat Encounter Management

#### Combat Tracker UI
- **Combatant List**
  - Initiative order (descending)
  - Name and thumbnail
  - Initiative value (editable)
  - HP bar and current/max display
  - Active effects/conditions icons
  - Defeated status

- **Turn Management**
  - Current turn highlighted
  - Begin combat button
  - Next turn button
  - Previous turn button
  - End combat button
  - Round counter

- **Combatant Controls**
  - Roll initiative (individual or group)
  - Roll all NPCs
  - Roll all PCs
  - Hide from players
  - Mark defeated
  - Remove from combat

#### Initiative System
- **Roll Methods**
  - Standard: roll for each combatant
  - Group initiative: same roll for groups
  - Fixed initiative: pre-set values
  - Manual entry

- **Initiative Formulas** (system-specific)
  - D&D 5e: `1d20 + DEX modifier`
  - Custom formula support
  - Advantage/disadvantage

- **Tie-breakers**
  - Highest ability score (DEX)
  - Re-roll
  - Alphabetical
  - Token ID

### 6.2 Advanced Combat Features

#### Turn Tracking
- **Turn Indicators**
  - Token border highlight (current turn)
  - Pulsing animation
  - Movement path preview
  - Used action indicators

- **Round-based Effects**
  - Duration tracking (rounds, turns)
  - Auto-decrement on turn/round
  - Expiration notifications
  - Concentration checks

#### Combat Automation
- **Turn Workflows**
  - Start-of-turn hooks (regeneration, DoT)
  - End-of-turn hooks (effect expiration)
  - Round start/end hooks
  - Custom automation scripts

- **Combat Extensions** (optional modules)
  - Group initiative mode
  - Phase-based combat (Marvel, Star Trek)
  - Simultaneous turns
  - Reverse initiative order
  - Hidden initiative for NPCs

---

## PHASE 7: AUDIO/VIDEO & COLLABORATION (Month 7-8)

### 7.1 Audio System

#### Ambient Audio
- **Playlist Management**
  - Create/edit playlists
  - Drag-drop audio files
  - Loop/shuffle controls
  - Volume per track
  - Fade in/out durations

- **Scene Audio** (automatic triggers)
  - Background music per scene
  - Ambient sounds (wind, water, fire)
  - Auto-play on scene load
  - Volume zones

#### Audio Sharing
- **DM Audio Broadcast** (Owlbear pattern)
  - Share system audio to players
  - YouTube/Spotify integration
  - Volume control
  - Mute/solo tracks

- **Sound Effects**
  - Trigger sounds via macros
  - Spatial audio (distance-based volume)
  - One-shot vs. looping
  - Audio library (door, sword, spell, etc.)

### 7.2 Voice/Video Integration

#### Built-in WebRTC (Optional)
- **Peer-to-Peer Architecture**
  - Simple-Peer for WebRTC
  - STUN/TURN server support
  - Camera and microphone selection
  - Audio-only mode
  - Push-to-talk option
  - Voice activation threshold

- **AV Dock**
  - Resizable video tiles
  - Hide/show individual feeds
  - Minimize to audio-only
  - Connection status indicators

#### Third-Party Integration
- **Discord Rich Presence**
  - Show current game/scene
  - Invite links
  - Player status

- **External Voice Recommendations**
  - Discord (most common)
  - Zoom, Google Meet, Teams
  - Separate audio layer (simplicity)

---

## PHASE 8: GAME SYSTEM INTEGRATION (Month 8-10)

### 8.1 System Architecture

#### System Abstraction Layer
- **System Definition**
  - `system.json` manifest file
  - Name, version, compatibility
  - Template data models
  - Style overrides

- **Data Templates**
  - Actor templates (character, NPC, vehicle)
  - Item templates (weapon, armor, spell, consumable)
  - Custom attributes and resources

#### D&D 5e System (Reference Implementation)

**Actor Data Model:**
- Abilities: STR, DEX, CON, INT, WIS, CHA
- Skills with proficiency bonuses
- Saving throws
- HP, AC, initiative modifiers
- Class, level, proficiency bonus
- Movement speeds
- Senses (darkvision, etc.)

**Item Data Model:**
- Weapons: damage dice, type, properties
- Armor: AC calculation, stealth disadvantage
- Spells: level, school, components, description
- Features: class/racial abilities
- Consumables: charges, uses

**Roll Integration:**
- Ability checks: `1d20 + ability mod + proficiency`
- Attack rolls: `1d20 + ability mod + proficiency`
- Damage rolls: weapon dice + ability mod
- Saving throws: `1d20 + ability mod + proficiency`
- Skill checks: `1d20 + skill mod`

### 8.2 Compendium System

#### Compendium Packs
- **Types**
  - Actors (monsters, NPCs)
  - Items (equipment, spells)
  - Scenes (pre-made maps)
  - Journals (lore, rules)
  - Roll tables (loot, encounters)
  - Macros (automation)

- **Pack Management**
  - Create/edit packs
  - Import/export
  - Search and filter
  - Drag-to-world
  - Lock/unlock for editing

#### Official Content
- **SRD Integration**
  - Monsters (Basic Rules)
  - Spells (Basic Rules)
  - Equipment and magic items
  - Conditions and rules

- **Premium Content** (if licensed)
  - Adventure modules with pre-built scenes
  - Monster Manual imports
  - Xanathar's, Tasha's, etc.

---

## PHASE 9: JOURNAL & ORGANIZATION (Month 10-11)

### 9.1 Journal System

#### Journal Entries
- **Rich Text Editor** (ProseMirror)
  - Text formatting (bold, italic, underline)
  - Headers (H1-H6)
  - Lists (ordered, unordered)
  - Tables with editing
  - Images (inline or reference)
  - Links to actors, items, scenes
  - Collapsible sections

- **Journal Pages** (multi-page documents)
  - Text pages
  - Image pages
  - PDF pages
  - Map pages (embedded scenes)

#### Organization Features
- **Folder Hierarchy**
  - Nested folders
  - Color coding
  - Collapse/expand
  - Drag-drop reorganization

- **Sharing & Permissions**
  - Show to players (share button)
  - Observer permissions
  - Private notes (GM only)

### 9.2 Drawing & Annotation Tools

#### Drawing Tools (Owlbear simplicity)
- **Shape Tools**
  - Freehand (pen)
  - Line
  - Rectangle
  - Circle/Ellipse
  - Polygon

- **Drawing Properties**
  - Color picker
  - Line width (1-20px)
  - Fill opacity
  - Stroke style (solid, dashed)
  - Layer (above/below tokens)

#### Measurement Tools
- **Ruler** (distance calculation)
  - Click-drag for straight line
  - Waypoints for complex paths
  - Display in grid units (feet, meters)
  - Snap to grid toggle

- **Templates** (spell areas)
  - Circle (Fireball)
  - Cone (Burning Hands)
  - Rectangle (Wall of Fire)
  - Ray (Lightning Bolt)
  - Highlight affected grid spaces
  - Token selection within template

---

## PHASE 10: PERFORMANCE & POLISH (Month 11-12)

### 10.1 Optimization

#### Frontend Performance
- **Canvas Optimization**
  - Culling (off-screen object skip)
  - Sprite batching
  - Texture atlasing
  - Shader caching
  - Dynamic resolution scaling

- **Asset Loading**
  - Lazy loading (viewport-based)
  - Progressive image loading
  - Placeholder thumbnails
  - Preload critical assets
  - Cache management (IndexedDB)

#### Backend Performance
- **Database Query Optimization**
  - Prepared statements
  - Index optimization
  - Query result caching (Redis)
  - Batch queries

- **WebSocket Optimization**
  - Message batching (100ms window)
  - Compression (per-message deflate)
  - Binary protocols for large data
  - Connection pooling

### 10.2 User Experience

#### Onboarding
- **First-Time User Tutorial**
  - Interactive walkthrough
  - Highlight key features
  - Sample scene/actors
  - Video tutorials

- **Templates & Presets**
  - Starter world templates
  - Example characters
  - Common macros
  - Map packs

#### Accessibility
- **Keyboard Navigation**
  - All features keyboard accessible
  - Customizable hotkeys
  - Focus indicators

- **Screen Reader Support**
  - ARIA labels
  - Alt text for images
  - Semantic HTML

- **Color Blindness Modes**
  - Alternative color schemes
  - High contrast mode
  - Customizable UI colors

### 10.3 Mobile Support (Stretch Goal)

#### Responsive Design
- **Touch Optimization**
  - Pinch to zoom
  - Two-finger pan
  - Long-press for context menu
  - Gesture controls

- **Mobile UI Adjustments**
  - Collapsible sidebars
  - Simplified controls
  - Bottom navigation
  - Larger touch targets

---

## PHASE 11: EXTENSION SYSTEM (Month 12+)

### 11.1 Module/Plugin Architecture

#### Module System (Foundry pattern)
- **Module Manifest** (`module.json`)
  - Name, version, author
  - Dependencies
  - Compatibility ranges
  - Scripts, styles, languages

- **Module Registry**
  - Browse modules
  - Install/update/uninstall
  - Compatibility checking
  - Community ratings

#### API Exposure
- **Core APIs**
  - Document API (CRUD operations)
  - Canvas API (rendering, layers)
  - Socket API (real-time events)
  - UI API (dialogs, notifications)
  - Dice API (roll expressions)

- **Hook System**
  - Pre/Post hooks for all operations
  - Custom hook registration
  - Async hook support

### 11.2 Extension Use Cases

**Popular Extension Ideas:**
- Alternative character sheets
- Advanced automation (Better Rolls, MIDI QoL)
- Drag ruler (show movement cost)
- Token effects (magic effects, weather)
- Advanced fog tools
- Shared vision modes
- Token health auras
- Custom dice skins
- Sound effects packs
- Module-specific integrations (D&D Beyond)

---

## KEY DESIGN PRINCIPLES

✅ **Power & Flexibility**: Full JavaScript API, modular architecture
✅ **Document-based data**: Clean, extensible, versionable
✅ **Dynamic lighting**: WebGL-powered, performant
✅ **Comprehensive dice system**: Extensive modifier support
✅ **Self-hosted**: Data ownership, offline capable
✅ **Extensibility**: Rich module ecosystem
✅ **Quick setup**: Import and play in <10 minutes
✅ **Intuitive UI**: Minimal chrome, obvious controls
✅ **Smart grid alignment**: Visual tools, auto-detection
✅ **No accounts required**: Share link, instant join
✅ **Room isolation**: Each game is independent
✅ **Mobile-friendly**: Touch-optimized

### Hybrid Innovations
🚀 **Progressive complexity**: Start simple, opt-in to advanced features
🚀 **Preset profiles**: "Simple Mode" vs "Advanced Mode" toggle
🚀 **Smart defaults**: Auto-configure common scenarios
🚀 **Guided workflows**: Wizards for map import, character creation
🚀 **Real-time collaboration**: Multiple GMs, shared editing
🚀 **Cloud-first**: Optional self-hosting for power users

---

## TECHNICAL SPECIFICATIONS

### Backend (Rust)
- **Framework**: Actix-Web for HTTP + WebSocket
- **Database**: PostgreSQL with Diesel ORM
- **Cache**: Redis for pub/sub and session storage
- **Storage**: S3-compatible (MinIO for self-hosted)
- **Auth**: JWT tokens, OAuth2 for social login

### Frontend (Vue.js)
- **Framework**: Vue 3 with Composition API
- **State**: Pinia for global state
- **Rendering**: PixiJS v7 for WebGL canvas
- **UI**: Tailwind CSS for styling
- **Build**: Vite for fast HMR development

### Infrastructure (BOUND TO CHANGE)
- **Hosting**: Kubernetes for scaling
- **CDN**: CloudFront for assets
- **Monitoring**: Prometheus + Grafana
- **Logging**: Structured logs to ELK stack

---

## SUCCESS METRICS

**DM Empowerment:**
- Time to first game: <15 minutes
- Map import success rate: >95%
- Module installation: one-click
- Pre-built content: 100+ SRD creatures/spells

**Player Immersion:**
- Dynamic lighting enabled: >70% of games
- Token response time: <50ms
- Fog of war reveals: smooth, no lag
- Audio sync: <200ms latency

**Technical Performance:**
- WebSocket reconnection: <2s
- Canvas FPS: 60fps with 50+ tokens
- Database query time: <10ms p95
- Asset load time: <3s for large maps

**Community Adoption:**
- Module ecosystem: 50+ modules in 6 months
- User retention: >60% monthly active
- Support tickets: <5% of user base
- Documentation coverage: 100% of features
