# Extended testing framework
Common testing patterns for ansible roles.

## [Requirements](https://github.com/r-pufky/ansible_test/blob/main/meta/main.yml)
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.data](https://github.com/r-pufky/ansible_collection_data) collection.

## Role Variables
Variables are passed directly to the task being called.

[assertions](https://github.com/r-pufky/ansible_test/tree/main/tasks/assertions.yml)
[cache_git](https://github.com/r-pufky/ansible_test/tree/main/tasks/cache_git.yml)
[cache_url](https://github.com/r-pufky/ansible_test/tree/main/tasks/cache_url.yml)
[copy](https://github.com/r-pufky/ansible_test/tree/main/tasks/copy.yml)
[create_cache](https://github.com/r-pufky/ansible_test/tree/main/tasks/create_cache.yml)
[file](https://github.com/r-pufky/ansible_test/tree/main/tasks/file.yml)
[lineinfile](https://github.com/r-pufky/ansible_test/tree/main/tasks/lineinfile.yml)
[remote_file_diff](https://github.com/r-pufky/ansible_test/tree/main/tasks/remote_file_diff.yml)
[sysctl](https://github.com/r-pufky/ansible_test/tree/main/tasks/sysctl.yml)
[template](https://github.com/r-pufky/ansible_test/tree/main/tasks/template.yml)

## Usage
Only use this role during Molecule testing and never for live roles. See each
task for detailed argument list. Tasks must be explicitly called.

### Assertion Testing

#### assertions (assert logic)
Verify assertion logic for roles are trigger correctly.

Assertion tests must be called during the **coverage** step; otherwise failures
will never allow Molecule to reach the verification step.

[assertions](https://github.com/r-pufky/ansible_test/tree/main/tasks/assertions.yml)

converge.yml
``` yaml
- name: 'role my_role fails when mutually exclusive assertion logic fails'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'assertions.yml'
  vars:
    test_name: 'test mutually-exclusive gate'
    test_role: 'r_pufky.my_role'
    test_fail_message: 'my_var, my_var2 did not trigger mutually exclusive assertion.'
    my_var: true
    my_other_var: 'alternative'
```

### Test Caching

#### cache_git (source GIT file caching)
Download and cache a source-only git repository for dynamic testing.

Downloads a GIT source asset locally.

[cache_url](https://github.com/r-pufky/ansible_test/tree/main/tasks/cache_git.yml)

prepare.yml
``` yaml
- name: 'Cache balloon hashing'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'cache_git.yml'
  vars:
    test_name: 'Cache balloon hashing'
    test_cache:
      '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") ~ "/molecule/cache" }}'
    test_dest: 'balloon'
    test_repo: 'https://github.com/nachonavarro/balloon-hashing'
    test_retries: 5
    test_delay: 5
```

#### cache_url (remote URL file caching)
Download and cache a remote file for dynamic testing.

Downloads a remote URL asset cache location if it does not exist.

[cache_url](https://github.com/r-pufky/ansible_test/tree/main/tasks/cache_url.yml)

prepare.yml
``` yaml
- name: 'Cache fonts-test-font.ttf'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'cache_url.yml'
  vars:
    test_name: 'Cache fonts-test-font.ttf'
    test_cache:
      '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") ~ "/molecule/cache" }}'
    test_dest: 'fonts-test-font.ttf'
    test_url: 'https://github.com/google/fonts/blob/main/ufl/ubuntu/Ubuntu-Regular.ttf'
    test_mode: '0644'
    test_retries: 5
    test_delay: 5
```

#### create_cache (create caching location)
Create testing cache location for dynamic testing files.

[create_cache](https://github.com/r-pufky/ansible_test/tree/main/tasks/create_cache.yml)

prepare.yml
``` yaml
- name: 'Prepare cache'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'create_cache.yml'
  vars:
    test_name: 'Prepare cache'
    test_cache:
      '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") ~ "/molecule/cache" }}'
```

### Test Frameworks
Standardized interfaces for common file/directory test with diff options.

#### file (existence and permissions)
Test file existence and permissions. May be applied to file **or** directories.

[file](https://github.com/r-pufky/ansible_test/tree/main/tasks/file.yml)

verify.yml
``` yaml
- name: 'Verify permissions'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'file.yml'
  vars:
    test_name: '{{ "Verify permissions | " ~ item }}'
    test_file: '{{ item }}'
    test_owner: '305'
    test_group: '305'
    test_mode: '0755'
    test_state: 'directory'
    test_diff: false
  loop:
    - '/var/lib/forgejo/gitea-repositories'
    - '/var/lib/forgejo/data/tmp/uploads'
    - '/var/lib/forgejo/tmp/local-repo'
```

#### copy (static file testing)
Static test file contents and permissions.

Copies a known good testing file to remote system and validates target file is
the same.

[copy](https://github.com/r-pufky/ansible_test/tree/main/tasks/copy.yml)

verify.yml
``` yaml
- name: 'Verify cert.pem'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'copy.yml'
  vars:
    test_name: 'Verify cert.pem'
    test_src: '{{
        lookup("env", "MOLECULE_PROJECT_DIRECTORY") ~
        "/molecule/cache/cert.pem"
      }}'
    test_remote_src: false
    test_file: '/etc/mail/cert.pem'
    test_owner: 'mail'
    test_group: 'mail'
    test_mode: '0400'
    test_diff: false
```

#### lineinfile (existence of line in file)
Test existence of line in file.

[lineinfile](https://github.com/r-pufky/ansible_test/tree/main/tasks/lineinfile.yml)

verify.yml
``` yaml
- name: 'Verify | assert postgres settings'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'lineinfile.yml'
  vars:
    test_name: '{{ "Verify | assert postgres settings | " ~ item }}'
    test_file: '/etc/forgejo/app.ini'
    test_line: '{{ item }}'
    test_state: 'present'
    test_diff: false
  loop:
    - 'DB_TYPE = postgres'
    - 'HOST = 127.0.0.1'
    - 'NAME = forgejo'
    - 'USER = root'
    - 'PASSWD = `test`'
```

#### remote_file_diff (dynamic runtime file testing)
Remote test file contents and permissions.

Tests requiring run-time options (e.g. injected hashes that must exist but
are random per-run) should use this instead of copy test to verify file
contents are the same. Both files must exist on the remote system (generally
requiring the source of truth file to be rendered on the remote host in
converge).

[remote_file_diff](https://github.com/r-pufky/ansible_test/tree/main/tasks/remote_file_diff.yml)

verify.yml
``` yaml
- name: 'Verify | app.ini'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'remote_file_diff.yml'
  vars:
    test_name: 'Verify | app.ini'
    test_remote_src: '/tmp/app.ini'
    test_file: '/etc/forgejo/app.ini'
    test_owner: 'git'
    test_group: 'git'
    test_mode: '0640'
    test_diff: false
```

### sysctl (sysctl kernel option)
Test sysctl settings.

[sysctl](https://github.com/r-pufky/ansible_test/tree/main/tasks/sysctl.yml)

verify.yml
``` yaml
- name: 'Verify sysctl'
  ansible.builtin.include_role:
    name: 'r_pufky.data.test'
    tasks_from: 'sysctl.yml'
  vars:
    test_name: 'Verify sysctl'
    test_option: 'net.ipv4.tcp_congestion_control'
    test_value: 'bbr'
    test_diff: false
```

#### template (remote template)
Remote test file against template.

[template](https://github.com/r-pufky/ansible_test/tree/main/tasks/template.yml)

verify.yml
``` yaml
    - name: 'Verify /etc/network/interfaces'
      ansible.builtin.include_role:
        name: 'r_pufky.data.test'
        tasks_from: 'template.yml'
      vars:
        test_name: 'Verify /etc/network/interfaces'
        test_src: 'template.j2'
        test_file: '/etc/network/interfaces'
        test_owner: 'root'
        test_group: 'root'
        test_mode: '0644'
        test_diff: true
        test_content: |
          # This file describes the network interfaces available on your system
          # and how to activate them. For more information, see interfaces(5).

          {{ _test_network_vagrant }}
          source /etc/network/interfaces.d/*
```

## Development
Configure [environment](https://r-pufky.github.io/ansible_docs/ansible/environment)

Run all unit tests:
``` bash
molecule test --all
```

### Releases
[Semantic versioning](https://semver.org/spec/v2.0.0).

 Release | Debian | Ansible | Notes
---------|--------|---------|-------
 4.x.x   | 13     | 2.18+   | Migrate to r_pufky.data.
 3.x.x   | 13     | 2.18+   | Ansible 2.20, semantic versioning.
 2.x.x   | 13     | 2.18    | Migrate to Debian Trixie.
 1.x.x   | 12     | 2.18    | Ansible 2.18 support.
 0.x.x   | 12     | 2.11    | Initial commit.

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_test/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
