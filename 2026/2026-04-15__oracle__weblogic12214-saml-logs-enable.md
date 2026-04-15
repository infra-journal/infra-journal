# Enabling or Disabling SAML detailed logs on Oracle Weblogic

## Applies To
- **Product:** Oracle weblogic 12.2.1.4
- **Platform:** All platforms (Windows/Linux)

## Step-by-Step Instructions

### PART 1
#### Step 1 - Login into WebLogic Server Administration Console.

#### Step 2 - Acquire lock, clicking on "Lock & Edit" button on change center, located at the upper left part of the screen.

#### Step 3 - Navigate to Environment > Servers

#### Step 4 - Select the server where you will activate the debugs

#### Step 5 - Navigate to Debug Tab and Expand weblogic tab

#### Step 6 - Locate the following debugs and click the check box on the left side.

> securtity -> Atn  
security -> Saml2  
servlet -> internal

#### Click Enable on the upper part of the screen
#### Activate changes
#### Expand webLogic -> servlet and enable DebugHttp and DebugURLResolution from here so that debugs are set on them. Save and Activate Changes

### PART 2 (Change the debug level of log file)
#### Step 1 - Navigate to: Environment > Servers

#### Step 2 - Select the server where you will activate the debugs

#### Step 3 - Navigate to Logging > General

#### Step 4 - Scroll down and hit Advanced

#### Step 5 - Locate: Log file > Severity Level and make sure that the dropdown beside of it is set to "Debug"

#### Step 6 - Locate: Minimum severity to log, and change it to "Debug"

#### Step 7 - Locate: Standard out  file > Severity Level and make sure that the dropdown beside of it is set to "Debug"

#### Step 8 - Locate: Minimum severity to log, and change it to "Debug"

#### Activate changes

| Log Type | Default Value |
| --- | --- |
| Minimum severity to log | Info |
| Log file severity level | Trace |
| Standard Out: <br> Severity Level: | Notice |
| Domain log broadcaster <br> Severity Level: | Notice | 

> Reboot Managed Server after making above changes. 


