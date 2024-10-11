s3Dgraphy Library Documentation
===============================

.. contents::
   :depth: 2
   :local:

Introduction
------------

Welcome to the documentation for the **s3Dgraphy** library. `s3Dgraphy` is a Python-based library designed to manage, manipulate, and export complex graph data structures, particularly tailored for integration with Blender for 3D modeling and visualization purposes. This documentation provides a comprehensive overview of the library's functionalities, data structures, mapping to the CIDOC CRM (Conceptual Reference Model), and the structure of the exported JSON files.

**Key Features:**
- **Graph Management:** Create, load, and manage multiple graphs efficiently.
- **Node and Edge Handling:** Support for various node types (e.g., GeoPosition, Epoch) and diverse edge types.
- **JSON Export:** Export graph data into a structured JSON format for interoperability.
- **CIDOC CRM Mapping:** Align graph data structures with the CIDOC CRM for standardized cultural heritage information representation.

Graph Data Structure
--------------------

At the core of `s3Dgraphy` is its robust graph data structure, which comprises **nodes** and **edges**. This section delves into the specifics of each component, detailing the various node types and the semantics of different edge types.

### Nodes

Nodes represent entities within the graph. Each node has a unique identifier, a type, a name, and associated data. The library supports multiple node types to encapsulate different kinds of information.

#### GeoPositionNode

**Purpose:**  
Represents the geographical position of the entire graph. Since each graph is associated with a single geographical location, there is only one `GeoPositionNode` per graph.

**Attributes:**
- **node_id (str):** Unique identifier for the node.
- **name (str):** Fixed as `"geo_position"`.
- **node_type (str):** Fixed as `"geo_position"`.
- **data (dict):** Contains geographical metadata.

**Data Fields:**
- **epsg (int):** EPSG code representing the coordinate system.
- **shift_x (float):** Shift along the X-axis.
- **shift_y (float):** Shift along the Y-axis.
- **shift_z (float):** Shift along the Z-axis.

**Implementation:**

.. code-block:: python

    # s3Dgraphy/geo_position_node.py

    from .node import Node

    class GeoPositionNode(Node):
        """
        Represents the geographical position of the graph.
        """
        def __init__(self, node_id, epsg=4326, shift_x=0.0, shift_y=0.0, shift_z=0.0):
            super().__init__(node_id=node_id, name="geo_position", node_type="geo_position")
            self.data = {
                "epsg": epsg,
                "shift_x": shift_x,
                "shift_y": shift_y,
                "shift_z": shift_z
            }

        def to_dict(self):
            return {
                "type": self.node_type,
                "name": self.name,
                "data": self.data
            }

#### EpochNode

**Purpose:**  
Represents geological epochs within the graph, allowing for temporal segmentation and analysis.

**Attributes:**
- **node_id (str):** Unique identifier for the node.
- **name (str):** Name of the epoch.
- **node_type (str):** Fixed as `"epoch"`.
- **data (dict):** Contains epoch-specific metadata.

**Data Fields:**
- **min (float):** Minimum Y-coordinate in the VisualGraph.
- **max (float):** Maximum Y-coordinate in the VisualGraph.
- **start (float):** Start time (absolute).
- **end (float):** End time (absolute).
- **color (str):** Hex color code representing the epoch.

**Implementation:**

.. code-block:: python

    # s3Dgraphy/epoch_node.py

    from .node import Node

    class EpochNode(Node):
        """
        Represents a geological epoch within the graph.
        """
        def __init__(self, node_id, name, min_coord, max_coord, start_time, end_time, color="#FFFFFF"):
            super().__init__(node_id=node_id, name=name, node_type="epoch")
            self.data = {
                "min": min_coord,
                "max": max_coord,
                "start": start_time,
                "end": end_time,
                "color": color
            }

        def to_dict(self):
            return {
                "type": self.node_type,
                "name": self.name,
                "data": self.data
            }

#### PropertyNode

**Purpose:**  
Encapsulates properties associated with stratigraphic nodes, such as temporal data or physical characteristics.

**Attributes:**
- **node_id (str):** Unique identifier for the node.
- **name (str):** Name of the property (e.g., `"start_time"`, `"end_time"`, `"existence"`).
- **node_type (str):** Fixed as `"property"`.
- **data (dict):** Contains property-specific metadata.

**Data Fields:**
- **description@it (str):** Description in Italian.
- **author (str):** Author identifier (e.g., ORCID).
- **time_start (float, optional):** Start time (applicable for temporal properties).
- **time_end (float, optional):** End time (applicable for temporal properties).
- **url (str, optional):** Associated URL.

**Implementation:**

.. code-block:: python

    # s3Dgraphy/property_node.py

    from .node import Node

    class PropertyNode(Node):
        """
        Represents a property associated with a stratigraphic node.
        """
        def __init__(self, node_id, name, description, author, time_start=None, time_end=None, url=None):
            super().__init__(node_id=node_id, name=name, node_type="property")
            self.data = {
                "description@it": description,
                "author": author,
                "url": url
            }
            if time_start is not None:
                self.data["time_start"] = time_start
            if time_end is not None:
                self.data["time_end"] = time_end

        def to_dict(self):
            return {
                "type": self.node_type,
                "name": self.name,
                "data": self.data
            }

#### ShiftNode

**Purpose:**  
Represents spatial shifts or transformations applied to the graph or its components.

**Attributes:**
- **node_id (str):** Unique identifier for the node.
- **name (str):** Fixed as `"shift"`.
- **node_type (str):** Fixed as `"shift"`.
- **data (dict):** Contains shift-specific metadata.

**Data Fields:**
- **shift_x (float):** Shift along the X-axis.
- **shift_y (float):** Shift along the Y-axis.
- **shift_z (float):** Shift along the Z-axis.

**Implementation:**

