---
layout: page
title: Certifications & Labs
icon: fas fa-stream
order: 6
---

<style>
  .cert-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 20px;
    margin-top: 20px;
  }
  .cert-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    background: var(--card-bg, rgba(128, 128, 128, 0.05)); /* Adapts to Chirpy dark/light mode */
    padding: 15px;
    border-radius: 10px;
    border: 1px solid var(--border-color, rgba(128, 128, 128, 0.2));
    transition: transform 0.2s ease;
  }
  .cert-card:hover {
    transform: translateY(-5px);
  }
  .cert-card img {
    width: 80px;
    height: 80px;
    object-fit: contain;
    margin-bottom: 15px;
    box-shadow: none !important; /* Overrides default Chirpy image shadows */
  }
  .cert-title {
    font-size: 0.9rem;
    font-weight: 600;
    line-height: 1.3;
  }
</style>

<div class="cert-grid">
  
  <!-- Cisco Section -->
  <div class="cert-card">
    <img src="/assets/img/cisco-linux.png" alt="Linux Essentials">
    <span class="cert-title">Cisco NDG Linux Essentials</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/cisco-security.png" alt="Network Security">
    <span class="cert-title">Cisco Network Security</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/cisco-intro.png" alt="Intro to Networks">
    <span class="cert-title">Cisco Introduction To Networks</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/cisco-routing.png" alt="Switching and Routing">
    <span class="cert-title">Cisco Switching and Routing</span>
  </div>

  <!-- AWS & Containers Section -->
  <div class="cert-card">
    <img src="/assets/img/aws-fargate.png" alt="AWS Fargate">
    <span class="cert-title">AWS Fargate Foundational</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/aws-lambda.png" alt="AWS Lambda">
    <span class="cert-title">AWS Lambda Foundations</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/containers.png" alt="Intro to Containers">
    <span class="cert-title">Introduction To Containers</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/docker.png" alt="Docker">
    <span class="cert-title">Docker Basics for DevOps</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/kubernetes.png" alt="Kubernetes">
    <span class="cert-title">Kubernetes Basics for DevOps</span>
  </div>

  <!-- Automation & DevOps Section -->
  <div class="cert-card">
    <img src="/assets/img/ansible.png" alt="Ansible">
    <span class="cert-title">Ansible for the Absolute Beginner</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/terraform.png" alt="Terraform">
    <span class="cert-title">Terraform for the Absolute Beginner</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/jenkins.png" alt="Jenkins">
    <span class="cert-title">Jenkins for Beginners</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/git.png" alt="Git">
    <span class="cert-title">Git Basics for DevOps</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/python.png" alt="Python">
    <span class="cert-title">Python for the Absolute Beginner</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/devops.png" alt="DevOps Mastery">
    <span class="cert-title">DevOps Mastery</span>
  </div>
  <div class="cert-card">
    <img src="/assets/img/devops-prereq.png" alt="DevOps Prerequisite">
    <span class="cert-title">DevOps Prerequisite Course</span>
  </div>

</div>
