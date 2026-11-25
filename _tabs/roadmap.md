---
title: Roadmap
icon: fas fa-graduation-cap
order: 5
mermaid: true
---

## Career Progression

```mermaid
flowchart TD
    A[Diploma in Network Security<br/>& Hands-on Labs] --> B[MIS Administrator]
    B --> C[Application Support Engineer]
    C --> D[Application Support Specialist]
    C --> E[DevOps Engineer]
    D -.-> F[Site Reliability Engineer - SRE]
    E --> F

    style A fill:#2d3748,stroke:#4a5568,color:#fff
    style B fill:#2d3748,stroke:#4a5568,color:#fff
    style C fill:#2b6cb0,stroke:#63b3ed,color:#fff
    style D fill:#1a202c,stroke:#a0aec0,stroke-dasharray: 5 5,color:#e2e8f0
    style E fill:#1a202c,stroke:#a0aec0,stroke-dasharray: 5 5,color:#e2e8f0
    style F fill:#1a202c,stroke:#68d391,stroke-dasharray: 5 5,color:#68d391
```

> **Note:** Solid boxes represent completed or active milestones. Dashed boxes represent target paths.
{: .prompt-info }

---

## Technical Skills

<style>
  .skills-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1.1rem;
    margin-top: 1rem;
  }
  .skill-card {
    background: var(--card-bg, #1e1e1e);
    border: 1px solid var(--border-color, #333);
    border-radius: 10px;
    padding: 1.2rem;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
  }
  .skill-card.target-card {
    border: 1.5px dashed #68d391;
  }
  .skill-card h4 {
    margin-top: 0;
    margin-bottom: 0.85rem;
    font-size: 1rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  .badge-group {
    display: flex;
    flex-wrap: wrap;
    gap: 0.45rem;
    align-items: center;
  }
  .badge-group img {
    margin: 0 !important;
    border-radius: 4px;
    height: 26px;
  }
</style>

<div class="skills-grid">
  <div class="skill-card">
    <h4><i class="fas fa-server"></i> OS &amp; System Admin</h4>
    <div class="badge-group">
      <img src="https://img.shields.io/badge/Red_Hat-EE0000?style=for-the-badge&logo=redhat&logoColor=white" alt="Red Hat">
      <img src="https://img.shields.io/badge/Rocky_%2F_CentOS-10B981?style=for-the-badge&logo=rockylinux&logoColor=white" alt="Rocky and CentOS">
      <img src="https://img.shields.io/badge/Ubuntu_Server-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu Server">
      <img src="https://img.shields.io/badge/Windows_Server-0078D6?style=for-the-badge&logo=windows&logoColor=white" alt="Windows Server">
      <img src="https://img.shields.io/badge/Active_Directory-005A9E?style=for-the-badge&logo=windows&logoColor=white" alt="Active Directory">
      <img src="https://img.shields.io/badge/Group_Policy_(GPO)-2B6CB0?style=for-the-badge" alt="GPO">
      <img src="https://img.shields.io/badge/IIS-0078D6?style=for-the-badge" alt="IIS">
      <img src="https://img.shields.io/badge/VMware_%2F_Hyper--V-607078?style=for-the-badge&logo=vmware&logoColor=white" alt="VMware and Hyper-V">
      <img src="https://img.shields.io/badge/Patching_%26_Backup-4A5568?style=for-the-badge" alt="Patching and Backup">
    </div>
  </div>

  <div class="skill-card">
    <h4><i class="fas fa-network-wired"></i> Networking &amp; Security</h4>
    <div class="badge-group">
      <img src="https://img.shields.io/badge/DNS-2B6CB0?style=for-the-badge" alt="DNS">
      <img src="https://img.shields.io/badge/DHCP-2B6CB0?style=for-the-badge" alt="DHCP">
      <img src="https://img.shields.io/badge/TCP%2FIP_%26_Subnetting-2C5282?style=for-the-badge" alt="TCP/IP and Subnetting">
      <img src="https://img.shields.io/badge/VLANs-2C5282?style=for-the-badge" alt="VLANs">
      <img src="https://img.shields.io/badge/Firewalls-C53030?style=for-the-badge" alt="Firewalls">
      <img src="https://img.shields.io/badge/VPNs-2F855A?style=for-the-badge" alt="VPNs">
      <img src="https://img.shields.io/badge/SSH_Hardening-1A202C?style=for-the-badge&logo=gnubash&logoColor=white" alt="SSH Hardening">
      <img src="https://img.shields.io/badge/SSL%2FTLS-2F855A?style=for-the-badge&logo=letsencrypt&logoColor=white" alt="SSL/TLS">
    </div>
  </div>

  <div class="skill-card">
    <h4><i class="fas fa-tools"></i> App Support &amp; Ops</h4>
    <div class="badge-group">
      <img src="https://img.shields.io/badge/Log_Analysis-4A5568?style=for-the-badge" alt="Log Analysis">
      <img src="https://img.shields.io/badge/Root_Cause_Analysis-DD6B20?style=for-the-badge" alt="RCA">
      <img src="https://img.shields.io/badge/ITIL-3182CE?style=for-the-badge" alt="ITIL">
      <img src="https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white" alt="SQL">
      <img src="https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white" alt="Nginx">
      <img src="https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache">
      <img src="https://img.shields.io/badge/API_Troubleshooting-6B46C1?style=for-the-badge&logo=postman&logoColor=white" alt="API Troubleshooting">
    </div>
  </div>

  <div class="skill-card target-card">
    <h4><i class="fas fa-rocket"></i> Target: DevOps &amp; SRE</h4>
    <div class="badge-group">
      <img src="https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Bash">
      <img src="https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white" alt="PowerShell">
      <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
      <img src="https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white" alt="Ansible">
      <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
      <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes">
      <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS">
      <img src="https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Azure">
      <img src="https://img.shields.io/badge/Git_%26_CI%2FCD-F05032?style=for-the-badge&logo=git&logoColor=white" alt="Git and CI/CD">
      <img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white" alt="Prometheus">
      <img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white" alt="Grafana">
    </div>
  </div>
</div>