.. code-block:: python

    # s3Dgraphy/shift_node.py

    from .node import Node

    class ShiftNode(Node):
        """
        Represents a spatial shift applied to the graph.
        """
        def __init__(self, node_id, shift_x=0.0, shift_y=0.0, shift_z=0.0):
            super().__init__(node_id=node_id, name="shift", node_type="shift")
            self.data = {
                "shift_x": shift_x,
                "shift_y": shift_y,
                "shift_z": shift_z
            }

        def to_dict(self):
            return {
                "type": self.node_type,
                "name": self.name,
                "data": self.data
            }

#### Other Node Types

`s3Dgraphy` supports various other node types to represent different entities and relationships within the graph. These include:

- **CombinerNode:** Represents logical combinations or inferences.
- **ExtractorNode:** Represents data extraction processes or sources.
- **DocumentNode:** Represents documents or references.
- **StratigraphicNode (US):** Represents stratigraphic units or layers.

Each of these node types inherits from the base `Node` class and includes specific attributes and data fields pertinent to their roles.

### Edges

Edges define the relationships between nodes in the graph. Each edge has a unique identifier, a source node, a target node, and a type that defines the nature of the relationship.

#### Edge Types and Their Meanings

`s3Dgraphy` categorizes edges into various types to represent different kinds of relationships. Understanding these edge types is crucial for interpreting the graph's structure and semantics.

1. **line:** Represents chronological or temporal relationships, indicating that one event or entity precedes another.
   
   *Example:* A restoration process occurring after the creation of a structure.

2. **dashed:** Represents data provenance or scientific provenance, indicating the origin or source of data.
   
   *Example:* A property node derived from a specific stratigraphic unit.

3. **dotted:** Represents entities or objects traveling through time, indicating temporal changes or movements.
   
   *Example:* An object that changes its properties over different epochs.

4. **double_line:** Represents contemporaneity between two stratigraphic units or entities.
   
   *Example:* Two geological layers existing simultaneously.

5. **dashed_dotted:** Represents conflicting properties or hypotheses, indicating mutual exclusivity.
   
   *Example:* Two differing temporal datings of the same stratigraphic unit.

6. **TBD:** Represents undefined or yet-to-be-determined relationships. This category serves as a placeholder for edge types that are not yet classified.

**Note:** Edge types are essential for maintaining the semantic integrity of the graph and facilitating meaningful data interpretations.

Mapping to CIDOC CRM
--------------------

The **CIDOC Conceptual Reference Model (CRM)** is a formal ontology intended to facilitate the integration, mediation, and interchange of heterogeneous cultural heritage information. Mapping `s3Dgraphy`'s graph data structures to CIDOC CRM ensures interoperability with other systems adhering to this standard.

### Overview of CIDOC CRM

**CIDOC CRM** is a standard developed by the International Committee for Documentation (CIDOC) of the International Council of Museums (ICOM). It provides a framework for defining the concepts and relationships used in cultural heritage documentation.

**Key Features:**
- **Entity Classes:** Defines classes like E1 CRM Entity, E2 Temporal Entity, E4 Period, E5 Event, etc.
- **Relationships:** Defines relationships like P4 has time-span, P7 took place at, etc.
- **Extensibility:** Allows for extension through specialized ontologies.

**Relevance to `s3Dgraphy`:**  
By aligning `s3Dgraphy`'s nodes and edges with CIDOC CRM classes and relationships, the library can ensure that its data is semantically rich and interoperable with other cultural heritage information systems.

### s3Dgraphy to CIDOC CRM Mapping

Below is a proposed mapping between `s3Dgraphy`'s graph elements and CIDOC CRM concepts. This mapping serves as a guideline and can be adjusted based on specific project requirements.

#### Nodes Mapping

+-----------------------+------------------------+-----------------------------------------------+
| **s3Dgraphy Node Type** | **CIDOC CRM Class**      | **Description**                               |
+=======================+========================+===============================================+
| **GeoPositionNode**   | E53 Place              | Represents a spatial location.               |
+-----------------------+------------------------+-----------------------------------------------+
| **EpochNode**         | E52 Time-Span          | Represents a temporal span or period.        |
+-----------------------+------------------------+-----------------------------------------------+
| **StratigraphicNode (US)** | E4 Period            | Represents a specific geological period.      |
+-----------------------+------------------------+-----------------------------------------------+
| **PropertyNode**      | E86 Degree of Confidence | Represents properties or attributes with varying levels of certainty. |
+-----------------------+------------------------+-----------------------------------------------+
| **CombinerNode**      | E24 Physical Man-Made Thing | Represents logical combinations or inferences (custom mapping needed). |
+-----------------------+------------------------+-----------------------------------------------+
| **ExtractorNode**     | E5 Event               | Represents data extraction events or sources.|
+-----------------------+------------------------+-----------------------------------------------+
| **DocumentNode**      | E62 Document           | Represents documents or references.          |
+-----------------------+------------------------+-----------------------------------------------+
| **ShiftNode**         | E53 Place (with modifiers) | Represents spatial shifts or transformations.|
+-----------------------+------------------------+-----------------------------------------------+

**Notes:**
- **GeoPositionNode:** Corresponds to E53 Place as it defines a specific geographical location associated with the graph.
- **EpochNode:** Aligns with E52 Time-Span, encapsulating the temporal characteristics of geological epochs.
- **StratigraphicNode (US):** Can be mapped to E4 Period, representing geological periods within the graph.
- **PropertyNode:** Could be linked to E86 Degree of Confidence for properties that have varying levels of certainty or completeness.
- **CombinerNode:** May require a custom subclass of existing CIDOC CRM classes or relationships to accurately represent logical combinations.
- **ExtractorNode:** Maps to E5 Event as it represents actions or processes that extract data.
- **DocumentNode:** Directly maps to E62 Document, representing textual or multimedia references.
- **ShiftNode:** Maps to E53 Place with additional modifiers to indicate spatial transformations.

#### Edges Mapping

