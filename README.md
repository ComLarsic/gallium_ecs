# Gallium
A simple entity-component-system crate for rust with serialization support

## Usage
You can include the library using carge:
```toml
[dependencies]
gallium_ecs = "0.2.0"
gallium_ecs_derive = "0.1.1
```

### Components
Components are simple structs that can de defined using the `component` attribute.
Note that all the data a component uses needs to be serializable by [serde](https://github.com/serde-rs/serde)

```rust
#[gallium::component]
pub struct ExampleComponent {
  data_1: i32,
  data_2: String,
}
```

### Entities
Entities can be build in two ways, the builder method or just by using the functions.

```rust
// The function method
let mut func_entity = Entity::new();
// Add a component to the entity
func_entity.add_component(ExampleComponent {});

// The builder method
let build_entity = EntityBuilder::new()
// Add a component to the entity
.with(ExampleComponent {})
.build();

```

You can fetch components in an entity like so:
```rust
// Returns a RefMut<ExampleComponent>
let example_component = entity.get_component::<ExampleComponent>().unwrap();
```

### Systems
Systems are serializable structs that implement the system trait,
implementation of system requires the usage of the `system` attribute for simplicity.

The `tick` function gets called everytime the systems with the specified tag get ticked.

The `on_event` function gets called everytime an event gets dispatched into the scene.

```rust
#[derive(Serialize, Deserialize)]
pub struct ExampleSystem

#[gallium::system]
impl System for ExampleSystem {
  fn tick(&mut self, _scene: &mut Scene, _world: &mut World) {
    // System tick code here
  }
  
  fn on_event(&self, _scene: &mut Scene, _world: &mut World, _tag: &str, _data: &mut dyn std::any::Any) {
    // Event handling here
  }
}

```

### Scenes
Scenes are serializable objects that hold entities and systems to be run.
Just like entites, these can be created using the builder method or using the functions.

```rust
// The function method
let mut func_scene = Scene::new();
// Add an entity to the scene
func_scene.add_entity(entity);
func_scene.add_system("init", ExampleSystem {});

// The builder method
let build_entity = SystemBuilder::new()
.with_entity(entity)
.with_system("init", ExampleSystem {})
.build();

```
#### Systems
You can tick the systems in the scenes like so:
```rust
let mut scene = SceneBuilder::new()
// Add a system to the scene
.with_system("init", ExampleSystem {})
.build();

// Tick all systems with the 'init' tag
scene.tick_systems("init");
```
you can also dispatch events over all the systems in the scene:
```rust
let mut scene = SceneBuilder::new()
// Add a system to the scene
.with_system("init", ExampleSystem {})
.build();

// Dispatch the event
let example_data = ... // Any type can be dispatched as data
scene.dispatch_event("example_event", &example_data); // The data has to be dispatched as a reference
```

#### Entities
To fetch a specific entity, you need to use its id:
```rust
let entity = scene.get_entity(0).unwrap(); // Returns a RefMut<Entity>
```

To fetch a list of entity with specific components, you can get all the entities and filter through the list:
```rust
// Returns a Vec<RefMut<Entity>>
let entities = scene.get_entities()
.are_active() // Make sure to only fetch active components
.with_component::<ExampleComponent>(); // Only fetch entities with specified components
```

### Worlds
If you have multiple scenes in your game the better way of managing them is using worlds.
```rust
// Create the world
let mut world = World::new();

// Set the scene to the first scene
world.set_scene(Scene::import_ron("scene_1.ron").unwrap());
// Tick the systems in the first scene
world.tick_systems("test");

// Set the scene to the second scene
world.set_scene(Scene::import_ron("scene_2.ron").unwrap());
// Tick the systems in the second scene
world.tick_systems("test");
```

### Serialization
Both entities and scenes have methods to serialize into [ron](https://github.com/ron-rs/ron) files.
```rust
// --- Serializing an entity ---
let entity = EntityBuilder::new()
// Add a serailizable component
.with(ExampleComponent{})
.build();
// Dump the entity to a ron-file
entity.export_ron("path_to_file").unwrap();

// --- Serializing a scene ---
let scene = SceneBuilder::new()
// Add the entity to the scene
.with(entity)
.build();
// Dump to scene to a file
scene.export_ron("path_to_file").unwrap();
```

Both can be deserialized aswell.
```rust
// Deserialize an entity
let entity = Entity::import_ron("path_to_file").unwrap();

// Deserialize a scene
let scene = Scene::import_ron("path_to_file").unwrap();
```
