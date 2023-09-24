# Running a protocol
To launch a protocol using the Bpod console, click 'Play' for launch manager.

### `RunProtocol()`
**Description**

Runs, or stops a Bpod behavior protocol.

Before using RunProtocol(), a protocol folder must exist in /Bpod Local/Protocols/.

Also, a test subject must have been added for the protocol:

- In the launch manager, use the '+' button, OR 
- Create a folder: /Bpod Local/Data/MyTestSubjectName/MyProtocolName/

You must run Bpod; before using RunProtocol().

**Syntax**

To start a protocol:
```matlab
RunProtocol('Start', ProtocolName, SubjectName, [SettingsName])
```

To stop a running protocol:
```matlab
RunProtocol('Stop')
```

**Parameters**

- ProtocolName: A string specifying the name of the protocol, as it would appear in the launch manager.
    - Do not include a path or file extension; for instance, to run the Operant protocol use 'Operant'.
    - New protocols can be created from the launch manager.
- SubjectName: A string specifying the test subject name, as it would appear in the launch manager
    - Do not include a path or file extension; for instance, to run Rat232, use 'Rat232'.
    - New subjects can be created from the launch manager.
- SettingsName: An optional string argument to specify a settings file.
    - If omitted, the protocol's default settings file is used (by default, this is an empty struct). 
    - Do not include a path or file extension; for instance, to load Rat232's 'Easy.mat' settings file for Operant, use 'Easy'.
    - New settings files can be created from the launch manager.

**Returns**

- None

**Examples**

This code starts a new session using the 'OdorTest' protocol, for test subject 'SniffMaster' using the protocol's default settings file.  

```matlab
RunProtocol('Start', 'OdorTest', 'SniffMaster');
```


This code starts a new session using the 'OdorTest' protocol, for test subject 'SniffMaster' using settings file /Bpod Local/Data/SniffMaster/OdorTest/ProtocolSettings/'BrutallyDifficult.mat'.  
```matlab
RunProtocol('Start', 'OdorTest', 'SniffMaster', 'BrutallyDifficult');
```