Edges in `s3Dgraphy` define relationships between nodes and can be mapped to CIDOC CRM relationships as follows:

+---------------------+------------------------+-------------------------------------------------------+
| **s3Dgraphy Edge Type** | **CIDOC CRM Relationship** | **Description**                                       |
+=====================+========================+=======================================================+
| **line**            | P4 has time-span        | Indicates chronological relationships.               |
+---------------------+------------------------+-------------------------------------------------------+
| **dashed**          | P22 has modifier        | Represents data provenance or source.                |
+---------------------+------------------------+-------------------------------------------------------+
| **dotted**          | P1 is identified by     | Represents temporal changes or movements.            |
+---------------------+------------------------+-------------------------------------------------------+
| **double_line**     | P106 is composed of     | Represents contemporaneity between entities.         |
+---------------------+------------------------+-------------------------------------------------------+
| **dashed_dotted**   | P2 has type             | Represents conflicting properties or hypotheses.     |
+---------------------+------------------------+-------------------------------------------------------+
| **TBD**             | Custom Relationship     | Placeholder for undefined relationships.             |
+---------------------+------------------------+-------------------------------------------------------+

**Notes:**
- **line (P4 has time-span):** Captures the chronological sequencing of events or entities.
- **dashed (P22 has modifier):** Represents provenance, linking data to its origin.
- **dotted (P1 is identified by):** Indicates temporal changes or the passage of time affecting entities.
- **double_line (P106 is composed of):** Captures simultaneous existence or compositional relationships.
- **dashed_dotted (P2 has type):** Represents conflicting attributes or mutually exclusive properties.
- **TBD:** Serves as a placeholder for relationships that are yet to be classified or defined.

**Custom Relationships:**  
For edge types that do not have a direct correspondence in CIDOC CRM, custom relationships or extensions to the CRM ontology may be necessary to accurately capture their semantics.

JSON Export Structure
---------------------

The `export_json.py` module in `s3Dgraphy` enables the export of graph data into a structured JSON format. This section elucidates the expected JSON structure, its components, and how each part corresponds to the graph data.

### Overall JSON Layout

The exported JSON adheres to a hierarchical structure, encapsulating all relevant data in organized sections. The primary sections are:

- **context:** Currently empty, reserved for future contextual metadata.
- **multigraph:** Contains all loaded graphs, each identified by a unique ID.

**Basic Structure:**

.. code-block:: json

    {
        "context": {},
        "multigraph": {
            "graph1": {
                // Graph details
            },
            "graph2": {
                // Graph details
            }
            // Additional graphs...
        }
    }

### Detailed Breakdown

#### Context Section

- **Description:**  
  A placeholder for contextual metadata related to the entire export. Currently empty but can be extended in the future to include global settings, export parameters, or other relevant information.

- **Structure:**

.. code-block:: json

    "context": {}

#### Multigraph Section

- **Description:**  
  Contains multiple graphs, each identified by its unique `graph_id`. Each graph encompasses its metadata, nodes, and edges.

- **Structure:**

.. code-block:: json

    "multigraph": {
        "graph1": {
            // Graph details
        },
        "graph2": {
            // Graph details
        }
        // Additional graphs...
    }

#### Individual Graph Structure

Each graph within the `multigraph` section follows a consistent structure, comprising metadata, nodes, and edges.

**Structure:**

.. code-block:: json

    "graph1": {
        "name@it": "Acropoli",
        "description@it": "Modello 3D Acropoli di Segni",
        "audio@it": ["it.mp3","it2.mp3"],
        "video@it": ["it.mp4"],
        "image@it": ["it.jpg"],
        "data": {
            "geo_position": {
                "epsg": 3004,
                "shift_x": 0,
                "shift_y": 0,
                "shift_z": 0
            },
            "epochs": {
                "Geologic": {
                    "min": 1262.9586181640625,
                    "max": 1312.9586181640625,
                    "start": -1000,
                    "end": -574,
                    "color": "#BCBCBC"
                },
                "Archaic": {
                    "min": 1050.9586181640625,
                    "max": 1262.9586181640625,
                    "start": -575,
                    "end": -480,
                    "color": "#339966"
                }
            }
        },
        "nodes": {
            // Nodes
        },
        "edges": {
            // Edges
        }
    }

**Components:**

1. **Metadata Fields:**
   - **name@it (str):** Italian name of the graph.
   - **description@it (str):** Italian description of the graph.
   - **audio@it (list):** List of audio file paths in Italian.
   - **video@it (list):** List of video file paths in Italian.
   - **image@it (list):** List of image file paths in Italian.

2. **Data Section:**
   - **geo_position (dict):** Contains geographical metadata, including EPSG code and spatial shifts.
   - **epochs (dict):** Contains geological epochs, each with their temporal and visual attributes.

3. **Nodes Section:**
   - **nodes (dict):** Contains all nodes within the graph, each identified by a unique key.

4. **Edges Section:**
   - **edges (dict):** Categorizes edges based on their types, each containing a list of relationships.

### Example JSON Structure

Below is a comprehensive example illustrating the structure of an exported JSON file.

