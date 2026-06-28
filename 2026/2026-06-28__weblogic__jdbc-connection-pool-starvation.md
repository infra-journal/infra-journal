# Incident Report: Dynamic JDBC Connection Pool Starvation under Peak Load

## 📋 Problem Description
During a period of high concurrent user traffic, an Oracle ADF application deployed on WebLogic Server experienced performance degradation. Multiple users encountered application freezes and page-load crashes. Intermittently under this heavy strain, the server failed to cleanly switch application context between rapid requests, causing a temporary state overlap where a user was briefly presented with data caching from a previous active session thread during page restoration.

### Extracted Application Log Signature:
```text
<Warning> <oracle.dfw.incident> <DFW-40125> <incident flood controlled with Problem Key "ADFC-00032 [ADFc]">
<Warning> <oracle.adfinternal.view.faces.context.RichExceptionHandler> <BEA-000000> <ADF_FACES-60098:Faces lifecycle receives unhandled exceptions in phase RESTORE_VIEW 
javax.el.ELException: oracle.jbo.DMLException: JBO-26061: Error while opening JDBC connection.
    at javax.el.BeanELResolver.getValue(BeanELResolver.java:304)
    at com.sun.faces.el.DemuxCompositeELResolver._getValue(DemuxCompositeELResolver.java:176)
```

---

## 🔍 Root Cause Analysis
Forensic analysis of the WebLogic Managed Server logs and real-time JDBC metrics isolated the issue completely to **infrastructure resource limits**. The application code itself performed correctly, but hit physical environment constraints.

### The Breakdown:
1. **Hard Connection Ceiling:** The JDBC Data Source (`TESTDS`) was configured with a hardcoded `Maximum Capacity` of **150 connections**.
2. **Resource Exhaustion:** Under peak traffic, concurrent application requests exceeded 150. WebLogic threw a `PoolLimitSQLException` as verified below:
   ```text
   Caused By: weblogic.jdbc.extensions.PoolLimitSQLException: weblogic.common.resourcepool.ResourceLimitException: No resources currently available in pool TESTDS to allocate to applications...
   ```
3. **Lifecycle Interruption:** Because available pool connections dropped to 0, the Oracle ADF framework encountered an unhandled exception mid-lifecycle during the `RESTORE_VIEW` phase. This abrupt termination prevented standard post-request thread cleanup routines from executing. 
4. **Context Overlap:** Due to default WebLogic pooling configurations (`Seconds to Trust an Idle Pool Connection = 10`), the server immediately recycled the affected thread pipeline to process the next incoming request. Because the previous lifecycle was cut short by the database dropout, un-cleared memory states momentarily overlapped into the next active session thread.

### Database Headroom Verification:
A configuration check on the backend Oracle Database confirmed that the database itself was never under stress and has massive infrastructure headroom to support higher thresholds:
* **`processes` Limit:** 1,200
* **`sessions` Limit:** 1,822

---

## 🛠️ Solution & Remediation Plan
The resolution requires zero application code alterations or patches. It is fully achieved via infrastructure scaling and enforcing strict connection-clearing isolation guards in the WebLogic Server Administration Console.

### Step 1: Scale Connection Pool Pipeline
1. Navigate to **Services** -> **Data Sources** -> **`TESTDS`** -> **Configuration** -> **Connection Pool**.
2. Click **Lock & Edit** in the Change Center.
3. Update the capacity configurations to absorb future traffic peaks safely utilizing the database's available headroom:
   * **Maximum Capacity:** Change from `150` to **`300`** (or `400` based on target load).
   * **Initial Capacity:** Leave at **`150`** (Prevents initialization spikes on the DB).
   * **Minimum Capacity:** Leave at **`150`**.

### Step 2: Enable Advanced Session Isolation Guards
Scroll down to the **Advanced** section of the Connection Pool settings and enforce strict real-time clearing parameters:

* **Test Connections On Reserve:** **`[X] Enabled (Checked)`**  
  * *Impact:* Forces WebLogic to thoroughly test and clear connection contexts *every single time* a user requests a page.
* **Test Table Name:** **`SQL SELECT 1 FROM DUAL`**  
  * *Impact:* Uses a native, optimized JDBC 4.0 hardware-level ping via Oracle’s dummy table. It introduces zero observable CPU latency.
* **Seconds to Trust an Idle Pool Connection:** Change from `10` to **`0`**  
  * *Impact:* Eliminates the 10-second reuse loophole, guaranteeing that no connection skips verification under rapid concurrent usage.
* **Connection Creation Retry Frequency:** Change from `0` to **`10`**  
  * *Impact:* Provides a fallback buffer for WebLogic to retry connection attempts rather than instantly crashing a user session if a minor queue forms.

### Step 3: Apply & Activate
1. Click **Save** at the bottom of the page.
2. Click **Activate Changes** in the Change Center.

> 💡 **Operational Note:** While capacity changes apply dynamically, testing structural adjustments might prompt a restart warning. To apply these configurations immediately on a production environment without system downtime, navigate to the data source **Control** tab, select the pool, execute a **Force Suspend**, and then click **Resume** to instantly reinitialize the connection pools with the new settings active.
