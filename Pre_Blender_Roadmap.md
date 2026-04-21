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

Scene graph architecture defines how object hierarchies, transforms, and dependencies are organized and evaluated across a 3D scene.

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

Data-oriented design focuses on memory layout and iteration patterns to maximize performance in large geometry and evaluation workloads.

Key ideas:

- Struct-of-arrays vs array-of-structs tradeoffs
- Cache locality and contiguous iteration
- Reducing pointer chasing in hot loops
- Data layout chosen for performance-critical passes

Why this matters in Blender:

- Mesh evaluation, modifiers, and geometry pipelines are performance-sensitive.
- Memory layout decisions strongly affect viewport responsiveness and render preparation speed.

### 1.3 Node-Based Architecture

Node-based architecture models computation as connected units of typed inputs and outputs, enabling modular and scalable graph execution.

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

Plugin architecture allows new behaviors to be added through registration, metadata, and callbacks without rewriting the application core.

Key ideas:

- Runtime registration systems
- Interface tables and callbacks
- Dynamic dispatch by type and operation
- Metadata-driven exposure to tools and scripting

Why this matters in Blender:

- Operators, add-ons, and many extensible subsystems rely on registration plus metadata.
- RNA bridges C-side data and behavior into Python-facing APIs.

### 1.5 Dependency Graph Architecture

Dependency graph architecture controls update order by tracking relationships and reevaluating only what changed.

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

The strategy pattern packages interchangeable algorithms behind one interface so behavior can vary without changing caller logic.

Common use:

- Modifiers
- Constraints
- Shader and geometry operations

Concept:

- Multiple interchangeable algorithms implement a shared behavioral contract.

### 2.2 Factory Pattern

The factory pattern centralizes object creation, making type-driven instantiation and registration-based construction predictable.

Common use:

- Creating RNA types
- Node instance creation
- Operator and type registration flows

Concept:

- Centralized construction logic creates concrete objects from a descriptor or type id.

### 2.3 Visitor Pattern

The visitor pattern separates traversal from operations, allowing new processing steps over complex structures without modifying node types.

Common use:

- Traversing graph-like structures
- Performing operations without embedding logic in every node type

Concept:

- Separate traversal logic from operations over heterogeneous elements.

### 2.4 Command Pattern

The command pattern represents actions as executable units, which makes undo and redo practical and consistent.

Common use:

- Operator execution model
- Undo and redo stacks

Concept:

- Encapsulate actions as commands with execute, undo, and redo semantics.

### 2.5 Observer Pattern

The observer pattern propagates state changes through notifications so dependent systems stay synchronized with minimal coupling.

Common use:

- Dependency update signaling
- UI redraw and notification systems

Concept:

- Changes in one subsystem notify dependent subsystems without tight coupling.

### 2.6 Builder Pattern

The builder pattern creates complex objects in stages, improving readability and correctness for multi-step configuration.

Common use:

- Defining RNA properties in steps
- Programmatic construction of complex node trees

Concept:

- Build complex objects incrementally with clear staged configuration.

### 2.7 Dataflow Pattern

Dataflow pattern treats processing as data moving through connected operators, where links define execution dependencies.

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

Linear algebra provides the language for transforms, orientations, projections, and coordinate conversions in graphics systems.

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

Geometry covers the shape-level math used for intersections, surface properties, and robust mesh operations.

You should know:

- Surface normals and tangent spaces
- Barycentric coordinates
- Ray-triangle and ray-mesh intersection
- Mesh topology basics

Why this matters in Blender:

- Geometry processing, snapping, picking, modifiers, and shading rely on geometric computations.

### 3.3 Numerical Methods

Numerical methods ensure interpolation, curves, and iterative computations remain stable and accurate under floating-point limits.

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

Graph algorithms provide the rules for ordering, validating, and traversing dependency relationships in complex systems.

You should know:

- Topological sorting
- Cycle detection
- Dependency-driven evaluation

Why this matters in Blender:

- Core to dependency graph scheduling and deterministic update order.

### 4.2 Mesh Algorithms

Mesh algorithms manipulate topology and geometry efficiently for modeling, procedural operations, and cleanup workflows.

You should know:

- Topology traversal strategies
- Edge loop and edge ring detection
- Subdivision logic
- Boolean and remeshing fundamentals

Why this matters in Blender:

- Editing tools, modifiers, and procedural geometry operations are built on these primitives.

### 4.3 Spatial Algorithms

Spatial algorithms accelerate geometric queries by organizing 3D data for fast intersection and nearest-neighbor lookups.

You should know:

- Bounding volume hierarchy (BVH)
- KD-tree usage patterns
- Spatial partitioning and nearest-neighbor queries
- Ray casting acceleration techniques

Why this matters in Blender:

- Used in rendering, viewport picking, geometry queries, and physics intersections.

### 4.4 Rendering Algorithms

Rendering algorithms define how light and materials are computed to produce images in real-time and offline pipelines.

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

GPU fundamentals explain how geometry, shaders, and render targets flow through the graphics hardware pipeline.

You should know:

- Vertex buffers and index buffers
- Shader stages and shader programs
- Uniforms, storage buffers, and resource binding
- Framebuffers and render targets
- Render passes and state changes

### 5.2 Rendering Pipelines

Rendering pipelines describe the sequence and data layout decisions used to balance visual quality and performance.

You should know:

- Forward rendering
- Deferred rendering
- Hybrid rendering approaches

### 5.3 Shader Languages and Intermediate Forms

Shader languages and intermediate forms define how shading logic is authored, compiled, and translated across GPU backends.

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

Half-edge style structures represent mesh connectivity explicitly, enabling safe and efficient topology editing.

You should know:

- Vertex, edge, loop, and face connectivity
- Efficient topology edits
- Euler-style topology operations

Why this matters in Blender:

- Modeling tools and mesh editing workflows rely on stable and efficient topology mutation.

### 6.2 Attribute and Custom Data Layers

Attribute layer systems store rich per-element data and keep it consistent as geometry is transformed and regenerated.

You should know:

- Layered storage for per-element data
- Dynamic schemas for UVs, colors, normals, and custom attributes
- Data propagation rules across operations

Why this matters in Blender:

- Attribute-heavy workflows (especially geometry nodes) depend on robust layer management.

### 6.3 Node Trees

Node tree data structures encode nodes, sockets, and links so graph-based tools can validate and evaluate correctly.

You should know:

- Node, socket, and link representations
- Typed connections and validation
- Execution context and evaluation metadata

Why this matters in Blender:

- A major portion of modern Blender workflows is node-driven.

### 6.4 Spatial Trees (BVH and Related Structures)

Spatial trees trade preprocessing cost for much faster runtime queries in rendering, physics, and geometry tools.

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

Reflection metadata describes types and properties at runtime so systems can inspect, edit, and expose data dynamically.

You should know:

- Runtime type metadata
- Property descriptors and registration
- Introspection and dynamic access patterns
- API exposure from native code to scripting layers

Why this matters in Blender:

- RNA is the bridge between internal data definitions and user-facing/API-visible behavior.

### 7.2 Binary Serialization Concepts (DNA)

Binary serialization concepts ensure structured data can be stored, loaded, and migrated across versions reliably.

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
