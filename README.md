# To use:

`ansible-playbook --ask-sudo-pass playbook.yml`

## example playbook

```yml
- hosts: localhost
  become: no
  vars:
      app_store:
          - Telegram
          - Xcode
      brew_apps:
          - mas
          - thefuck
          - tcpflow
          - bash
          - htop
          - smartmontools
          - qrencode
          - locateme
  tasks:
    - homebrew: name={{ item }}
      with_items: "{{ brew_apps }}"
      tags: brew
      
    - block:
        - name: search appstore
          shell: mas search {{ item }}|grep --color -E '^\d+\s+{{ item }}$'
          with_items: "{{ app_store }}"
          register: appstore
      
        - name: show apps
          debug: msg="{{ appstore.results|map(attribute='stdout')|list }}"
      
        - name: install apps from the appstore
          shell: mas install {{ item.split()[0] }}
          with_items: "{{ appstore.results|map(attribute='stdout')|list }}"
      tags: appstore

- hosts: localhost
  become: no
  roles:
    - role: ansible.osx/apps
      install_virtualbox: yes
      install_flux: yes
      tags: apps

```

`mas` is used for app store installations, `locateme` is required for flux setup.