.. code-block:: json

    {
        "context": {},
        "multigraph": {
            "AcropoliGraph": {
                "name@it": "Acropoli",
                "description@it": "Modello 3D Acropoli di Segni",
                "audio@it": ["acropoli_it.mp3", "acropoli_it2.mp3"],
                "video@it": ["acropoli_it.mp4"],
                "image@it": ["acropoli_it.jpg"],
                "data": {
                    "geo_position": {
                        "epsg": 3004,
                        "shift_x": 100.0,
                        "shift_y": 200.0,
                        "shift_z": 300.0
                    },
                    "epochs": {
                        "Geologic": {
                            "min": 1262.9586,
                            "max": 1312.9586,
                            "start": -1000,
                            "end": -574,
                            "color": "#BCBCBC"
                        },
                        "Archaic": {
                            "min": 1050.9586,
                            "max": 1262.9586,
                            "start": -575,
                            "end": -480,
                            "color": "#339966"
                        }
                    }
                },
                "nodes": {
                    "geo_AcropoliGraph": {
                        "type": "geo_position",
                        "name": "geo_position",
                        "data": {
                            "epsg": 3004,
                            "shift_x": 100.0,
                            "shift_y": 200.0,
                            "shift_z": 300.0
                        }
                    },
                    "US01": {
                        "type": "US",
                        "name": "US01",
                        "data": {
                            "description@it": "Banco di roccia vergine",
                            "url": null,
                            "rel_time": 1307.9586,
                            "end_rel_time": 2024
                        }
                    },
                    "USD10.start::n0": {
                        "type": "property",
                        "name": "start_time",
                        "data": {
                            "description@it": " ",
                            "author": "orcID",
                            "time_start": 1500,
                            "time_end": 1650
                        }
                    },
                    "USD10.end::n0": {
                        "type": "property",
                        "name": "end_time",
                        "data": {
                            "description@it": " ",
                            "author": "orcID",
                            "time_start": 1700,
                            "time_end": 1750
                        }
                    },
                    "USD10.existence::n0": {
                        "type": "property",
                        "name": "existence",
                        "data": {
                            "description@it": " ",
                            "author": "ORCID"
                        }
                    },
                    "C.01": {
                        "type": "combiner",
                        "name": "C.01",
                        "data": {
                            "author": "orcID",
                            "description@it": " "
                        }
                    },
                    "n0::n16": {
                        "type": "extractor",
                        "name": "D.02.01",
                        "data": {
                            "description@it": " ",
                            "icon": false,
                            "url": "D.02.01.jpg",
                            "src": ""
                        }
                    },
                    "D.32.1": {
                        "type": "extractor",
                        "name": "D.32.1",
                        "data": {
                            "description@it": " ",
                            "author": " ",
                            "url_type": "image",
                            "url": "image.jpeg"
                        }
                    },
                    "n0::n15": {
                        "type": "document",
                        "name": "D.02",
                        "data": {
                            "description@it": "La chiesa di San Pietro in un'incisione di E. Dodwell (1834)",
                            "url_type": "image",
                            "url": "D.02.jpg"
                        }
                    }
                },
                "edges": {
                    "line": [
                        {
                            "from": "USV105",
                            "to": "SF01"
                        }
                    ],
                    "dashed": [
                        {
                            "from": "USD10.start::n0",
                            "to": "US01"
                        },
                        {
                            "from": "USD10.end::n0",
                            "to": "US01"
                        },
                        {
                            "from": "USD10.existence::n0",
                            "to": "US01"
                        }
                    ],
                    "dotted": [
                        {
                            "from": "n0::n17",
                            "to": "n0::n16"
                        }
                    ],
                    "double_line": [
                        {
                            "from": "n0::n17",
                            "to": "n0::n16"
                        }
                    ],
                    "dashed_dotted": [
                        {
                            "from": "n0::n17",
                            "to": "n0::n16"
                        }
                    ],
                    "TBD": [
                        {
                            "from": "n0::n17",
                            "to": "n0::n16"
                        }
                    ]
                }
            }
        }
    }

### Explanation of Each Section and Field

#### Context

- **Purpose:**  
  Currently reserved for contextual metadata related to the export process or global settings. It is empty in the current implementation but can be extended for future use cases.

- **Example:**

.. code-block:: json

    "context": {}

#### Multigraph

- **Purpose:**  
  Encapsulates all graphs managed by `s3Dgraphy`. Each graph is identified by a unique `graph_id`.

- **Example:**

.. code-block:: json

    "multigraph": {
        "AcropoliGraph": {
            // Graph details
        },
        // Additional graphs...
    }

#### Graph Details

Each graph contains comprehensive details, including metadata, nodes, and edges.

- **Metadata Fields:**
  - **name@it (str):** The name of the graph in Italian.
  - **description@it (str):** A description of the graph in Italian.
  - **audio@it (list):** List of audio file paths relevant to the graph in Italian.
  - **video@it (list):** List of video file paths relevant to the graph in Italian.
  - **image@it (list):** List of image file paths relevant to the graph in Italian.

- **Data Section:**
  - **geo_position (dict):** Geographical metadata associated with the graph.
  - **epochs (dict):** Geological epochs defined within the graph.

- **Nodes Section:**
  - **nodes (dict):** Contains all nodes within the graph, each identified by a unique key.

- **Edges Section:**
  - **edges (dict):** Categorizes edges based on their types, each containing a list of relationships.

**Example:**

.. code-block:: json

    "AcropoliGraph": {
        "name@it": "Acropoli",
        "description@it": "Modello 3D Acropoli di Segni",
        "audio@it": ["acropoli_it.mp3", "acropoli_it2.mp3"],
        "video@it": ["acropoli_it.mp4"],
        "image@it": ["acropoli_it.jpg"],
        "data": {
            "geo_position": {
                "epsg": 3004,
                "shift_x": 100.0,
                "shift_y": 200.0,
                "shift_z": 300.0
            },
            "epochs": {
                "Geologic": {
                    "min": 1262.9586,
                    "max": 1312.9586,
                    "start": -1000,
                    "end": -574,
                    "color": "#BCBCBC"
                },
                "Archaic": {
                    "min": 1050.9586,
                    "max": 1262.9586,
                    "start": -575,
                    "end": -480,
                    "color": "#339966"
                }
            }
        },
        "nodes": {
            // Nodes
        },
        "edges": {
            // Edges
        }
    }

#### Nodes

- **Purpose:**  
  Each node represents an entity within the graph, such as geographical positions, geological epochs, properties, or documents.

- **Structure:**

.. code-block:: json

    "nodes": {
        "node_id": {
            "type": "node_type",
            "name": "node_name",
            "data": {
                // Node-specific data
            }
        },
        // Additional nodes...
    }

