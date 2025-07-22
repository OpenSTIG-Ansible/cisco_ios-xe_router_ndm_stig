# Ansible Role: Cisco IOS XE NDM STIG

This role automates the application of the [Cisco IOS XE Router NDM Security Technical Implementation Guide (STIG)](https://public.cyber.mil/stigs/downloads/?_dl_facet_stigs=network-perimeter-ndm). It is designed to be modular, allowing you to enable or disable specific STIG controls through variables.

## Description

This Ansible role configures a Cisco IOS XE device to be compliant with the DISA STIG. Each STIG requirement (V-Key) is implemented as a task or a block of tasks. The role is designed to be idempotent where possible, meaning it will only make changes to the device if the configuration is not already compliant.

## Requirements

-   **Ansible:** `ansible-core >= 2.11`
-   **Ansible Collections:** `cisco.ios`

You can install the required collection using Ansible Galaxy:

```bash
ansible-galaxy collection install cisco.ios
```

## Role Variables

This role uses two primary files for variable management: `defaults/main.yml` and `vars/main.yml`.

### `defaults/main.yml`

This file contains role-wide default variables that are generally static, such as the standard DoD login banner. You typically do not need to edit this file.

### `vars/main.yml`

This is the main file you will interact with to control the role's behavior. It is structured to allow granular control over which STIGs are applied.

#### How it Works

Each configurable STIG control is represented by a dictionary keyed by its `V-Key` (e.g., `V-215855`). Inside each dictionary, you will find:

1.  A `run` flag: Set this to `true` to enable the task(s) for that STIG. By default, most are set to `false`.
2.  Configuration parameters: These are the specific values needed for the configuration, such as IP addresses, server names, or passwords.

**To enable a STIG control, you must:**
1.  Find the corresponding V-Key block in `vars/main.yml`.
2.  Change `run: false` to `run: true`.
3.  Update the placeholder parameter values with the correct settings for your environment.

If a task does not require environment-specific variables, it will run automatically without needing a corresponding entry in `vars/main.yml`.

## Idempotency

Ansible tasks are idempotent if they ensure that running the same task multiple times results in the same state, only making changes on the first run if the system is not in the desired state.

-   **Idempotent Tasks:** Most tasks in this role are idempotent. They check the device's running configuration before applying any changes. If the device is already compliant, the task will be reported as `ok` (green) and no changes will be made.
-   **Non-Idempotent Tasks:** A few tasks are **not** idempotent due to the nature of the configuration they apply, which usually involves sensitive data like passwords or encrypted keys. The device stores these values as secure hashes that cannot be read back directly. Because Ansible cannot compare the existing value with the one in the variable file, it must apply the configuration on every run to ensure it is set correctly. These tasks will always report as `changed` (yellow).

## Usage

1.  **Configure Inventory:** Create an inventory file (e.g., `inventory`) with the connection details for your router(s).

    ```ini
    [routers]
    router1 ansible_host=192.168.1.1
    ```

2.  **Edit Variables:** Open `vars/main.yml` and enable the STIG controls you wish to apply by setting `run: true` and updating the required variables.

3.  **Create Playbook:** Create a playbook file (e.g., `playbook.yml`) to execute the role.

    ```yaml
    ---
    - name: Apply Cisco IOS XE STIG
      hosts: routers
      gather_facts: false
      connection: network_cli
      roles:
        - cisco_ios-xe_router_ndm_stig
    ```

4.  **Run Playbook:** Execute the playbook from your terminal.

    ```bash
    ansible-playbook -i inventory playbook.yml
    ```

5.  **Running Specific STIGs with Tags:**
    All tasks are tagged with their corresponding `V-Key`, `STIG ID`, and `CAT` level (e.g., `CAT II`). This allows you to run specific checks or categories of checks.

    For example, to run only the task for `V-215855`:
    ```bash
    ansible-playbook -i inventory playbook.yml --tags "V-215855"
    ```

    To run all Category I checks:
    ```bash
    ansible-playbook -i inventory playbook.yml --tags "CAT I"
    ```

## STIG Task Summary

The table below provides a summary of each implemented STIG, its idempotency status, and any required actions in the `vars/main.yml` file.

| V-Key / STIG ID                                                                                                                              | Description                                                         | Idempotent? | Variable Driven? | Action Required in `vars/main.yml`                                                                                  |
| -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------- | ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| **V-215807** / SRG-APP-000001                                                                                                                 | Limit concurrent sessions and secure VTY lines.                     | Yes         | No               | None.                                                                                                               |
| **V-215808, V-215809, V-215810, V-215811, V-215815, V-215819, V-215834, V-215848, V-215850** / CISC-ND-000090 and others                          | Enable configuration change logging.                                | Yes         | No               | None.                                                                                                               |
| **V-215813** / CISC-ND-000150                                                                                                                  | Block logins after multiple failed attempts.                        | Yes         | No               | None.                                                                                                               |
| **V-215814** / CISC-ND-000160                                                                                                                  | Configure DoD login banner.                                         | Yes         | Yes (Default)    | The banner is in `defaults/main.yml`. No action is needed unless you wish to customize it.                         |
| **V-215817** / CISC-ND-000280                                                                                                                  | Configure timestamps for logging.                                   | Yes         | No               | None.                                                                                                               |
| **V-215818** / CISC-ND-000290                                                                                                                  | Check for `log-input` on ACL deny statements.                       | N/A         | No               | See [Manual STIG Controls](#manual-stig-controls) section below.                                                    |
| **V-215820, V-215821, V-215822** / CISC-ND-000380, CISC-ND-000390, CISC-ND-000460                                                               | Restrict file system access to privilege level 15.                  | Yes         | No               | None.                                                                                                               |
| **V-215823** / CISC-ND-000470                                                                                                                  | Disable unnecessary and nonsecure services.                         | Yes         | No               | None.                                                                                                               |
| **V-215826, V-215827, V-215828, V-215829, V-215830, V-215831** / CISC-ND-000550 and others                                                      | Configure common criteria password policies.                        | Yes         | No               | None.                                                                                                               |
| **V-215832** / CISC-ND-000620                                                                                                                  | Enable `service password-encryption`.                               | Yes         | No               | None.                                                                                                               |
| **V-215833** / CISC-ND-000720                                                                                                                  | Configure session timeouts for HTTP, console, and VTY.              | Yes         | No               | None.                                                                                                               |
| **V-215836** / CISC-ND-000980                                                                                                                  | Configure logging buffer size and level.                            | Yes         | Yes              | Set `V-215836.run: true` and define the buffer size and level.                                                      |
| **V-215837** / CISC-ND-001000                                                                                                                  | Generate alerts for critical audit failure events.                  | Yes         | No               | None.                                                                                                               |
| **V-215838** / CISC-ND-001030                                                                                                                  | Synchronize clock with unauthenticated NTP sources.                 | Yes         | Yes              | Set `V-215838.run: true` and provide a list of NTP servers. Skipped if `V-215843` (authenticated NTP) is enabled. |
| **V-215841** / CISC-ND-001130                                                                                                                  | Configure SNMPv3 with authentication (auth).                        | **No**      | Yes              | Set `V-215841.run: true` and provide SNMPv3 group, user, and password details.                                   |
| **V-215842** / CISC-ND-001140                                                                                                                  | Configure SNMPv3 with encryption (priv).                            | **No**      | Yes              | Set `V-215842.run: true` and provide SNMPv3 details. Supersedes V-215841.                                        |
| **V-215843** / CISC-ND-001150                                                                                                                  | Authenticate NTP sources with a cryptographic key.                  | **No**      | Yes              | Set `V-215843.run: true` and provide NTP key details. Supersedes V-215838.                                      |
| **V-215844** / CISC-ND-001200                                                                                                                  | Enforce SSHv2 and FIPS-validated HMAC.                              | Yes         | No               | None.                                                                                                               |
| **V-215845** / CISC-ND-001210                                                                                                                  | Configure strong SSH encryption algorithms.                         | Yes         | No               | None.                                                                                                               |
| **V-215849** / CISC-ND-001260                                                                                                                  | Enable logging for successful and failed logins.                    | Yes         | No               | None.                                                                                                               |
| **V-215854** / CISC-ND-001370                                                                                                                  | Use at least two RADIUS servers for authentication.                 | **Partial** | Yes              | Set `V-215854.run: true` and provide RADIUS server details. The `key` is not idempotent.                          |
| **V-215855** / CISC-ND-001410                                                                                                                  | Back up the configuration when changes occur using an EEM applet.   | Yes         | Yes              | Set `V-215855.run: true` and provide the SCP server IP.                                                             |
| **V-215856** / CISC-ND-001440                                                                                                                  | Obtain public key certificates from an approved CA.                 | Yes         | Yes              | Set `V-215856.run: true` and provide the PKI trustpoint name and enrollment URL.                                   |
| **V-220139** / CISC-ND-001450                                                                                                                  | Send log data to at least two syslog servers.                       | Yes         | Yes              | Set `V-220139.run: true` and provide a list of at least two syslog server IPs.                                    |

## Manual STIG Controls

Some STIG requirements cannot be safely or fully automated. For these controls, this role may provide tasks that check for compliance and report on the findings, but manual intervention is required to remediate them.

| V-Key / STIG ID        | Description                                     | Reason for Manual Implementation                                                                                                                  | Manual Check / Remediation Guidance                                                                                                                                                                                                                                                                                             |
| ---------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **V-215812** / CISC-ND-000140 | Restrict management access via an ACL.          | Defining a management network is highly specific to the environment. Automating this without user-provided details is unsafe and likely to cause lockouts. | **Check:** Run `show running-config \| section access-list|line vty` and verify that an access-list is defined to permit only management traffic and that it is applied to the VTY lines using `access-class <ACL_NAME> in`. <br/> **Fix:** Create an access list that permits your management network and denies all other traffic, then apply it to the VTY lines (e.g., `line vty 0 4`, `access-class MANAGEMENT_NET in`). |
| **V-215818** / CISC-ND-000290 | Use `log-input` on ACL deny statements.         | Safely modifying complex ACLs is difficult to automate without a deep understanding of the network's traffic flow. An incorrect change could block critical traffic. | The role includes a task that checks for extended ACLs containing `deny` statements that are missing the `log-input` parameter. <br/><br/> **Check:** `show running-config \| include ip access-list` <br/> **Fix:** Manually add the `log-input` keyword to the end of each required deny statement within the relevant ACLs. |
| **V-215824** / CISC-ND-000490 | Maintain only one local "account of last resort". | Creating and managing specific user accounts is a sensitive operation and highly dependent on organizational policy. This role avoids managing local users to prevent unintended security consequences. | **Check:** `show running-config \| include ^username` to list local users. Ensure only one privileged account exists for emergency access. Also, verify that `local` is configured as a fallback authentication method (e.g., `aaa authentication login default group tacacs+ local`), which is handled by V-215854. <br/> **Fix:** Manually create one privileged local user and remove any others. |
| **V-220140** / CISC-ND-001470 | Run a supported IOS XE release.                 | Verifying software support requires checking an external resource (Cisco's website). An OS upgrade is a major operational change that requires careful planning and should not be automated by a compliance role. | **Check:** Run `show version` and compare the output against the End-of-Life notices on [Cisco's support website](https://www.cisco.com/c/en/us/support/ios-nx-os-software). <br/> **Fix:** Follow organizational procedures to plan and execute an IOS XE upgrade to a supported version. |
