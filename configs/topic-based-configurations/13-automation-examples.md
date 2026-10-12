# Automation & Programmability - Examples

Illustrative snippets for the automation topics in the guide. Adapt hosts, credentials, and modules to your lab.

## 1. Ansible playbook (`cisco.ios` collection)

`inventory.ini`
```ini
[routers]
R1 ansible_host=192.168.99.1

[routers:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=cisco.ios.ios
ansible_user=admin
ansible_password={{ vault_password }}
```

`vlans.yml`
```yaml
---
- name: Configure VLANs on switches
  hosts: switches
  gather_facts: false
  tasks:
    - name: Ensure VLANs exist
      cisco.ios.ios_vlans:
        config:
          - vlan_id: 10
            name: SALES
          - vlan_id: 20
            name: ENGINEERING
        state: merged

    - name: Save config
      cisco.ios.ios_config:
        save_when: modified
```
Run: `ansible-playbook -i inventory.ini vlans.yml`

## 2. REST/RESTCONF with curl

```bash
curl -k -u admin:'<password>' \
  -H "Accept: application/yang-data+json" \
  https://192.168.99.1/restconf/data/ietf-interfaces:interfaces
```

## 3. Python `requests` (RESTCONF GET)

```python
import requests, urllib3
urllib3.disable_warnings()          # lab only - self-signed cert

url = "https://192.168.99.1/restconf/data/ietf-interfaces:interfaces"
headers = {"Accept": "application/yang-data+json"}
resp = requests.get(url, headers=headers, auth=("admin", "<password>"), verify=False)
resp.raise_for_status()
for intf in resp.json()["ietf-interfaces:interfaces"]["interface"]:
    print(intf["name"], intf.get("enabled"))
```

## 4. Enabling RESTCONF on IOS XE (lab)

```text
ip http secure-server
restconf
username admin privilege 15 secret <password>
```

## 5. Concepts mapped

| Concept | Where it shows up |
|---|---|
| REST verbs (GET/POST/PUT/PATCH/DELETE) | curl/requests calls |
| JSON data format | RESTCONF responses |
| YANG models | `ietf-interfaces` path |
| Agentless, push-based config management | Ansible over SSH |
| Infrastructure as code | Playbooks in Git |