**Example:**

.. code-block:: json

    "nodes": {
        "geo_AcropoliGraph": {
            "type": "geo_position",
            "name": "geo_position",
            "data": {
                "epsg": 3004,
                "shift_x": 100.0,
                "shift_y": 200.0,
                "shift_z": 300.0
            }
        },
        "US01": {
            "type": "US",
            "name": "US01",
            "data": {
                "description@it": "Banco di roccia vergine",
                "url": null,
                "rel_time": 1307.9586,
                "end_rel_time": 2024
            }
        }
        // Additional nodes...
    }

#### Edges

- **Purpose:**  
  Defines relationships between nodes, categorized by their types.

- **Structure:**

.. code-block:: json

    "edges": {
        "edge_type": [
            {
                "from": "source_node_id",
                "to": "target_node_id"
            },
            // Additional edges...
        ],
        // Additional edge types...
    }

**Example:**

.. code-block:: json

    "edges": {
        "line": [
            {
                "from": "USV105",
                "to": "SF01"
            }
        ],
        "dashed": [
            {
                "from": "USD10.start::n0",
                "to": "US01"
            }
        ],
        "TBD": [
            {
                "from": "n0::n17",
                "to": "n0::n16"
            }
        ]
        // Additional edges...
    }

Usage
-----

This section outlines how to utilize the `export_json.py` module within the `s3Dgraphy` library to export graph data into JSON format. It also covers integration with Blender through a dedicated Blender operator.

### Exporting Graphs to JSON

The `export_json.py` module provides functions to export all loaded graphs into a structured JSON format and save them to a specified file path.

#### Functions:

1. **`export_graphs_to_json`**

   - **Description:**  
     Exports all graphs managed by `MultiGraphManager` into a dictionary conforming to the specified JSON structure.

   - **Returns:**  
     `dict` representing the exported data.

2. **`save_export_to_json`**

   - **Description:**  
     Saves the exported graph data into a JSON file at the specified file path.

   - **Arguments:**
     - `export_data (dict)`: The data to be exported.
     - `filepath (str)`: The destination file path for the JSON file.

#### Implementation:

.. code-block:: python

    # s3Dgraphy/export_json.py

    import json
    from .multigraph import get_all_graph_ids, get_graph

    def export_graphs_to_json():
        """
        Exports all loaded graphs into a JSON-compliant dictionary.
        
        Returns:
            dict: The exported graph data.
        """
        export_data = {
            "context": {},
            "multigraph": {}
        }

        graph_ids = get_all_graph_ids()
        for graph_id in graph_ids:
            graph = get_graph(graph_id)
            if graph is None:
                continue  # Skip if graph not found

            # Extract basic graph information
            graph_dict = {
                "name@it": graph.name.get("it", ""),
                "description@it": graph.description.get("it", ""),
                "audio@it": graph.audio.get("it", []),
                "video@it": graph.video.get("it", []),
                "image@it": graph.image.get("it", []),
                "data": {
                    # Extract geo_position node
                    "geo_position": {},
                    "epochs": graph.data.get("epochs", {})
                },
                "nodes": {},
                "edges": {
                    "line": [],
                    "dashed": [],
                    "dotted": [],
                    "double_line": [],
                    "dashed_dotted": [],
                    "TBD": []
                }
            }

            # Extract geo_position node
            geo_nodes = [node for node in graph.nodes if node.node_type == "geo_position"]
            if geo_nodes:
                geo_node = geo_nodes[0]  # Assuming only one geo_position node per graph
                graph_dict["data"]["geo_position"] = geo_node.data

            # Extract nodes
            for node in graph.nodes:
                node_dict = {
                    "type": node.node_type,
                    "name": node.name,
                    "data": node.data
                }
                graph_dict["nodes"][node.node_id] = node_dict

            # Extract edges
            for edge in graph.edges:
                edge_info = {
                    "from": edge.edge_source,
                    "to": edge.edge_target
                }
                if edge.edge_type in graph_dict["edges"]:
                    graph_dict["edges"][edge.edge_type].append(edge_info)
                else:
                    # If edge type is undefined, add under "TBD"
                    graph_dict["edges"]["TBD"].append(edge_info)

            export_data["multigraph"][graph_id] = graph_dict

        return export_data

    def save_export_to_json(export_data, filepath):
        """
        Saves the exported graph data to a JSON file.
        
        Args:
            export_data (dict): The data to export.
            filepath (str): The file path where the JSON will be saved.
        """
        with open(filepath, 'w', encoding='utf-8') as json_file:
            json.dump(export_data, json_file, ensure_ascii=False, indent=4)
        print(f"Exported JSON successfully saved to {filepath}")

### Integration with Blender

To facilitate seamless integration with Blender, an operator is provided that leverages the `export_json.py` module to export graph data directly from the Blender interface.

#### Blender Operator: `EM_export_JSON`

**Purpose:**  
Allows users to export graphs to JSON through Blender's export menu.

**Implementation:**

.. code-block:: python

    # s3Dgraphy/export_json_operator.py

    import bpy
    from bpy.props import StringProperty
    from bpy_extras.io_utils import ExportHelper
    from .export_json import export_graphs_to_json, save_export_to_json

    class EM_export_JSON(bpy.types.Operator, ExportHelper):
        """
        Operator to export graphs to JSON format.
        """
        bl_idname = "export.em_json"
        bl_label = "Export Graphs to JSON"
        bl_description = "Esporta i grafi in formato JSON"
        bl_options = {"REGISTER", "UNDO"}

        # Define the default file extension
        filename_ext = ".json"

        filter_glob: StringProperty(
            default="*.json",
            options={'HIDDEN'},
            maxlen=255,
        )

        def execute(self, context):
            # Export the graphs to JSON
            export_data = export_graphs_to_json()
            # Save the JSON to the specified filepath
            save_export_to_json(export_data, self.filepath)
            self.report({'INFO'}, f"Exported JSON successfully saved to {self.filepath}")
            return {'FINISHED'}

    def menu_func_export(self, context):
        self.layout.operator(EM_export_JSON.bl_idname, text="Export Graphs to JSON")

    def register():
        bpy.utils.register_class(EM_export_JSON)
        bpy.types.TOPBAR_MT_file_export.append(menu_func_export)

    def unregister():
        bpy.utils.unregister_class(EM_export_JSON)
        bpy.types.TOPBAR_MT_file_export.remove(menu_func_export)

    if __name__ == "__main__":
        register()

