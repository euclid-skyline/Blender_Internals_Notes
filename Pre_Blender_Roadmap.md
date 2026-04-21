# Pre-Blender Knowledge Roadmap

## Table of Contents

1. [Application Architecture](#1-application-architecture)
2. [Design Patterns](#2-design-patterns)
3. [Mathematics](#3-mathematics)
4. [Algorithms](#4-algorithms)
5. [3D Engine Fundamentals](#5-3d-engine-fundamentals)
6. [Data Structures](#6-data-structures)
7. [Compiler and Reflection Concepts](#7-compiler-and-reflection-concepts)

This document describes the core pre-Blender knowledge needed before contributing to Blender source code. Think of it as the technical backpack you carry before climbing Blender's codebase.

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

Key ideas:

- Struct-of-arrays vs array-of-structs tradeoffs
- Cache locality and contiguous iteration
- Reducing pointer chasing in hot loops
- Data layout chosen for performance-critical passes

Why this matters in Blender:

- Mesh evaluation, modifiers, and geometry pipelines are performance-sensitive.
- Memory layout decisions strongly affect viewport responsiveness and render preparation speed.

### 1.3 Node-Based Architecture

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

Key ideas:

- Runtime registration systems
- Interface tables and callbacks
- Dynamic dispatch by type and operation
- Metadata-driven exposure to tools and scripting

Why this matters in Blender:

- Operators, add-ons, and many extensible subsystems rely on registration plus metadata.
- RNA bridges C-side data and behavior into Python-facing APIs.

### 1.5 Dependency Graph Architecture

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

Common use:

- Modifiers
- Constraints
- Shader and geometry operations

Concept:

- Multiple interchangeable algorithms implement a shared behavioral contract.

### 2.2 Factory Pattern

Common use:

- Creating RNA types
- Node instance creation
- Operator and type registration flows

Concept:

- Centralized construction logic creates concrete objects from a descriptor or type id.

### 2.3 Visitor Pattern

Common use:

- Traversing graph-like structures
- Performing operations without embedding logic in every node type

Concept:

- Separate traversal logic from operations over heterogeneous elements.

### 2.4 Command Pattern

Common use:

- Operator execution model
- Undo and redo stacks

Concept:

- Encapsulate actions as commands with execute, undo, and redo semantics.

### 2.5 Observer Pattern

Common use:

- Dependency update signaling
- UI redraw and notification systems

Concept:

- Changes in one subsystem notify dependent subsystems without tight coupling.

### 2.6 Builder Pattern

Common use:

- Defining RNA properties in steps
- Programmatic construction of complex node trees

Concept:

- Build complex objects incrementally with clear staged configuration.

### 2.7 Dataflow Pattern

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

You should know:

- Surface normals and tangent spaces
- Barycentric coordinates
- Ray-triangle and ray-mesh intersection
- Mesh topology basics

Why this matters in Blender:

- Geometry processing, snapping, picking, modifiers, and shading rely on geometric computations.

### 3.3 Numerical Methods

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

You should know:

- Topological sorting
- Cycle detection
- Dependency-driven evaluation

Why this matters in Blender:

- Core to dependency graph scheduling and deterministic update order.

### 4.2 Mesh Algorithms

You should know:

- Topology traversal strategies
- Edge loop and edge ring detection
- Subdivision logic
- Boolean and remeshing fundamentals

Why this matters in Blender:

- Editing tools, modifiers, and procedural geometry operations are built on these primitives.

### 4.3 Spatial Algorithms

You should know:

- Bounding volume hierarchy (BVH)
- KD-tree usage patterns
- Spatial partitioning and nearest-neighbor queries
- Ray casting acceleration techniques

Why this matters in Blender:

- Used in rendering, viewport picking, geometry queries, and physics intersections.

### 4.4 Rendering Algorithms

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

You should know:

- Vertex buffers and index buffers
- Shader stages and shader programs
- Uniforms, storage buffers, and resource binding
- Framebuffers and render targets
- Render passes and state changes

### 5.2 Rendering Pipelines

You should know:

- Forward rendering
- Deferred rendering
- Hybrid rendering approaches

### 5.3 Shader Languages and Intermediate Forms

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

You should know:

- Vertex, edge, loop, and face connectivity
- Efficient topology edits
- Euler-style topology operations

Why this matters in Blender:

- Modeling tools and mesh editing workflows rely on stable and efficient topology mutation.

### 6.2 Attribute and Custom Data Layers

You should know:

- Layered storage for per-element data
- Dynamic schemas for UVs, colors, normals, and custom attributes
- Data propagation rules across operations

Why this matters in Blender:

- Attribute-heavy workflows (especially geometry nodes) depend on robust layer management.

### 6.3 Node Trees

You should know:

- Node, socket, and link representations
- Typed connections and validation
- Execution context and evaluation metadata

Why this matters in Blender:

- A major portion of modern Blender workflows is node-driven.

### 6.4 Spatial Trees (BVH and Related Structures)

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

You should know:

- Runtime type metadata
- Property descriptors and registration
- Introspection and dynamic access patterns
- API exposure from native code to scripting layers

Why this matters in Blender:

- RNA is the bridge between internal data definitions and user-facing/API-visible behavior.

### 7.2 Binary Serialization Concepts (DNA)

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
