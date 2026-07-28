

# What is SIEM ?
**SIEM** (Security Information and Event Management) is a centralized security tool that collects and analyzes log data from across your entire network in one place.


## How SIEM Helps in a SOC ?

- Gathers logs from all network devices into a single dashboard for complete visibility.
    
- Correlates separate suspicious events in real time to spot hidden attacks automatically.
    
- Delivers immediate context like affected IPs and timestamps to speed up threat containment.

# What is Wazuh ?

**Wazuh** is a free, open-source SIEM and XDR (Extended Detection and Response) platform.

## Wazuh architecture
The Wazuh architecture relies on four modular components to handle end-to-end telemetry processing:

The **Wazuh Agent** runs on endpoints to collect system logs, monitor file integrity, and scan for vulnerabilities. It ships this telemetry to the **Wazuh Server**, the central engine that normalizes data, compares events against detection rules, and triggers security alerts. Processed events are stored in the **Wazuh Indexer**, a high-performance search engine that indexes logs for fast querying and long-term storage. Finally, analysts use the **Wazuh Dashboard**, a web-based UI, to query logs, visualize threat metrics, and manage the platform.
	![[Pasted image 20260726201802.png]]

## Wazuh Instalation and Configuration :
Wazuh provides pre-built virtual appliance images in OVA format. We can download the latest version from the official Wazuh website and then import then import the virtual machine image directly into VirtualBox. However, since I'm using QEMU for this project and it doesn't natively support OVA archives, the file must first be extracted and converted to QCOW2 format.

```bash
tar -xvf wazuh-*.ova
qemu-img convert -f vmdk -O qcow2 wazuh-*.vmdk wazuh-server.qcow2
```

  1. Start the VM.

  2. Log in using the default credentials
```
user: wazuh-user
password: wazuh
```


  3. Once logged in , we grab the server's IP address
```
ip a
```


  4. Open the web dashboard in the browser (`https://<WAZUH_IP>`) and log in using the default credentials (admin:admin) .
![[Pasted image 20260726215204.png]]
![[Pasted image 20260726215713.png]]

## Wazuh Agent Installation :
The **Wazuh Agent** is a lightweight security service installed on endpoints (such as Windows, Linux, or macOS machines) that continuously collects system logs, monitors file integrity, tracks active processes, and scans for vulnerabilities. It encrypts this security telemetry and securely sends it back to the central Wazuh server for real-time analysis and detection.

Now we need to deploy a Wazuh agent on Windows, which we do by generating the deployment script directly from our server's web interface and executing it on our target machine:

- We open the Wazuh dashboard in our browser and navigate to the **Agents** section to add a new Windows agent.
    
- We enter our Wazuh server's IP address in the configuration prompt so the agent knows where to send telemetry.
    
  
  ![[Pasted image 20260727163608.png]]
	![[Pasted image 20260727163654.png]]
	
	
- We copy the generated PowerShell command from the dashboard.
- We open PowerShell as Administrator on our Windows machine and paste the command to automatically download, configure, and register the agent.
	![[Pasted image 20260727172913.png]]

* Once the installation command finishes executing, we return to our Wazuh dashboard to verify that the target endpoint has successfully connected to our manager.
  ![[Pasted image 20260727173637.png]]


# Conclusion

With the server running and the agent connected, our SIEM lab is fully set up. We now have real-time visibility over our Windows endpoint and are ready to observe security alerts during simulated attacks.