#### Adding the Operator to Blender's Export Menu

To make the `EM_export_JSON` operator accessible from Blender's export menu, include the `menu_func_export` function in the library's initialization.

**Example:**

.. code-block:: python

    # s3Dgraphy/__init__.py

    from .export_json_operator import EM_export_JSON, menu_func_export

    def register():
        # Register other classes...
        bpy.utils.register_class(EM_export_JSON)
        bpy.types.TOPBAR_MT_file_export.append(menu_func_export)

    def unregister():
        # Unregister other classes...
        bpy.utils.unregister_class(EM_export_JSON)
        bpy.types.TOPBAR_MT_file_export.remove(menu_func_export)

**Usage in Blender:**

1. **Access the Export Menu:**
   - Navigate to `File > Export` in Blender's top menu.

2. **Select "Export Graphs to JSON":**
   - Choose the "Export Graphs to JSON" option to open the file save dialog.

3. **Specify the Export Path:**
   - Choose the desired location and filename for the exported JSON file.

4. **Execute the Export:**
   - Click the "Export Graphs to JSON" button to perform the export.

Best Practices
--------------

Adhering to best practices ensures the integrity, maintainability, and scalability of your graph data within `s3Dgraphy`. Below are key recommendations for optimal usage.

### Data Consistency

- **Unique Identifiers:**  
  Ensure that every node and edge has a unique identifier (`node_id` and `edge_id`, respectively) to prevent conflicts and maintain clear relationships.

- **Uniform Data Structures:**  
  Maintain consistent data structures across nodes and edges to facilitate predictable behavior during import/export and integration processes.

### Managing Unique IDs

- **Automated ID Generation:**  
  Utilize automated mechanisms to generate unique IDs, especially when adding nodes programmatically, to avoid manual errors.

- **ID Naming Conventions:**  
  Adopt clear and descriptive naming conventions for IDs to enhance readability and traceability.

  *Example:*  
  - `geo_AcropoliGraph` for a GeoPositionNode related to the Acropoli graph.
  - `USD10.start::n0` for a PropertyNode representing the start time of node `USD10`.

### Validation

- **Graph Integrity:**  
  Implement validation checks to ensure that all nodes and edges conform to expected types and relationships, preventing malformed graphs.

- **JSON Schema Validation:**  
  Before utilizing the exported JSON, validate it against a predefined JSON schema to ensure structural correctness.

### Logging and Debugging

- **Implement Logging:**  
  Utilize Python's `logging` module instead of `print` statements for more flexible and configurable logging.

  *Example:*

  .. code-block:: python

      import logging

      # Configure the logger
      logging.basicConfig(level=logging.INFO)
      logger = logging.getLogger(__name__)

      class Graph:
          def add_node(self, node: Node, overwrite=False) -> Node:
              logger.info(f"Attempting to add node: {node.node_id}, overwrite: {overwrite}")
              # Rest of the method...

- **Debugging Tools:**  
  Incorporate debugging tools and practices, such as unit tests and exception handling, to streamline the development and maintenance process.

### Documentation

- **Comprehensive Documentation:**  
  Maintain up-to-date and thorough documentation for all classes, methods, and modules to facilitate ease of use and onboarding for new developers or stakeholders.

- **Inline Comments:**  
  Use inline comments judiciously to explain complex logic or decisions within the codebase.

### Extensibility

- **Modular Design:**  
  Structure the library in a modular fashion, allowing for easy extension and integration of new node types, edge types, or functionalities.

- **Custom Relationships:**  
  When mapping to external ontologies like CIDOC CRM, allow for the creation of custom relationships or extensions to accommodate specialized requirements.

Future Enhancements
-------------------

The `s3Dgraphy` library is designed with scalability and flexibility in mind. Future enhancements may include:

1. **Enhanced CIDOC CRM Mapping:**  
   - Develop more comprehensive mappings between `s3Dgraphy` nodes/edges and CIDOC CRM classes/relationships.
   - Implement automated mapping tools or converters.

2. **Advanced Export Options:**  
   - Support additional export formats (e.g., RDF, XML).
   - Implement selective export capabilities based on user-defined criteria.

3. **Interactive Visualization:**  
   - Integrate visualization tools to render graphs within Blender or other platforms.
   - Provide real-time graph editing and visualization features.

4. **API Integration:**  
   - Develop APIs to allow external applications to interact with `s3Dgraphy`'s graph data programmatically.

5. **User Interface Enhancements:**  
   - Enhance Blender's interface with more intuitive controls for graph management and export operations.

6. **Data Validation and Cleaning:**  
   - Incorporate advanced data validation and cleaning tools to maintain high data quality.

Conclusion
----------

The `s3Dgraphy` library offers a robust framework for managing complex graph data structures, particularly suited for integration with Blender for 3D modeling and visualization. By adhering to best practices and leveraging standardized models like CIDOC CRM, `s3Dgraphy` ensures interoperability, scalability, and semantic richness of your data.

**Key Takeaways:**
- **Structured Data Management:**  
  Efficiently manage nodes and edges with clear types and relationships.

- **Standardized Interoperability:**  
  Align graph data with CIDOC CRM for enhanced interoperability with cultural heritage information systems.

