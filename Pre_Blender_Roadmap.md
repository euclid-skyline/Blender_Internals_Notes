# Pre-Blender Knowledge Roadmap<!-- omit from toc -->

This document describes the core pre-Blender knowledge needed before contributing to Blender source code. Think of it as the technical backpack you carry before climbing Blender's codebase.

## Table of Contents<!-- omit from toc -->

- [1. Application Architecture](#1-application-architecture)
  - [1.1 Scene Graph Architecture](#11-scene-graph-architecture)
  - [1.2 Data-Oriented Design (DOD)](#12-data-oriented-design-dod)
  - [1.3 Node-Based Architecture](#13-node-based-architecture)
  - [1.4 Plugin and Extension Architecture](#14-plugin-and-extension-architecture)
  - [1.5 Dependency Graph Architecture](#15-dependency-graph-architecture)
- [2. Design Patterns](#2-design-patterns)
  - [2.1 Strategy Pattern](#21-strategy-pattern)
  - [2.2 Factory Pattern](#22-factory-pattern)
  - [2.3 Visitor Pattern](#23-visitor-pattern)
  - [2.4 Command Pattern](#24-command-pattern)
  - [2.5 Observer Pattern](#25-observer-pattern)
  - [2.6 Builder Pattern](#26-builder-pattern)
  - [2.7 Dataflow Pattern](#27-dataflow-pattern)
- [3. Mathematics](#3-mathematics)
  - [3.1 Linear Algebra](#31-linear-algebra)
  - [3.2 Geometry](#32-geometry)
  - [3.3 Numerical Methods](#33-numerical-methods)
- [4. Algorithms](#4-algorithms)
  - [4.1 Graph Algorithms](#41-graph-algorithms)
  - [4.2 Mesh Algorithms](#42-mesh-algorithms)
  - [4.3 Spatial Algorithms](#43-spatial-algorithms)
  - [4.4 Rendering Algorithms](#44-rendering-algorithms)
- [5. 3D Engine Fundamentals](#5-3d-engine-fundamentals)
  - [5.1 GPU Concepts](#51-gpu-concepts)
  - [5.2 Rendering Pipelines](#52-rendering-pipelines)
  - [5.3 Shader Languages and Intermediate Forms](#53-shader-languages-and-intermediate-forms)
- [6. Data Structures](#6-data-structures)
  - [6.1 Half-Edge and Editable Mesh Models (BMesh Concepts)](#61-half-edge-and-editable-mesh-models-bmesh-concepts)
  - [6.2 Attribute and Custom Data Layers](#62-attribute-and-custom-data-layers)
  - [6.3 Node Trees](#63-node-trees)
  - [6.4 Spatial Trees (BVH and Related Structures)](#64-spatial-trees-bvh-and-related-structures)
- [7. Compiler and Reflection Concepts](#7-compiler-and-reflection-concepts)
  - [7.1 Reflection Concepts (RNA)](#71-reflection-concepts-rna)
  - [7.2 Binary Serialization Concepts (DNA)](#72-binary-serialization-concepts-dna)
- [Closing Note](#closing-note)

---

## 1. Application Architecture

Blender is not just a single-purpose 3D tool. It combines multiple architectures in one codebase:

- Scene graph engine
- Node-based computation engine
- Real-time renderer
- Offline (batch) renderer
- Python API and scripting layer
- Add-on and plugin system
- Reflection system (RNA)
- Binary serialization system (DNA)
- Dependency graph scheduler
- Multi-backend GPU abstraction layer

To navigate Blender source effectively, you should be comfortable with the architectural concepts below.

### 1.1 Scene Graph Architecture

- What it is: A structure that organizes objects in parent-child hierarchies and evaluates transforms and dependencies through that hierarchy.
- Why it matters: Correct transform propagation is foundational for object placement, animation, and constraints.
- Where to apply it in Blender: Object transform evaluation, parenting logic, and depsgraph-driven world matrix updates.

Key ideas:

- Hierarchical transforms
- Parent-child relationships
- Local space vs world space
- Dependency propagation through transform chains
- Lazy evaluation of derived state

Why this matters in Blender:

- Object transform evaluation depends on hierarchy and dependency propagation.
- Core object and dependency evaluation code assumes clear understanding of space conversions and update order.

### 1.2 Data-Oriented Design (DOD)

- What it is: A performance-oriented design approach that prioritizes data layout and cache-friendly iteration over class-like abstraction.
- Why it matters: Large meshes and modifier stacks must run fast and predictably under heavy workloads.
- Where to apply it in Blender: Mesh evaluation paths, CustomData access, and tight loops in geometry and modifier code.

Key ideas:

- Struct-of-arrays vs array-of-structs tradeoffs
- Cache locality and contiguous iteration
- Reducing pointer chasing in hot loops
- Data layout chosen for performance-critical passes

Why this matters in Blender:

- Mesh evaluation, modifiers, and geometry pipelines are performance-sensitive.
- Memory layout decisions strongly affect viewport responsiveness and render preparation speed.

### 1.3 Node-Based Architecture

- What it is: A graph model where nodes perform operations and typed sockets define data flow between operations.
- Why it matters: It enables modular authoring, reuse, and scalable computation pipelines.
- Where to apply it in Blender: Geometry Nodes, Shader Nodes, and Compositor node tree evaluation and execution.

Key ideas:

- Directed acyclic graph (DAG) models
- Typed sockets and links
- Lazy or partial evaluation
- Node registration and discovery
- Execution context separation

Why this matters in Blender:

- Geometry Nodes, Shader Nodes, and Compositor Nodes all depend on this model.
- Understanding node graph evaluation makes debugging and extending node systems much easier.

### 1.4 Plugin and Extension Architecture

- What it is: An extensibility model based on registration, metadata, and callback interfaces.
- Why it matters: It allows features to be added without destabilizing or rewriting core systems.
- Where to apply it in Blender: Operators, add-ons, custom tools, and RNA-backed runtime type exposure.

Key ideas:

- Runtime registration systems
- Interface tables and callbacks
- Dynamic dispatch by type and operation
- Metadata-driven exposure to tools and scripting

Why this matters in Blender:

- Operators, add-ons, and many extensible subsystems rely on registration plus metadata.
- RNA bridges C-side data and behavior into Python-facing APIs.

### 1.5 Dependency Graph Architecture

- What it is: A directed dependency system that tracks relationships and schedules evaluation in valid order.
- Why it matters: It prevents stale results and avoids unnecessary full-scene recalculation.
- Where to apply it in Blender: Animation updates, constraints, modifiers, and scene-wide change propagation.

Key ideas:

- DAG construction from scene relationships
- Topological scheduling
- Dirty tagging and invalidation
- Incremental re-evaluation
- Parallel execution where possible

Why this matters in Blender:

- Depsgraph is central to animation, constraints, modifiers, and scene updates.
- Bugs in dependency handling often appear as stale data, wrong evaluation order, or update storms.

---

## 2. Design Patterns

Blender is largely written in C and C++, but many object-oriented design patterns are implemented through structs, function pointers, and registration systems.

### 2.1 Strategy Pattern

- What it is: A pattern that encapsulates multiple interchangeable algorithms behind a common interface.
- Why it matters: It simplifies swapping behavior without modifying call sites.
- Where to apply it in Blender: Modifier implementations, constraint solvers, and interchangeable operation backends.

Common use:

- Modifiers
- Constraints
- Shader and geometry operations

Concept:

- Multiple interchangeable algorithms implement a shared behavioral contract.

### 2.2 Factory Pattern

- What it is: A creation pattern that builds concrete objects from type information through centralized logic.
- Why it matters: It keeps instantiation consistent and decouples creation from usage.
- Where to apply it in Blender: RNA type creation, node instantiation, and operator/type registration flows.

Common use:

- Creating RNA types
- Node instance creation
- Operator and type registration flows

Concept:

- Centralized construction logic creates concrete objects from a descriptor or type id.

### 2.3 Visitor Pattern

- What it is: A pattern that applies operations to structure elements without embedding the operation in each element type.
- Why it matters: It improves extensibility when traversing heterogeneous structures.
- Where to apply it in Blender: Node tree traversals, dependency graph walks, and analysis or export passes.

Common use:

- Traversing graph-like structures
- Performing operations without embedding logic in every node type

Concept:

- Separate traversal logic from operations over heterogeneous elements.

### 2.4 Command Pattern

- What it is: A pattern that wraps actions as command objects or units with executable behavior.
- Why it matters: It enables reliable undo/redo and action history management.
- Where to apply it in Blender: Operator execution lifecycle and undo stack integration.

Common use:

- Operator execution model
- Undo and redo stacks

Concept:

- Encapsulate actions as commands with execute, undo, and redo semantics.

### 2.5 Observer Pattern

- What it is: A publish-subscribe pattern where observers react to state changes in observed subjects.
- Why it matters: It keeps systems synchronized while reducing direct coupling.
- Where to apply it in Blender: Notifier systems, dependency updates, and UI redraw signaling.

Common use:

- Dependency update signaling
- UI redraw and notification systems

Concept:

- Changes in one subsystem notify dependent subsystems without tight coupling.

### 2.6 Builder Pattern

- What it is: A staged construction pattern for assembling complex objects step by step.
- Why it matters: It improves clarity and reduces errors in multi-part configuration.
- Where to apply it in Blender: RNA property definitions and programmatic node tree assembly.

Common use:

- Defining RNA properties in steps
- Programmatic construction of complex node trees

Concept:

- Build complex objects incrementally with clear staged configuration.

### 2.7 Dataflow Pattern

- What it is: A computational model where connected operators transform data as it flows through a graph.
- Why it matters: It makes dependency-driven evaluation and modular graph design natural.
- Where to apply it in Blender: Geometry Nodes execution, shader graph evaluation, and compositor pipelines.

Common use:

- Geometry Nodes
- Shader graph execution
- Compositor pipelines

Concept:

- Data moves through connected processing units where links define flow and dependencies.

---

## 3. Mathematics

Strong math fundamentals are mandatory for graphics, simulation, and animation code.

### 3.1 Linear Algebra

- What it is: The mathematical foundation for vectors, matrices, rotations, and coordinate space conversions.
- Why it matters: Nearly all transform and camera computations depend on it.
- Where to apply it in Blender: Object transforms, rigging math, camera projection, and viewport calculations.

You should know:

- Vectors and vector operations
- Matrices and matrix multiplication
- Quaternions and rotation composition
- Transform composition and decomposition
- Inverse transforms and coordinate conversion
- Projection matrices and camera space

Why this matters in Blender:

- Transform evaluation, camera projection, rigging, and viewport math all depend on linear algebra.

### 3.2 Geometry

- What it is: The study of shape, surface properties, and geometric relationships in 2D/3D space.
- Why it matters: Robust geometric operations are required for modeling tools and shading correctness.
- Where to apply it in Blender: Intersection tests, normals/tangents, snapping, picking, and mesh operations.

You should know:

- Surface normals and tangent spaces
- Barycentric coordinates
- Ray-triangle and ray-mesh intersection
- Mesh topology basics

Why this matters in Blender:

- Geometry processing, snapping, picking, modifiers, and shading rely on geometric computations.

### 3.3 Numerical Methods

- What it is: Techniques for stable approximation, interpolation, and iterative computation with floating-point numbers.
- Why it matters: Numerical instability can produce jitter, drift, and non-deterministic behavior.
- Where to apply it in Blender: Animation interpolation, spline evaluation, constraints, and simulation stepping.

You should know:

- Interpolation methods (LERP, SLERP)
- Curve and spline evaluation
- Floating-point precision and error accumulation
- Numerical stability in iterative systems

Why this matters in Blender:

- Animation curves, drivers, constraints, and simulation steps require numerically stable implementations.

---

## 4. Algorithms

Blender combines classic algorithms with domain-specific optimizations.

### 4.1 Graph Algorithms

- What it is: Algorithms for traversing, ordering, and validating node/edge relationships in directed graphs.
- Why it matters: Correct dependency order is essential for deterministic evaluation.
- Where to apply it in Blender: Depsgraph construction, topological ordering, and cycle detection.

You should know:

- Topological sorting
- Cycle detection
- Dependency-driven evaluation

Why this matters in Blender:

- Core to dependency graph scheduling and deterministic update order.

### 4.2 Mesh Algorithms

- What it is: Algorithms that edit mesh connectivity and geometry for modeling and procedural workflows.
- Why it matters: Efficient topology operations directly affect tool responsiveness and result quality.
- Where to apply it in Blender: BMesh editing, subdivision, booleans, remeshing, and loop/ring selection logic.

You should know:

- Topology traversal strategies
- Edge loop and edge ring detection
- Subdivision logic
- Boolean and remeshing fundamentals

Why this matters in Blender:

- Editing tools, modifiers, and procedural geometry operations are built on these primitives.

### 4.3 Spatial Algorithms

- What it is: Algorithms and data structures that accelerate spatial queries in 3D scenes.
- Why it matters: Naive geometric queries do not scale to production scene complexity.
- Where to apply it in Blender: BVH ray casts, KD-tree nearest searches, picking, and collision-related queries.

You should know:

- Bounding volume hierarchy (BVH)
- KD-tree usage patterns
- Spatial partitioning and nearest-neighbor queries
- Ray casting acceleration techniques

Why this matters in Blender:

- Used in rendering, viewport picking, geometry queries, and physics intersections.

### 4.4 Rendering Algorithms

- What it is: Methods for simulating light transport and material response to produce final pixels.
- Why it matters: Image quality and render performance are both determined by algorithm choice.
- Where to apply it in Blender: Eevee raster techniques, Cycles path tracing, and shadowing/light integration code.

You should know:

- Rasterization pipeline basics
- Ray tracing and path tracing foundations
- Shadow mapping concepts
- Physically based rendering (PBR) principles

Why this matters in Blender:

- Eevee and Cycles use different but related rendering models that share core graphics ideas.

---

## 5. 3D Engine Fundamentals

Before reading GPU and renderer internals, build a solid graphics pipeline foundation.

### 5.1 GPU Concepts

- What it is: Core concepts for how graphics hardware processes geometry and shading workloads.
- Why it matters: Correct buffer, shader, and state management is critical for rendering correctness and speed.
- Where to apply it in Blender: GPU module code, draw manager integration, and backend-specific resource handling.

You should know:

- Vertex buffers and index buffers
- Shader stages and shader programs
- Uniforms, storage buffers, and resource binding
- Framebuffers and render targets
- Render passes and state changes

### 5.2 Rendering Pipelines

- What it is: The ordered rendering stages and data layout strategy used to produce frames.
- Why it matters: Pipeline architecture determines performance, memory usage, and achievable visual features.
- Where to apply it in Blender: Eevee frame passes, deferred/forward tradeoffs, and real-time viewport rendering flow.

You should know:

- Forward rendering
- Deferred rendering
- Hybrid rendering approaches

### 5.3 Shader Languages and Intermediate Forms

- What it is: Source and intermediate representations used to author and translate GPU shader programs.
- Why it matters: Cross-platform rendering depends on consistent shader translation and backend compatibility.
- Where to apply it in Blender: GLSL authoring, backend translation paths, and SPIR-V or platform shader compilation.

You should know:

- GLSL basics
- Metal shading concepts
- SPIR-V as an intermediate representation concept

Why this matters in Blender:

- Blender's GPU layer abstracts multiple backends, but backend constraints and shader translation details still affect engine behavior.

---

## 6. Data Structures

Blender's performance and flexibility depend on choosing the right structures for geometry and runtime systems.

### 6.1 Half-Edge and Editable Mesh Models (BMesh Concepts)

- What it is: Connectivity-centric mesh structures that store explicit relations between vertices, edges, loops, and faces.
- Why it matters: Reliable topology editing requires fast, consistent neighborhood access.
- Where to apply it in Blender: BMesh edit operations, topology mutation tools, and mesh editing operators.

You should know:

- Vertex, edge, loop, and face connectivity
- Efficient topology edits
- Euler-style topology operations

Why this matters in Blender:

- Modeling tools and mesh editing workflows rely on stable and efficient topology mutation.

### 6.2 Attribute and Custom Data Layers

- What it is: Flexible layered storage for per-element mesh or geometry attributes.
- Why it matters: Modern procedural workflows depend on preserving and transforming attributes correctly.
- Where to apply it in Blender: UV/color/normal data handling, geometry node attributes, and modifier data propagation.

You should know:

- Layered storage for per-element data
- Dynamic schemas for UVs, colors, normals, and custom attributes
- Data propagation rules across operations

Why this matters in Blender:

- Attribute-heavy workflows (especially geometry nodes) depend on robust layer management.

### 6.3 Node Trees

- What it is: Graph data structures that represent nodes, typed sockets, links, and evaluation metadata.
- Why it matters: Strong graph structure is required for validation, scheduling, and execution correctness.
- Where to apply it in Blender: Node editors, node graph serialization, and execution preparation.

You should know:

- Node, socket, and link representations
- Typed connections and validation
- Execution context and evaluation metadata

Why this matters in Blender:

- A major portion of modern Blender workflows is node-driven.

### 6.4 Spatial Trees (BVH and Related Structures)

- What it is: Hierarchical spatial partitioning structures that accelerate query operations in 3D space.
- Why it matters: They make rendering and geometric query workloads tractable at scale.
- Where to apply it in Blender: Ray tracing acceleration, physics broadphase queries, and geometry node spatial lookups.

You should know:

- Tree construction tradeoffs
- Query performance characteristics
- Update and rebuild strategies

Why this matters in Blender:

- Spatial acceleration structures appear throughout rendering, physics, and geometry queries.

---

## 7. Compiler and Reflection Concepts

Blender's DNA/RNA pipeline combines reflection-like metadata with binary file compatibility concerns.

### 7.1 Reflection Concepts (RNA)

- What it is: Runtime metadata that describes types, properties, and access rules for data introspection.
- Why it matters: It enables dynamic tooling, scripting exposure, and consistent UI/property behavior.
- Where to apply it in Blender: RNA definitions, Python API exposure, and property-driven UI integration.

You should know:

- Runtime type metadata
- Property descriptors and registration
- Introspection and dynamic access patterns
- API exposure from native code to scripting layers

Why this matters in Blender:

- RNA is the bridge between internal data definitions and user-facing/API-visible behavior.

### 7.2 Binary Serialization Concepts (DNA)

- What it is: Binary data layout and persistence rules for storing and restoring structured application state.
- Why it matters: File compatibility and safe version migration depend on serialization discipline.
- Where to apply it in Blender: .blend read/write paths, struct versioning, and pointer remapping during load.

You should know:

- Struct layout awareness
- Version compatibility strategies
- Pointer remapping and loading logic
- Backward compatibility constraints

Why this matters in Blender:

- Blender's file format and data migration behavior depend on stable serialization contracts.

---

## Closing Note

If you become comfortable with these seven areas, Blender source will feel far less intimidating. You do not need perfect mastery before your first contribution, but this roadmap dramatically shortens the learning curve and helps you read code with the right mental models from day one.
