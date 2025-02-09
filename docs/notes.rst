Version 1.1.0
=============

**Agent 1.1.0 | Client 1.1.1**

This version include the following changes in both Agent and Client:

* Agent 1.1.0:
    * Add
        * Message fragmentation.
        * P2P communication.
        * API.
        * Time synchronization.
        * Windows discovery support.
        * New unitary tests.
        * API documentation.
        * Logger.
        * Command Line Interface.
        * Centralized middleware.
        * Remove Asio dependency.
    * Change
        * CMake approach.
        * Server's thread pattern.
    * Fix
        * Serial transport.

* Client 1.1.1:
    * Add
        * Message fragmentation.
        * Time synchronization.
        * Windows discovery support.
        * New unitary tests.
        * API documentation.
        * Raspberry Pi support.
    * Change
        * Memory usage improvement.
        * CMake approach.
        * Discovery API.
        * Examples usage.
    * Fix
        * Acknack reading.
        * User data bad aligment.

Previous Versions
-----------------

Version 1.0.3
=============

**Agent 1.0.3 | Client 1.0.2**

This version includes the following changes in both Agent and Client:

* Agent 1.0.3:
    * Fast RTPS version upgraded to 1.7.2.
    * Baud rate support improvements.
    * Bugfixes.

* Client 1.0.2:
    * Uses new Fast RTPS 1.7.2 XML format.
    * Add Raspberry Pi toolchain.
    * Fix bugs.

Version 1.0.2
=============

**Agent 1.0.2 | Client 1.0.1**

This version includes the following changes in the Agent:

* Agent 1.0.2:
    * Fast RTPS version upgraded to 1.7.0.
    * Added dockerfile.
    * Documentation fixes.

Version 1.0.1
=============

**Agent 1.0.1 | Client 1.0.1**

This release includes the following changes in both Agent and Client:

* Agent 1.0.1:
    * Fixed Windows installation.
    * Fast CDR version upgraded.
    * Simplified CMake code.
    * Bug fixes.

* Client 1.0.1:
    * Fixed Windows configuration.
    * MicroCDR version upgraded.
    * Cleaned unused code.
    * Fixed documentation.
    * Bug fixes.

Version 1.0.0
=============

This release includes the following features:

* Extended C topic code generation tool (strings, sequences and n-dimensional arrays).
* Discovery profile.
* Native write access profile (without using *eProsima Micro XRCE-DDS Gen*)
* Creation and configuration by XML.
* Creation by reference.
* Added `REUSE` flag at create entities.
* Added prefix to functions.
* Transport stack modification.
* More tests.
* Reorganized project.
* Bug fixes.
* API changes.

Version 1.0.0Beta2
==================

This release includes the following features:

* Reliability.
* Stream concept (best-effort, reliable).
* Multiples streams of same type.
* Configurable data delivery control.
* C Topic example code generation tool.