- **Flexible Export Mechanism:**  
  Export graph data into a well-defined JSON structure, facilitating integration with various tools and platforms.

- **Extensibility and Scalability:**  
  Design the library to accommodate future enhancements and evolving project requirements.

By following the guidelines and leveraging the functionalities provided by `s3Dgraphy`, stakeholders can ensure the effective management and utilization of graph-based data within their projects.

Appendix
--------

### A. JSON Schema Example

To ensure that exported JSON files adhere to the expected structure, consider defining a JSON schema. This schema can be used for validation purposes.

.. code-block:: json

    {
        "$schema": "http://json-schema.org/draft-07/schema#",
        "title": "s3Dgraphy Export Schema",
        "type": "object",
        "properties": {
            "context": {
                "type": "object"
            },
            "multigraph": {
                "type": "object",
                "additionalProperties": {
                    "type": "object",
                    "properties": {
                        "name@it": { "type": "string" },
                        "description@it": { "type": "string" },
                        "audio@it": {
                            "type": "array",
                            "items": { "type": "string" }
                        },
                        "video@it": {
                            "type": "array",
                            "items": { "type": "string" }
                        },
                        "image@it": {
                            "type": "array",
                            "items": { "type": "string" }
                        },
                        "data": {
                            "type": "object",
                            "properties": {
                                "geo_position": {
                                    "type": "object",
                                    "properties": {
                                        "epsg": { "type": "integer" },
                                        "shift_x": { "type": "number" },
                                        "shift_y": { "type": "number" },
                                        "shift_z": { "type": "number" }
                                    },
                                    "required": ["epsg", "shift_x", "shift_y", "shift_z"]
                                },
                                "epochs": {
                                    "type": "object",
                                    "additionalProperties": {
                                        "type": "object",
                                        "properties": {
                                            "min": { "type": "number" },
                                            "max": { "type": "number" },
                                            "start": { "type": "number" },
                                            "end": { "type": "number" },
                                            "color": { "type": "string" }
                                        },
                                        "required": ["min", "max", "start", "end", "color"]
                                    }
                                }
                            },
                            "required": ["geo_position", "epochs"]
                        },
                        "nodes": {
                            "type": "object",
                            "additionalProperties": {
                                "type": "object",
                                "properties": {
                                    "type": { "type": "string" },
                                    "name": { "type": "string" },
                                    "data": { "type": "object" }
                                },
                                "required": ["type", "name", "data"]
                            }
                        },
                        "edges": {
                            "type": "object",
                            "properties": {
                                "line": {
                                    "type": "array",
                                    "items": {
                                        "type": "object",
                                        "properties": {
                                            "from": { "type": "string" },
                                            "to": { "type": "string" }
                                        },
                                        "required": ["from", "to"]
                                    }
                                },
                                "dashed": {
                                    "type": "array",
                                    "items": {
                                        "type": "object",
                                        "properties": {
                                            "from": { "type": "string" },
                                            "to": { "type": "string" }
                                        },
                                        "required": ["from", "to"]
                                    }
                                },
                                "dotted": {
                                    "type": "array",
                                    "items": {
                                        "type": "object",
                                        "properties": {
                                            "from": { "type": "string" },
                                            "to": { "type": "string" }
                                        },
                                        "required": ["from", "to"]
                                    }
                                },
                                "double_line": {
                                    "type": "array",
                                    "items": {
                                        "type": "object",
                                        "properties": {
                                            "from": { "type": "string" },
                                            "to": { "type": "string" }
                                        },
                                        "required": ["from", "to"]
                                    }
                                },
                                "dashed_dotted": {
                                    "type": "array",
                                    "items": {
                                        "type": "object",
                                        "properties": {
                                            "from": { "type": "string" },
                                            "to": { "type": "string" }
                                        },
                                        "required": ["from", "to"]
                                    }
                                },
                                "TBD": {
                                    "type": "array",
                                    "items": {
                                        "type": "object",
                                        "properties": {
                                            "from": { "type": "string" },
                                            "to": { "type": "string" }
                                        },
                                        "required": ["from", "to"]
                                    }
                                }
                            },
                            "required": ["line", "dashed", "dotted", "double_line", "dashed_dotted", "TBD"]
                        }
                    },
                    "required": ["name@it", "description@it", "audio@it", "video@it", "image@it", "data", "nodes", "edges"]
                }
            }
        },
        "required": ["context", "multigraph"]
    }

Usage Workflow Example
----------------------

To illustrate how `s3Dgraphy` functions within a typical workflow, consider the following sequence:

1. **Graph Creation:**
   - Define nodes (e.g., GeoPositionNode, StratigraphicNode, PropertyNode).
   - Define edges to establish relationships between nodes.

2. **Graph Import:**
   - Use `GraphMLImporter` to parse a GraphML file and populate the graph data structure within `s3Dgraphy`.

3. **Automated Property Addition:**
   - `GraphMLImporter` ensures that all stratigraphic nodes have associated `start_time` and `end_time` properties, either through existing PropertyNodes or by creating them based on connected EpochNodes.

4. **JSON Export:**
   - Utilize `export_json.py` to export the graph data into a structured JSON file.
   - Integrate the export process within Blender using the provided Blender operator for seamless data handling.

5. **Data Utilization:**
   - Use the exported JSON for further processing, visualization, or integration with other systems adhering to CIDOC CRM.

.. mermaid::

    graph LR
        A[GraphML File] -->|Import| B[GraphMLImporter]
        B --> C[Graph Data Structure]
        C --> D[Ensure Properties]
        D --> E[Export to JSON]
        E --> F[Save JSON File]
        F --> G[Blender Integration]

Troubleshooting Tips
--------------------

1. **Duplicate Nodes or Edges:**
   - **Issue:** Nodes or edges are duplicated during import.
   - **Solution:**  
     - Ensure unique `node_id` and `edge_id` values.
     - Utilize the `overwrite` parameter when adding nodes to handle existing entries.

