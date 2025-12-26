---
title: "Automating Cisco IOS Upgrades: A Step-by-Step Ansible Guide (Bundle Mode)"
seoTitle: "Automate Cisco IOS Upgrade with Ansible Guide"
seoDescription: "Automate Cisco IOS upgrades with Ansible for efficient, error-free updates across devices. Follow this step-by-step guide"
datePublished: Mon Dec 22 2025 09:38:08 GMT+0000 (Coordinated Universal Time)
cuid: cmjgyqqz4000102jsaymv3trw
slug: automating-cisco-ios-xe-upgrade-ansible
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1766556881784/2554d2bb-0a3d-40f9-8611-2ad6086ab463.png
tags: tutorial, ansible, networking, cisco, netdevops, network-automation

---

## Introduction

Upgrading network devices manually is one of the most tedious tasks for any Network man, it involves repetitive steps: checking flash storage, setting up a SCP server, copying files, and praying that the console cable doesn’t disconnect.

When you have to upgrade 50 or 100 switches/routers, manual work is not an option. It is slow and prone to **human error**. In this post, I’ll share how to write Ansible playbook to automate this process efficiently, use Ansible Automation Platform for orchestration and manage the workflow.

For this blog, I assume your **Ansible Automation Platform** is already up and running.

## Diagram

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766289223837/447e74d1-7016-4340-968c-9c79a6436119.png align="center")

## The logic: The 3-phase Workflow

Instead of writing a single playbook, I prefer to split the process into three distinct phases. This approach helps me isolate issues and ensures that if something fails, it fails early and safely.

**1\. Phase 1: Preparation & Staging**

* Health Checks: Version, Storage, Register, Routing.
    
* Staging: Transfer image & Verify MD5
    

**2\. Phase 2: Activation (The Upgrade)**

* Backup configuration
    
* Boot & Reload
    

**3\. Phase 3: Post-checks**

* Validate version
    
* Compare with Pre-check
    
    > *💡* ***Why use AAP for this?*** *By leveraging* *Ansible Automation Platform (AAP), we can wrap these three distinct phases into a single* *Workflow Template*. This allows us to use the *Workflow Visualizer* *to drag-and-drop logic, add approval nodes easily, and—most importantly—if one phase fails (e.g., Staging), we can fix it and retry just that node without re-running the whole process.*
    

## The Orchestration

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766323881986/afceced9-2717-4a08-875f-34ca9b65d4ec.png align="center")

My logic: The “Human-in-the-Loop” Strategy

* **Phase 1 (Pre-checks):** Runs automatically, it gathers data, checks storage, and stages image. While Ansible is copying the file, I simply grab a cup of coffee and relax ☕, this is the longest process.
    
* **The Safety Pause (Approval Node):** This is critical. The workflow pauses here. It waits for me to review the Pre-check report. If I see any red flags (like full storage or unstable routing), I deny the job. Nothing get rebooted.
    
* **Phase 2 (The Upgrade):** Only triggers after I manually click "Approve". This ensures the reboot happens exactly when I want it
    
* **Visual Debugging:** If Phase 3 turns red, I know instantly that the Post-check failed, without digging through thousands of log lines.
    

## The Implementation

For this demonstration, I utilized a **Cisco C8000v** virtual router running **IOS XE**. The workflow aims to perform an upgrade from version **17.06.03** to **17.15.04c**.

> Additionally, to keep the lab simple, I configured the **AAP controller itself** to act as the backend **SCP Server** for hosting and transferring the image files.

Here is the core logic for the Pre-check phase. My approach is to capture a full snapshot of the hardware inventory and current network state, including the routing table and interface status. I check each component individually and transfer the image file from the SCP Server, saving all these parameters as `extra_vars` to ensure a precise comparison during the post-check phase.

```yaml
- name: Upgrade Pre-Check Playbook for Cisco IOS and IOS-XE Devices
  hosts: all
  gather_facts: no
  vars:
    ansible_command_timeout: 10800
    scp_server: "{{ scp_server }}"
    scp_username: "{{ scp_username }}"
    scp_password: "{{ scp_password }}"
    new_ios_version: "{{ new_ios_version }}"
    new_ios_image: "{{ new_image_name }}"
    new_image_md5: "{{ new_image_md5 }}"

  tasks:
    - name: Get all elements hardware device
      cisco.ios.ios_facts:
        gather_subset: hardware
      register: pre_upgrade_facts

    - name: Display pre-upgrade facts
      debug:
        var: pre_upgrade_facts
```

