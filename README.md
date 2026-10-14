# **Software-Installment-Request--ServiceNow**

An end-to-end ServiceNow Service Catalog and Flow Designer implementation designed to automate employee software installation requests, dynamic manager approvals, fulfillment task generation, and automated email notifications.

---

## **Project Overview**

This project automates the IT Service Management (ITSM) lifecycle for software provisioning in an enterprise environment using ServiceNow. 

When an employee submits a **Software Installation Request** via the Service Portal (`/sp`), the request dynamically routes through an automated 5-step Flow Designer workflow. The workflow manages multi-stage email notifications, manager approvals, and catalog task assignment for IT fulfillment teams.

---

## **System Architecture & Workflow Summary**

```
[ Service Portal Submits Request ]
               │
               ▼
   [ Trigger: Service Catalog ]
               │
               ├─────────────────────────────────────────┐
               ▼                                         ▼
   [ 1. Ask for Manager Approval ]         [ 2. Send Email: Approval Needed ]
   (Assigned to Abel Tuter)               (To: abel.tuter@example.com)
               │
               ▼
   [ 3. Send Email: Request Received ]
   (To: abel.tuter@example.com)
               │
               ▼ (Upon Approval)
   [ 4. Create Catalog Task (SCTASK) ]
   (Assigned to Group: Software)
               │
               ▼
   [ 5. Send Email: Task Assignment ]
   (To: fred.luddy@example.com)
```

---

## **Key Features**

* **Self-Service Catalog Item:** Accessible on Service Portal with mandatory variables for Software Name, Business Justification, and Urgency.
* **Dynamic Manager Approval Routing:** Flow Designer evaluates the `Requested for -> Manager` field on the user profile to assign approvals dynamically.
* **Automated Multi-Stage Email Notifications:**
  1. **Approval Request Email:** Sent to the manager upon submission.
  2. **Submission Confirmation Email:** Sent to the requester with the RITM reference.
  3. **Task Assignment Email:** Sent to the Software assignment group once approved.
* **Fulfillment Task Generation:** Automatically creates an `SCTASK` record assigned to the **Software** assignment group upon approval.

---

##** Prerequisites & Configuration Steps**

### 1. Catalog Item Setup
* **Name:** Software Installation Request
* **Catalog:** Service Catalog | **Category:** Software
* **Variables:**
  * `Software Name` (Single Line Text - Mandatory)
  * `Business Justification` (Multi Line Text - Mandatory)
  * `Urgency` (Select Box: Low, Medium, High)

### 2. User & Manager Profile Configuration
To avoid `Email has no recipients` errors during flow execution:
1. Open `sys_user.list`.
2. Ensure the submitting user's record has a populated **Manager** field (e.g., `Abel Tuter`).
3. Ensure the Manager's profile (`Abel Tuter`) contains a valid **Email** address (e.g., `abel.tuter@example.com`).

---

## **Flow Designer Canvas Implementation**

**Flow Name:** `Software Installation Request Flow`  
**Trigger:** `Service Catalog`

### **Executed Workflow Steps:**
1. **Ask for Approval:** 
   * **Approval Conditions:** `Approve when Requested for -> Manager approves`
2. **Send Email (Approval Needed):**
   * **To:** `Trigger -> Requested Item Record -> Requested for -> Manager`
   * **Subject:** `Approval Needed: Software Installation Request - [RITM Number]`
3. **Send Email (Request Confirmation):**
   * **To:** `Trigger -> Requested Item Record -> Requested for`
   * **Subject:** `Your Software Installation Request Has Been Received - [RITM Number]`
4. **Create Catalog Task:**
   * **Table:** Catalog Task (`sc_task`)
   * **Assignment Group:** `Software`
   * **Short Description:** `Install requested software for user`
5. **Send Email (Task Assignment Notification):**
   * **To:** `Trigger -> Catalog Task Record -> Assignment Group`
   * **Subject:** `Catalog Task [Task Number] has been assigned to group Software`

---

## **Testing & Execution Verification**

1. **Submit Request:** Submitted order via Service Portal (`/sp`), generating item **`RITM00100..`**.
2. **Approval Verification:** Checked `sys_approval.list` and approved the pending request assigned to **Abel Tuter**.
3. **Task Verification:** Checked `sc_task.list` and confirmed task **`SCTASK00100..`** was created and routed to the **Software** group.
4. **Email Queue Verification:** Verified outbound logs in `sys_email.list`:
   * `Approval Needed: Software Installation Request - RITM00100..` (`abel.tuter@example.com`)
   * `Your Software Installation Request Has Been Received - RITM00100..` (`abel.tuter@example.com`)
   * `Catalog Task SCTASK00100.. has been assigned to group Software` (`fred.luddy@example.com`)

---