2. **Missing GeoPositionNode:**
   - **Issue:** The exported JSON lacks the `geo_position` node.
   - **Solution:**  
     - Verify that the graph contains a `GeoPositionNode`.
     - Ensure that the import process correctly adds the `geo_position` node if absent.

3. **Incorrect Epoch Data:**
   - **Issue:** Epochs have incorrect or missing data fields.
   - **Solution:**  
     - Validate the GraphML file to ensure all necessary data fields are present.
     - Check the `EpochNode` instantiation for correct data assignment.

4. **Export Failure:**
   - **Issue:** The JSON export process fails or generates invalid JSON.
   - **Solution:**  
     - Inspect the console for error messages.
     - Validate the graph data structure before export.
     - Use the provided JSON schema to identify structural issues.

5. **CIDOC CRM Mapping Errors:**
   - **Issue:** Graph elements do not correctly map to CIDOC CRM concepts.
   - **Solution:**  
     - Review the mapping table to ensure accurate correspondence.
     - Extend or customize CIDOC CRM relationships as needed for complex relationships.

Extending the Library
---------------------

To accommodate additional requirements or data types, `s3Dgraphy` can be extended by introducing new node or edge types. Follow these steps to add new components:

1. **Define the New Node or Edge Class:**
   - Create a new Python class inheriting from the base `Node` or `Edge` class.
   - Define necessary attributes and methods.

2. **Update the Graph Class:**
   - Implement methods to handle the addition and management of the new node or edge types.
   - Ensure integration with existing functionalities like importers and exporters.

3. **Update Export Logic:**
   - Modify `export_json.py` to include the new node or edge types in the export process.
   - Ensure the JSON structure accommodates the new components.

4. **Document the New Components:**
   - Update the documentation to include descriptions and usage examples of the new node or edge types.

**Example: Adding a New Node Type `AnalysisNode`:**

.. code-block:: python

    # s3Dgraphy/analysis_node.py

    from .node import Node

    class AnalysisNode(Node):
        """
        Represents an analysis conducted on a stratigraphic unit.
        """
        def __init__(self, node_id, analysis_type, results, author, date):
            super().__init__(node_id=node_id, name=analysis_type, node_type="analysis")
            self.data = {
                "analysis_type": analysis_type,
                "results": results,
                "author": author,
                "date": date
            }

        def to_dict(self):
            return {
                "type": self.node_type,
                "name": self.name,
                "data": self.data
            }

Glossary
--------

- **Node:** An entity or object within a graph.
- **Edge:** A connection or relationship between two nodes in a graph.
- **CIDOC CRM:** A standard ontology for cultural heritage information.
- **GeoPositionNode:** A node representing geographical position.
- **EpochNode:** A node representing a geological epoch.
- **PropertyNode:** A node representing a property or attribute.
- **ShiftNode:** A node representing spatial transformations.
- **CombinerNode:** A node representing logical combinations or inferences.
- **ExtractorNode:** A node representing data extraction processes.
- **DocumentNode:** A node representing documents or references.
- **StratigraphicNode (US):** A node representing stratigraphic units or layers.
- **Export Operator:** A Blender operator that facilitates exporting graph data to JSON.

Frequently Asked Questions (FAQ)
-------------------------------

**Q1: Can `s3Dgraphy` handle multiple graphs simultaneously?**  
**A1:** Yes, `s3Dgraphy` supports managing multiple graphs through the `MultiGraphManager`. Each graph is identified by a unique `graph_id` and can be individually manipulated and exported.

**Q2: How do I add a new node type to the graph?**  
**A2:** Define a new node class inheriting from the base `Node` class, implement necessary attributes and methods, update the `Graph` class to manage the new node type, and ensure that the export logic accommodates the new nodes.

**Q3: What happens if the JSON export fails?**  
**A3:** Check Blender's console for error messages, ensure that all nodes and edges have unique identifiers, and validate the graph's integrity before attempting the export again.

**Q4: How does `s3Dgraphy` ensure data consistency during import and export?**  
**A4:** The library implements validation checks, manages unique identifiers, and maintains consistent data structures across nodes and edges to ensure data integrity.

**Q5: Is it possible to customize the JSON export structure?**  
**A5:** Yes, the `export_json.py` module can be modified to adjust the JSON structure as needed. Ensure that any changes align with the intended use cases and maintain compatibility with downstream processes.

References
----------

1. **CIDOC CRM Official Documentation:**  
   `CIDOC CRM <http://www.cidoc-crm.org/cidoc-crm/>`_

2. **JSON Schema Documentation:**  
   `JSON Schema <https://json-schema.org/>`_

3. **Blender Python API:**  
   `Blender API Documentation <https://docs.blender.org/api/current/>`_

4. **Python Logging Module:**  
   `Python Logging <https://docs.python.org/3/library/logging.html>`_

Contact and Support
-------------------

For further assistance, feature requests, or to report bugs, please contact the development team at `emanuel.demetrescu@cnr.it <mailto:emanuel.demetrescu@cnr.it> `_.

License
-------

This project is licensed under the MIT License - see the `LICENSE <LICENSE>`_ file for details.

Contributing
------------

Contributions are welcome! Please fork the repository and submit a pull request with your enhancements.

Changelog
---------

**v1.0.0**
- Initial release.
- Support for GeoPositionNode, EpochNode, PropertyNode, ShiftNode.
- JSON export functionality.
- Blender integration for exporting JSON.

**v1.1.0**
- Added CombinerNode and ExtractorNode.
- Enhanced CIDOC CRM mapping.
- Improved documentation and examples.

**v1.2.0**
- Implemented automated property addition for stratigraphic nodes.
- Added ShiftNode functionality.
- Enhanced error handling and logging.

**v1.3.0**
- Extended support for additional node types.
- Improved JSON export structure.
- Added unit tests for export functionality.

End of Documentation
