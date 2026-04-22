# Pre-Blender Knowledge Roadmap<!-- omit from toc -->

This document describes the core pre-Blender knowledge needed before contributing to Blender source code. Think of it as the technical backpack you carry before climbing Blender's codebase.

## Table of Contents<!-- omit from toc -->

- [Recommended Reading Order](#recommended-reading-order)
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

---

## Recommended Reading Order

Use this order to reduce context switching and build concepts in layers:

1. Application Architecture: Build your mental map of how Blender evaluates and coordinates systems.
2. Data Structures: Learn how core data is represented before reading algorithms that mutate it.
3. Mathematics: Refresh the math used by transforms, geometry, and evaluation code.
4. Algorithms: Map core algorithmic patterns to depsgraph, mesh, and rendering workloads.
5. 3D Engine Fundamentals: Understand GPU and pipeline concepts before deep rendering internals.
6. Compiler and Reflection Concepts: Learn DNA/RNA to understand data definition, IO, and API exposure.
7. Design Patterns: Revisit patterns to recognize recurring architecture choices in production code.

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

Source directory references:

- source/blender/blenkernel
- source/blender/depsgraph
- source/blender/nodes
- source/blender/windowmanager
- source/blender/editors

### 1.1 Scene Graph Architecture

- What it is: A structure that organizes objects in parent-child hierarchies and evaluates transforms and dependencies through that hierarchy.
- Why it matters: Correct transform propagation is foundational for object placement, animation, and constraints.
- Where to apply it in Blender: Object transform evaluation and parenting in BKE, with world matrix update scheduling in DEG (depsgraph).

Key ideas:

**Hierarchical transforms:** A hierarchical transform is a matrix applied to an object relative to its parent, so each object's final position is the product of its own transform composed with all ancestor transforms. Learn this by manually multiplying parent and child TRS (Translation × Rotation × Scale) matrices and observing how a change at the root affects the entire chain. Important subjects include 4×4 matrix composition, local-to-world matrix derivation, and how transform order affects the result.

**Parent-child relationships:** A parent-child relationship means a child object inherits its parent's transform, so moving the parent moves all descendants automatically. Learn this by building a small scene tree where re-parenting or re-ordering nodes changes evaluated positions. Important subjects include transform inheritance rules, bind-pose vs current-pose concepts in rigging, and how reparenting requires space conversion to preserve visual position.

**Local space vs world space:** Local space is the coordinate system relative to an object's own origin, while world space is relative to the global scene origin. Learn this by converting vectors between spaces using the model matrix and its inverse. Important subjects include the model matrix, view matrix, inverse transforms, change-of-basis operations, and how Blender stores both ob->object_to_world and the local matrix per object.

**Dependency propagation through transform chains:** When a parent's transform changes, every descendant must recompute its world transform in correct hierarchical order before any downstream system reads the result. Learn this by tracing a single position change from a root object down through several levels of children and verifying update order. Important subjects include dirty propagation, topological update ordering, and how the depsgraph tracks transform relations to drive correct re-evaluation.

**Lazy evaluation of derived state:** Lazy evaluation defers recomputing derived data, such as a world matrix, until something actually requests it and the data is marked dirty, avoiding redundant work. Learn this through cache-invalidation and mark-dirty patterns: flag data stale when inputs change, then recompute only on first read. Important subjects include dirty flags, on-demand evaluation, the difference between authoritative source data and cached derived data, and how Blender's DEG decides when to skip re-evaluation.

Why this matters in Blender:

- Object transform evaluation depends on hierarchy and dependency propagation.
- Core object and dependency evaluation code assumes clear understanding of space conversions and update order.

### 1.2 Data-Oriented Design (DOD)

- What it is: A performance-oriented design approach that prioritizes data layout and cache-friendly iteration over class-like abstraction.
- Why it matters: Large meshes and modifier stacks must run fast and predictably under heavy workloads.
- Where to apply it in Blender: Mesh and modifier hot paths in BKE, plus CustomData and attribute storage in DNA/BKE mesh data.

Key ideas:

**Struct-of-arrays vs array-of-structs tradeoffs:** Struct-of-arrays (SoA) stores each field of a record type in its own contiguous array, while array-of-structs (AoS) stores all fields together per element; SoA is faster when only one or two fields are accessed per iteration, while AoS is friendlier when all fields are needed together. Learn this by benchmarking a simple vertex loop in both layouts and comparing cache miss counts with a profiler. Important subjects include memory bandwidth, CPU cache line width (typically 64 bytes), SIMD vectorization requirements, and when mixed or hybrid layouts make sense.

**Cache locality and contiguous iteration:** Cache locality means keeping data that is accessed together physically adjacent in memory so each cache line fetch is fully used rather than wasted on unused fields. Learn this by measuring loop throughput over contiguous arrays vs scattered allocations and observing the performance difference. Important subjects include L1/L2/L3 cache sizes, cache line utilization, hardware prefetching, and the cost of random vs sequential access patterns.

**Reducing pointer chasing in hot loops:** Pointer chasing happens when each element holds a pointer to the next data item, forcing the CPU to wait for each memory fetch before it knows the next address. Learn this by refactoring a linked-list traversal to use a flat index array and measuring the speedup. Important subjects include linked list vs flat array tradeoffs, handle/index-based indirection systems, and how to identify pointer-chasing bottlenecks with a memory profiler.

**Data layout chosen for performance-critical passes:** Choosing the right layout for a workload means analyzing which fields are read or written together in a given algorithm and arranging them to minimize wasted bandwidth and maximize vectorization. Learn this by profiling a mesh evaluation pass, identifying which attributes are hot, and reorganizing them into tighter arrays. Important subjects include hot-and-cold data splitting, attribute-of-arrays patterns, profiler-guided layout decisions, and how Blender's CustomData and geometry attribute systems balance flexibility against layout efficiency.

Why this matters in Blender:

- Mesh evaluation, modifiers, and geometry pipelines are performance-sensitive.
- Memory layout decisions strongly affect viewport responsiveness and render preparation speed.

### 1.3 Node-Based Architecture

- What it is: A graph model where nodes perform operations and typed sockets define data flow between operations.
- Why it matters: It enables modular authoring, reuse, and scalable computation pipelines.
- Where to apply it in Blender: Node definitions and execution in NOD/BKE, including Geometry Nodes, Shader Nodes, and Compositor node trees.

Key ideas:

**Directed acyclic graph (DAG) models:** A DAG is a graph of nodes connected by directed edges with no cycles, used to represent computation where outputs of some nodes become inputs of others. Learn this by implementing a simple node evaluator that topologically sorts nodes before executing them, rejecting any cycle with an error. Important subjects include graph theory fundamentals, topological sort algorithms, cycle detection via DFS, and how acyclicity is validated when users connect nodes.

**Typed sockets and links:** Sockets define the data type a node inputs or outputs, and links only connect compatible socket types, ensuring the graph carries well-typed data throughout evaluation. Learn this by building a small type system for node connections and implementing type-checking on every link creation attempt. Important subjects include type systems, implicit type conversion rules, socket validation logic, and how Blender handles type mismatches or coercions between linked nodes.

**Lazy or partial evaluation:** Lazy evaluation defers a node's computation until its output is actually needed; partial evaluation limits recomputation only to the nodes downstream of a change. Learn this by building a pull-based evaluator where requesting a final output triggers recursive upstream evaluation, then adding a result cache to skip unchanged nodes. Important subjects include demand-driven evaluation, memoization, incremental computation, and how Blender's node systems decide which nodes are dirty after an edit.

**Node registration and discovery:** Node registration is the mechanism by which each node type announces itself to the system at startup so it can be instantiated by name, enumerated in menus, and serialized to disk by type identifier. Learn this by building a simple node type registry keyed on a string identifier, then implementing a factory that creates instances from that key. Important subjects include factory and registry patterns, self-registering modules, initialization order, and how Blender's NOD module enumerates available node types.

**Execution context separation:** An execution context is the isolated set of state, inputs, and resources threaded through a node graph during one evaluation pass, keeping each evaluation independent from others running concurrently or at different times. Learn this by wrapping all mutable evaluation state into a context object passed explicitly rather than stored globally. Important subjects include context objects, evaluation scope, resource and memory isolation, and how Blender's geometry node execution context carries field inputs, geometry data, and evaluation settings through the tree.

Why this matters in Blender:

- Geometry Nodes, Shader Nodes, and Compositor Nodes all depend on this model.
- Understanding node graph evaluation makes debugging and extending node systems much easier.

### 1.4 Plugin and Extension Architecture

- What it is: An extensibility model based on registration, metadata, and callback interfaces.
- Why it matters: It allows features to be added without destabilizing or rewriting core systems.
- Where to apply it in Blender: Operator/tool registration in WM/ED, add-on exposure through RNA, and runtime type metadata in RNA.

Key ideas:

**Runtime registration systems:** A runtime registration system lets new types, operators, or behaviors declare themselves at startup or load time by inserting descriptors into a shared registry, so the core never needs to know about extensions in advance. Learn this by building a string-keyed registry that maps type names to creation functions and calling it from a module initializer. Important subjects include factory registration, self-registering modules, initialization and teardown order, and how Blender's WM and RNA register operators, panel types, and property groups.

**Interface tables and callbacks:** Interface tables are structs of function pointers where each slot defines one behavioral aspect of a type, allowing different implementations to be swapped simply by assigning a different struct. Learn this by implementing a C-style vtable manually, filling it with concrete function addresses, and calling behavior through the table. Important subjects include function pointer syntax, vtable layout, callback safety and lifetime, and how Blender's ModifierTypeInfo and ConstraintTypeInfo structs expose all type-specific behavior as function pointer slots.

**Dynamic dispatch by type and operation:** Dynamic dispatch selects which implementation to invoke based on runtime type information rather than a compile-time type, routing calls through a registry or table lookup. Learn this by implementing a dispatch table keyed on a type enum and routing all type-specific calls through it. Important subjects include type IDs, dispatch tables, tagged unions, and how Blender resolves at runtime which modifier evaluation function or node execution path to call for a given type.

**Metadata-driven exposure to tools and scripting:** Metadata describes properties and behaviors in a declarative, structured way so they can be automatically surfaced in the UI, Python API, and operator search without manual wiring per property. Learn this by building a simple property descriptor system where each property declares its name, type, range, and tooltip, then driving UI rendering from those descriptors. Important subjects include property descriptors, annotation and reflection systems, and how Blender's RNA metadata drives automatic Python API generation, UI panel rendering, and operator property display.

Why this matters in Blender:

- Operators, add-ons, and many extensible subsystems rely on registration plus metadata.
- RNA bridges C-side data and behavior into Python-facing APIs.

### 1.5 Dependency Graph Architecture

- What it is: A directed dependency system that tracks relationships and schedules evaluation in valid order.
- Why it matters: It prevents stale results and avoids unnecessary full-scene recalculation.
- Where to apply it in Blender: Dependency scheduling and change propagation in DEG, feeding BKE animation, constraints, and modifiers.

Key ideas:

**DAG construction from scene relationships:** The dependency graph is built by scanning scene objects, their parents, constraints, drivers, and modifiers, then creating directed evaluation nodes with edges representing data dependencies between them. Learn this by writing a simple graph builder that turns a list of objects with parent and constraint relationships into a set of nodes and directed edges. Important subjects include relation types (transform, geometry, parameter), node creation from scene data, and how Blender's DEG builder traverses the scene to produce its internal node-relation graph.

**Topological scheduling:** Topological scheduling orders evaluation nodes so every node is guaranteed to run only after all nodes it depends on have already completed, producing a valid linear sequence from a partial order. Learn this by implementing both Kahn's algorithm and a DFS post-order approach on a sample graph and comparing results. Important subjects include Kahn's algorithm, DFS post-order traversal, scheduling queues, and how DEG uses topological order to build its task execution sequence.

**Dirty tagging and invalidation:** Dirty tagging marks a node or piece of data as out-of-date when any of its inputs change, so only stale data gets recomputed rather than the entire scene. Learn this by building a graph where changing one node sets a dirty flag on all downstream nodes, and evaluation only runs for dirty nodes. Important subjects include dirty bit propagation, invalidation scope control, minimal invalidation strategies, and how Blender tags depsgraph component nodes dirty in response to RNA property changes or user edits.

**Incremental re-evaluation:** Incremental evaluation limits recomputation to only the subgraph made dirty by a change, skipping all clean nodes and reusing their cached outputs. Learn this by combining dirty propagation with a cache per node so clean nodes return their stored result immediately. Important subjects include change detection, affected-subgraph extraction, cache validity rules, and how DEG avoids full-scene recomputation after small localized edits by evaluating only the dirty component path.

**Parallel execution where possible:** Independent nodes in the evaluation graph that share no dependencies can be evaluated concurrently on separate threads, multiplying throughput on multi-core hardware. Learn this by identifying sets of nodes with no shared dependency edges and dispatching them to a thread pool simultaneously. Important subjects include thread safety, task graph scheduling, work queues, data race prevention, and how Blender's DEG uses the task scheduler to run independent object evaluations in parallel during a scene update.

Why this matters in Blender:

- Depsgraph is central to animation, constraints, modifiers, and scene updates.
- Bugs in dependency handling often appear as stale data, wrong evaluation order, or update storms.

---

## 2. Design Patterns

Blender is largely written in C and C++, but many object-oriented design patterns are implemented through structs, function pointers, and registration systems.

Source directory references:

- source/blender/blenkernel
- source/blender/modifiers
- source/blender/nodes
- source/blender/windowmanager
- source/blender/editors

### 2.1 Strategy Pattern

- What it is: A pattern that encapsulates multiple interchangeable algorithms behind a common interface.
- Why it matters: It simplifies swapping behavior without modifying call sites.
- Where to apply it in Blender: Strategy-style dispatch in BKE modifier and constraint systems, and selected kernel backends in intern modules.

Common use:

- Modifiers
- Constraints
- Shader and geometry operations

Concept:

- Multiple interchangeable algorithms implement a shared behavioral contract.

### 2.2 Factory Pattern

- What it is: A creation pattern that builds concrete objects from type information through centralized logic.
- Why it matters: It keeps instantiation consistent and decouples creation from usage.
- Where to apply it in Blender: Type construction in RNA, node type registration in NOD, and operator registration in WM.

Common use:

- Creating RNA types
- Node instance creation
- Operator and type registration flows

Concept:

- Centralized construction logic creates concrete objects from a descriptor or type id.

### 2.3 Visitor Pattern

- What it is: A pattern that applies operations to structure elements without embedding the operation in each element type.
- Why it matters: It improves extensibility when traversing heterogeneous structures.
- Where to apply it in Blender: Visitor-style traversals in NOD node trees, DEG relation/eval walks, and export passes in IO modules.

Common use:

- Traversing graph-like structures
- Performing operations without embedding logic in every node type

Concept:

- Separate traversal logic from operations over heterogeneous elements.

### 2.4 Command Pattern

- What it is: A pattern that wraps actions as command objects or units with executable behavior.
- Why it matters: It enables reliable undo/redo and action history management.
- Where to apply it in Blender: Command flow in WM operators and ED undo integration, including invoke/exec/cancel paths.

Common use:

- Operator execution model
- Undo and redo stacks

Concept:

- Encapsulate actions as commands with execute, undo, and redo semantics.

### 2.5 Observer Pattern

- What it is: A publish-subscribe pattern where observers react to state changes in observed subjects.
- Why it matters: It keeps systems synchronized while reducing direct coupling.
- Where to apply it in Blender: Observer patterns in WM notifiers, DEG update signaling, and ED/UI redraw triggers.

Common use:

- Dependency update signaling
- UI redraw and notification systems

Concept:

- Changes in one subsystem notify dependent subsystems without tight coupling.

### 2.6 Builder Pattern

- What it is: A staged construction pattern for assembling complex objects step by step.
- Why it matters: It improves clarity and reduces errors in multi-part configuration.
- Where to apply it in Blender: Staged definitions in RNA properties and builder-like creation of node graphs in NOD/BKE.

Common use:

- Defining RNA properties in steps
- Programmatic construction of complex node trees

Concept:

- Build complex objects incrementally with clear staged configuration.

### 2.7 Dataflow Pattern

- What it is: A computational model where connected operators transform data as it flows through a graph.
- Why it matters: It makes dependency-driven evaluation and modular graph design natural.
- Where to apply it in Blender: Dataflow execution in NOD/BKE for Geometry Nodes, shader graphs, and compositor evaluation.

Common use:

- Geometry Nodes
- Shader graph execution
- Compositor pipelines

Concept:

- Data moves through connected processing units where links define flow and dependencies.

---

## 3. Mathematics

Strong math fundamentals are mandatory for graphics, simulation, and animation code.

Source directory references:

- source/blender/blenlib
- source/blender/blenkernel
- source/blender/geometry
- source/blender/draw

### 3.1 Linear Algebra

- What it is: The mathematical foundation for vectors, matrices, rotations, and coordinate space conversions.
- Why it matters: Nearly all transform and camera computations depend on it.
- Where to apply it in Blender: Math utilities in BLI, transform/rigging logic in BKE, and projection math in draw and camera code.

You should know:

**Vectors and vector operations:** Start by searching for "vector dot product cross product projection normalization" and work through small 2D and 3D examples until direction, length, angle, and projection feel automatic. To make this second nature, focus on vector addition/subtraction, dot products for angle/alignment tests, cross products for orthogonal directions, and normalization for direction-only quantities.

**Matrices and matrix multiplication:** Search for "4x4 transformation matrix graphics" and "matrix multiplication order model view projection" so you understand how transforms are represented and combined. The important subjects are matrix layout, row-major vs column-major conventions, multiplication order, homogeneous coordinates, and why multiplying in the wrong order produces visually wrong transforms.

**Quaternions and rotation composition:** Search for "quaternion intuition graphics" and "quaternion vs Euler angles" before diving into formulas, because the intuition matters more than memorizing symbols. You want to know how quaternions avoid gimbal lock, how quaternion multiplication composes rotations, how to convert to and from matrices/Euler angles, and how interpolation with SLERP behaves.

**Transform composition and decomposition:** Search for "TRS decomposition graphics" and "compose transform translate rotate scale" to learn how full object transforms are built and broken back into parts. The important subjects are translation-rotation-scale order, extracting rotation and scale from a matrix, handling non-uniform scale, and understanding where decomposition becomes numerically awkward.

**Inverse transforms and coordinate conversion:** Search for "inverse transform local to world world to local" and practice converting points, directions, and normals between spaces. You should understand matrix inverse basics, why points and directions are transformed differently, how inverse-transpose relates to normals, and how coordinate conversion is used in picking, constraints, and parenting.

**Projection matrices and camera space:** Search for "perspective projection matrix graphics" and "view matrix camera space clip space" to understand how a 3D scene becomes a 2D image. The important subjects are view space, clip space, NDC, perspective divide, near/far planes, field of view, and the model-view-projection pipeline.

Why this matters in Blender:

- Transform evaluation, camera projection, rigging, and viewport math all depend on linear algebra.

### 3.2 Geometry

- What it is: The study of shape, surface properties, and geometric relationships in 2D/3D space.
- Why it matters: Robust geometric operations are required for modeling tools and shading correctness.
- Where to apply it in Blender: Geometry kernels in BLI/BKE, mesh normal/tangent code, and snapping/picking in ED/GPU draw paths.

You should know:

**Surface normals and tangent spaces:** Search for "vertex normal face normal tangent space" and "normal mapping tangent bitangent" to learn how surfaces carry orientation information for lighting and shading. The important subjects are geometric normals vs shading normals, averaged vertex normals, tangent-bitangent-normal frames, and how UV layout affects tangent generation.

**Barycentric coordinates:** Search for "barycentric coordinates triangle interpolation" and "point in triangle barycentric" because barycentric thinking shows up in interpolation, hit testing, and rasterization. You should understand how weights inside a triangle are computed, how they interpolate attributes, and how they are used to detect whether a point lies inside or outside a triangle.

**Ray-triangle and ray-mesh intersection:** Search for "Moller Trumbore algorithm" and "ray mesh intersection BVH" to understand the standard geometric tests behind picking and ray casting. Important subjects are parametric ray form, triangle plane intersection, hit distance t, front-face vs back-face handling, and how acceleration structures reduce the cost of testing many triangles.

**Mesh topology basics:** Search for "manifold vs non-manifold mesh" and "mesh adjacency vertices edges faces" to build an intuition for how geometry is connected beyond just coordinates. Important subjects are adjacency, boundary edges, poles, loops, rings, manifoldness, and why topological validity matters for booleans, subdivision, and editing tools.

Why this matters in Blender:

- Geometry processing, snapping, picking, modifiers, and shading rely on geometric computations.

### 3.3 Numerical Methods

- What it is: Techniques for stable approximation, interpolation, and iterative computation with floating-point numbers.
- Why it matters: Numerical instability can produce jitter, drift, and non-deterministic behavior.
- Where to apply it in Blender: FCurve interpolation in BKE animation, curve/spline evaluation in BKE, and numerics in simulation intern modules.

You should know:

**Interpolation methods (LERP, SLERP):** Search for "LERP vs SLERP graphics" and practice interpolating scalars, vectors, and rotations to see where each method is appropriate. The important subjects are linear interpolation, spherical interpolation, parameter t, interpolation artifacts, and why rotations need different handling than positions.

**Curve and spline evaluation:** Search for "Bezier curve evaluation", "Hermite spline", and "Catmull-Rom spline" so you can compare the most common curve families. Focus on control points, tangents, continuity, parameterization, and sampling because these ideas recur in animation curves, paths, and shape tools.

**Floating-point precision and error accumulation:** Search for "floating point precision graphics" and "catastrophic cancellation numerical analysis" to understand why mathematically correct code can still behave badly on a computer. The important subjects are machine epsilon, rounding error, cancellation, loss of significance, and the difference between float and double tradeoffs.

**Numerical stability in iterative systems:** Search for "numerical stability simulation" and "integration error iterative methods" to see how small errors compound over repeated updates. Important subjects include stable vs unstable recurrence, timestep sensitivity, error propagation, clamping, damping, and why iterative solvers or simulations can explode without careful formulation.

Why this matters in Blender:

- Animation curves, drivers, constraints, and simulation steps require numerically stable implementations.

---

## 4. Algorithms

Blender combines classic algorithms with domain-specific optimizations.

Source directory references:

- source/blender/depsgraph
- source/blender/bmesh
- source/blender/modifiers
- source/blender/geometry
- source/blender/render

### 4.1 Graph Algorithms

- What it is: Algorithms for traversing, ordering, and validating node/edge relationships in directed graphs.
- Why it matters: Correct dependency order is essential for deterministic evaluation.
- Where to apply it in Blender: Core graph algorithms in DEG relation building, topological sort, and cycle handling.

You should know:

**Topological sorting:** Search for "topological sort Kahn algorithm" and "DFS topological sort" to learn the two standard ways to linearize a dependency graph. The important subjects are partial order, indegree counting, DFS post-order, and how valid evaluation order emerges from directed acyclic dependencies.

**Cycle detection:** Search for "cycle detection directed graph DFS" and "strongly connected components overview" to understand how invalid dependency loops are discovered. Important subjects are visited states, recursion stack detection, SCC intuition, and how cycle reporting helps explain invalid graph structures to users and developers.

**Dependency-driven evaluation:** Search for "dependency graph evaluation" and "incremental graph recomputation" to learn how updates are triggered by changed inputs rather than brute-force reevaluation. Focus on dirty propagation, evaluation scheduling, cached results, and the difference between building dependency relations and executing them.

Why this matters in Blender:

- Core to dependency graph scheduling and deterministic update order.

### 4.2 Mesh Algorithms

- What it is: Algorithms that edit mesh connectivity and geometry for modeling and procedural workflows.
- Why it matters: Efficient topology operations directly affect tool responsiveness and result quality.
- Where to apply it in Blender: Topology algorithms in BMesh/BKE, subdivision and boolean code in modifiers, and edit tools in ED mesh.

You should know:

**Topology traversal strategies:** Search for "mesh adjacency traversal" and "half-edge traversal" to learn how algorithms walk from one element to neighboring elements efficiently. Important subjects are vertex-edge-face adjacency, iterator patterns, local neighborhood queries, and how traversal choice affects performance and correctness.

**Edge loop and edge ring detection:** Search for "edge loop edge ring mesh" and study examples in quad-dominant meshes until the pattern becomes visually obvious. The important subjects are opposite-edge traversal in quads, poles and termination cases, manifold assumptions, and why loops/rings matter for selection and modeling tools.

**Subdivision logic:** Search for "Catmull-Clark subdivision explained" and "Loop subdivision basics" to learn how coarse control cages become smooth surfaces. Focus on refinement rules, limit surfaces, crease handling, vertex repositioning, and how subdivision changes topology as well as geometry.

**Boolean and remeshing fundamentals:** Search for "mesh boolean robustness" and "surface remeshing overview" to understand why these topics are algorithmically difficult. Important subjects are intersection classification, winding or inside-outside tests, topology cleanup, voxel vs surface remeshing, and numerical robustness under imperfect input meshes.

Why this matters in Blender:

- Editing tools, modifiers, and procedural geometry operations are built on these primitives.

### 4.3 Spatial Algorithms

- What it is: Algorithms and data structures that accelerate spatial queries in 3D scenes.
- Why it matters: Naive geometric queries do not scale to production scene complexity.
- Where to apply it in Blender: Spatial acceleration in BLI BVH/KDTree, picking and ray cast utilities in ED/BKE, and render/physics query code.

You should know:

**Bounding volume hierarchy (BVH):** Search for "BVH construction traversal" and "surface area heuristic BVH" to understand how geometry is grouped hierarchically for fast culling. Important subjects are bounding boxes, splitting heuristics, traversal order, leaf size tradeoffs, and refit vs rebuild strategies.

**KD-tree usage patterns:** Search for "KD-tree nearest neighbor" and "KD-tree radius search" to learn where axis-aligned partition trees are still a strong fit. Focus on point set partitioning, nearest and k-nearest queries, balanced vs unbalanced trees, and how KD-trees differ from BVHs in use cases.

**Spatial partitioning and nearest-neighbor queries:** Search for "spatial partitioning data structures" and compare grids, octrees, BVHs, and KD-trees in terms of query type and update cost. Important subjects are partitioning strategy, update overhead, distance metrics, pruning, and choosing the right structure for the query pattern.

**Ray casting acceleration techniques:** Search for "ray casting acceleration structure" and "packet ray traversal" to understand how repeated intersection queries are made practical. Important subjects are broadphase vs narrowphase tests, traversal pruning, coherent rays, and how acceleration structures reduce the number of expensive primitive intersections.

Why this matters in Blender:

- Used in rendering, viewport picking, geometry queries, and physics intersections.

### 4.4 Rendering Algorithms

- What it is: Methods for simulating light transport and material response to produce final pixels.
- Why it matters: Image quality and render performance are both determined by algorithm choice.
- Where to apply it in Blender: Rendering algorithms in Eevee and Cycles, plus shared GPU/DRW light and shadow integration paths.

You should know:

**Rasterization pipeline basics:** Search for "graphics pipeline vertex fragment rasterization" and "clip space to screen space" to learn how triangles become pixels in real time. Important subjects are vertex processing, clipping, rasterization, interpolation, depth testing, blending, and the order in which GPU stages run.

**Ray tracing and path tracing foundations:** Search for "ray tracing basics" and "path tracing Monte Carlo" to understand the difference between single-ray visibility and stochastic light transport simulation. Focus on primary/secondary rays, recursion, BRDF sampling, noise vs convergence, and why path tracing produces realism at higher computational cost.

**Shadow mapping concepts:** Search for "shadow mapping explained" and "shadow map bias acne peter panning" to learn the standard real-time shadow technique and its tradeoffs. Important subjects are light-space depth maps, comparison sampling, resolution artifacts, bias tuning, and filtering methods such as PCF.

**Physically based rendering (PBR) principles:** Search for "PBR metallic roughness workflow" and "energy conservation BRDF" to understand the material model used by modern renderers. Important subjects are albedo, metallic, roughness, Fresnel, microfacet BRDFs, energy conservation, and the difference between artistic controls and physical meaning.

Why this matters in Blender:

- Eevee and Cycles use different but related rendering models that share core graphics ideas.

---

## 5. 3D Engine Fundamentals

Before reading GPU and renderer internals, build a solid graphics pipeline foundation.

Source directory references:

- source/blender/gpu
- source/blender/draw
- source/blender/render
- intern/cycles

### 5.1 GPU Concepts

- What it is: Core concepts for how graphics hardware processes geometry and shading workloads.
- Why it matters: Correct buffer, shader, and state management is critical for rendering correctness and speed.
- Where to apply it in Blender: GPU abstraction and resource APIs in GPU module, draw scheduling in DRW, and backend bindings per platform.

You should know:

**Vertex buffers and index buffers:** Search for "vertex buffer index buffer graphics" and build a minimal triangle renderer so you can see how mesh data is fed to the GPU. Important subjects are vertex layout, attribute formats, indexed drawing, memory upload, and why index reuse reduces bandwidth.

**Shader stages and shader programs:** Search for "vertex shader fragment shader pipeline" and then expand to geometry, compute, or mesh shading only after the core is clear. Important subjects are programmable stages, stage inputs/outputs, shader linkage, compilation, and how stages cooperate to transform and shade primitives.

**Uniforms, storage buffers, and resource binding:** Search for "uniform buffer vs storage buffer" and "descriptor binding graphics API" to understand how shaders access external data. Focus on binding models, constant vs large structured data, update frequency, alignment rules, and how incorrect binding causes hard-to-debug rendering failures.

**Framebuffers and render targets:** Search for "framebuffer render target graphics" and "offscreen rendering" to understand where rendering results are written before final presentation. Important subjects are color and depth attachments, multisampling, offscreen passes, texture-backed targets, and why render-to-texture is central to modern pipelines.

**Render passes and state changes:** Search for "render pass graphics API" and "pipeline state changes performance" to understand how workloads are grouped and submitted efficiently. Important subjects are batching, state sorting, render pass boundaries, barriers or synchronization concepts, and the performance cost of excessive state churn.

### 5.2 Rendering Pipelines

- What it is: The ordered rendering stages and data layout strategy used to produce frames.
- Why it matters: Pipeline architecture determines performance, memory usage, and achievable visual features.
- Where to apply it in Blender: Pipeline orchestration in DRW/Eevee frame passes, with GPU state transitions and viewport render flow.

You should know:

**Forward rendering:** Search for "forward rendering advantages disadvantages" and implement or study a small forward-lit scene first because it is the easiest pipeline to reason about. Important subjects are per-object lighting cost, transparency friendliness, MSAA compatibility, and how shading cost scales with light count.

**Deferred rendering:** Search for "deferred rendering G-buffer" to understand how geometry and lighting are split into separate stages. Important subjects are G-buffer design, screen-space lighting, bandwidth cost, memory usage, transparency limitations, and why deferred rendering excels with many dynamic lights.

**Hybrid rendering approaches:** Search for "hybrid renderer raster ray tracing" and "forward plus clustered lighting" to see how real engines mix ideas rather than staying pure. Focus on when a renderer mixes rasterization with ray tracing, or forward with tiled/clustered lighting, and how those hybrids balance quality, speed, and complexity.

### 5.3 Shader Languages and Intermediate Forms

- What it is: Source and intermediate representations used to author and translate GPU shader programs.
- Why it matters: Cross-platform rendering depends on consistent shader translation and backend compatibility.
- Where to apply it in Blender: Shader authoring in GPU shader sources, backend compilation paths, and platform translation layers.

You should know:

**GLSL basics:** Search for "GLSL tutorial vertex fragment shader" and write a minimal shader pair to understand syntax, inputs, outputs, and built-in variables. Important subjects are shader language types, interpolation qualifiers, texture sampling, coordinate conventions, and compiling/debugging shader code.

**Metal shading concepts:** Search for "Metal Shading Language basics" and compare its resource and stage model to GLSL so you understand the backend translation problem. Important subjects are argument buffers or resource bindings, stage functions, language syntax differences, and Apple-platform pipeline conventions.

**SPIR-V as an intermediate representation concept:** Search for "SPIR-V overview" and "shader IR compilation pipeline" to understand why engines often target an intermediate form instead of a source language directly. Important subjects are IR design, cross-compilation, validation, optimization passes, and how an intermediate representation helps support multiple backends from one source pipeline.

Why this matters in Blender:

- Blender's GPU layer abstracts multiple backends, but backend constraints and shader translation details still affect engine behavior.

---

## 6. Data Structures

Blender's performance and flexibility depend on choosing the right structures for geometry and runtime systems.

Source directory references:

- source/blender/bmesh
- source/blender/makesdna
- source/blender/blenkernel
- source/blender/nodes
- source/blender/blenloader

### 6.1 Half-Edge and Editable Mesh Models (BMesh Concepts)

- What it is: Connectivity-centric mesh structures that store explicit relations between vertices, edges, loops, and faces.
- Why it matters: Reliable topology editing requires fast, consistent neighborhood access.
- Where to apply it in Blender: Editable topology structures in BMesh and BKE mesh editing, plus operator-driven edits in ED.

You should know:

**Vertex, edge, loop, and face connectivity:** Search for "half-edge mesh data structure" and "BMesh loops explained" to understand how editable mesh systems represent neighborhood relationships explicitly. Important subjects are adjacency, loop vs edge roles, face boundaries, and why connectivity representation matters more than raw positions in edit-mode tools.

**Efficient topology edits:** Search for "editable mesh operations data structure" and study how split, collapse, dissolve, and extrude can be implemented without rebuilding the whole mesh. Important subjects are local updates, invariant preservation, element reuse, and keeping adjacency consistent after every edit.

**Euler-style topology operations:** Search for "Euler operators mesh topology" to learn the classical primitive operations from which more complex modeling commands are built. Important subjects are make/kill vertex-edge-face style operations, topological validity, and how complex edits can be decomposed into a safe sequence of primitive steps.

Why this matters in Blender:

- Modeling tools and mesh editing workflows rely on stable and efficient topology mutation.

### 6.2 Attribute and Custom Data Layers

- What it is: Flexible layered storage for per-element mesh or geometry attributes.
- Why it matters: Modern procedural workflows depend on preserving and transforming attributes correctly.
- Where to apply it in Blender: Attribute storage in DNA/BKE CustomData, geometry node attributes in NOD/BKE, and modifier propagation logic.

You should know:

**Layered storage for per-element data:** Search for "mesh attribute layers" and "per-vertex per-face-corner data" to understand how geometry stores multiple parallel attribute channels. Important subjects are attribute domains, storage indirection, memory ownership, and why not every attribute belongs to the same element type.

**Dynamic schemas for UVs, colors, normals, and custom attributes:** Search for "dynamic attribute schema geometry nodes" and study how new named attributes can be added without changing the core struct definition. Important subjects are schema flexibility, typed attribute storage, domain-specific attributes, and how named layers are discovered and managed.

**Data propagation rules across operations:** Search for "attribute propagation mesh operations" and think through what should happen to UVs, normals, and generic attributes when topology changes. Important subjects are interpolation, copying vs recomputing, domain adaptation, conflict resolution, and how procedural systems preserve semantic data through edits.

Why this matters in Blender:

- Attribute-heavy workflows (especially geometry nodes) depend on robust layer management.

### 6.3 Node Trees

- What it is: Graph data structures that represent nodes, typed sockets, links, and evaluation metadata.
- Why it matters: Strong graph structure is required for validation, scheduling, and execution correctness.
- Where to apply it in Blender: Node graph structures in NOD/BKE, editor integration in ED, and serialization support tied to DNA data.

You should know:

**Node, socket, and link representations:** Search for "node graph data model" and sketch a minimal in-memory representation of nodes, sockets, and links before reading production code. Important subjects are graph ownership, stable IDs, connection endpoints, and how serialization stores graph structure.

**Typed connections and validation:** Search for "node graph type checking" and "graph validation rules" to learn how a graph rejects invalid links before execution. Important subjects are socket compatibility, implicit conversions, validation passes, and distinguishing authoring-time errors from execution-time errors.

**Execution context and evaluation metadata:** Search for "execution context graph evaluation" and "node metadata scheduling" to understand what extra information an evaluator needs beyond raw graph structure. Important subjects are evaluation flags, caching metadata, dirty state, context objects, and runtime information attached to nodes or sockets.

Why this matters in Blender:

- A major portion of modern Blender workflows is node-driven.

### 6.4 Spatial Trees (BVH and Related Structures)

- What it is: Hierarchical spatial partitioning structures that accelerate query operations in 3D space.
- Why it matters: They make rendering and geometric query workloads tractable at scale.
- Where to apply it in Blender: BVH-like acceleration in render/geometry code, physics broadphase in simulation modules, and spatial lookups in NOD/BKE.

You should know:

**Tree construction tradeoffs:** Search for "BVH build quality vs speed" and "spatial tree construction heuristics" to understand why the fastest builder does not always produce the fastest queries. Important subjects are build time, tree quality, split heuristics, leaf size, and the runtime consequences of those choices.

**Query performance characteristics:** Search for "nearest neighbor complexity BVH KD-tree" and compare common query types such as ray hits, nearest neighbor, overlap, and range search. Important subjects are asymptotic complexity, pruning efficiency, coherence, and how data distribution changes practical performance.

**Update and rebuild strategies:** Search for "BVH refit vs rebuild" and "dynamic spatial acceleration structure" to learn how mutable scenes complicate tree maintenance. Important subjects are full rebuilds, incremental refits, partial updates, scene coherence, and the quality degradation that can accumulate if updates are too cheap for too long.

Why this matters in Blender:

- Spatial acceleration structures appear throughout rendering, physics, and geometry queries.

---

## 7. Compiler and Reflection Concepts

Blender's DNA/RNA pipeline combines reflection-like metadata with binary file compatibility concerns.

Source directory references:

- source/blender/makesdna
- source/blender/makesrna
- source/blender/blenloader
- source/blender/python

### 7.1 Reflection Concepts (RNA)

- What it is: Runtime metadata that describes types, properties, and access rules for data introspection.
- Why it matters: It enables dynamic tooling, scripting exposure, and consistent UI/property behavior.
- Where to apply it in Blender: Type and property reflection in RNA, Python bindings in bpy integration, and UI property panels via RNA metadata.

You should know:

**Runtime type metadata:** Search for "reflection runtime type metadata" and study how a program can describe its own types without hardcoding every use site. Important subjects are type descriptors, field metadata, enum/property metadata, and how runtime systems query this information to build tools.

**Property descriptors and registration:** Search for "property descriptor system" and "registered properties reflection" to learn how properties are declared once and reused everywhere. Important subjects are property names, types, ranges, defaults, callbacks, registration tables, and how descriptors drive UI, serialization, and scripting.

**Introspection and dynamic access patterns:** Search for "introspection dynamic property access" and practice writing code that reads or writes fields by descriptor instead of direct struct access. Important subjects are lookup by name or identifier, reflective iteration over properties, dynamic setters/getters, and the tradeoff between flexibility and static safety.

**API exposure from native code to scripting layers:** Search for "binding native code to Python" and "reflection driven API generation" to understand how C/C++ data becomes script-visible. Important subjects are marshaling, lifetime ownership, reference policies, method/property exposure, and how metadata can reduce manual wrapper code.

Why this matters in Blender:

- RNA is the bridge between internal data definitions and user-facing/API-visible behavior.

### 7.2 Binary Serialization Concepts (DNA)

- What it is: Binary data layout and persistence rules for storing and restoring structured application state.
- Why it matters: File compatibility and safe version migration depend on serialization discipline.
- Where to apply it in Blender: Binary layout and file IO in DNA/Blendfile code, with versioning and pointer remap logic during load.

You should know:

**Struct layout awareness:** Search for "binary struct layout padding alignment" and "ABI struct layout" to understand why binary formats depend on exact data layout rules. Important subjects are padding, alignment, field ordering, fixed-size types, and why even a small layout change can break serialized compatibility.

**Version compatibility strategies:** Search for "file format version migration" and "backward compatible binary format" to learn how old data is upgraded safely. Important subjects are schema evolution, version tags, migration code, optional fields, and strategies for preserving compatibility while the internal runtime model changes.

**Pointer remapping and loading logic:** Search for "pointer remapping serialized file load" and "relocation table binary loader" to understand how in-memory references are reconstructed after reading raw bytes. Important subjects are address independence, ID mapping, relocation, deferred fix-up passes, and how loaders rebuild object graphs safely.

**Backward compatibility constraints:** Search for "backward compatibility file format engineering" and study why format decisions become long-lived obligations. Important subjects are compatibility guarantees, migration cost, deprecated fields, round-tripping old files, and how engineering tradeoffs change when legacy support must be preserved for years.

Why this matters in Blender:

- Blender's file format and data migration behavior depend on stable serialization contracts.
