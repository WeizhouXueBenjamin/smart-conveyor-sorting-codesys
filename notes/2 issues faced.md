# Two issues face in the project setup, along with my thinkings

## Project overview

I used CODESYS Control Win V3 as a SoftPLC and connected it to a Factory I/O conveyor-sorting scene through OPC UA.

This was a local simulation project. It did not involve a physical PLC or a real production system.

During the setup, I ran into two main problems:

- Factory I/O could not connect to the CODESYS OPC UA server.
- After the connection worked, Factory I/O showed an `Object reference not set to an instance of an object` error.

I fixed the connection settings and rebuilt the OPC UA tag mappings. After that, Factory I/O could connect and exchange all 18 signals with CODESYS.

## Problem 1: Factory I/O could not connect

The PLC program was running in CODESYS, but Factory I/O could not establish an OPC UA connection.

At first, I checked several possible causes:

- Whether CODESYS Control Win was running
- Whether the endpoint was correct: `opc.tcp://localhost:4840`
- Whether port 4840 was available
- Whether the PLC variables were included in Symbol Configuration
- Whether the problem was related to certificates or user permissions

I initially looked for the permission controls under Communication Settings, but later found that the relevant controls were under Access Rights.

The main cause was that my current CODESYS runtime required device user management, while the Factory I/O sample was trying to connect anonymously. The sample appeared to have been made for an older CODESYS setup with different default security settings.

## How I fixed it

For this local simulation, I made the following changes in CODESYS:

- Opened `Device → Change Runtime Security Policy`
- Changed Device User Management to optional
- Enabled anonymous login
- Opened Access Rights
- Gave `Anonymous_OPCUAServer` the required permissions under `RemoteConnections → OPCUAServer`
- Rebuilt and downloaded the PLC application
- Confirmed that the SoftPLC was in `RUN`

I kept the administrator account and did not place any passwords in the project or screenshots.

After these changes, Factory I/O connected to the OPC UA server and displayed all 18 `FIO` variables. This showed that the endpoint, permissions and Symbol Configuration were working.

Anonymous access was only used because this was a simulation running on one computer. For a real industrial system, I would use authenticated access and only give each user the permissions they need.

## Problem 2: Factory I/O showed a null-reference error

Once the connection problem was fixed, Factory I/O showed this error:

`Object reference not set to an instance of an object`

The message did not clearly explain what was wrong. However, Factory I/O could already browse all 18 CODESYS variables. This made me think the OPC UA connection itself was working and the problem was more likely related to the saved mappings in the scene.

The official Factory I/O scene already contained mappings from the environment in which the sample was created. Those saved mappings did not work correctly with the variables being provided by my current CODESYS runtime.

## How I fixed it

- Saved a copy of the original Factory I/O scene
- Disconnected the OPC UA driver
- Used `CLEAR` to remove the old mappings
- Browsed `opc.tcp://localhost:4840` again
- Filtered for `FIO` and confirmed that all 18 variables were available
- Recreated the input and output mappings manually
- Reconnected the driver and returned Factory I/O to `RUN`

After rebuilding the mappings, the null-reference error disappeared and the 18 signals could be exchanged between CODESYS and Factory I/O. No PLC code changes were needed to fix these two setup problems.

## What I learned

- A PLC being in `RUN` does not mean the whole connection is working.
- It is useful to check the setup in a simple order: runtime, endpoint, permissions, symbols, mappings and then machine movement.
- If Factory I/O can browse the PLC variables, the basic OPC UA connection is probably working.
- Older sample projects may not match the security settings or saved mappings used by a newer runtime.
- Saving a copy before clearing mappings makes it easier to undo changes.
- It is better to change one setting at a time and check what changed.
- A communication fix does not automatically mean the control sequence is correct. The conveyor and turntable logic still need to be tested separately.

## What I would do next time

- Check the runtime version and the sample instructions before starting.
- Confirm that the SoftPLC is running and that the OPC UA endpoint is reachable.
- Check whether Factory I/O can browse the expected 18 variables before changing the PLC code.
- Rebuild old mappings if the sample was created with a different version.
- Test the scene from a clean restart and watch one complete sorting cycle.
