#  VM Backup and Restoration on Proxmox VE (Disaster Recovery)

**Project**: Implementation of a Business Continuity Plan (BCP) and Disaster Recovery Plan (DRP) in a nested virtualized environment.

##  Overview

This project demonstrates the implementation of a resilient infrastructure using Proxmox Virtual Environment (VE). The objective is to master the two pillars of data protection:

- **Snapshots**: For quick versioning and testing
- **Backups**: For protection against total data loss

The project was carried out in a **Nested Virtualization** environment, simulating a real datacenter from a standard workstation.

##  Lab Architecture

The infrastructure is based on a layered architecture:

- **Physical Host**: Windows 11 PC (Wi-Fi/Ethernet Network)
- **Level 1 Hypervisor**: VMware Workstation Pro/Player (NAT/Bridge Configuration)
- **Level 2 Hypervisor**: Proxmox VE 8.x (Static IP)
- **Target VM (Victim)**: Ubuntu Server 24.04 LTS
```mermaid
graph TD;
    A[Physical PC Windows] -->|Hosts| B[VMware Workstation];
    B -->|Virtualizes| C[Proxmox VE];
    C -->|Hosts| D[VM Ubuntu Server];
    C -->|Stores Backups| E[Local Disk];
```

## ⚙️ Prerequisites & Configuration

To allow Proxmox to run within VMware, specific configuration was required:

- Enable nested virtualization (Virtualize Intel VT-x/EPT or AMD-V/RVI)
- Adapted network configuration (Bridge or NAT) for Proxmox internet access

<img width="750" height="718" alt="image" src="https://github.com/user-attachments/assets/e6e84f8f-27b3-4b10-99dd-c41b764cc4dd" />


## VM UBUNTU installation
<img width="1908" height="711" alt="image" src="https://github.com/user-attachments/assets/3cca6f90-5367-4ab4-bc9d-2e55675c4cc9" />
<img width="1906" height="982" alt="image" src="https://github.com/user-attachments/assets/cb39deb5-9249-4ff5-882e-1cd011f29f43" />
## test file 
<img width="781" height="89" alt="image" src="https://github.com/user-attachments/assets/61163112-6050-4505-b575-af48de43d640" />




##  Part 1: Snapshots (Short-term Protection)

**Scenario**: Critical system modification (Simulating human error)

1. Creation of a critical file on the Ubuntu VM (`important_data.txt`)
2. Taking a snapshot named "Stable-State"
3. **Incident**: Accidental deletion of the file using the `rm` command
4. **Resolution**: Rollback via Proxmox

| Action | Result |
|--------|--------|
| Snapshot Taken | 🟢 Success (State frozen) |
| File Deletion | 🔴 File lost |
| Rollback | 🟢 System restored in < 10s |

<img width="950" height="908" alt="image" src="https://github.com/user-attachments/assets/ecf2ad34-f3c9-44b7-91fa-db24971129a4" />

File deletion : <img width="447" height="67" alt="image" src="https://github.com/user-attachments/assets/56d9b72c-3c67-45bf-a290-eb8d70c4784b" />

ROllback : 
<img width="649" height="165" alt="image" src="https://github.com/user-attachments/assets/968a0022-7577-46c0-8048-b2f0ada79186" />

After the RollbACK : <img width="443" height="52" alt="image" src="https://github.com/user-attachments/assets/48bbe23a-5641-413f-a5e2-2cc67c3291cf" />




##  Part 2: Backups (Long-term Protection)

**Scenario**: Total server crash or VM deletion (Disaster Recovery)

1. Backup storage configuration (`vzdump` on local)
2. Full backup execution (Snapshot Mode, ZSTD Compression)
3. **Major Incident**: Complete deletion of VM 100 (Simulating disk crash)
4. **Resolution**: Complete restoration from backup archive

### Technical Comparison

| Feature | Snapshot 📸 | Backup 💾 |
|---------|------------|-----------|
| Storage | Differential (on VM disk) | Compressed archive `.vma.zst` (Independent) |
| Independence | Depends on original disk | Autonomous (can be moved) |
| Use Case | Before update / Testing | Disaster / Archiving / Ransomware |

 backup log :  <img width="781" height="490" alt="image" src="https://github.com/user-attachments/assets/a1b65540-e5a8-4599-9b52-355c59b0e8a9" />

 deleting the VM : <img width="511" height="236" alt="image" src="https://github.com/user-attachments/assets/9e2acd00-1ac5-4130-9651-86aa8f004255" />


 VM restored via backup <img width="936" height="903" alt="image" src="https://github.com/user-attachments/assets/87f72564-9e5d-47d2-ab33-f3659dea74c7" />


##  Automation

To guarantee the RPO (Recovery Point Objective) rule, a scheduled task was created:

- **Frequency**: Every 30 minutes (for testing)
- **Retention**: Keep only the last 2 copies (to save space)
- 
 schedule backup every 30 min : <img width="1602" height="208" alt="image" src="https://github.com/user-attachments/assets/96144cef-401d-463e-9861-a8df9a48acca" />

##  Conclusion

This project validated:

- The feasibility of nested virtualization for complex labs
- The reliability of Proxmox's snapshot mechanism (based on QCOW2/LVM)
- The robustness of `vzdump` full backups for disaster recovery


### Possible Future Improvements

- Implementation of Proxmox Backup Server (PBS) for deduplication
- Sending backups to external NAS or Cloud (3-2-1 Rule)


