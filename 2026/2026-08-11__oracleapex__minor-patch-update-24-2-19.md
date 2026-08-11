# Oracle APEX 24.2 to 24.2.19 Patch Set Bundle Upgrade Guide

This guide outlines the end-to-end steps to safely apply the official Oracle Patch Set Bundle (Patch 37366599) to update your Oracle APEX environment to version 24.2.19 on an Oracle Linux VM running ORDS Standalone mode.

---

## Phase 1: Download the Patch
1. Log into your **My Oracle Support (MOS)** account.
2. Navigate to the **Patches & Updates** tab.
3. Search for **Patch Number 37366599** (the APEX 24.2 Bundle).
4. Download the compressed `p37366599_2420_Generic.zip` archive to your local machine.

---
## Phase 2: Stage and Extract the Files
1. Use an SFTP client or secure copy tool (`scp`) to move the `.zip` file onto your Oracle Linux VM under the `/tmp` directory.
2. Log into your VM terminal as the `oracle` user.
3. Move to the directory and unpack the archive:
   ```bash
   cd /tmp
   unzip p37366599_2420_Generic.zip
   ```
   *Note: This extraction creates a subfolder named `/tmp/patch` containing the update scripts and partial static images. ex: /tmp/37366599*
4. Since scp moved the file with default user (which is generally **opc** on OCI) so execute the below command to change ownership
    ```bash
     sudo chown oracle:oinstall p37366599_2420_Generic.zip
     ```


---
## Phase 3: Stop the Frontend Service
To prevent file locking during the image sync and to block active user sessions from corrupting mid-patch, stop the ORDS standalone service:
```bash
sudo systemctl stop ords
```

I use tmux to start/stop ORDS and it stays active in the background
```bash
tmux attach-session -t ords
exit
```

---

## Phase 4: Run the Database Patch Script
1. Navigate into the freshly extracted patch directory:
   ```bash
   cd /tmp/patch
   ```
   *Note: in my case: cd /tmp/37366599*
   
2. Launch SQL*Plus and establish a connection as the administrative database user `SYSDBA`:
   ```bash
   sqlplus / as sysdba
   ```
3. **[Conditional]** If your APEX metadata resides inside a Pluggable Database (PDB), explicitly switch container sessions into that active PDB (skip this step if using a standard non-CDB):
   ```sql
   ALTER SESSION SET CONTAINER = YOUR_PDB_NAME;
   ```
   
   *i use PDB for apex so I entered my Apex PDB above and switched to it*

4. Execute the correct APEX patch installation driver script discovered in the package:
   ```sql
   @catpatch.sql
   ```
   *Note: Allow the routine to finish execution. This will take a few minutes as it compiles all changed `.sql` and `.plb` files.*
5. Once completed, exit out of your active database session:
   ```sql
   EXIT;
   ```

---

## Phase 5: Merge Updated Static Images
The patch image folder only contains updated "delta" files rather than the entire library. Do **not** wipe out your target folder. Instead, use the following recursive merge command to overwrite only the changed files while preserving the remaining base structure:

I copied the whole images folder to be on the safer side
```bash
cd /home/oracle/apex/
mkdir images_old
cp -r images/* images_old/
```
then executed the below command.

```bash
cp -r /tmp/37366599/images/* /home/oracle/apex/images
```



---

## Phase 6: Restart Frontend and Verify
1. Start your ORDS service proxy handler engine back up:
   ```bash
   sudo systemctl start ords
   ```
   as I use tmux so I did below commands
   ```bash
   tmux new -s ords
   sudo ~/ords/bin/ords --config /etc/ords/conf/ serve
   ```

    *Detach from the tmux session without terminating the process:
    Press Ctrl + b, then release both keys and press d.*


2. Verify the system version update by running a query in SQL*Plus (connected to your PDB if applicable):
   ```sql
   SELECT patch_version, installed_on 
   FROM apex_patches 
   WHERE patch_number = 37366599;
   ```
   *Alternatively, log into the workspace interface and navigate to the **Help > About** menu dashboard to verify the release displays **24.2.19**.*







