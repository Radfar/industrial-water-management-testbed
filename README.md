# Industrial Water Management & Automation Testbed

**Multi-vendor industrial automation, SCADA, OT/IT integration, and Building Automation interoperability testbed**

> A practical engineering testbed for developing and validating industrial automation architectures across PLCs, SCADA, industrial communication protocols, IIoT technologies, databases, web applications, and Building Automation Systems.

---

## 1. Project Overview

This project is a long-term **Industrial Water Management & Automation Testbed** designed to reproduce a realistic industrial water-management process while allowing different automation technologies to be developed, integrated, and evaluated independently.

The same underlying process model is implemented across multiple control platforms and connected through open communication technologies.

The project currently combines:

* PLC and SoftPLC control
* VFD-driven pumping
* Valve and process control
* Flow, level, and moisture feedback
* Multi-vendor automation platforms
* SCADA/HMI systems
* OPC UA
* BACnet/IP
* Web-based SCADA
* Node.js integration services
* PostgreSQL
* Industrial networking
* IoT hardware
* Arduino / ESP8266
* Future MQTT / Node-RED / IIoT integration
* Future cloud and data analytics capabilities

The objective is not to reproduce one vendor's automation solution, but to investigate how heterogeneous **OT systems can participate in a common process and information architecture**.

---

# 2. System Architecture

The testbed is being developed as a layered architecture:

```text
                         INDUSTRIAL WATER PROCESS
                                  │
                 ┌────────────────┼────────────────┐
                 │                │                │
                 ▼                ▼                ▼
          Siemens S7-1500   Rockwell Micro850   CODESYS
             Zone 01           Zone 02          Zone 03
                 │                │                │
                 └────────────────┼────────────────┘
                                  │
                              OPC UA
                                  │
                                  ▼
                         ┌─────────────────┐
                         │   SCADA LAYER   │
                         │                 │
                         │ Ignition        │
                         │ Web SCADA       │
                         └────────┬────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
             Industrial IT                BAS Integration
                    │                           │
                    ▼                           ▼
             Node.js / APIs              BACnet/IP
             PostgreSQL                       │
                    │                         ▼
                    │                    BACnet Clients
                    │                    / BAS Platforms
                    │
                    ▼
             Future IIoT Layer
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       MQTT     Node-RED   VFD / Edge
          │
          ▼
       Cloud / Data
       Analytics
```

The architecture is intentionally modular. Individual technologies can be replaced or extended without changing the underlying water-management process model.

---

# 3. Multi-Vendor Control Platforms

The same water-management concept is being implemented across different control environments.

### Zone 01 — Siemens

**Technology**

* Siemens S7-1500
* TIA Portal
* PLCSIM Advanced
* Siemens G120X VFD
* OPC UA

Purpose:

> Demonstrate a conventional industrial PLC/VFD automation architecture using the Siemens ecosystem.

---

### Zone 02 — Rockwell Automation

**Technology**

* Allen-Bradley Micro850
* Connected Components Workbench (CCW)
* Catalog: `2080-LC50-48QWB`

Purpose:

> Demonstrate the same process model using a Rockwell Micro800/Micro850 control platform.

---

### Zone 03 — CODESYS

**Technology**

* CODESYS Control Win
* IEC 61131-3 Structured Text
* OPC UA

Zone 03 currently serves as an important integration environment because its process data is connected simultaneously to:

* Ignition
* Web SCADA
* OPC UA clients
* OPC UA–BACnet/IP gateway

---

# 4. SCADA & HMI Systems

The testbed intentionally uses more than one supervisory technology.

## Ignition

Ignition provides the industrial SCADA environment for the testbed.

It is used to investigate:

* SCADA architecture
* OPC UA connectivity
* Tag structures
* UDT concepts
* HMI/faceplate concepts
* Multi-vendor supervisory integration

---

## Web SCADA

A separate web-based SCADA platform has been developed using:

* Node.js
* Express
* HTML
* CSS
* JavaScript
* OPC UA
* PostgreSQL
* REST APIs

The Web SCADA project is maintained separately:

**[Web SCADA Repository](https://github.com/Radfar/building-irrigation-scada)**

The master repository documents its role in the overall architecture rather than duplicating its source code.

---

# 5. OPC UA Integration

OPC UA is used as an important interoperability layer between the control systems and higher-level applications.

Typical architecture:

```text
PLC / SoftPLC
      │
      ▼
   OPC UA
      │
      ▼
SCADA / Integration Layer
      │
      ├── Ignition
      ├── Web SCADA
      └── Node.js Gateway
```

The project investigates the use of OPC UA for:

* Real-time process data
* Control commands
* Cross-vendor integration
* SCADA connectivity
* Application-level integration

---

# 6. OPC UA → BACnet/IP Gateway

One of the major milestones of the project is the development of a software-based **OPC UA–BACnet/IP gateway**.

The gateway connects industrial process data to the Building Automation ecosystem without requiring a dedicated hardware protocol converter.

```text
CODESYS
   │
   │ OPC UA
   ▼
Node.js Web SCADA / Gateway
   │
   │ BACnet/IP
   ▼
BACnet Client / BAS
```

The dedicated implementation is maintained here:

**[OPC UA–BACnet/IP Gateway Repository](https://github.com/Radfar/opcua-bacnet-gateway)**

### Validated capabilities

The current prototype has demonstrated:

* BACnet/IP device startup
* BACnet device discovery
* Who-Is / I-Am
* BACnet object communication
* ReadProperty
* WriteProperty
* Live OPC UA process data acquisition
* BACnet exposure of process data
* BACnet command transfer
* BACnet → OPC UA write-back
* Independent BACnet client validation
* Real-time process response

The current validation architecture is:

```text
CODESYS Water Process
        │
        │ OPC UA
        ▼
Node.js Gateway
        │
        │ BACnet/IP
        ▼
Independent BACnet Test Environment
        │
        ▼
Read / Write Validation
```

### Validation Video

The recorded demonstration will be maintained in:

`media/videos/opcua-bacnet-gateway-validation.mp4`

The demonstration shows the complete path from a BACnet command through the gateway to the CODESYS process and back to live BACnet-readable process data.

---

# 7. IoT & Embedded Water-Control Hardware

The physical/embedded side of the project is being developed separately from the industrial PLC testbed.

The IoT hardware project uses:

* Arduino
* ESP8266
* Wireless valve nodes
* Main controller
* Flow measurement
* Tank-level monitoring
* Wireless communication
* Remote control concepts

Dedicated repository:

**[Wireless Watering System / IoT Hardware Repository](https://github.com/Radfar/Wireless-Watering-System)**

This branch of the project investigates how distributed embedded devices can participate in the larger water-management architecture.

The long-term objective is to connect the embedded/edge layer with the industrial SCADA and IIoT architecture.

---

# 8. Canonical Water Process Model

Although different technologies are used, the project attempts to maintain a common logical representation of the water-management process.

Typical process signals include:

### Commands

* AUTO
* START
* STOP
* FAULT_RESET

### Process Feedback

* RUNNING
* VALVE
* FLOW
* MOISTURE
* FAULT
* LOW_FLOW

### Status

* PERMISSIVE
* AUTO_ACTIVE
* FLOW_OK
* PROCESS_OK
* COMM_OK

### Setpoints

* MOISTURE_LOW_SP
* MOISTURE_HIGH_SP

This common process model allows different PLCs, SCADA platforms, gateways, and software applications to participate in the same conceptual system.

---

# 9. Repository Architecture

This repository is the **master documentation and architecture repository**.

Implementation projects remain separated into their own repositories.

```text
industrial-water-management-testbed
│
├── Documentation
├── Architecture
├── Process Model
├── Engineering Decisions
├── Research
├── Validation Results
├── Media
└── Links to Implementation Repositories
```

Related implementation repositories:

| Component                        | Repository                                                                                           |
| -------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Master Testbed / Documentation   | [industrial-water-management-testbed](https://github.com/Radfar/industrial-water-management-testbed) |
| Web SCADA                        | [building-irrigation-scada](https://github.com/Radfar/building-irrigation-scada)                     |
| OPC UA–BACnet/IP Gateway         | [opcua-bacnet-gateway](https://github.com/Radfar/opcua-bacnet-gateway)                               |
| IoT / Wireless Watering Hardware | [Wireless-Watering-System](https://github.com/Radfar/Wireless-Watering-System)                       |

This separation keeps implementation repositories focused while allowing this repository to document the **complete engineering system**.

---

# 10. IIoT & Edge Integration — Next Development Stage

The next major stage of the project will extend the architecture from industrial automation and protocol integration toward **Industrial IoT (IIoT)**.

Planned technologies include:

* Node-RED
* MQTT
* Edge data processing
* VFD data integration
* Industrial gateways
* Time-series / operational data
* Cloud connectivity
* Remote monitoring
* Data analytics

The planned architecture is:

```text
PLC / VFD / Sensors
        │
        ▼
   Industrial OT
        │
        ▼
    OPC UA / Other
        │
        ▼
   Node.js / Edge
        │
        ├───────────────┐
        ▼               ▼
     MQTT           Node-RED
        │               │
        └───────┬───────┘
                ▼
        Edge Data Layer
                │
                ▼
             Cloud
                │
        ┌───────┴────────┐
        ▼                ▼
   Dashboards       Analytics / AI
```

### Planned investigations

1. Connect VFD operating data to the IIoT layer.
2. Collect industrial process data through Node.js.
3. Publish selected data through MQTT.
4. Use Node-RED for edge-level data processing and routing.
5. Investigate cloud connectivity.
6. Develop historical data and monitoring capabilities.
7. Investigate analytics and AI applications for water-management processes.

This stage will extend the project from **OT interoperability** toward a broader **OT/IT/IIoT architecture**.

---

# 11. Building Automation Integration Roadmap

The BACnet/IP gateway establishes the first BAS-facing integration layer.

Future investigation may include:

```text
Industrial Automation
        │
        ▼
      OPC UA
        │
        ▼
     Node.js
        │
        ▼
    BACnet/IP
        │
        ▼
   Building Automation
        │
        ▼
     Niagara N4
```

A future validation stage may investigate interoperability with Niagara 4 or another BAS platform.

This is intentionally treated as a future validation stage rather than a prerequisite for the current BACnet/IP milestone.

---

# 12. Research & Engineering Objectives

The project is being used to investigate practical questions around:

* Multi-vendor automation
* Industrial interoperability
* OT/IT integration
* SCADA architecture
* Open industrial protocols
* OPC UA
* BACnet/IP
* BAS integration
* Web-based industrial applications
* IIoT architectures
* Edge computing
* MQTT
* Industrial data integration
* Cloud-connected automation
* Future AI-assisted industrial analytics

The project is therefore both a technical development environment and a long-term engineering research portfolio.

---

# 13. Documentation

Detailed engineering documentation is maintained under:

```text
docs/
```

Current documentation includes:

* System architecture
* Canonical irrigation/water-zone logic
* Engineering decisions
* PLC implementations
* SCADA integration
* OPC UA integration
* BACnet/IP integration
* Research and publication material

Media and demonstrations are maintained under:

```text
media/
```

---

# 14. Project Status

### Completed / Validated

* [x] Multi-vendor water-management process concept
* [x] Siemens control environment
* [x] Rockwell Micro850 environment
* [x] CODESYS SoftPLC environment
* [x] OPC UA integration
* [x] Ignition SCADA integration
* [x] Web SCADA platform
* [x] Node.js integration layer
* [x] PostgreSQL data layer
* [x] BACnet/IP gateway prototype
* [x] BACnet device discovery
* [x] ReadProperty validation
* [x] WriteProperty validation
* [x] BACnet → OPC UA control
* [x] Independent BACnet test environment
* [x] Bidirectional OPC UA ↔ BACnet/IP demonstration

### In Progress / Planned

* [ ] BACnet object/data-model refinement
* [ ] Niagara 4 / BAS interoperability validation
* [ ] IIoT edge architecture
* [ ] Node-RED integration
* [ ] MQTT integration
* [ ] VFD data integration
* [ ] Cloud connectivity
* [ ] Historical/analytical data pipeline
* [ ] Advanced analytics / AI applications

---

# 15. Long-Term Vision

The long-term objective is to evolve the testbed into a compact but realistic demonstration of a modern industrial water-management architecture:

```text
┌──────────────────────────────────────────────────────────┐
│                  WATER MANAGEMENT PROCESS                │
└───────────────────────────┬──────────────────────────────┘
                            │
              PLC / VFD / Sensors / IoT
                            │
                            ▼
┌──────────────────────────────────────────────────────────┐
│                    INDUSTRIAL OT                          │
│ Siemens │ Rockwell │ CODESYS │ Embedded Controllers      │
└───────────────────────────┬──────────────────────────────┘
                            │
                       OPC UA / OT
                            │
                            ▼
┌──────────────────────────────────────────────────────────┐
│                 SCADA / INTEGRATION                      │
│ Ignition │ Web SCADA │ Node.js │ PostgreSQL              │
└───────────────┬───────────────────────┬──────────────────┘
                │                       │
             BACnet/IP                MQTT
                │                       │
                ▼                       ▼
       Building Automation          IIoT / Edge
                │                       │
                ▼                       ▼
            BAS / N4                  Cloud
                                        │
                                        ▼
                                Analytics / AI
```

The project is deliberately being developed incrementally, with each layer independently validated before extending the architecture.

---

## Author

**Alireza Radfar**

Industrial Automation & Electronics Engineer

Focus areas:

**PLC | SCADA/HMI | VFD | Industrial Networks | OPC UA | BACnet | OT/IT Integration | IIoT | Web SCADA | Building Automation**

---

## License

See [LICENSE](LICENSE).