You can view the full Source Code for Phase 1 here: [01\_upgrade\_pre\_check.yaml](https://github.com/haviethung3004/ansible-cisco-ios-upgrade/blob/main/01_upgrade_pre_check.yaml)

Moving to **Phase 2**, we execute the actual upgrade. This involves backing up the running configuration, setting the new boot system, saving the config, and finally reloading the router. The task then pauses to wait for the device to come back online.

```yaml
---
- name: Upgrade Bundle Mode IOS Device
  hosts: all
  gather_facts: yes
  vars:
    ansible_command_timeout: 30
    scp_server: "{{ scp_server }}"
    scp_username: "{{ scp_username }}"
    scp_password: "{{ scp_password }}"
    new_ios_version: "{{ new_ios_version }}"
    new_ios_image: "{{ new_image_name }}"
    new_image_md5: "{{ new_image_md5 }}"
    
  tasks:
    - name: Backup configuration before upgrade
      cisco.ios.ios_command:
        commands: 
          - command: copy running-config startup-config
            prompt: 'Destination filename \[startup-config\]?'
            answer: "\n"
      register: backup_result
```

You can access the Source Code for Phase 2 here: [02\_upgrade\_bundle\_mode.yaml](https://github.com/haviethung3004/ansible-cisco-ios-upgrade/blob/main/02_upgrade_bundle_mode.yaml)

In **Phase 3**, we validate the upgrade. The script compares the current network state against the "snapshot" taken in Phase 1. This ensures the new version is active and confirms there are no unintended changes to the routing table, interface status,…

```yaml
---
- name: Post-check after IOS Upgrade Playbook
  hosts: all
  gather_facts: false
  vars:
    ansible_command_timeout: 30
    scp_server: "{{ scp_server }}"
    scp_username: "{{ scp_username }}"
    scp_password: "{{ scp_password }}"
    new_ios_version: "{{ new_ios_version }}"
    new_ios_image: "{{ new_image_name }}"
    new_image_md5: "{{ new_image_md5 }}"
  tasks:
    - name: Gather IOS version after upgrade
      cisco.ios.ios_facts:
        gather_subset: hardware
      register: post_upgrade_facts
    
    - name: Validate IOS version
      ansible.builtin.assert:
        that:
          - "{{ post_upgrade_facts.ansible_facts.ansible_net_version == new_ios_version }}"
        fail_msg: "IOS version after upgrade does not match expected version."
        success_msg: "IOS version after upgrade matches expected version."
```

The full code for this validation phase is available here: [03\_upgrade\_post\_check.yaml](https://github.com/haviethung3004/ansible-cisco-ios-upgrade/blob/main/03_upgrade_post_check.yaml)

Once the individual templates are ready, we proceed to **Ansible Automation Platform (AAP)** to orchestrate them. I created a new Workflow Template that combines these three Job Templates into a single pipeline.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766394013146/239b7ebe-cf98-4454-b6d2-44ff1c13e0d6.png align="center")

As shown below, I added an **Approval Node** right after the Pre-check (Node 1) completes. This allows a human engineer to review the pre-check status before authorizing the actual reboot.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766394150838/8240d124-d1f5-4623-bd6d-23117252faaf.png align="center")

Merged 3 job template as 3 node into 1 workflow, and add aproval node after completed Node 1

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766394345772/0b193cc9-6e36-4d2a-b716-5275d46f1e0e.png align="center")

Note: Remember to replace extra\_vars with your actual lab environment details.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766394502767/ac9f889d-f8ad-4e57-a649-d51d5fe76211.png align="center")

Finally, launch the job, grab a cup of coffee, and watch the automation do the heavy lifting!

## The Outcome

After sipping my coffee, I returned to the desk to find the workflow paused exactly where expected: the **Approval Node**.

I quickly reviewed the output from Node 1 (Pre-check) to ensure the device was ready.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766395327798/ff664c18-d753-4b79-afab-8979d3cff64a.png align="center")

Satisfied with the results, I clicked **"Approve"** to authorize the reboot.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766395391140/6d50ec23-8910-4795-a009-a58fff8ff520.png align="center")

Once approved, the automation continued to execute Phase 2 and Phase 3 without any manual intervention. There is nothing quite like seeing that satisfying **"All Green"** status on the Ansible Automation Platform dashboard:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766395455596/1dd15308-dac5-42ce-b55b-8f0149d40ced.png align="center")

Finally, the post-check comparison confirmed the upgrade was successful with **zero unintended changes** to the **routing table** or **interface statuses**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766395522699/05bdaefe-774d-4ffd-a61a-99332883756c.png align="center")

## Conclusion

By automating this workflow, we’ve transformed a high-risk, tedious night shift task into a predictable, click-of-a-button process. Not only does this save time, but it also ensures consistency across hundreds of devices—something manual typing can never guarantee.

> If you have any questions about the playbooks or the logic, feel free to drop a comment below!