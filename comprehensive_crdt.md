comprehensive_crdt
==================
- Category: Learning
- Tags: 
- Created: 2025-01-31T16:23:38-08:00

``https://inria.hal.science/inria-00555588/document``

## Introduction

## Background and System Model

### Atoms and Objects
- A process may store **atoms** and **objects**
	- **Atoms** are immutable primitives
	- **Objects** are mutable containers for primitives. Made up of...
		- **Identity** - unique identifier for the object
			- Two objects with the same identity in different processes are called **replicas**
		- **Content** (AKA payload)
			- Any number of atoms or objects
		- **Initial state** - what does the object initialize as?
		- **Operations**, utilized through an interface

### Operations
- Our environment consists of unspecified **clients** that can:
	- Query information
		- This happens locally (between client and **source replica**)
	- Modify/update information
		- Split into two phases
			- Client calls operation at source, which may cause initial processing
			- Update is transmitted asynchronously to all replicas - **downstream** part
		- Two styles of updating information
			- **State-based** replication
				- 
			- **Operation-based** replication (AKA op-based)
